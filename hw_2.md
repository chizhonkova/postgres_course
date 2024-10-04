# Homework 2

1. В первой сессии создать таблицу и наполнить данными:
```
CREATE TABLE developers (name TEXT);
INSERT INTO developers
VALUES
    ('Peter Eisentraut'),
    ('Andres Freund'),
    ('Magnus Hagander'),
    ('Jonathan Katz'),
    ('Tom Lane');
SELECT * FROM developers;
```
Результат:
```
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
(5 rows)
```

2. Посмотреть текущий уровень изоляции:
```
postgres=> SHOW TRANSACTION ISOLATION LEVEL;
 transaction_isolation 
-----------------------
 read committed
(1 row)
```

3. В обеих сессиях начать транзакции:
```
BEGIN;
```

4. В первой сессии добавить запись:
```
postgres=*> INSERT INTO developers VALUES ('Bruce Momjian');
INSERT 0 1
postgres=*> select * from developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
 Bruce Momjian
(6 rows)
```

5. Во второй сессии запросить все записи:
```
postgres=*> SELECT * FROM developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
(5 rows)
```

6. Запись во второй сессии не видна, потому что уровень изоляции read committed не позволяет увидеть незакоммиченные данные.

7. Завершить транзакцию в первой сессии:
```
COMMIT;
```

8. Во второй сессии запросить все записи:
```
postgres=*> SELECT * FROM developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
 Bruce Momjian
(6 rows)
```

9. Новая запись видна, потому что данных уровень изоляции позволяет увидеть данные, которые были закоммичены другими транзакциями до начала запроса. Цитата из документации: `When a transaction uses this isolation level, a SELECT query sees only data committed before the query began`.

10. Завершить транзакцию во второй сессии:
```
COMMIT;
```

11. Начать в обеих сессиях транзакции с уровнем изоляции repeatable read:
```
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

12. В первой сессии добавить новую запись:
```
postgres=*> INSERT INTO developers VALUES ('Dave Page');
INSERT 0 1
postgres=*> SELECT * FROM developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
 Bruce Momjian
 Dave Page
(7 rows)
```

13. Сделать выбор всех записей во второй сессии:
```
postgres=*> SELECT * FROM developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
 Bruce Momjian
(6 rows)
```

14. Запись во второй сессии не видна, потому что уровень изоляции repeatable read не позволяет увидеть незакоммиченные данные.

15. Завершить транзакцию в первой сессии:
```
COMMIT;
```

16. Сделать выбор всех записей во второй сессии:
```
postgres=*> SELECT * FROM developers;
       name       
------------------
 Peter Eisentraut
 Andres Freund
 Magnus Hagander
 Jonathan Katz
 Tom Lane
 Bruce Momjian
(6 rows)
```

17. Новая запись не видна, потому что уровень изоляции repeatable read позволяет видеть только те данные, которые были закоммичены до начала транзакции. Цитата: `The Repeatable Read isolation level only sees data committed before the transaction began; it never sees either uncommitted data or changes committed by concurrent transactions during the transaction's execution.`
