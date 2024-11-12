# Homework 8

1. Сгенерировать таблицу с JSONB документами:
```
CREATE TABLE jsonb_data AS
SELECT id, (SELECT jsonb_object_agg(j, 2 * j) FROM generate_series(1, 1000) j) AS data
FROM generate_series(1, 10000) id;

SELECT relname AS toast_table_name,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
       pg_size_pretty(pg_relation_size(oid)) AS relation_size,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_size,
       pg_size_pretty(pg_total_relation_size(oid) - pg_relation_size(oid)) AS bloat_size
FROM pg_class
WHERE oid = 'jsonb_data'::regclass;
 toast_table_name | total_size | relation_size | toast_size | bloat_size 
------------------+------------+---------------+------------+------------
 jsonb_data       | 80 MB      | 512 kB        | 78 MB      | 79 MB
(1 row)
```

2. Создать индекс
```
CREATE INDEX idx_jsonb_data ON jsonb_data USING GIN (data);

SELECT relname AS toast_table_name,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
       pg_size_pretty(pg_relation_size(oid)) AS relation_size,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_size,
       pg_size_pretty(pg_total_relation_size(oid) - pg_relation_size(oid)) AS bloat_size
FROM pg_class
WHERE oid = 'jsonb_data'::regclass;
 toast_table_name | total_size | relation_size | toast_size | bloat_size 
------------------+------------+---------------+------------+------------
 jsonb_data       | 127 MB     | 512 kB        | 78 MB      | 126 MB
(1 row)
```

3. Обновить 1 из полей в json
```
UPDATE jsonb_data
SET data = jsonb_set(data, '{1}', '"Some text"');
```

4. Убедиться в блоатинге TOAST
```
SELECT relname AS toast_table_name,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
       pg_size_pretty(pg_relation_size(oid)) AS relation_size,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_size,
       pg_size_pretty(pg_total_relation_size(oid) - pg_relation_size(oid)) AS bloat_size
FROM pg_class
WHERE oid = 'jsonb_data'::regclass;
 toast_table_name | total_size | relation_size | toast_size | bloat_size 
------------------+------------+---------------+------------+------------
 jsonb_data       | 226 MB     | 1024 kB       | 156 MB     | 225 MB
(1 row)
```

5. Придумать методы избавится от него и проверить на практике
```
VACUUM FULL jsonb_data;

SELECT relname AS toast_table_name,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
       pg_size_pretty(pg_relation_size(oid)) AS relation_size,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_size,
       pg_size_pretty(pg_total_relation_size(oid) - pg_relation_size(oid)) AS bloat_size
FROM pg_class
WHERE oid = 'jsonb_data'::regclass;
 toast_table_name | total_size | relation_size | toast_size | bloat_size 
------------------+------------+---------------+------------+------------
 jsonb_data       | 126 MB     | 512 kB        | 78 MB      | 126 MB
(1 row)
```
