# Основные операторы PostgreSQL

### 1. Создать таблицы со следующими структурами и загрузить данные из csv-файлов. Детали приведены ниже.

```postgresql
create table customer_20250101(
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

create table transaction_20250101(
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

#### Задание 1. Вывести все уникальные бренды, у которых стандартная стоимость выше 1500 долларов.
```postgresql
select distinct brand
from transaction_20250101 t
where t.standard_cost > 1500;
```
![image](https://github.com/user-attachments/assets/f525f702-c93f-4298-8a8c-25998fcc60e2)

#### Задание 2. Вывести все подтвержденные транзакции за период '2017-04-01' по '2017-04-09' включительно.
```postgresql
select *
from transaction_20250101 t
where t.order_status = 'Approved'
and (t.transaction_date >= '2017-04-01' and t.transaction_date <= '2017-04-09')
order by t.transaction_date asc;
```
![image](https://github.com/user-attachments/assets/72cd8895-57c2-41df-b9d5-9966f5a68ce9)

#### Задание 3. Вывести все профессии у клиентов из сферы IT или Financial Services, которые начинаются с фразы 'Senior'.
```postgresql
select distinct c.job_title
from customer_20250101 c
where (c.job_industry_category = 'IT' or c.job_industry_category = 'Financial Services')
	  and c.job_title like 'Senior%';
```
![image](https://github.com/user-attachments/assets/75ca4a8f-efdd-4879-aa0f-f82c4ba88076)

#### Задание 4. Вывести все бренды, которые закупают клиенты, работающие в сфере Financial Services.
```postgresql
select distinct t.brand
from transaction_20250101 t 
left join customer_20250101 c on t.customer_id = c.customer_id 
where c.job_industry_category = 'Financial Services';
```
![image](https://github.com/user-attachments/assets/46c1fe08-9877-42f8-bdf1-6ec4182e7dae)

#### Задание 5. Вывести 10 клиентов, которые оформили онлайн-заказ продукции из брендов 'Giant Bicycles', 'Norco Bicycles', 'Trek Bicycles'.
```postgresql
select distinct c.customer_id, c.first_name, c.last_name 
from customer_20250101 c 
left join transaction_20250101 t on c.customer_id = t.customer_id 
where t.online_order = 'True' 
	  and t.brand in ('Giant Bicycles', 'Norco Bicycles', 'Trek Bicycles')
limit 10;
```
![image](https://github.com/user-attachments/assets/3f8771cb-c288-4333-9eb5-cd274a8a00da)

#### Задание 6. Вывести всех клиентов, у которых нет транзакций.
```postgresql
select distinct c.customer_id, c.first_name, c.last_name  
from customer_20250101 c
full outer join transaction_20250101 t on c.customer_id = t.customer_id 
where t.transaction_id is null;
```
![image](https://github.com/user-attachments/assets/f588b6a8-8559-47b7-9d34-cfe06f63f34d)

#### Задание 7. Вывести всех клиентов из IT, у которых транзакции с максимальной стандартной стоимостью.
```postgresql
select distinct c.customer_id, c.first_name, c.last_name  
from customer_20250101 c
left join transaction_20250101 t on c.customer_id = t.customer_id 
where c.job_industry_category = 'IT' 
	  and t.standard_cost = (
                    select max(t.standard_cost)
                    from transaction_20250101 t
);
```
![image](https://github.com/user-attachments/assets/d28e7a6c-2146-4d67-8c85-f79e6983ac84)

#### Задание 8. Вывести всех клиентов из сферы IT и Health, у которых есть подтвержденные транзакции за период '2017-07-07' по '2017-07-17'.
```postgresql
select distinct c.customer_id, c.first_name, c.last_name  
from customer_20250101 c
left join transaction_20250101 t on c.customer_id = t.customer_id 
where c.job_industry_category in ('IT ', 'Health')
	  and t.order_status = 'Approved'
	  and t.transaction_date::date between '2017-07-07' and '2017-07-17';
```
![image](https://github.com/user-attachments/assets/f4ad1917-2df9-467d-949b-fefcb63b19ab)
