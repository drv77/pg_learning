# MVCC, vacuum и autovacuum

Подключаемся к виртуальной машине в Яндекс облаке и подготавливаем ее к выполнению домашнего задания

> sudo pg_ctlcluster 14 main stop

> sudo pg_dropcluster 14 main

> sudo pg_createcluster 14 main

> sudo pg_ctlcluster 14 main start

Устанавливаем параметры для кластера в файле **postgresql.conf** и перезагружаем службы postgres

>max_connections = 40
>
>shared_buffers = 1GB
>
>effective_cache_size = 3GB
>
>maintenance_work_mem = 512MB
>
>checkpoint_completion_target = 0.9
>
>wal_buffers = 16MB
>
>default_statistics_target = 500
>
>random_page_cost = 4
>
>effective_io_concurrency = 2
>
>work_mem = 6553kB
>
>min_wal_size = 4GB
>
>max_wal_size = 16GB

>>sudo service postgresql restart

Далее под пользователем **postgres** выполняем команду pgbench, предназначенную для тестирования производительности БД postgres

> sudo su postgres

> pgbench -i postgres

![image](https://user-images.githubusercontent.com/116566498/202180285-0f132265-b471-4a38-a60b-fec81e0d3cdf.png)

Запускаем команду

> pgbench -c8 -P 60 -T 600 -U postgres postgres

![image](https://user-images.githubusercontent.com/116566498/202182970-85fa3810-0408-49ed-bad7-ecb168a28ed8.png)

Посмотри текущие настройки autovacuum

> SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'autovacuum%';

![image](https://user-images.githubusercontent.com/116566498/202186535-02e80091-d4e8-4dbe-9d87-70d1bfca8fb5.png)

Теперь выставим в файле **postgresql.conf** агрессивные настройки autovacuum

![image](https://user-images.githubusercontent.com/116566498/202210852-a9dce62c-a4fd-4baa-94ff-b596af83a998.png)

Но, к сожалению, мне не удалось добиться максимально ровных значений tps. 
Результат нагрузочного тестирования после изменения настроек мало отличается от тестирования с настройками по умолчанию.

![image](https://user-images.githubusercontent.com/116566498/202211598-a8448883-3621-4a22-8ec4-2bedf012eb30.png)



