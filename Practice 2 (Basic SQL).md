Завдання:

1.	Розгорнути MySQL RDS як службу СУБД в Amazon Web Services та прикріпити скріншот, що демонструє унікальний endpoint.
 
2.	На створеному екземплярі СУБД створити базу Northwind з скрита за [посиланням](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/northwindextended/Northwind.MySQL5.sql).
Прикріпити скріншот з MySQL Workbench.
 
3.	Використавши SELECT та не використовуючи FROM вивести на екран назву виконавця та пісні, яку ви слухали останньою. Імена колонок вказати як Artist та Title. 
```sql
select 'Never Gonna Give You Up' as Title, 'Rick Astley' as Artist;
```
4.	Вивести вміст таблиці Order Details, замінивши назви атрибутів OrderID та ProductID на OrderNumber та ProductNumber.
```sql
select OrderID as OrderNumber, ProductID as ProductNumber 
from `Order Details`;
```
5.	З таблиці співробітників вивести всіх співробітників, що мають заробітну платню більше 2000.00, проте меншу за 3000.00. Результат відобразити у вигляді двох колонок, де перша – це конкатенація звернення (TitleOfCourtesy), прізвища та імені. Друга колонка – це заробітна плата відсортована у порядку зростання.
```sql
select concat_ws(' ', TitleOfCourtesy, LastName, FirstName), Salary
from Employees
where Salary between 2000 and 3000
order by salary asc;
```
6.	Вивести назву всіх продуктів, що продаються у банках (jar), відсортувати за алфавітом.
```sql
select ProductName 
from Products
where QuantityPerUnit like '%jar%'
order by ProductName asc;
```
7.	Використовуючи базу Northwind вивести всі замовлення, що були здійснені замовниками Island Trading та Queen Cozinha.
```sql
select ShipName,
 ShipAddress,
 ShipCity,
 ShipRegion,
 ShipPostalCode,
 ShipCountry,
 OrderDate,
 RequiredDate
 ShippedDate,
 ShipVia,
 Freight
from Orders
where CustomerID in (
	select CustomerID
	from Customers 
	where CompanyName = 'Queen Cozinha' 
		or CompanyName = 'Island Trading');
```
8.	Вивести всі назви та кількість на складі продуктів, що належать до категорій Dairy Products, Grains/Cereals та Meat/Poultry.
```sql
select ProductName, UnitsInStock
from Products
where CategoryID in (
	select CategoryID
	from Categories
	where CategoryName in ('Dairy Products', 'Grains/Cereals', 'Meat/Poultry'));
```
9.	Вивести всі замовлення, де вартість одиниці товару 50.00 та вище. Відсортувати за номером, позбавитися дублікатів.
```sql
select ShipName,
 ShipAddress,
 ShipCity,
 ShipRegion,
 ShipPostalCode,
 ShipCountry,
 OrderDate,
 RequiredDate
 ShippedDate,
 ShipVia,
 Freight
from Orders 
where OrderId in (
	select distinct OrderID 
	from `Order Details`
	where UnitPrice > 50)
order by OrderID ASC;
```
10.	Відобразити всіх постачальників, де контактною особою є власник, або менеджер і є продукти, що зараз знаходяться на стадії поставки.
```sql
select 
	CompanyName,
	ContactName,
    Country,
    Phone
from Suppliers
where ((ContactTitle like '%Manager%')
	or (ContactTitle like 'Owner'))
    and SupplierID in (
		select distinct SupplierID 
		from Products 
		where UnitsOnOrder > 0);
```
11.	Вивести всіх замовників з Мексики, де контактною особою є власник, а доставка товарів відбувалася через Federal Shipping.
```sql
select CompanyName
from Customers
where ContactTitle = 'Owner' 
	and Country = 'Mexico'
    and CustomerID in (
		select distinct CustomerID 
		from Orders
		where ShipVia in (
			select ShipperID
			from Shippers
			where CompanyName = 'Federal Shipping'));
```
Матеріали для повторення:

[Cloud Databases (practice) pt2 - відео заняття](https://youtu.be/czFZ8VQpGLk)
[Cloud Databases (practice) pt3 - відео заняття](https://youtu.be/x0Jfq9qpKdU)
[Предикати порівняння (рос.)](http://sql-ex.ru/help/select2.php)
[Предикат LIKE (рос.)](http://www.sql-tutorial.ru/ru/book_predicate_like.html)
[Функції по роботі з текстом (англ.)](https://www.tutorialspoint.com/sql/sql-string-functions.htm)
[Підзапити (укр.)](http://moonexcel.com.ua/%D1%83%D1%80%D0%BE%D0%BA%D0%B8-sql9-%D0%BF%D1%96%D0%B4%D0%B7%D0%B0%D0%BF%D0%B8%D1%82%D0%B8_ua)

