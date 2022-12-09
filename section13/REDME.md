# Нагрузочное тестирование и тюнинг PostgreSQL

Пересоздаем кластер в виртуальной машине из предыдущих домашних заданий с помощью команд **pg_dropcluster**, **pg_createclaster**

Подключаемся к кластеру и создаем базу данны **test**

Далее выполним нагрузочное тестирование с настройками по умолчанию с помощью утилиты pgbench. Инициализируем необходимые нам таблицы в БД

>pgbench -i test
> 
>pgbench -c 50 -j 2 -P 10 -T 30 test

Результаты тестирования:

![image](https://user-images.githubusercontent.com/116566498/206741379-7fa93586-9f04-488f-a46b-45b39138fff4.png)

Настроим кластер PostgreSQL 14 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины.

## Настройка с помощью утилиты [pgtune](https://pgtune.leopard.in.ua/#/)

Генерируем параметры кластера и прописываем их в отделтный файл. Ссылаемся на файл через параметр **include** гланого конфигурацтонного файла **postgresql.conf**

>max_connections = 100
>
>shared_buffers = 512MB
>
>effective_cache_size = 1536MB
>
>maintenance_work_mem = 128MB
>
>checkpoint_completion_target = 0.9
>
>wal_buffers = 16MB
>
>default_statistics_target = 100
>
>random_page_cost = 1.1
>
>effective_io_concurrency = 200
>
>work_mem = 1310kB
>
>min_wal_size = 1GB
>
>max_wal_size = 4GB

Перезапускаем кластер и выполняем повторное тестирование. Результат:

![image](https://user-images.githubusercontent.com/116566498/206745673-a632f62f-8480-40d5-9238-47d4afac2445.png)

Включаем режим асинхронной записи. Для этого изменяем добавляем настройку в файле, перезапускаеи службу postgres и повторяем тестирование.

> synchronous_commit = off

![image](https://user-images.githubusercontent.com/116566498/206747052-4342710e-1ff5-479f-b2eb-c1f17e945db4.png)

Количество обрабатываемых транзакций в секунду увеличилось.

