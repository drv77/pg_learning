# Секционирование

Разворачиваем демонстрационную БД 

> su postgres\
> psql\
> create database demo;\
> ALTER USER postgres PASSWORD 'postgres';\
> psql demo < /tmp/demo-medium-20170815.sql\

Устанавливаем расширение timescaldb с помощью скрипта на странице https://packagecloud.io/timescale/timescaledb

> curl -s https://packagecloud.io/install/repositories/timescale/timescaledb/script.deb.sh | sudo bash \
> sudo apt update \
> sudo apt install timescaledb-2-postgresql-14 \
> sudo timescaledb-tune \
> sudo systemctl restart postgresql \
> sudo -u postgres psql \
> CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE; \

![image](https://user-images.githubusercontent.com/116566498/213841379-9aa0bb78-7007-413b-a5e9-cac082f9ee83.png)

Выполняем секционирование таблицы flights по статье https://docs.timescale.com/timescaledb/latest/how-to-guides/migrate-data/same-db/

> CREATE TABLE flights_new (LIKE flights INCLUDING DEFAULTS INCLUDING CONSTRAINTS INCLUDING INDEXES); \
> SELECT create_hypertable('flights_new', 'scheduled_departure', chunk_time_interval => INTERVAL '1 week'); \
> INSERT INTO flights_new SELECT * FROM flights; \

При выполнении create_hypertable возникает ошибка, связанная с созданием уникального индекса в таблице. описание проблемы в стсатье https://docs.timescale.com/timescaledb/latest/how-to-guides/hypertables/hypertables-and-unique-indexes/
Пока не знаю как правильно решить данную проблему, но для выполнения задачи добавим в индес исходной таблицы столбец, по которому выполняем партицирование таблицы, и завершим секционирование.

> ALTER TABLE bookings.flights DROP CONSTRAINT flights_pkey CASCADE; \
> ALTER TABLE bookings.flights ADD CONSTRAINT flights_pkey PRIMARY KEY (flight_id, scheduled_departure); \
> SELECT create_hypertable('flights_new', 'scheduled_departure', chunk_time_interval => INTERVAL '1 week'); \

![image](https://user-images.githubusercontent.com/116566498/213843681-ff1a328e-b59f-4a82-bbda-e384c7a81602.png)

Заменим исходную таблицу на новую.

> DROP TABLE flights CASCADE; \
> ALTER TABLE flights_new RENAME TO flights; \

Проверяем

> select * from flights;

![image](https://user-images.githubusercontent.com/116566498/213843985-b580c511-127b-46ae-abcd-fc38356dbac9.png)









