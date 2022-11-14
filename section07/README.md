# Логический уровень PostgreSQL

Используем виртуальную машину, созданную для предудущих домашних заданий. Подключаемся к ней и пересоздаем кластер Postgres-14.

> sudo pg_ctlcluster 14 main stop

> sudo pg_dropcluster 14 main

> sudo pg_createcluster 14 main

> sudo pg_ctlcluster 14 main start

Заходим в кластер под пользователем **postgres** и создаем БД **testdb**

> sudo -u postgres psql

> CREATE DATABASE testdb;

\c testdb

Создаем новую схему **testnm** и новую таблицу **t1** с одной колонкой **c1** типа **integer**. Вставляем в таблицо одно значение.

> CREATE SCHEMA testnm;

> CREATE TABLE t1(c1 integer);

> INSERT INTO t1 values(1);

Создаем новую роль **readonly** и даем новой роли право на подключение к базе данных **testdb**, даем новой роли право на использование схемы **testnm**,
даем новой роли право на **select** для всех таблиц схемы **testnm**

> CREATE role readonly;

> grant connect on DATABASE testdb TO readonly;

> grant usage on SCHEMA testnm to readonly;

> grant SELECT on all TABLEs in SCHEMA testnm TO readonly;

Создаем пользователя **testread** с паролем **test123**

> CREATE USER testread with password 'test123';

Дайем роль **readonly** пользователю **testread**

> grant readonly TO testread;

Заходим под пользователем **testread** в базу данных **testdb** и пробуем выполнить запрос

> \c testdb testread

И получаем ошибку

![image](https://user-images.githubusercontent.com/116566498/201661309-90940908-85d0-43ee-a8e4-68733031108b.png)

Необходимо в файле **pg_hba.conf** изменить настройку peer на md5 для локального входа для всех пользователей кроме **postgres**, и перезапускаем службу postgres

> sudo service postgresql restart

Повторяем подключение

> \c testdb testread
 
Выполняем запрос

> SELECT * FROM t1;

В результате видим, что у пользователя **testread** нет прав чтения таблицы

![image](https://user-images.githubusercontent.com/116566498/201664402-a4e4091b-9535-4490-94ce-9465bd00e471.png)

Так получилось потому, что права мы дали на схему **testnm**, а таблицу создали в схеме **public**

исправляем

> \c testdb postgres

> drop TABLE t1;

> CREATE TABLE testnm.t1(c1 integer);

> INSERT INTO testnm.t1 values(1);

Выполняем запрос

> SELECT * FROM testnm.t1;

![image](https://user-images.githubusercontent.com/116566498/201665537-29c5fc8d-3a70-49bc-9f3e-4da5cd799746.png)

Упользователя **testread** опять нет прав чтения таблицы, так как новая таблица создана позже назначения прав. Для исправления проблемы выплняем команды

> \c testdb postgres;

> ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;

> \c testdb testread;

Проверяем еще раз

> SELECT * FROM testnm.t1;

![image](https://user-images.githubusercontent.com/116566498/201668531-789e9b94-bec6-401a-9696-eca1c0317394.png)

Данные из таблицы не считались, так как команда **ALTER default** действеут только на вновь создаваемые таблицы. Необходимо еще раз пересоздать таблицу.

После пересоздания таблицы данные считались.

![image](https://user-images.githubusercontent.com/116566498/201669430-66f227bf-e5e7-4e4c-a8a6-9b3a753fdb64.png)

Попробуем выполнить команды

> create table t2(c1 integer); 
> 
> insert into t2 values (2);

![image](https://user-images.githubusercontent.com/116566498/201673493-42b7d2e3-02e0-40ad-a6e2-7636d38b174a.png)

Видим, что таблица принадлежит схеме **public**. Так получось потому, что при создании таблицы не указана схема. 
Схема **public** создается в каждой базе данных и по умолчанию таблица создается в данной ней, если указать конкретную схему.

> \c testdb postgres; 
> revoke CREATE on SCHEMA public FROM public; 
> revoke all on DATABASE testdb FROM public; 
> \c testdb testread; 

![image](https://user-images.githubusercontent.com/116566498/201677688-4c26e684-5b86-4172-948a-79982ee56a77.png)

Теперь не получается создать таблицу в схеме **public**, так как нет полномочий.

![image](https://user-images.githubusercontent.com/116566498/201678210-fc2573e6-bfdb-4d5d-99fd-8b18c1da63ef.png)

Данные вставились в таблицу **t2**, находящейся в схеме **public**, так как у роли **testread** есть полномочия на запись в схеме **public** (запись запрещена только в схеме **testnm**).









