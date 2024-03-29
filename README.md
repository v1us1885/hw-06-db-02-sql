# Домашнее задание к занятию 2. «SQL» -Филатов А. В.

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

## Решение 1
```
version: '3'
services:
  postgres:
    image: postgres:12
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data
      - backups:/backups
volumes:
  pgdata:
  backups:

```


## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

## Решение 2
![spisokbd](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/spisokBD.png)
![d_orders](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/d_orders.png)
![d_clients](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/d_clients.png)
![sql-user_privileges](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/sql-user_privileges.png)
![dp_orders_clients](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/dp_orders_clients.png)
## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

## Решение 3
```
-- Вставка данных в таблицу orders
INSERT INTO orders (id, наименование, цена)
VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);

-- Вставка данных в таблицу clients
INSERT INTO clients (id, фамилия, страна_проживания, заказ)
VALUES (1, 'Иванов Иван Иванович', 'USA', 1), (2, 'Петров Петр Петрович', 'Canada', 2), (3, 'Иоганн Себастьян Бах', 'Japan', 3), (4, 'Ронни Джеймс Дио', 'Russia', 4), (5, 'Ritchie Blackmore', 'Russia', 5);
```
![count_orders_clients](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/count_orders_clients.png)

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

## Решение 4
```
UPDATE clients 
SET заказ = (SELECT id FROM orders WHERE наименование = 'Книга') 
WHERE фамилия = 'Иванов Иван Иванович';

UPDATE clients 
SET заказ = (SELECT id FROM orders WHERE наименование = 'Монитор') 
WHERE фамилия = 'Петров Петр Петрович';

UPDATE clients 
SET заказ = (SELECT id FROM orders WHERE наименование = 'Гитара') 
WHERE фамилия = 'Иоганн Себастьян Бах';
```

```
SELECT c.фамилия, o.наименование 
FROM clients c 
JOIN orders o ON c.заказ = o.id 
WHERE c.заказ IS NOT NULL;
```
![sql-buy](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/sql-buy.png)

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Решение 5
![explain](https://github.com/v1us1885/hw-06-db-02-sql/blob/main/explain.png)

Результат выполнения EXPLAIN представляет собой план выполнения запроса, который оптимизатор запросов PostgreSQL использует для выполнения вашего SQL-запроса. Вот разбор:   
  - Hash Join: Это тип соединения между таблицами, в данном случае, использован Hash Join.
  - cost=11.57..38.77: Это оценка стоимости выполнения запроса. Важно для оптимизатора.
  - rows=70: Оценка числа строк, которые будут возвращены после выполнения запроса.
  - width=548: Ожидаемая ширина строки результата.
  - Hash Cond: (o.id = c."заказ"): Условие слияния хэшей для соединения таблиц.
  - Seq Scan on orders o (cost=0.00..22.00 rows=1200 width=36): Последовательное сканирование таблицы orders с оценкой стоимости.
  - Hash (cost=10.70..10.70 rows=70 width=520): Операция хэширования для соединения.
  - Seq Scan on clients c (cost=0.00..10.70 rows=70 width=520): Последовательное сканирование таблицы clients с оценкой стоимости.
  - Filter: ("заказ" IS NOT NULL): Фильтр для строк, где поле "заказ" не является NULL.   
План выполнения выглядит довольно эффективно. Hash Join используется для соединения таблиц, что обычно хорошо для производительности. Оценки стоимости и количества строк также выглядят разумно.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

## Решение 6

```
sudo docker exec -it 7887773041bf sh -c 'pg_dumpall -c -U admin > /backups/backup.sql'
```
```
sudo docker cp 6c79bc0aea8d:/backups/backup.sql /home/devops/backup.sql
```
```
sudo docker-compose down
```
```
sudo docker-compose up -d
```
```
cat /home/devops/backup.sql | sudo docker exec -i 6c79bc0aea8d psql -U admin -d test_db
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
 
