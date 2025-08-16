# Домашнее задание к занятию «Индексы»

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

---

```sql
SELECT
    ROUND(
        (SUM(t.data_length + t.index_length) / SUM(t.data_length)) * 100, 2
    ) AS index_to_table_ratio
FROM
    information_schema.tables t
WHERE
    t.table_schema = 'sakila';
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

---
```sql
SELECT concat(c.last_name, ' ', c.first_name) as full_name,
       sum(p.amount) as customer_total
FROM payment p
INNER JOIN rental r ON p.payment_date = r.rental_date
INNER JOIN customer c ON r.customer_id = c.customer_id
INNER JOIN inventory i ON i.inventory_id = r.inventory_id
INNER JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00' 
  AND p.payment_date < '2005-07-31 00:00:00'
GROUP BY c.customer_id, c.last_name, c.first_name
```

Вот краткий список узких мест, которые были устранены:

Неявные соединения (FROM ... WHERE ...) → заменены на явные INNER JOIN, что позволяет использовать индексы и улучшает производительность.

Фильтрация по дате с использованием date(p.payment_date) = '2005-07-30' → заменена на точное условие с временным диапазоном, что позволяет использовать индекс на поле payment_date.

Использование DISTINCT для уникальности записей → заменено на GROUP BY, что позволяет более эффективно агрегировать данные.

Избыточные или неэффективные условия соединения были убраны или улучшены, упрощая запрос.

Эти изменения улучшили производительность запроса за счет более точных фильтров и эффективных соединений.


### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

---

GiST (Generalized Search Tree) — для сложных типов данных и многомерных запросов.

GIN (Generalized Inverted Index) — для массивов, JSON и полнотекстового поиска.

SP-GiST (Space-partitioned GiST) — для пространственных и многомерных данных.

BRIN (Block Range INdex) — для очень больших таблиц с упорядоченными данными.

Bloom Index — для быстрого поиска по нескольким столбцам.

Hash Index — для быстрого поиска по равенству (ограниченно используется).