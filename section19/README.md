# Сбор и использование статистики

## Подготовка

Используем для выполнения задания демонстрационную БД с сайта PostgresPro. Скачиваем по ссылке https://edu.postgrespro.ru/demo-medium.zip

Подключемся к БД postgres, создаем БД demo и восстанавливаем ее из файла:

> CREATE DATABASE DEMO;

> psql demo < /tmp/demo-medium-20170815.sql

Далее для удобства установим DBeaver и подключимся к виртуальной машине в яндекс-облаке по ip-адресу. Для этого необходимо также установить пароль роли БД postgres:

> ALTER USER postgres PASSWORD 'postgres';

Предварительно удалим два индекса из таблицы БД flights. Команда для примера:

> ALTER TABLE bookings.flights DROP CONSTRAINT flights_pkey CASCADE;

## Использование индекса

Сначала выполним поиск по таблице без использования индекса

> explain

> SELECT flight_id, flight_no, scheduled_departure, scheduled_arrival, departure_airport, arrival_airport, status, aircraft_code, actual_departure, actual_arrival

> FROM bookings.flights

> where departure_airport = 'SVO';

Результат запроса:
> Seq Scan on flights  (cost=0.00..1612.80 rows=5919 width=63)

Видим, что используется последовательный обход строк таблицы.

![image](https://user-images.githubusercontent.com/116566498/210939198-8f11304c-8bd3-47e7-ad73-dd8d2da2d8b4.png)

Создаем индекс на поле departure_airport

> CREATE INDEX flights_departure_airport_idx ON bookings.flights (departure_airport);

Повторно выполним запрос и посмотрим на результат:
![image](https://user-images.githubusercontent.com/116566498/210941025-53c3a285-bf09-4272-b0e8-31da542c82bc.png)

Видим, что теперь для выборки строк используется индекс.

## Реализация индекса для полнотекстового поиска

Выбираем дял работы с полнотекстовыми индексами таблицу tickets и удаляем текущие индексы:

 > ALTER TABLE bookings.tickets DROP CONSTRAINT tickets_pkey CASCADE;

Выполняем поиск без индекса:
> explain

> SELECT ticket_no, passenger_name, contact_data

> FROM bookings.tickets

> where passenger_name like '%ELENA AFANASEVA%';

Результат:

![image](https://user-images.githubusercontent.com/116566498/210943621-e48ca5cd-f402-426f-9c79-9ca090c05669.png)

Теперь создаем индекс с типом gin
> CREATE EXTENSION pg_trgm;

> CREATE EXTENSION btree_gin;

> CREATE INDEX passenger_name_idx ON bookings.tickets using GIN (passenger_name gin_trgm_ops);

Повторно выполняем запрос и получаем результат:

![image](https://user-images.githubusercontent.com/116566498/210948344-3867e3d7-626a-4620-8a66-abffe32737c4.png)







