Завдання:

Використовуючи [безкоштовну хмарну версію MongoDB](https://www.mongodb.com/try)*, або розгорнувши [локальний кластер СУБД в Docker](https://hub.docker.com/_/mongo) виконайте наступні завдання:
### 1.	Створити базу даних з назвою, що відповідає вашій групі (приклад: ip96).
Створити колекцію назвою якої буде ваше прізвища та ініціали латиницею (nedashkivskyi_ya).
Переглянути існуючі в БД колекції.
```javascript
use it91
db.createCollection("terentiev_hv")
show collections
```
### 2.	Додати в новостворену колекцію дев’ять співробітників аналогічним тим, що знаходяться в таблиці Employees реляційної бази даних Northwind. 
Необхідні ключі: `LastName`, `FirstName`, `Title`, `HireDate`.
```javascript
db.terentiev_hv.insertMany([
	{
		"LastName":'Davolio',
		"FirstName":'Nancy',
		"Title":'Sales Representative',
		"HireDate":new Date("1992-05-01")
	},
	{
		"LastName":'Fuller',
		"FirstName":'Andrew',
		"Title":'Vice President, Sales',
		"HireDate":new Date('1992-08-14')
	},
	{
		"LastName":'Peacock',
		"FirstName":'Margaret',
		"Title":'Sales Representative',
		"HireDate":new Date('1993-05-03')
	},
	{
		"LastName":'Buchanan',
		"FirstName":'Steven',
		"Title":'Sales Manager',
		"HireDate":new Date('1993-10-17')
	},
	{
		"LastName":'Suyama',
		"FirstName":'Michael',
		"Title":'Sales Representative',
		"HireDate":new Date('1993-10-17')
	},
	{
		"LastName":'King',
		"FirstName":'Robert',
		"Title":'Sales Representative',
		"HireDate":new Date('1994-01-02')
	},
	{
		"LastName":'Callahan',
		"FirstName":'Laura',
		"Title":'Inside Sales Coordinator',
		"HireDate":new Date('1994-03-05')
	},
	{
		"LastName":'Dodsworth',
		"FirstName":'Anne',
		"Title":'Sales Representative',
		"HireDate":new Date('1994-11-15')
	},
	{
		"LastName":'Leverling',
		"FirstName":'Janet',
		"Title":'Sales Representative',
		"HireDate":new Date('1992-04-01')
	},
 ])
 ```
### 3.	Знайти один (якщо таких декілька) документ з відомостями про співробітника Robert King. 
```javascript
db.terentiev_hv.findOne({
	"LastName":'King',
	"FirstName":'Robert'
})
```
### 4.	Відобразити дані з усіма співробітникам, що мають посаду Sales Representative. Відсортувати результати за прізвищем.
```javascript
db.terentiev_hv.
	find({"Title":"Sales Representative"}).
	sort({"LastName":1})
```
### 5.	Знайти кількість співробітників, що приєдналися до компанії в 1993 році.
```javascript
db.terentiev_hv.countDocuments({
   "HireDate": {
	$gte: new Date("1993-01-01"),
	$lt: new Date("1994-01-01")
	}
})
```
або
```javascript
db.terentiev_hv.countDocuments({ "$expr": { "$eq": [{"$year":"$HireDate"}, 1993]}})
```
### 6.	Аналогічно до даних в реляційній базі Northwind усім співробітникам, що проживають в США додати ключ з країною проживання.
```javascript
db.terentiev_hv.updateMany(
	{
		$or: [
			{
				"LastName":'Davolio',
				"FirstName":'Nancy'
			},
			{
				"LastName":'Fuller',
				"FirstName":'Andrew'
			},
			{
				"LastName":'Peacock',
				"FirstName":'Margaret'
			},
			{
				"LastName":'Leverling',
				"FirstName":'Janet'
			},
			{
				"LastName":'Callahan',
				"FirstName":'Laura'
			}
		]
	},
	{ $set: {"Country":'USA'}}
)

db.terentiev_hv.updateMany(
	{
		$or: [
			{
				"LastName":'Buchanan',
				"FirstName":'Steven'
			},
			{
				"LastName":'Suyama',
				"FirstName":'Michael'
			},
			{
				"LastName":'King',
				"FirstName":'Robert'
			},
			{
				"LastName":'Dodsworth',
				"FirstName":'Anne'
			}
		]
	},
	{ $set: {"Country": "UK"}}
)
```

### 7.	Додати себе як директора з датою прийняття на роботу, що відповідає даті виконання завдання. Додатково вказати `Country`, `BirthDate` та `City` (місто проживання).
```javascript
db.terentiev_hv.insertOne({
	"LastName":'Terentiev',
	"FirstName":'Hlib',
	"Title":'Director',
	"HireDate": new Date("2021-12-14"),
	"Country":'UA',
	"BirthDate": new Date("2001-11-25"),
	"City":'Kyiv'
})
```
### 8.	Співробітнику Janet Leverling додати новий ключ, що буде включати три вкладених документи з обробленими замовленнями (ключі `OrderId`, `OrderDate`, `OrderSum`). Взяти будь-які три реальних замовлення з бази Northwind. 
```javascript
db.terentiev_hv.updateOne(
	{
		"LastName":'Leverling',
		"FirstName":'Janet',
	},
	{ $set: {
		"ProcessedOrders": [
			{
				"OrderID":"10251",
				"OrderDate": new Date("1996-07-08"),
				"OrderSum":41,
			},
			{
				"OrderID":"10253",
				"OrderDate": new Date("1996-07-10"),
				"OrderSum":102,
			},
			{
				"OrderID":"10256",
				"OrderDate": new Date("1996-07-15"),
				"OrderSum":27,
			},
		]
	}}
)
```
### 9.	Видалити усі документи, де ключ `Country` не існує.
```javascript
db.terentiev_hv.deleteMany({"Country": {$exists:false}})
```
### 10.	Видалити усі документи з колекції, потім саму колекцію та базу даних.
```javascript
db.terentiev_hv.deleteMany({})

db.terentiev_hv.drop()

db.dropDatabase()
```

*будь-ласка впевніться, що ви використовуєте Free Tier: M0 Sandbox (General) - AZURE Netherlands (westeurope).

### Матеріали для повторення:
[Cloud Databases (Lecture) pt9 (Документо-орієнтовані СУБД, MongoDB)](https://www.youtube.com/watch?app=desktop&v=FM0w4BVmxK4&feature=youtu.be)   
[Cloud Databases (practice) pt11 (Querying MongoDB)](https://www.youtube.com/watch?app=desktop&v=U1nUwwBN_VI&feature=youtu.be)   
[MongoDB University (англомовні курси від розробників СУБД)](https://university.mongodb.com/)   
