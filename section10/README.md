Используем виртуальную машину, созданную для предудущих домашних заданий. Подключаемся к ней и пересоздаем кластер Postgres-14.

Настраиваем сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд.

> ALTER SYSTEM SET log_lock_waits = on;

> ALTER SYSTEM SET deadlock_timeout = 200;

> select pg_reload_conf();

Создадим таблицу

> CREATE TABLE accounts(acc_no integer PRIMARY KEY, amount numeric);
> 
> INSERT INTO accounts VALUES (1, 100.00), (2, 200.00), (3, 300.00);

Запускаем один и тот же запрос, обновляющий данные в одной строке, в двух сенсах

> BEGIN;
> UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;

Видим, что в логе появилась запись об этом

![image](https://user-images.githubusercontent.com/116566498/203547736-8127304a-9595-44c8-8ed3-4fe452beed65.png)

Выведем информацию из таблицы блокировок

> select * from pg_locks WHERE relation = 'accounts'::regclass;

![image](https://user-images.githubusercontent.com/116566498/203554802-1f598e1e-f33f-41d9-9a26-672ec554b4bd.png)

Или в более удобном виде

> SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'accounts'::regclass;

![image](https://user-images.githubusercontent.com/116566498/203555010-30713546-f4d6-46b9-9956-395a5d4594c8.png)

Из результата запроса видим, что транзакция из сеанса 2063 ждет завершения транзакции сеанса 1616, а из сеанса 2080 из 2063.

также в ситуации можно разобраться с помощью информации из лога

![image](https://user-images.githubusercontent.com/116566498/203556867-01c17f23-7bd1-4d97-a835-78accf77e3c9.png)



