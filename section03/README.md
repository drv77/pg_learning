# Установка PostgreSQL

Устанавливаем виртуальную машину с Ubuntu 22.04 в Яндекс облаке со стандартными настройками. Натсраиваем подключение ssh, как в домашнем задании №1. Устанавиваем docker
> sudo curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER

Создаем каталог для файлов Postgres
> sudo mkdir /var/lib/postgres

Создаем docker-сеть
> sudo docker network create pg-net

Разворачиваем контейнер сервера Postgres с подключением к созданной сети
> sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

Запускаем отдельный контейнер с клиентом в общей сети с БД
> sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres

Создаем базу БД fio и добавляем в нее пару строк (как в предыдущем домашнем задании)
> CREATE DATABASE fio;
>
> \c fio;
>
> create table persons(id serial, first_name text, second_name text);
>
> insert into persons(first_name, second_name) values('ivan', 'ivanov');
> 
> insert into persons(first_name, second_name) values('petr', 'petrov');

Подключаемся к контейнеру с сервером с внешней консоли (в моем случае с подсистемы linux в Windows 10)
> psql -p 5432 -U postgres -h 51.250.110.41 -d postgres -W

Подключение успешно выполнено.
![image](https://user-images.githubusercontent.com/116566498/198812501-10b86c7d-c961-4e29-b133-e6c79aacd636.png)
