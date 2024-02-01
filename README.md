# Домашнее задание к занятию «Индексы»

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

#### Ответ на задание 1.

```SQL
mysql> select sum(data_length/1024) AS "Total table, Kb", sum(index_length/1024) AS "Total index, Kb", sum((data_length + index_length)/1024) AS TOTAL, sum(data_length/1024)/sum((data_length + index_length)/1024) * 100 AS "Tables %", sum(index_length/1024)/sum((data_length + index_length)/1024) * 100 AS "Index %" from information_schema.tables where table_schema='sakila';
+-----------------+-----------------+-----------+-------------+-------------+
| Total table, Kb | Total index, Kb | TOTAL     | Tables %    | Index %     |
+-----------------+-----------------+-----------+-------------+-------------+
|       4272.0000 |       2336.0000 | 6608.0000 | 64.64891041 | 35.35108959 |
+-----------------+-----------------+-----------+-------------+-------------+
1 row in set (0.00 sec)


```

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

#### Ответ на задание 2.
Из минусов текущего запроса:
 * сильно громоздкий;
 * совсем не человекочитаемый. Излишне запутан;
 * использует в себе сущности, не имеющие прямого отношения к сути;
 * перегружен лишними конструкциями типа over partition by;
 * все это создает лишние join'ы, влияющие на итоговое время выполнения запроса;

Оптимизированный запрос:
```SQL
select distinct c.customer_id, concat(c.last_name, ' ', c.first_name) as cust_name, sum(p.amount) from customer c inner join payment p on c.customer_id=p.customer_id where date(p.payment_date)='2005-07-30' group by c.customer_id;
```
Запрос работает без созданного индекса payment_date. Обошелся базовым набором индексов.
Диагностика EXPLAIN ANALYZE это подтверждает.

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
