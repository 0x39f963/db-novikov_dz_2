# Домашнее задание №2  
Новиков Иван

 

---

## 1. Развернуты таблицы для выполнения ДЗ. 

Напоминаю, у меня PostgreSQL и pgAdmin были развернуты в Docker под WSL2.
Данные были скорректированы / очищены.

**1.1. Таблица customer**

```sql
CREATE TABLE customer (
    customer_id integer PRIMARY KEY,
    first_name text,
    last_name text,
    gender text,
    dob date,
    job_title text,
    job_industry_category text,
    wealth_segment text,
    deceased_indicator boolean,
    owns_car boolean,
    address text,
    postcode text,
    state text,
    country text,
    property_valuation integer
);
```

Скрины: 
https://disk.yandex.ru/i/GFLyYtYqAZSHUQ

https://disk.yandex.ru/i/2a1bVPiot1cccw  - успешно


**1.2. Таблица product**

```sql
CREATE TABLE product (
    product_id integer PRIMARY KEY,
    brand text,
    product_line text,
    product_class text,
    product_size text,
    list_price numeric(10,2),
    standard_cost numeric(10,2)
);
```

В CSV цены вида 1386.84, 60.34 и т.д., поэтому использовал numeric(10,2)

**Проблема**

product_id в сырых данных имеет множественные дубли, делаю по инструкции от преподалавателя:

https://disk.yandex.ru/i/A2wUh2bWDW35nw


```sql
CREATE TABLE product_cor AS
SELECT *
FROM (
    SELECT
        p.*,
        row_number() OVER (
            PARTITION BY product_id
            ORDER BY list_price DESC
        ) AS rn
    FROM product p
) t
WHERE rn = 1;
```

Скрин:  https://disk.yandex.ru/i/2Y-xu_oiuuWCmA

удаляю служ-ю колонку, следовательно таблицу product_cor будет иметь формат как product, толдько без дублей.

```sql
ALTER TABLE product_cor DROP COLUMN rn;
```

довожу до ума таблицу product_cor, далее переименовываю таблицы.

```sql
ALTER TABLE product_cor 
    ADD CONSTRAINT product_cor_pkey
    PRIMARY KEY (product_id);
	
ALTER TABLE product RENAME TO product_raw;
ALTER TABLE product_cor RENAME TO product;
```
Скрин:  https://disk.yandex.ru/i/bkZLftKh0BDfdw 




**1.3. Таблица orders**

```sql
CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    customer_id integer,
    order_date date,
    online_order boolean,
    order_status text
);
```
Скрин:
https://disk.yandex.ru/i/LrjgX-MqcM0Ctg

Проблемы импорта: 360 пустых строк (boolean) из 19640 строк CSV:

https://disk.yandex.ru/i/aFGpW4nbFn-hAQ 

В итоге я проставил всем пустым строкам NULL в CSV руками (отфильтровав и протянув)
https://disk.yandex.ru/i/I8YilwekPlsXWg 

Псоле чего импортнул с параметром NULL String: https://disk.yandex.ru/i/m6Bx8ZeRg6jF0w
Импортированные данные: https://disk.yandex.ru/i/PsAGvGf9KrZR5w


**1.4. Таблица order_items**

```sql
CREATE TABLE order_items (
    order_item_id integer PRIMARY KEY,
    order_id integer,
    product_id integer,
    quantity numeric(10,2),
    item_list_price_at_sale numeric(10,2),
    item_standard_cost_at_sale numeric(10,2)
);
```

Скрин:
https://disk.yandex.ru/i/ocF11OFrXryfgw



## 2. Задания по запросам

**Запрос 1.**
*Вывести все уникальные бренды, у которых есть хотя бы один продукт со стандартной стоимостью выше 1500 долларов, и который был продан как минимум 1000 раз (суммарное количество)*

```sql


SELECT DISTINCT
    p.brand
FROM product p

JOIN order_items oi
    ON oi.product_id = p.product_id WHERE p.standard_cost > 1500

GROUP BY
    p.brand,
    p.product_id

HAVING
SUM(oi.quantity) >= 1000

ORDER BY
    p.brand;

```
**Результат:**
| brand          |
|----------------|
| Giant Bicycles |
| OHM Cycles     |


Скриншоты: https://disk.yandex.ru/i/1ORpP_pj1Qdzmw


.



**Запрос 2.** 
*Для каждого дня в диапазоне с 2017-04-01 по 2017-04-09 включительно вывести количество подтвержденных онлайн-заказов и количество уникальных клиентов, совершивших эти заказы*

