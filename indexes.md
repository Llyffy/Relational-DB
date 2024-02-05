# Домашнее задание к занятию «Индексы»

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ к заданию 1:

```
select (SUM(INDEX_LENGTH) / NULLIF(SUM(DATA_LENGTH + INDEX_LENGTH), 0)) * 100 AS index_percentage
from information_schema.tables
where table_schema = 'sakila';
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

### Ответ к заданию 2:

```
select distinct concat(c.last_name, ' ', c.first_name) as customer_name, sum(p.amount) over (partition by c.customer_id, f.title) as total_amount
from payment p
join rental r on p.payment_date = r.rental_date and p.customer_id = r.customer_id
join inventory i on r.inventory_id = i.inventory_id
join film f on i.film_id = f.film_id
join customer c on r.customer_id = c.customer_id
WHERE p.payment_date >= '2005-07-30' AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY);
```

Скриншотом не совсем удобно поэтому я просто выведу мой explain analyze:

```
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=14.5..14.6 rows=599 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=14.5..14.5 rows=599 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=12.6..14.3 rows=634 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=12.6..12.7 rows=634 loops=1)
                -> Stream results  (cost=3029 rows=91.6) (actual time=0.0748..12.3 rows=634 loops=1)
                    -> Nested loop inner join  (cost=3029 rows=91.6) (actual time=0.0707..11.9 rows=634 loops=1)
                        -> Nested loop inner join  (cost=2996 rows=91.6) (actual time=0.0669..11 rows=634 loops=1)
                            -> Nested loop inner join  (cost=2964 rows=91.6) (actual time=0.0638..10 rows=634 loops=1)
                                -> Nested loop inner join  (cost=2316 rows=1833) (actual time=0.0567..8.23 rows=634 loops=1)
                                    -> Filter: ((p.payment_date >= TIMESTAMP'2005-07-30 00:00:00') and (p.payment_date < <cache>(('2005-07-30' + interval 1 day))))  (cost=1674 rows=1833) (actual time=0.0478..7.5 rows=634 loops=1)
                                        -> Table scan on p  (cost=1674 rows=16500) (actual time=0.0368..5.65 rows=16044 loops=1)
                                    -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=973e-6..997e-6 rows=1 loops=634)
                                -> Filter: (r.customer_id = p.customer_id)  (cost=0.25 rows=0.05) (actual time=0.00186..0.00271 rows=1 loops=634)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.25 rows=1.04) (actual time=0.00171..0.00252 rows=1.01 loops=634)
                            -> Single-row index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.251 rows=1) (actual time=0.00129..0.00132 rows=1 loops=634)
                        -> Single-row index lookup on f using PRIMARY (film_id=i.film_id)  (cost=0.251 rows=1) (actual time=0.00132..0.00135 rows=1 loops=634)

```


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

### Ответ к заданию 3:


*Приведите ответ в свободной форме.*
