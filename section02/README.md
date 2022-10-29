# Домашнее задание. SQL и реляционные СУБД. Введение в PostgreSQL

## Создание и подготовка виртуальной машины
Для выполнения задания используем Яндекс облако. Создаем проект postgres2022-197704

Далее создаем инстанс виртуальной машины на Ubuntu 22.04 LTS с названием postgres и настройками по умолчанию. При создании машины необходимо ввести сгенерированный ключ для ssh-подключения. Для генерации ключа и подключения к виртуальной машине устанавливаем на своей учебной машине также Ubuntu 22.04.1 из магазина приложений Microsoft store. 

Генерируем ключ:
> sudo apt update
>
> sudo apt upgrade
>
> ssh-keygen -t rsa
>
> eval `ssh-agent -s`
>
> ssh-add .ssh/id_rsa
>
> cat .ssh/id_rsa.pub

Копируем с экрана ключ и вставляем в окне создания виртуальной машины. Подключаемся:
> ssh roman@51.250.110.45

Устанавливаем postgres:
> sudo apt update
> 
> sudo apt upgrade
>
>  sudo apt-get -y install postgresql

## Подготовка базы данных

Заходим в двух консолях под пользователем postgres
> sudo -u postgres psql

Отключаем auto commit в обеих консолях:
> \set AUTOCOMMIT OFF

Создаем в первой сессии базу с таблицей и заполняем ее:
> begin;

> CREATE DATABASE fio;
>
> \c fio;
>
> create table persons(id serial, first_name text, second_name text);
> 
> insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
> 
> insert into persons(first_name, second_name) values('petr', 'petrov'); 
> 
> commit;

Смотрим текущий уровень изоляции транзакций:
> show transaction isolation level;

По умолчанию установлен уровень изоляции транзакций _read commited_

## Проверка выполнения транзакций БД при уровне изоляции _read commited_

В первой сессии выполняем:
> begin;

> insert into persons(first_name, second_name) values('sergey', 'sergeev');

Во второй сессии:
> select * from persons;

В выводе результатов запроса новая запись не видна, так как в первой сесси не выполнена команда COMMIT

Если в первой консоли теперь завершить транзакцию командой COMMIT и выполнить SELECT повторно, то запись появляется в результе выполнения запроса.

## Проверка выполнения транзакций БД при уровне изоляции транзакций _repeatable read_

В обеих консолях начинаем транзцакции с уровнем изоляции _repeatable read_:
> begin;
>
> set transaction isolation level repeatable read;

Далее в первой консоли:
> insert into persons(first_name, second_name) values('sveta', 'svetova');
>
> commit;

Во второй консоли:
> select * from persons;

Новая запись не отображается в результатах вывода команды SELECT, так как на данном уровне изоляции транзакций отображаются данные в момент начала транзакции во второй консоли. В этот момент новые данные еще не были добавлены в таблицу. Мне лучше всего этот процесс помогла понять идея снапшотов.

Завершаем транзакцию во второй консоли командой COMMIT и выполняем повторно SELECT:
> select * from persons;

Теперь новая запись отображается в результатах, так как на данный момент данные уже содержатся в таблице.

[Видеоролик](https://www.youtube.com/watch?v=gOB3hpAVIIQ&t=319s), объясняющий уровни изоляции транзакций.
