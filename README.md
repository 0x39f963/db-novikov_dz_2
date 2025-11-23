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



...
пром табл
...



**1.4. Таблица orders**

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


**1.5. Таблица order_items**

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

-- empty 1

```

Скриншоты:



**Запрос 2.** 
*Для каждого дня в диапазоне с 2017-04-01 по 2017-04-09 включительно вывести количество подтвержденных онлайн-заказов и количество уникальных клиентов, совершивших эти заказы*

```sql

-- empty 2

```

Скриншоты:



**Запрос 3.** 
*Вывести профессии для клиентов, которые: находятся в сфере 'IT' И их профессия начинается с Senior, находятся в сфере 'Financial Services' и их профессия начинается с Lead. При этом для обоих пунктов учесть, что возраст клиентов должен быть старше 35 лет. Использовать UNION ALL для объединения 2 пунктов*

```sql

-- empty 3

```

Скриншоты:



**Запрос 4.** 
*Вывести бренды, которые были куплены клиентами из сферы Financial Services, но НЕ были куплены клиентами из сферы IT*

```sql

-- empty 4

```

Скриншоты:


**Запрос 5.** 
*Вывести 10 клиентов (ID, имя, фамилия), которые совершили наибольшее количество онлайн-заказов (в штуках) брендов Giant Bicycles, Norco Bicycles, Trek Bicycles, при условии, что они активны и имеют оценку имущества (property_valuation) выше среднего по их штату*

```sql

-- empty 5

```

Скриншоты:


**Запрос 6.** 
*Вывести всех клиентов (ID, имя, фамилия), у которых нет подтвержденных онлайн-заказов за последний год, но при этом они владеют автомобилем и их сегмент благосостояния не Mass Customer.*


```sql

-- empty 6

```

Скриншоты:


**Запрос 7.** 
*Вывести всех клиентов из сферы IT (ID, имя, фамилия), которые купили 2 из 5 продуктов с самой высокой list_price в продуктовой линейке Road*

```sql

-- empty 7

```

Скриншоты:



**Запрос 8.** 
*Вывести клиентов (ID, имя, фамилия, сфера деятельности) из сферы IT или Health, которые совершили не менее 3 подтвержденных заказов в период 2017-01-01 по 2017-03-01 и при этом их общий доход от этих заказов превышает 10000 долларов.*
*Разделить вывод на две группы (IT и Health) с помощью UNION*


```sql

-- empty 8

```

Скриншоты:

