# Homework 1

Тестовая база данных:
```
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
```

Посчитать число поездок:

```
select count(*) from book.tickets;
```

Результат:
```
5185505
```
