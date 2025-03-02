## Группировка данных и оконные функции

### 1. Создать таблицы со следующими структурами и загрузить данные из csv-файлов

```postgresql
create table customer_20250301(
    customer_id integer PRIMARY key not null,
    first_name text,         
    last_name text,             
    gender text,                 
    DOB date,                    
    job_title text,               
    job_industry_category text,   
    wealth_segment text,          
    deceased_indicator text,      
    owns_car text,               
    address text,              
    postcode integer,            
    state text,          
    country text,                
    property_valuation integer       
);

create table transaction_20250301(
    transaction_id integer PRIMARY key not null,
    product_id integer,
    customer_id integer,
    transaction_date date,    
    online_order text,        
    order_status text,        
    brand text,               
    product_line text,       
    product_class text,      
    product_size text,        
    list_price float,          
    standard_cost float      
);
```

### 2. Выполнить следующие запросы:

#### Задание 1. Вывести распределение (количество) клиентов по сферам деятельности, отсортировав результат по убыванию количества.
```postgresql
select count(customer_id) as customer_count, job_industry_category
from customer_20250301 c
group by c.job_industry_category 
order by customer_count desc;
```
![image](https://github.com/user-attachments/assets/5367aaec-9233-4423-91cd-6e92127baa6d)

#### Задание 2. Найти сумму транзакций за каждый месяц по сферам деятельности, отсортировав по месяцам и по сфере деятельности.
```postgresql
select date_trunc('month', t.transaction_date) as month, sum(t.list_price) as transaction_sum, c.job_industry_category
from transaction_20250301 t 
left join customer_20250301 c 
on t.customer_id = c.customer_id
group by date_trunc('month', t.transaction_date), c.job_industry_category
order by month, c.job_industry_category;
```
![image](https://github.com/user-attachments/assets/17c935d8-5eb0-47e8-a8e3-c9b769c08db5)

#### Задание 3. Вывести количество онлайн-заказов для всех брендов в рамках подтвержденных заказов клиентов из сферы IT.
```postgresql
select count(t.online_order) as order_count, t.brand
from transaction_20250301 t 
left join customer_20250301 c 
on t.customer_id = c.customer_id
where t.order_status = 'Approved'
	   and t.online_order = 'True'
	   and c.job_industry_category = 'IT'
group by t.brand
order by order_count;
```
![image](https://github.com/user-attachments/assets/b4330eae-de6c-4373-aacf-66b4bd87d9d7)

#### Задание 4. Найти по всем клиентам сумму всех транзакций (list_price), максимум, минимум и количество транзакций, отсортировав результат по убыванию суммы транзакций и количества клиентов. Выполнить двумя способами: используя только group by и используя только оконные функции. Сравнить результат.
```postgresql
select c.customer_id, sum(list_price) as sum_price, min(list_price) as min_price, max(list_price) as max_price, count(t.transaction_date) as trans_count
from transaction_20250301 t 
inner join customer_20250301 c 
on t.customer_id  = c.customer_id
group by c.customer_id
order by sum_price desc, trans_count desc;
```
![image](https://github.com/user-attachments/assets/29dccca1-9bf6-4eda-8128-fed954e1a227)

```postgresql
with all_clients_tr_sum as (
	select distinct *,
			row_number() over(partition by c.customer_id order by c.customer_id desc) as row_numb,
			sum(t.list_price) over(partition by c.customer_id) as sum_price,
			min(t.list_price) over(partition by c.customer_id) as min_price,
			max(t.list_price) over(partition by c.customer_id) as max_price,
			count(t.transaction_id) over(partition by c.customer_id) as trans_count
	from transaction_20250301 t
	inner join customer_20250301 c on t.customer_id = c.customer_id
)
select *
from all_clients_tr_sum
where row_numb = 1
order by sum_price desc, trans_count desc;
```
![image](https://github.com/user-attachments/assets/7136c61d-d4d2-4563-8ac7-560d658566b4)


Запрос с помощью команды group by агрегирует данные и возвращает одну строку по каждому клиенту, в то время как запрос с оконными функциями не агрегирует данные, а добавляет значения к каждой транзакции.

#### Задание 5. Найти имена и фамилии клиентов с минимальной/максимальной суммой транзакций за весь период (сумма транзакций не может быть null). Напишите отдельные запросы для минимальной и максимальной суммы.
```postgresql
select c.first_name, c.last_name, min(t.list_price) as min_price
from customer_20250301 c 
inner join transaction_20250301 t 
on c.customer_id = t.customer_id
group by c.first_name, c.last_name
having min(t.list_price) = (
	select min(t.list_price) as min_price
	from transaction_20250301 t
);
```
![image](https://github.com/user-attachments/assets/ba762ee1-8368-4951-a5df-f98163b2c18d)

```postgresql
select c.first_name, c.last_name, max(t.list_price) as max_price
from customer_20250301 c 
inner join transaction_20250301 t 
on c.customer_id = t.customer_id
group by c.first_name, c.last_name
having max(t.list_price) = (
	select max(t.list_price)
	from transaction_20250301 t
);
```
![image](https://github.com/user-attachments/assets/bda2708b-e7fd-48ee-8613-a800f5a928da)

#### Задание 6. Вывести только самые первые транзакции клиентов. Решить с помощью оконных функций.
```postgresql
with first_clients_trans as (
    select t.*, row_number() over(partition by c.customer_id order by t.transaction_date) as rownumb
    from transaction_20250301 t
    inner join customer_20250301 c on t.customer_id = c.customer_id
)
select *
from first_clients_trans
where rownumb = 1;
```
![image](https://github.com/user-attachments/assets/512b01bd-61d5-4dfc-9aa8-c50c96efd2b7)

#### Задание 7. Вывести имена, фамилии и профессии клиентов, между транзакциями которых был максимальный интервал (интервал вычисляется в днях).
```postgresql
with client_trans as (
	select c.first_name, c.last_name, c.job_title, t.transaction_date,
	    lag(t.transaction_date) over (partition by c.customer_id order by t.transaction_date) as lag_date
	from transaction_20250301 t
	inner join customer_20250301 c on t.customer_id = c.customer_id
),
max_time_diff as (
    select *, transaction_date - lag_date as time_diff
    from client_trans
)
select first_name, last_name, job_title, time_diff
from max_time_diff
where time_diff is not null
order by time_diff desc;
```
![image](https://github.com/user-attachments/assets/349bb0d4-fe25-4fb7-8330-8050c2a682ee)
