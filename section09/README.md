# Журналы 

Используем виртуальную машину, созданную для предудущих домашних заданий. Подключаемся к ней и пересоздаем кластер Postgres-14.

sudo pg_ctlcluster 14 main stop

sudo pg_dropcluster 14 main

sudo pg_createcluster 14 main

sudo pg_ctlcluster 14 main start

Настраиваем выполнение контрольной точки раз в 30 секунд. Для этого изменяем настройку в файле **postgres.conf**  и перезапускаеи службу **postgres** 

> checkpoint_timeout = 30s

> sudo service postgresql restart

Под пользователем postgres выполняем команду pgbench, предназначенную для тестирования производительности БД postgres

> sudo su postgres

> pgbench -i postgres

Изначально БД postgres имеет два файла журнала по 16 Мбайт каждый, которые находятся в папке pg_wal. 

В процессе выполнения нагрузочного тестирования видим, что у нас в папке имеется четыре файла по 16 Мбайт, то есть между двумя контрольными точками генерируеися 64 Мбайта данных.

![image](https://user-images.githubusercontent.com/116566498/202848650-82cbe73d-e9a9-4a4e-81a8-6e034a648041.png)

В итоге, исходя из имени файла, объем журнальных данных составляет 30 (1E) * 16 Мбайт = 480 Мбайт

Посмотри статистику выполнения контрольных точек

>  SELECT * FROM pg_stat_bgwriter \gx

![image](https://user-images.githubusercontent.com/116566498/202849261-a986f1de-9cfc-4dca-b162-7eeac1ba8f46.png)

Видим, что все кроме одной контрольные точки выполнялись по расписанию (checkpoints_timed).

Стравним работу базы данных в сиснхронном и асинхронном режимах. По умолчанию включен синхронный режим.

>  show synchronous_commit;

![image](https://user-images.githubusercontent.com/116566498/202849884-fac66a02-1944-428d-8637-9d60a97176a2.png)

Результат бенчмарка в синхронном режиме:

![image](https://user-images.githubusercontent.com/116566498/202849933-21189a15-09ee-4d17-8693-ebedace04e55.png)

Включаем режим асинхронной записи. Для этого изменяем настройку в файле **postgres.conf**  и перезапускаеи службу **postgres** 

> synchronous_commit = off;

![image](https://user-images.githubusercontent.com/116566498/202854589-d00ff40e-c31e-4ae5-b5bd-73a21c42f4f0.png)

Параметр tps увеличился в несколько раз. В режиме off отсутствует ожидание записи транзакций журнала WAL на диск.




