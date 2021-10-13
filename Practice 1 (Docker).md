Завдання
1.	Встановити Docker та будь-яке графічне середовище адміністрування СУБД на вибір на вашому локальному ПК (SQL Server Management Studio, MySQL Workbench, pgAdmin).
```bash
sudo apt-get install docker
```
 
2.	Скачати один з наступних Linux-образів на вибір:
•	Microsoft SQL Server
•	Mysql
•	Postgres
```bash
docker pull postgres
```
3.	Переглянути всі скачані образи.
 
4.	Створити через консоль на вашому локальному ПК папку, назва якої є конкатенацією перших двох літер вашого імені та чотирьох літер вашого прізвища згідно закордонного паспорту, або якщо його нема – згідно офіційної української паспортної транслітерації.
Приклад: Yevhen Nedashkivskyi -> D:\yeneda
```bash
mkdir hltere
```
 
5.	Запустити контейнер з обраною СУБД під’єднавши до нього створену директорію в /var/lib. 
```bash
docker run --name postgre -e POSTGRES_PASSWORD=123  --mount type=bind,source=/home/hlib/sem5/cloud,target=/var/lib/postgresql/data posgres
```
6.	Зупинити контейнер.
```bash
docker stop postgre
```
7.	Вивести список всіх існуючих контейнерів.
```bash
docker container ls --all
``` 
8.	Повторно запустити контейнер в відкріпленому (detached) режимі, вказавши в якості зовнішнього порту для СУБД число, що дорівнює 50000 + ваш порядковий номер в групі.
```bash
docker run --name postgre –p 50018:5432 –d -e POSTGRES_PASSWORD=123  --mount type=bind,source=/home/hlib/sem5/cloud,target=/var/lib/postgresql/data posgres
``` 
9.	Під’єднатися до СУБД в контейнері з-за допомогою встановленого локально середовища адміністрування та розгорнути базу [Northwind](https://code.google.com/archive/p/northwindextended/downloads).
 
 
10.	З консолі виконати команду SELECT для будь-якої з таблиць розгорнутої бази.
```sql
select * from categories limit 10;
``` 



Матеріали для повторення:   
Відеозапис: [Cloud Databases (practice) pt1](https://youtu.be/7j4KStGX1j4)   
Корисні посилання      
[Docker CLI](https://docs.docker.com/engine/reference/run/)

