# Физический уровень PostgreSQL

Используем виртуальную машину, созданную для предудущих домашних заданий. Подключаемся к ней и производим установку Postgres-14.
> apt -y install postgresql-14

Проверяем наличие запущенного кластера командой **pg_lsclusters**.
И если кластера нет, то его необходимо создать  и запустить:
> sudo pg_createcluster 14 main

> sudo pg_cluster 14 main start

Кластер создан и работает:

![image](https://user-images.githubusercontent.com/116566498/200862464-994106eb-0847-49da-8314-86dd859960df.png)

Заходим из под пользователя postgres в psql и создаем таблицу с произвольным содержимым.
> create table test(c1 text);
> 
> insert into test values('1');

Проверяем создание таблицы запросом:
> select * from test;

Останавливаем кластер postgres:
> sudo -u postgres pg_ctlcluster 14 main stop

Создаем дополнительный диск для ВМ.

![image](https://user-images.githubusercontent.com/116566498/200866011-787b53f7-426d-4040-a67e-b43b13df49be.png)

Подключаем диск к ВМ

![image](https://user-images.githubusercontent.com/116566498/200866743-4296d82c-9ca3-44e2-9c0d-dbe7ce9a49d3.png)

![image](https://user-images.githubusercontent.com/116566498/200867078-ffa36dfa-9219-4305-bce6-3f8b2f6cac7a.png)

Инициализируем новый диск
> sudo parted /dev/vdb mklabel msdos
> 
> sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%

> sudo mkfs.ext4 -L datapartition /dev/vdb1

Монтируем диск
> sudo mkdir -p /mnt/data

> sudo mount -o defaults /dev/vdb1 /mnt/data

Проверяем командой **df -h**

![image](https://user-images.githubusercontent.com/116566498/200869783-4ffab474-6799-45d6-b997-8be644a60c66.png)

Смотрим UUID дискакомандой **sudo lsblk --fs** и добавляем строку в файле /etc/fstab для того, чтобы псоле перезагрузки ВМ диск автоматически подключался.
> UUID=d5bfe129-5027-4243-8794-8eb27c42ea4b /mnt/data ext4 defaults 0 2

Делаем пользователя postgres владельцем /mnt/data 
> sudo chown -R postgres:postgres /mnt/data/

Переносим содержимое /var/lib/postgres/14 в /mnt/data 
> sudo mv /var/lib/postgresql/14 /mnt/data

Стартуем кластер
>sudo -u postgres pg_ctlcluster 14 main start

![image](https://user-images.githubusercontent.com/116566498/200873394-19895906-6fb4-409f-90b4-67b17f0f9fa3.png)

Кластер не стартует, потому что в настройках postgres содержится ссылка на старое расположение файлов кластера.
Изменим настройку *data_directory = '/var/lib/postgresql/14/main'* в файле /etc/postgresql/14/main/postgresql.conf на значение *data_directory = '/mnt/data/14/main'*
Еще раз стартуем кластер. На этот раз кластер запустился.

![image](https://user-images.githubusercontent.com/116566498/200875177-7eea5575-53a0-4f4b-8cc0-824dadd49b7d.png)

Запускаем psql и проверяем данные
> select * from test;

Данные на месте

![image](https://user-images.githubusercontent.com/116566498/200875737-54442993-b9fa-4073-9986-ffee61d87dcc.png)




