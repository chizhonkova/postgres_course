# Homework 4

1. Создать таблицу и наполнить данными:
```
CREATE TABLE accounts (id INTEGER, account NUMERIC);
INSERT INTO accounts VALUES (0, 0), (1, 0), (2, 0);
SELECT * FROM accounts;
 id | account 
----+---------
  0 |       0
  1 |       0
  2 |       0
(3 rows)
```

2. Добиться дедлока.

В первой сессии:
```
BEGIN;
SELECT * FROM accounts WHERE id = 0 FOR UPDATE;
UPDATE accounts SET account = 2 WHERE id = 1;
```

Во второй сессии:
```
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET account = 3 WHERE id = 0;
```

Результат:
```
ERROR:  deadlock detected
DETAIL:  Process 114885 waits for ShareLock on transaction 1015; blocked by process 
```