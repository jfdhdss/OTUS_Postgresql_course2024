1. Создал в yc виртуальную машину
![vm](D:\Обучение\ДЗ1\vm.jpg)

2. Подключился к postgresql
	```
    kda@otus-db-kda-postgre:~$ psql -h 158.160.164.62 -U postgres
	Password for user postgres:
	psql (16.2 (Ubuntu 16.2-1.pgdg20.04+1))
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	Type "help" for help.
	postgres=# \l
														   List of databases
	   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	(3 rows)
    ```

3. Создал бд для работы
	```
    postgres=# create database work01;
	CREATE DATABASE
	postgres=# \l
														   List of databases
	   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 work01    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	(4 rows)
    ```
	
4. Создал таблицу для дз, наполнил данными, проверил текущий уровень изоляции. Read commited
	```
    postgres=# \c work01
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	You are now connected to database "work01" as user "postgres".
	work01=# \dt
	Did not find any relations.
	work01=# \set AUTOCOMMIT OFF
	work01=# create table persons(id serial, first_name text, second_name text);
	CREATE TABLE
	work01=# \dt
			  List of relations
	 Schema |  Name   | Type  |  Owner
	--------+---------+-------+----------
	 public | persons | table | postgres
	(1 row)
		work01=# insert into persons(first_name, second_name) values('ivan', 'ivanov');
	INSERT 0 1
	work01=# insert into persons(first_name, second_name) values('petr', 'petrov');
	INSERT 0 1
	work01=# commit;
	work01=# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	(2 rows)
	work01=# show transaction isolation level;
	 transaction_isolation
	-----------------------
	 read committed
	(1 row)
    ```

5. В первой сессии вставили строку
	```
    work01=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
	INSERT 0 1
    ```

6. Select во второй сессии. Вставленную строку в первой сессии не видим, потому что при таком уровне изоляции не видим изменений в не завершенных транзакциях.
	```
    work01=# show transaction isolation level;
	 transaction_isolation
	-----------------------
	 read committed
	(1 row)
		work01=# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	(2 rows)
    ```

7. Делаем commit в первой сессии и выбираем еще раз select во второй. Строка появляется. Первая транзакция завершена, изменения видны.
	```
    work01=# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	  3 | sergey     | sergeev
	(3 rows)
    ```

8. Изменил уровень изоляции на repeatable read
	```
    work01=# set transaction isolation level repeatable read;
	SET
	work01=*# SHOW TRANSACTION ISOLATION LEVEL;
	 transaction_isolation
	-----------------------
	 repeatable read
	(1 row)
    ```

9. В первой сессии вставляем строку
	```
    work01=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
	INSERT 0 1
    ```

10. Во второй сессии делаем select. Строку не видим. При этом уровне, видим данные на начало транзакции
	```
    work01=*# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	  3 | sergey     | sergeev
	(3 rows)
    ```

11. Делаем commit в первой сессии и выполняем select во второй. Строку не видим. При этом уровне, видим данные на начало транзакции.	
	```
    work01=*# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	  3 | sergey     | sergeev
	(3 rows)
    ```

12. Делаем commit во второй сессии и выполняем select. Строку видим. Завершили транзакцию и увидели изменения.	
	```
    work01=# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | ivan       | ivanov
	  2 | petr       | petrov
	  3 | sergey     | sergeev
	  4 | sveta      | svetova
	(4 rows)
    ```






