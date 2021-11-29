Завдання:

Використовуючи Azure SQL Database*:
### 1.	Створити базу даних, де назвою буде ваше прізвище. Впевнитись, що кодування бази даних підтримує українську мову.
```sql
create DATABASE Терентьєв
COLLATE SQL_Ukrainian_CP1251_CS_AS;
go
```

### 2.	Створити таблицю music, що матиме колонки з назвою виконавця, пісні, альбому та року релізу, самостійно обравши типи даних.

```sql
CREATE TABLE music (
    artist VARCHAR(20) NOT NULL,
    songName VARCHAR(20) NOT NULL,
    albumName VARCHAR(20),
    releaseYear INTEGER,
);
```

### 3.	Відредагувати створену таблицю додавши нову колонку з ідентифікатором, що буде первинним ключем та матиме властивість автоінкременту.
```sql
ALTER TABLE music ADD id INT PRIMARY KEY IDENTITY(1,1);
```
### 4.	Записати в таблицю music п’ять рядкова з піснями, що ви зараз слухаєте найчастіше. Серед них повинна бути хоча б одна з назвою чи виконавцем українською мовою.
```sql

INSERT INTO music (artist, songName)
VALUES 
('Воплі відоплясова', 'Весна'),
('ТНМК', 'Гранули'),
('AC/DC', 'TNT'),
('Metallica', 'Enter Sandman'),
('Rick Astley', 'Never gonna give up');
```

### 5.	Створити таблицю artists, що містить назви виконавців (музичних гуртів), країн їх походження, та рік заснування.
Додати до таблиці music нову колонку з ідентифікатором виконавця.
Заповнити відсутні значення шляхом пошуку відповідностей в назвах виконавців між таблицями.
Видалити назву виконавця з таблиці music.
Впевнитись, що неможливо додати посилання не неіснуючого виконавця.
```sql

CREATE TABLE artists (
    name VARCHAR(20) PRIMARY KEY NOT NULL,
    country VARCHAR(20),
    foundYear INT, 
);

ALTER TABLE music ADD artistId VARCHAR(20);

INSERT INTO artists (name) 
SELECT DISTINCT artist FROM music;

UPDATE music 
SET artistId = m.artist
FROM music AS m
WHERE artist = m.artist;

ALTER TABLE music DROP COLUMN artist;

ALTER TABLE music ADD CONSTRAINT fk_artist_name FOREIGN KEY (artistId) REFERENCES artists(name);

INSERT INTO music (songName, artistId)
VALUES ('Gimme!', 'ABBA');
```

### 6.	Створити представлення (view), що виводить всю інформацію про музичні треки виконавців зі Східної Європи.
```sql

UPDATE artists 
SET country = 'Ukraine'
WHERE name in ('Воплі відоплясова', 'ТНМК');

CREATE VIEW musicFromEastEurope
AS
SELECT 
    songName,
    artistId,
    albumName,
    releaseYear
FROM music
WHERE artistId in ( 
    SELECT name 
    FROM artists 
    WHERE country in (
                      'Ukraine',
                      'Belarus',
                      'Lithuania',
                      'Latvia',
                      'Estonia',
                      'Poland',
                      'Romania',
                      'Bulgaria',
                      'Slovakia',
                      'Hungary',
                      'Serbia',
                      'Croatia',
                      'Chezh Republic',
                      'Moldova'
    )
);
```

### 7.	Створити збережену процедуру, що у якості параметрів приймає назву треку та виконавця і заповнює відповідні таблиці. Не додавати записи, якщо вони вже є в базі.
```sql

CREATE PROCEDURE AddSong @trackName VARCHAR(20), @artist VARCHAR(20)
AS
BEGIN 
    IF NOT EXISTS (SELECT * FROM artists 
                    WHERE name = @artist)
    BEGIN
        INSERT INTO artists (name)
        VALUES (@artist)
    END

    IF NOT EXISTS (SELECT * FROM music
                    WHERE songName = @trackName 
                      AND artistId = @artist)
    BEGIN
        INSERT INTO music (songName, artistId)
        VALUES (@trackName, @artist)
    END
END
GO
```

### 8.	Створити скалярну функцію, що приймає ідентифікатори пісні та виконавця та повертає назву треку з конкатенацією альбому та року за зразком:
Shine On You Crazy Diamond (Wish You Were Here, 1975)
```sql

CREATE FUNCTION fn_GetSongWithAlbum (@songId INT, @artistId VARCHAR(20))
RETURNS VARCHAR(65)
AS 
BEGIN
    DECLARE @result VARCHAR(65)
    SELECT @result = (SELECT TOP 1 CONCAT(songName, ' (', COALESCE(albumName, 'Unknown album'), ', ', ISNULL(CONVERT(VARCHAR(4), releaseYear), 'N/A'), ')')
            FROM music
            WHERE id = @songId
            AND artistId = @artistId)
    RETURN @result
END
GO
```

### 9.	Створити табличну функцію, що повертає ті ж колонки, що і створене представлення, проте фільтрація по країнам відбувається згідно вхідного параметру.
```sql

CREATE FUNCTION fn_GetMusicByCountry (@country VARCHAR(20))
RETURNS TABLE 
AS 
RETURN SELECT 
            songName,
            artistId,
            albumName,
            releaseYear
        FROM music
        WHERE artistId in ( 
            SELECT name 
            FROM artists 
            WHERE country = @country)
GO
```

### 10.	Створити тригер, що при додаванні пісні буде перевіряти, що рік її релізу не менше за рік початку діяльності виконавця.
```sql

CREATE TRIGGER tr_CheckReleaseYear
ON music 
FOR INSERT
AS 
BEGIN 
    IF (SELECT (SELECT releaseYear FROM INSERTED) - (SELECT foundYear FROM artists WHERE name =(SELECT artistId FROM INSERTED))) < 0 
    BEGIN
        RAISERROR('release year is smaller that foundation year', 1, 0)
        ROLLBACK
    END
END
GO
ALTER TABLE music ENABLE TRIGGER tr_CheckReleaseYear
GO
```

**будь-ласка впевніться, що ви використовуєте Basic tier та видалили всі ресурси в кінці роботи*

Матеріали для повторення:  
[How to change an Azure SQL Database Collation](https://www.sqlshack.com/how-to-change-an-azure-sql-database-collation/)  
[CREATE TABLE (Transact-SQL) IDENTITY (Property)](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql-identity-property?view=sql-server-ver15)  
[Cloud Databases (practice) pt6 (INSERT, UPDATE, DELETE, TRUNCATE, CREATE TABLE)](https://www.youtube.com/watch?app=desktop&v=0QNTyaiUjYc&feature=youtu.be)  
[Cloud Databases (practice) pt7 (AZURE, PROCEDURE, UDF, SCALAR & TABLE FUNCTIONS)](https://www.youtube.com/watch?app=desktop&v=3ymEzDdgAN0&feature=youtu.be)  
[Cloud Databases (practice) pt8 (CURSOR, TRIGGER, QUERY OPTIMIZATION)](https://www.youtube.com/watch?app=desktop&v=OkqmeUfMVZg&feature=youtu.be)  
[Оператори INSERT, UPDATE, DELETE](http://fcit.tneu.edu.ua/navchannja/pidhotovka-do-pratsevlashtuvannia/56-web-rozrobka/mysql/579-mysql-insert)  
