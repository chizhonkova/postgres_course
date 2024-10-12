# Homework 3

1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк:
```
CREATE TABLE generated_texts
(
    id SERIAL PRIMARY KEY,
    text TEXT
);

INSERT INTO generated_texts (text)
SELECT md5(random()::text)
FROM generate_series(1, 1000000);

SELECT count(*) FROM generated_texts;
  count  
---------
 1000000
(1 row)
```

2. Посмотреть размер файла с таблицей:
```
SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 87 MB
(1 row)
```

3. 5 раз обновить все строчки и добавить к каждой строчке любой символ:
```
5 times: UPDATE generated_texts SET text = md5(random()::text);
UPDATE generated_texts SET text = concat(md5(random()::text), 'a');
```

4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
автовакуум:
```
SELECT last_autovacuum, n_dead_tup, relname from pg_stat_all_tables WHERE relname = 'generated_texts';
       last_autovacuum        | n_dead_tup |     relname     
------------------------------+------------+-----------------
 2024-10-12 15:45:15.52197+00 |    2001511 | generated_texts

SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 505 MB
(1 row)
```

5. Подождать некоторое время, проверяя, пришел ли автовакуум:
```
SELECT last_autovacuum, n_dead_tup, relname from pg_stat_all_tables WHERE relname = 'generated_texts';
        last_autovacuum        | n_dead_tup |     relname     
-------------------------------+------------+-----------------
 2024-10-12 15:45:59.123418+00 |          0 | generated_texts
(1 row)

SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 505 MB
(1 row)
```

6. 5 раз обновить все строчки и добавить к каждой строчке любой символ:
```
5 times: UPDATE generated_texts SET text = md5(random()::text);
UPDATE generated_texts SET text = concat(md5(random()::text), 'a');
```

7. Посмотреть размер файла с таблицей:
```
SELECT last_autovacuum, n_dead_tup, relname from pg_stat_all_tables WHERE relname = 'generated_texts';
        last_autovacuum        | n_dead_tup |     relname     
-------------------------------+------------+-----------------
 2024-10-12 15:50:01.369662+00 |    4007093 | generated_texts
(1 row)

SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 512 MB
(1 row)

VACUUM FULL generated_texts;
SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 87 MB
(1 row)
```

8. Отключить Автовакуум на конкретной таблице:
```
ALTER TABLE generated_texts SET (autovacuum_enabled = false);
```

9. 10 раз обновить все строчки и добавить к каждой строчке любой символ:
```
10 times: UPDATE generated_texts SET text = md5(random()::text);
UPDATE generated_texts SET text = concat(md5(random()::text), 'a');
```

10. Посмотреть размер файла с таблицей:
```
SELECT pg_size_pretty(pg_total_relation_size('generated_texts'));
 pg_size_pretty 
----------------
 846 MB
(1 row)

SELECT last_autovacuum, n_dead_tup, relname from pg_stat_all_tables WHERE relname = 'generated_texts';
        last_autovacuum        | n_dead_tup |     relname     
-------------------------------+------------+-----------------
 2024-10-12 15:51:11.683073+00 |   10998690 | generated_texts
(1 row)
```

11. Объясните полученный результат:

    В случае, когда autovacuum был включен, он помечал место, которое занимали мертвые строки, как свободное, и новые вставки заполняли его же, поэтому размер не превышал ~500MB. Также размер не уменьшался до 87MB, потому что autovacuum не возвращает эту память операционной системе, только помечает ее как свободную.

    В случае, когда autovacuum был выключен, мертвые строки не очищались, их примерное число составило 10998690.