```sql

SELECT
    order_date as day,
	COUNT (*) AS approved_online_orders, -- всего закаов
    COUNT (DISTINCT customer_id) AS uniq_clients -- уник клиенты 
	
FROM orders

WHERE
    order_date BETWEEN DATE '2017-04-01' AND DATE '2017-04-09'
    AND online_order = true
    AND order_status = 'Approved'
	
GROUP BY order_date ORDER BY order_date; -- собираем строки по дням и сортируем по ним же

```

Скриншоты:
https://disk.yandex.ru/i/xcFfZ21FNM4dug

**Результат:**
| day        | approved_online_orders | uniq_clients |
|------------|------------------------|--------------|
| 2017-04-01 | 37 | 37 |
| 2017-04-02 | 29 | 29 |
| 2017-04-03 | 27 | 27 |
| 2017-04-04 | 32 | 32 |
| 2017-04-05 | 33 | 32 |
| 2017-04-06 | 36 | 36 |
| 2017-04-07 | 24 | 24 |
| 2017-04-08 | 33 | 33 |
| 2017-04-09 | 30 | 30 |


.


**Запрос 3.** 
*Вывести профессии для клиентов, которые: находятся в сфере 'IT' И их профессия начинается с Senior, находятся в сфере 'Financial Services' и их профессия начинается с Lead. При этом для обоих пунктов учесть, что возраст клиентов должен быть старше 35 лет. Использовать UNION ALL для объединения 2 пунктов*

```sql

SELECT job_title FROM customer WHERE
    job_industry_category = 'IT'
    AND job_title LIKE 'Senior%'
	
    AND dob <= CURRENT_DATE - INTERVAL '35 years' -- :)

UNION ALL

SELECT job_title FROM customer WHERE
    job_industry_category = 'Financial Services'
    AND job_title LIKE 'Lead%'
    AND dob <= CURRENT_DATE - INTERVAL '35 years'

```

Скриншоты:
https://disk.yandex.ru/i/7Wi4jU0aiQN4Mw

**Результат:**

| job_title              |
|------------------------|
| Senior Sales Associate |
| Senior Developer       |

.

**Запрос 4.** 
*Вывести бренды, которые были куплены клиентами из сферы Financial Services, но НЕ были куплены клиентами из сферы IT*

```sql

SELECT DISTINCT p.brand FROM customer c
	JOIN orders o ON o.customer_id = c.customer_id
	JOIN order_items oi ON oi.order_id = o.order_id
	JOIN product p ON p.product_id = oi.product_id
WHERE c.job_industry_category = 'Financial Services' -- ORDER BY p.brand ASC

EXCEPT

SELECT DISTINCT p.brand FROM customer c
 	JOIN orders o ON o.customer_id = c.customer_id
 	JOIN order_items oi ON oi.order_id = o.order_id
    JOIN product p ON p.product_id = oi.product_id
WHERE c.job_industry_category = 'IT' -- ORDER BY p.brand ASC

```

Скриншоты:
https://disk.yandex.ru/i/ajrp59kAuF5zDQ

**Результат: нет таких брендов, которые купили первые и не купили вторые.**

Проверяем бренды по отдельности:
https://disk.yandex.ru/i/EYwpG8suhd72kw 



.



**Запрос 5.** 
*Вывести 10 клиентов (ID, имя, фамилия), которые совершили наибольшее количество онлайн-заказов (в штуках) брендов Giant Bicycles, Norco Bicycles, Trek Bicycles, при условии, что они активны и имеют оценку имущества (property_valuation) выше среднего по их штату*

```sql

SELECT c.customer_id, c.first_name, c.last_name,
	COUNT(DISTINCT o.order_id) AS online_order_count 
FROM customer c 

	JOIN orders o ON o.customer_id = c.customer_id
	JOIN order_items oi ON oi.order_id = o.order_id 
	JOIN product p ON p.product_id = oi.product_id

WHERE

    p.brand IN ('Giant Bicycles', 'Norco Bicycles', 'Trek Bicycles')

    AND o.online_order = true

	AND c.deceased_indicator = false
    
	-- оценка выше ср. по штату
    AND c.property_valuation > (SELECT AVG(c2.property_valuation) 
        FROM customer c2 WHERE c2.state = c.state)
		
GROUP BY c.customer_id, c.first_name, c.last_name
-- ! смотрим по убыв кол-ва заказов, чтобы нужные попали в отбор
ORDER BY online_order_count DESC, c.customer_id  
LIMIT 10;

```

Скриншоты: https://disk.yandex.ru/i/dqcl747aAY4yeQ 

**Результат:.**

