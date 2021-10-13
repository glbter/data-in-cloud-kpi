Завдання:
Використовуючи MySQL в AWS, що було створено під час попередньої лабораторної роботи, або створивши новий екземпляр замість видаленого:
### 1.	Знайти всіх співробітників, що ніколи не надавали знижок. Навіть якщо такі на даний момент відсутні.
```sql
select * 
from Employees 
where EmployeeID not in (
	select distinct EmployeeID
	from Orders
	where OrderID in (
		select OrderID
		from `Order Details` 
		where Discount != 0));
```
### 2.	Показати всі персональні дані з бази Northwind: повне ім’я, країну, місто, адресу, телефон. Звернути увагу, що ця інформація присутня в різних таблицях. 
```sql
select 
	ContactName,
	Country, 
	City, 
	Address, 
	Phone 
from Customers
union
select 
	concat_ws(' ', FirstName, LastName) as ContactName,
	Country,
	City,
	Address,
	HomePhone as Phone
from Employees
union
select 
	ContactName, 
	Country, 
	City, 
	Address, 
	Phone 
from Suppliers;
```
### 3.	Відобразити список всіх країн та міст, куди компанія робила відправлення. Позбавитися порожніх значень на дублікатів.
```sql
select distinct
	CompanyName
	ShipCountry,
	ShipCity
from Suppliers as s
join Products as p on s.SupplierID = p.SupplierID
join `Order Details` as od on p.ProductID = od.ProductID
join Orders as o on od.OrderID = o.OrderID 
where CompanyName is not null 
	and ShipCountry is not null 
	and ShipCity is not null
```
### 4.	Використовуючи базу Northwind вивести в алфавітному порядку назви продуктів та їх сумарну кількість в замовленнях.
```sql
select ProductName, sum(od.Quantity) as TotalInOrders
from Products as p 
join `Order Details` as od on p.ProductID = od.ProductID 
group by p.ProductID 
order by ProductName;
```
without joins
```sql
select 
	(select p.ProductName from Products as p where p.ProductID = od.ProductID) as ProductName,
	 sum(od.Quantity) as TotalInOrders
from `Order Details` as od
group by od.ProductID 
order by ProductName;
```
### 5.	Вивести імена всіх постачальників та сумарну вартість їх товарів, що зараз знаходяться на складі Northwind за умови, що ця сума більше $1000.
```sql
select 
	CompanyName,
    sum(UnitPrice * UnitsInStock) as TotalPrice
from Products as p 
join Suppliers as s on p.SupplierID = s.SupplierID
group by s.SupplierID
having TotalPrice > 1000;
```
without joins
```sql
select 
	(select CompanyName from Suppliers as s where p.SupplierID = s.SupplierID) as CompanyName,
    sum(UnitPrice * UnitsInStock) as TotalPrice
from Products as p
group by p.SupplierID
having TotalPrice > 1000;
```
### 6.	Знайти кількість замовлень, де фігурують товари з категорії «Сири». Результат має містити дві колонки: опис категорії та кількість замовлень.
```sql
select Description, count(OrderID) as AmountOfOrders
from Categories as c
join Products as p on p.CategoryID = c.CategoryID
join `Order Details` as od on od.ProductID = p.ProductID
where Description like 'Cheeses%';
```
without joins 
```sql
select 
	'Cheeses' as Description,
	 count(OrderID) as AmountOfOrders
from `Order Details` 
where ProductID in (
	select ProductID
	 from Products
	 where CategoryID in (
		select CategoryID
		 from Categories
		 where Description like 'Cheeses%'));
```
### 7.	Відобразити всі імена компаній-замовників та загальну суму всіх їх замовлень, враховуючи кількість товару та знижки. Показати навіть ті компанії, у яких замовлення відсутні. Позбавитися від відсутніх значень замінивши їх на нуль. Округлити числові результати до двох знаків після коми, відсортувати за алфавітом.
```sql
select 
	CompanyName, 
	round(sum(ifnull(((((od.UnitPrice * od.Quantity) * 
(1 - od.Discount)) / 100) * 100), 0)),2) as FullPrice
from Customers as c
left join Orders as o on c.CustomerID = o.CustomerID
left join `Order Details` as od on od.OrderID = o.OrderID

group by c.CustomerID
order by CompanyName;
```
### 8.	Вивести три колонки: співробітника (прізвище та ім’я, включаючи офіційне звернення), компанію, з якою співробітник найбільше працював згідно величини товарообігу (максимальна сума по усім замовленням в розрізі компанії), та ім’я представника компанії, додавши до останнього через кому посаду. Цікавить інформація тільки за 1998 рік.
```sql
select 
	concat_ws(' ', TitleOfCourtesy, LastName, FirstName) as FullName,
    max(prices.TotalPrice) as MaxPrice,
    concat(ContactName, ', ',ContactTitle) as ClientRepresentative
from Employees as e
join (
	select
		sum((((od.UnitPrice * od.Quantity) * (1 - od.Discount)) / 100) * 100) as TotalPrice,
        o.EmployeeID,
        o.CustomerID
	from Orders as o
	join `Order Details` as od on od.OrderID = o.OrderID
	where year(OrderDate) = 1998
	group by o.CustomerID, o.EmployeeID) 
as prices on prices.EmployeeID = e.EmployeeID
join Customers as c on c.CustomerID = prices.CustomerID
group by FullName;
```
### 9.	Вивести три колонки та три рядки. Колонки: Description, Key, Value. Рядки: 
ShippedDate, дата з максимальною кількістю відправлених замовлень, кількість відправлених замовлень на цю дату; 
Customer, замовник з максимальною кількістю відправлених замовлень, загальна кількість відправлених замовлень цьому замовнику; 
Shipper, перевізник з максимальною кількістю оброблених замовлень, загальна кількість відправлених через цього перевізника.
```sql
(select 
	'ShippedDate' as Description,
	ShippedDate as 'Key',
	count(OrderID) as Value
from Orders
where ShippedDate is not null
group by ShippedDate
 order by Value desc
limit 1)
union    
(select 
	'Customer' as Description,
	(select CompanyName from Customers as c where o.CustomerID = c.CustomerID) as 'Key',
	count(OrderID) as Value
from Orders as o
where ShippedDate is not null
group by CustomerID
order by Value desc
limit 1)
union
(select 
	'Shipper' as Description,
	(select CompanyName from Shippers where ShipVia = ShipperID) as 'Key',
	count(OrderID) as Value
from Orders
where ShippedDate is not null
group by ShipVia
order by Value desc
limit 1);
```
### 10.	Вивести найбільш популярній товари в розрізі країни. Показати: назву країни, назву продукту, загальну вартість поставок за весь час. Не використовувати функцій ранкування та партиціонування. 
```sql
select distinct
	o.ShipCountry as Country,
    (select ProductName from Products as pp where pp.ProductID = od.ProductID) as Product,
    (select sum((((UnitPrice * Quantity) * (1 - Discount)) / 100) * 100)
		From Orders o2
		join `Order Details` as od2 
		on o2.OrderID = od2.OrderID
        where o2.ShipCountry = o.ShipCountry 
			and od2.ProductID = od.ProductID) as TotalPrice
from Orders as o
join `Order Details` as od on o.OrderID = od.OrderID
where not exists (
	select distinct 1
	from Orders as o1
	join `Order Details` as od1 on o1.OrderID = od1.OrderID
    where (select sum((((UnitPrice * Quantity) * (1 - Discount)) / 100) * 100)
			From Orders o2
			join `Order Details` as od2 
			on o2.OrderID = od2.OrderID
			where o2.ShipCountry = o1.ShipCountry and o1.ShipCountry = o.ShipCountry 
				and od2.ProductID = od1.ProductID) > (
                
			select sum((((UnitPrice * Quantity) * (1 - Discount)) / 100) * 100)
			From Orders o2
			join `Order Details` as od2 
			on o2.OrderID = od2.OrderID
			where o2.ShipCountry = o.ShipCountry 
				and od2.ProductID = od.ProductID));
```
https://stackoverflow.com/questions/15603581/find-maximum-without-aggregate-function/56093134   
https://stackoverflow.com/questions/4259611/count-without-group


Матеріали для повторення:   
[Объединение (рос.)](http://www.sql-tutorial.ru/ru/book_union.html)   
[Объяснение SQL объединений JOIN: LEFT/RIGHT/INNER/OUTER (рос.)](http://www.skillz.ru/dev/php/article-Obyasnenie_SQL_obedinenii_JOIN_INNER_OUTER.html)   
[Предложение GROUP BY (рос.)](http://www.sql-tutorial.ru/ru/book_group_by_clause.html)   
[Агрегатні функції (укр.)](http://fcit.tneu.edu.ua/web-rozrobka/mysql/lektsiia-4-sql-select-ahrehatni-ta-hrupovi-funktsii)   


**hint: you can join tables on calculated values**   
you can join with `using(EmployeeID)`
you can use `select partition by over` [link](https://www.sqlshack.com/sql-partition-by-clause-overview/)
