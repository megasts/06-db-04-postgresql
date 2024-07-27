# Домашнее задание к занятию 4. «PostgreSQL» - `Шульгатый Станислав`

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
- подключения к БД,
- вывода списка таблиц,
- вывода описания содержимого таблиц,
- выхода из psql.

---
### Ответ

1. Вывод списка БД:
```
    \l[+]   [PATTERN]      list databases
```

2. Подключение к БД:
```
  \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "postgres")
```

3. Вывод списка таблиц:
```
  \dt[S+] [PATTERN]      list tables
```

4. Вывод описания содержимого таблиц:
```
  \d[S+]  NAME           describe table, view, sequence, or index
```

5. Выход из psql:
```
  \q                     quit psql
```
---

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

---
### Ответ

``` sql
test_database=# select t.tablename , t.attname , t.avg_width
from pg_catalog.pg_stats t
where t.tablename  = 'orders' 
order by t.avg_width desc 
limit 1;
```
![Screenshot_1](https://github.com/megasts/06-db-04-postgresql/blob/main/img/Screenshot_1.png)

---

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

---
### Ответ

Скрипт для разбиения таблицы на 2:

``` sql
alter table orders rename to orders_old;

create table orders (like orders_old);

create table orders_1 (
	check ( price > 499 ))
inherits (orders);

create table orders_2 (
	check ( price <= 499 ))
inherits (orders);

create rule orders_1 as on insert to orders
  where (price>499)
  do instead insert into orders_1 values(NEW.*);

create rule orders_2 as on insert to orders
  where (price<=499)
  do instead insert into orders_2 values(NEW.*);

insert into orders select * from orders_old;

drop table orders_old;
```

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?  Исключить ручное разбиени было можно на этапе проектирвоаниея системы предусмотрев создание таблиц необходимых для шардирования, но это не точно.

---

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

### Ответ
---
``` bash
pg_dump -U postgres test_database > /backup/test_dump_new.sql 
```
Файл бэкапа: [test_dump_new.sql](https://github.com/megasts/06-db-04-postgresql/blob/main/test_dump_new.sql)

Чтобы добавить уникальность значения столбца `title` для таблиц `test_database` необходимо добавить свойство `UNIQUE`:

``` sql
    title character varying(80) NOT NULL UNIQUE,
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

