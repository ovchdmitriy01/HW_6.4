**Домашнее задание к занятию "6.4. PostgreSQL"**

1. Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
```
postgres=# \l
```
- подключения к БД
```
postgres=# \c postgres postgres
```
- вывода списка таблиц
```
postgres=# \dt *.*
```
- вывода описания содержимого таблиц
```
postgres=# \d+ tablename
```
- выхода из psql
```
postgres=# \q
```

2. Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

```
test_database=# SELECT attname from pg_stats where tablename = 'orders' ORDER BY avg_width DESC LIMIT 1;
 attname
---------
 title
```

3. Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

```
BEGIN;
create table orders_new (
        id integer NOT NULL,
        title varchar(80) NOT NULL,
        price integer) partition by range(price);
    create table orders_1 partition of orders_new for values from (0) to (499);
    create table orders_2 partition of orders_new for values from (499) to (9999999);
    insert into orders_new (id, title, price) select * from orders;
COMMIT;
```

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?  
```
Можно, необходимо было на этапе проектирования учесть разбиение и создавать таблицу с указанием "partition ..."
```

4. Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

```
root@673ba49568de:/# pg_dump -h 192.168.1.121 -U postgres test_database > backup_data/dump.sql
Password:
root@673ba49568de:/# ls -l /backup_data/
total 16
-rw-r--r-- 1 root root 3499 May 27 22:03 dump.sql
-rw-r--r-- 1 1000 1000 4539 May 27 17:09 test_db.dump
-rw-rw-r-- 1 1000 1000 2082 May 27 18:01 test_dump.sql

Можно открыть бэкап-файл текстовым редактором и изменить строку описания столбца title.
"title character varying(80) NOT NULL UNIQUE"

```
