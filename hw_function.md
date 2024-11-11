# Homework 7

1. Создать таблицу с продажами
```
CREATE TABLE sales (
    product_id INTEGER,
    total_amount INTEGER,
    quantity INTEGER,
    sale_date TIMESTAMP
);

INSERT INTO sales VALUES
(0, 10, 1, TIMESTAMP '2004-01-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-02-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-03-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-04-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-05-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-06-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-07-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-08-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-09-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-10-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-11-19 10:23:54'),
(0, 10, 1, TIMESTAMP '2004-12-19 10:23:54'),
(0, 10, 1, NULL);
```

2. Реализовать функцию выбора трети года
```
CREATE OR REPLACE FUNCTION get_third(ts TIMESTAMP)
RETURNS INTEGER AS $$
BEGIN
    RETURN (EXTRACT(MONTH FROM ts)::INTEGER - 1) / 4 + 1;
END;
$$
 LANGUAGE plpgsql
 RETURNS NULL ON NULL INPUT;
```

3. Вызвать эту функцию в select из таблицы с продажами
```
postgres=# SELECT sale_date, get_third(sale_date) FROM sales;
      sale_date      | get_third 
---------------------+-----------
 2004-01-19 10:23:54 |         1
 2004-02-19 10:23:54 |         1
 2004-03-19 10:23:54 |         1
 2004-04-19 10:23:54 |         1
 2004-05-19 10:23:54 |         2
 2004-06-19 10:23:54 |         2
 2004-07-19 10:23:54 |         2
 2004-08-19 10:23:54 |         2
 2004-09-19 10:23:54 |         3
 2004-10-19 10:23:54 |         3
 2004-11-19 10:23:54 |         3
 2004-12-19 10:23:54 |         3
                     |          
(13 rows)
```
