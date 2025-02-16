# Создание и нормализация базы данных

### Задание 1. Продумать структуру базы данных и отрисовать в редакторе.

![image](https://github.com/user-attachments/assets/d5febf6f-8d23-4d8e-a360-22fbe3bede03)

### Задание 2. Нормализовать базу данных (1НФ — 3НФ), описав, к какой нормальной форме приводится таблица и почему таблица в этой нормальной форме изначально не находилась.

Почему исходные данные не были в 3НФ?

Исходная структура данных содержит информацию о транзакциях и клиентах в виде двух больших Эксель таблиц. Таблица уже находится в 1НФ и 2НФ, однако некоторые столбцы явно избыточны, 
есть транзитивная зависимость в данных, а так же в данных содержаться пропуски и дубли:

- В таблице клиентов существует транзитивная зависимость между postcode, state и country;
- В таблице транзакций происходит дублирование информации о продуктах, например: product_class, product_size, list_price, standard_cost;
- Нет четкой связи между клиентами, транзакциями и продуктами.

Анализируя поля таблицы customer, мы видим, что большинство полей напрямую зависят от customer_id и уникальны для каждого клиента. Однако, можно заметить, что поля, связанные 
с адресом (address, postcode, state, country), могут быть выделены в отдельную таблицу, чтобы уменьшить дублирование данных клиентов, живущих по одинаковому адресу.

Теперь приведем таблицу transaction к третьей нормальной форме (3НФ):

Удаление транзитивных зависимостей: В таблице transaction, поля, такие как brand, product_line, product_size, list_price, и standard_cost могут зависеть не напрямую 
от transaction_id, а от product_id. Эти поля лучше вынести в отдельную таблицу product.

В результате всех преобразований мы получили следующие 4 таблицы:

- location – информация об адресе клиентов
- customer – личные данные клиентов
- product – данные о товарах
- transaction – данные о транзакциях товаров


### Задание 3. Создать все таблицы в DBeaver, указав первичные ключи к таблицам, правильные типы данных, могут ли поля быть пустыми или нет.

```postgresql
create table location (
  location_id integer PRIMARY key not null,
  address text,
  postcode integer,
  state text,
  country text
);

create table customer (
  customer_id integer PRIMARY key not null,
  first_name varchar(50),
  last_name varchar(50),
  gender text,
  DOB text,
  job_title text,
  job_industry_category text,
  wealth_segment text,
  deceased_indicator boolean,
  owns_car boolean,
  location_id integer REFERENCES location(location_id),
  property_valuation integer
);

create table product (
  id integer PRIMARY KEY not null,
  product_id integer,
  brand varchar(50),
  product_line text,
  product_class text,
  product_size text,
  list_price float,
  standard_cost float
);

create table transaction (
  transaction_id integer PRIMARY KEY not null,
  product_transaction_id integer REFERENCES product(id),
  product_id integer,
  customer_id integer REFERENCES customer(customer_id),
  transaction_date text,
  online_order boolean,
  order_status text
);

```

### Задание 4. Загрузить данные в таблицы в соответствии с созданной структурой.

Таблица Customer
![image](https://github.com/user-attachments/assets/a51489d0-f429-4f4f-9549-9bea0d2dd841)

Таблица Location
![image](https://github.com/user-attachments/assets/11165ee2-aa45-490d-97bd-fb073e1c62a0)

Таблица Product
![image](https://github.com/user-attachments/assets/4d75cd03-efe6-4d7a-905a-467071cd58f7)

Таблица Transaction
![image](https://github.com/user-attachments/assets/1828feb7-2afc-4757-97d6-bc586218b452)