| customer_id | first_name | last_name   | online_order_count |
|-------------|------------|------------|---------------------|
| 787         | Norma      | Batrim     | 6                   |
| 1           | Laraine    | Medendorp  | 5                   |
| 273         | Nevile     | Abraham    | 5                   |
| 353         | Antonia    | Cardis     | 5                   |
| 1033        | Jacob      | Claringbold| 5                   |
| 1117        | Georgena   | Guilaem    | 5                   |
| 2072        | Margie     | Tillyer    | 5                   |
| 2498        | Rosana     | Emmatt     | 5                   |
| 2595        | Land       | Bangley    | 5                   |
| 2637        | Marcile    | Christley  | 5                   |

.


**Запрос 6.** 
*Вывести всех клиентов (ID, имя, фамилия), у которых нет подтвержденных онлайн-заказов за последний год, но при этом они владеют автомобилем и их сегмент благосостояния не Mass Customer.*


```sql

SELECT c.customer_id, c.first_name, c.last_name FROM customer c

LEFT JOIN orders o ON o.customer_id = c.customer_id
   AND o.online_order = true
   AND o.order_status = 'Approved'
   AND o.order_date BETWEEN '2017-01-01' AND '2017-12-31'
   
WHERE
    c.owns_car = true
    AND c.wealth_segment <> 'Mass Customer' 
    AND o.order_id IS NULL
ORDER BY c.customer_id;

```

напоминаю, что в данных у нас 2017 год:
https://disk.yandex.ru/i/aFv-_2riGgWiXg

Скриншоты результатов (всеголд получилось 173 строки):
https://disk.yandex.ru/i/TtCHxQQ0Jcvciw


.



**Запрос 7.** 
*Вывести всех клиентов из сферы IT (ID, имя, фамилия), которые купили 2 из 5 продуктов с самой высокой list_price в продуктовой линейке Road*

```sql

SELECT c.customer_id, c.first_name, c.last_name FROM customer c
	JOIN orders o ON o.customer_id = c.customer_id
	JOIN order_items oi ON oi.order_id = o.order_id 
	JOIN product p  ON p.product_id = oi.product_id

WHERE
    c.job_industry_category = 'IT'

    AND p.product_id IN ( SELECT product_id  FROM product
        WHERE product_line = 'Road'
        ORDER BY list_price DESC
        LIMIT 5 )
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING COUNT(DISTINCT p.product_id) >= 2

ORDER BY c.customer_id;

```

Скриншоты: https://disk.yandex.ru/i/hRT1WugMSWvShQ 

Результат: 

| customer_id | first_name | last_name  |
|-------------|------------|-----------|
| 604         | Mella      | Petrovsky |
| 983         | Shaylyn    | Riggs     |
| 1683        | Brenn      | Bacon     |
| 2469        | Kermie     | Hedger    |
| 3406        | Lucy       | Lackmann  |


.



**Запрос 8.** 
*Вывести клиентов (ID, имя, фамилия, сфера деятельности) из сферы IT или Health, которые совершили не менее 3 подтвержденных заказов в период 2017-01-01 по 2017-03-01 и при этом их общий доход от этих заказов превышает 10000 долларов.*
*Разделить вывод на две группы (IT и Health) с помощью UNION*


```sql

SELECT c.customer_id, c.first_name, c.last_name, c.job_industry_category AS industry
FROM customer c 
	JOIN orders o ON o.customer_id = c.customer_id
	JOIN order_items oi ON oi.order_id = o.order_id
WHERE c.job_industry_category = 'IT'
    AND o.order_status = 'Approved'
    AND o.order_date >= '2017-01-01'
    AND o.order_date <= '2017-03-01'
	
GROUP BY c.customer_id, c.first_name, c.last_name,
    c.job_industry_category
	
HAVING 
    COUNT(DISTINCT o.order_id) >= 3
    AND SUM(oi.quantity * oi.item_list_price_at_sale) > 10000

UNION

-- клиенты health
SELECT c.customer_id, c.first_name, c.last_name, c.job_industry_category AS industry

FROM customer c 
JOIN orders o  ON o.customer_id = c.customer_id 
JOIN order_items oi ON oi.order_id = o.order_id

WHERE c.job_industry_category = 'Health'
    AND o.order_status = 'Approved'
    AND o.order_date >= '2017-01-01'
    AND o.order_date <= '2017-03-01'
	
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name,
	
    c.job_industry_category
	
HAVING
    COUNT(DISTINCT o.order_id) >= 3
    AND  SUM(oi.quantity *  oi.item_list_price_at_sale) > 10000
	
ORDER BY
    industry,
    customer_id;

```


Скриншоты: https://disk.yandex.ru/i/iOLvyaAcdhXdMg
в результатах 38 строк *см. скриншот*


.
.

