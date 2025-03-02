# Найти имена и фамилии клиентов с минимальной/максимальной суммой транзакций за весь период (сумма транзакций не может быть null). Напишите отдельные запросы для минимальной и максимальной суммы.

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

