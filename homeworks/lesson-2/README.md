ДЗ делал с запуском бд в докер контейнере.

- запустить везде psql из под пользователя postgres
>psql --dbname=otus  --host=localhost --port=5432 --username=postgres
```
otus=# 
```
---

- выключить auto commit
>\echo :AUTOCOMMIT
>\set AUTOCOMMIT OFF
```
otus=# 
```
---

- сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
```
COMMIT
```
---

- посмотреть текущий уровень изоляции: show transaction isolation level;
>show transaction isolation level;
```
 transaction_isolation 
-----------------------
 read committed
(1 row)
```
---

- начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции.\
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');\
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Нет. Потому что при уровне изоляции read committed незакомиченные изменения не видны в других сессиях.
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
---

- завершить первую транзакцию - commit;\
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Да, после завершения транзакции изменения стали доступными и видны для всех сессиий.
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
---

- завершите транзакцию во второй сессии
```
otus=*# END;
COMMIT
```
---

- начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Нет. В repeatable read уровне так же не закомиченные изменения не будут видны в других сессиях, более того на момент первого запроса строки(на выборку или изменения)зафиксированные в рамках текущей транзакции не меняются .
otus=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)

otus=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
---

- завершить первую транзакцию - commit;
```
otus=*# commit;
COMMIT
```
---

- сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Нет. В repeatable read уровне при первом запроса зафиксированные строки на выборку или изменения не будут меняться в рамках текущей транзакции, поэтому результат выборки будет одним и тем же в рамках этой транзакции.
otus=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
---

- завершить вторую транзакцию
```
otus=*# commit;
COMMIT
```
---

 - сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Да. Перед выборкой в новой транзакции изменения уже были закомичены, соответственно они выбрались и зафиксировались в рамках текущей транзакции. 
otus=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```
