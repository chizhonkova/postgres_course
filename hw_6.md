# Homework 6

1. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов):
```
thai=# WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 4601.697 ms (00:04.602)
```

2. После добавления внешних индексов:
```
CREATE INDEX idx_ride_fk_schedule on book.ride(fkschedule);
CREATE INDEX idx_schedule_fk_route on book.schedule(fkroute);
CREATE INDEX idx_br_fkbsfrom on book.busroute(fkbusstationfrom);
CREATE INDEX idx_tickets_fk_ride on book.tickets(fkride);
CREATE INDEX idx_seat_fk_bus on book.seat(fkbus);

WITH all_place AS (                                             
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 4521.898 ms (00:04.522)
```

3. Индексы не помогли ускориться, в explain их нет:
```
 Limit  (cost=327732.53..327732.55 rows=10 width=56)
   ->  Sort  (cost=327732.53..328086.81 rows=141712 width=56)
         Sort Key: r.startdate
         ->  Group  (cost=322190.22..324670.18 rows=141712 width=56)
               Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
               ->  Sort  (cost=322190.22..322544.50 rows=141712 width=56)
                     Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                     ->  Hash Join  (cost=257113.19..305220.92 rows=141712 width=56)
                           Hash Cond: (r.fkbus = s_1.fkbus)
                           ->  Nested Loop  (cost=257108.08..303819.95 rows=141712 width=84)
                                 ->  Hash Join  (cost=257107.93..300447.37 rows=141712 width=24)
                                       Hash Cond: (s.fkroute = br.id)
                                       ->  Hash Join  (cost=257105.58..300046.75 rows=141712 width=24)
                                             Hash Cond: (r.fkschedule = s.id)
                                             ->  Merge Join  (cost=257062.18..299630.26 rows=141712 width=24)
                                                   Merge Cond: (r.id = t.fkride)
                                                   ->  Index Scan using ride_pkey on ride r  (cost=0.42..4534.42 rows=144000 width=16)
                                                   ->  Finalize GroupAggregate  (cost=257061.76..292964.44 rows=141712 width=12)
                                                         Group Key: t.fkride
                                                         ->  Gather Merge  (cost=257061.76..290130.20 rows=283424 width=12)
                                                               Workers Planned: 2
                                                               ->  Sort  (cost=256061.74..256416.02 rows=141712 width=12)
                                                                     Sort Key: t.fkride
                                                                     ->  Partial HashAggregate  (cost=218997.44..241514.43 rows=141712 width=12)
                                                                           Group Key: t.fkride
                                                                           Planned Partitions: 4
                                                                           ->  Parallel Seq Scan on tickets t  (cost=0.00..80582.27 rows=2160627 width=12)
                                             ->  Hash  (cost=25.40..25.40 rows=1440 width=8)
                                                   ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8)
                                       ->  Hash  (cost=1.60..1.60 rows=60 width=8)
                                             ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8)
                                 ->  Memoize  (cost=0.15..0.36 rows=1 width=68)
                                       Cache Key: br.fkbusstationfrom
                                       Cache Mode: logical
                                       ->  Index Scan using busstation_pkey on busstation bs  (cost=0.14..0.35 rows=1 width=68)
                                             Index Cond: (id = br.fkbusstationfrom)
                           ->  Hash  (cost=5.05..5.05 rows=5 width=12)
                                 ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12)
                                       Group Key: s_1.fkbus
                                       ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8)
```