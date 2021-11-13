Завдання:
Використовуючи MySQL в AWS, що було створено під час другої лабораторної роботи, або створивши новий екземпляр замість видаленого:
### 1.	Відобразити назву продукту та його вартість. Якщо товар відсутній на складі – відобразити замість вартості N/A.
```sql
select ProductName, if(UnitsInStock = 0, 'N/A', UnitPrice) as UnitPrice from Products;
```
### 2.	У дві колонки вивести співробітників, так їх керівників. Якщо керівник відсутній – повідомити про Self-managed. 
```sql
select 
	concat_ws(' ', FirstName, LastName) as EmployeeName,
    if(isnull(ReportsTo), 'Self-managed', 
    (
      select concat_ws(' ', FirstName, LastName) 
      from Employees e2 
      where e.ReportsTo = e2.EmployeeID)
    ) as ManagerName
from Employees e;
```
### 3.	Вивести номери замовлення та їх регіон відправки. Якщо такий відсутній – країну відправки. Якщо дати відправки немає – відобразити замість регіону фразу Not shipped.
```sql
select OrderID, 
if(isnull(ShippedDate), 'Not shipped', coalesce(ShipRegion, ShipCountry)) as Region
from Orders;
```
### 4.	Повернути з бази даних наступні дані: повне ім’я співробітника (однією колонкою), назву території, за яку він відповідає, до останньої через пробіл додати в дужках індикатор відповідного регіону (Nord, Sud, Est, Ovest). Приклад: Phoenix (Ovest).
```sql
select 
	concat_ws(' ', FirstName, LastName) as Employee,
    concat(trim(TerritoryDescription),' (',
		case 
			when trim(RegionDescription) = 'Westerns'
				then 'Ovest'
			when trim(RegionDescription) = 'Northern'
				then 'Nord'
			when trim(RegionDescription) = 'Southern'
				then 'Sud'
			when trim(RegionDescription) = 'Eastern'
				then 'Est'
		end
	,')') as Territory
from Employees e
join EmployeeTerritories using(EmployeeID)
join Territories using(TerritoryID)
join Region using(RegionID);
```
### 5.	Вивести по три найдешевших товару для кожної категорії.
```sql
select ProductName, CategoryName from (
	select *,
		(row_number() over (partition by CategoryName order by UnitPrice)) as number
		from Products
	join Categories using(CategoryID)
) as t
where number <= 3;
```
### 6.	Вивести наступну інформацію: Країна, Ранг. Відсортувати за рангом. Ранжування провести по загальній вартості товарів відправлених в цю країну.         
```sql
select 
	ShipCountry, 
	rank() over (order by sum((UnitPrice * Quantity) * (1 - Discount))) as PriceRank
from Orders
join `Order Details` using(OrderID)
group by ShipCountry
order by PriceRank;
```
### 7.	Вивести окремими стовпчиками прізвище та ім’я співробітників Northwind та контактних осіб замовників, що мають посади спільні для обох таблиць. В якості третьої колонки вивести саму посаду.
```sql
select
	substring_index(ContactName, ' ', 1) as FirstName,
	substring_index(ContactName, ' ', -1) as LastName,
	ContactTitle as Title
	from Customers
	where ContactTitle in (select Title from Employees)
union
select
	FirstName,
	LastName,
	Title
	from Employees
	where Title in (select ContactTitle from  Customers);
```  
щоб вивести групами по посадам

```sql
select 
	FirstName,
	LastName,
	Title
from (        
	select *, row_number() over (partition by Title)
	from (
		select
			substring_index(ContactName, ' ', 1) as FirstName,
			substring_index(ContactName, ' ', -1) as LastName,
			ContactTitle as Title
			from Customers
			where ContactTitle in (select Title from Employees)
		union
		select
			FirstName,
			LastName,
			Title
			from Employees
			where Title in (select ContactTitle from Customers)
	) t1
) t;
```
### 8.	Вивести прізвище та ім’я співробітника, рік, номер замовлення в базі, та яким за рік стало це замовлення для конкретного співробітника (починаючи нумерацію з одиниці).
```sql
select 
	LastName,
    FirstName,
    year(OrderDate) as Year,
	OrderID,
	row_number() over (partition by EmployeeID, year(OrderDate) order by OrderDate) as OrderInYear
from Employees
join Orders using(EmployeeID);
```
### 9.	Для кожного замовника знайти три замовлення з максимальною різницею між датою замовлення та датою відправлення. 
```sql
select 
	ContactName,
    OrderID
from (
	select *,
		row_number() over (partition by CustomerID order by ShippedDate - OrderDate desc) as Num
	from Customers
	join Orders using(CustomerID)
	where ShippedDate - OrderDate is not null
) t
where Num <= 3;
```
### 10.	Для кожного співробітника вивести другий десяток найбільш дорогих замовлень (тобто замовлення, що за загальною вартістю для конкретного співробітника будуть під номерами з 11 по 20).
```sql
select 
	LastName,
    FirstName,
	OrderID,
    TotalPrice
from (
	select *,
		row_number() over (partition by EmployeeID order by TotalPrice desc) as PriceOrder
	from (
		select *,
			sum((UnitPrice * Quantity) * (1 - Discount)) as TotalPrice
		from Employees
		join Orders using(EmployeeID)
		join `Order Details` using(OrderID)
		group by OrderID
	) t1
)t
where PriceOrder between 11 and 20;
```


Матеріали для повторення:
- [Cloud Databases (practice) pt4](https://www.youtube.com/watch?app=desktop&v=s2ZPPiAFFqs&feature=youtu.be) (NULL, JOIN, UNION, GROUP BY, агрегатні функції та текстові функції)
- [Cloud Databases (practice) pt5](https://www.youtube.com/watch?app=desktop&v=7pa9bP9zrnI&feature=youtu.be) (INTERSECT, EXCEPT, ISNULL/IFNULL, COALESCE, IF, CASE, ROW_NUMBER, RANK, DENSE_RANK)- 
- [Database Skill School 4](https://www.youtube.com/watch?app=desktop&v=9R1mC3YQKQ8&feature=youtu.be) (Агрегатні функції, GROUP BY, HAVING, текстові функції)
- [Database Skill School 5](https://www.youtube.com/watch?app=desktop&v=Zgm5qH9FhJU&feature=youtu.be) (Вкладеність підзапитів, віконні функції ROW_NUMBER, RANK)
- [Database Skill School 6](https://www.youtube.com/watch?app=desktop&v=9kbDpxBHEuc&feature=youtu.be) (INTERSECT, EXCEPT, ISNULL/IFNULL, COALESCE, IF, CASE)
- Функції CASE, IF, IFNULL, COALESCE (рос.)
