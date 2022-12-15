# Виды и устройство репликации в PostgreSQL

Пересоздаем кластер в виртуальной машине из предыдущих домашних заданий с помощью команд pg_dropcluster, pg_createclaster

Подключаемся к кластеру и устанавливаем логический уровень репликации и рестартуем кластер

> ALTER SYSTEM SET wal_level = logical;

> sudo pg_ctlcluster 14 main restart

Создаем базу данны cluster1 и таблицы test для записи и test2 для чтения

> CREATE DATABASE cluster1;

> create table test (id integer, fio char(10));

> create table test2 as select generate_series(1,10) as id, md5(random()::text)::char(10) as fio;

Создаем второй кластер main1 с базой данных cluster2 и также две таблицы - test для чтения и test2 для записи>

> sudo pg_createcluster 14 main1

> sudo pg_ctlcluster 14 main1 start

> su postgres

> psql -p 5433

> CREATE DATABASE cluster2;

> create table test2 (id integer, fio char(10));

> create table test as select generate_series(1,10) as id, md5(random()::text)::char(10) as fio;

> ALTER SYSTEM SET wal_level = logical;

> sudo pg_ctlcluster 14 main1 restart

## Репликация cluster1 -> cluster2

На первом кластере создаем публикацию:
> sudo -u postgres psql -p 5432

> \c cluster1;

> CREATE PUBLICATION cluster1 FOR TABLE test2;

Задаем пароль
> \password

Переключаемся во второй кластер и создаем подписку

> sudo -u postgres psql -p 5433

> CREATE SUBSCRIPTION test_sub 
CONNECTION 'host=localhost port=5432 user=postgres password=postgres dbname=cluster1' 
PUBLICATION cluster1 WITH (copy_data = true);

Посмотрим результат

![image](https://user-images.githubusercontent.com/116566498/207865002-5e8f3cd4-a12d-4141-a50a-d177233f137f.png)

## Репликация cluster2 -> cluster1

Создадим публикацию на втором кластере

> CREATE PUBLICATION cluster2 FOR TABLE test;

Задаем пароль
> \password

Переключаемся в первый кластер и создаем подписку

> sudo -u postgres psql -p 5432

> \c cluster1;

> CREATE SUBSCRIPTION test_sub 
CONNECTION 'host=localhost port=5433 user=postgres password=postgres dbname=cluster2' 
PUBLICATION cluster2 WITH (copy_data = true);

Смотрим результат

![image](https://user-images.githubusercontent.com/116566498/207866454-068ccfe9-dc17-4611-ba2a-c0498e7b2a1b.png)

Видим, что данные таблиц среплицированы.



