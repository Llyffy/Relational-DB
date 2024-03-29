# Домашнее задание к занятию «SQL. Часть 2»

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

---

Задание можно выполнить как в любом IDE, так и в командной строке.

### Задание 1

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию: 
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

---

### Ответ к заданию 1:

```
select concat(s.first_name, ' ', s.last_name) as full_name,
 count(c.customer_id) as customer_count,
 ci.city as city_name
 from staff s
 join store st ON s.store_id = st.store_id
 join address ad ON st.address_id = ad.address_id
 join customer c ON s.staff_id = c.store_id
 join city ci on ad.address_id = ci.city_id
 GROUP BY s.staff_id, st.store_id, ad.address_id, ci.city
 having customer_count > 300
 order by city_name;
```

---

### Задание 2

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

---

### Ответ к заданию 2:

```
select count(*) as movie_count
from film
where length > (select avg(length) from film);
```

---

### Задание 3

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

---

### Ответ к заданию 3:

```
select extract(month from payment_date) as payment_month,
sum(amount) as total_payments,
count(p.rental_id) as rental_count
from payment p
JOIN rental ON EXTRACT(MONTH FROM p.payment_date) = EXTRACT(MONTH FROM rental.rental_date)
group by payment_month
ORDER BY total_payments desc
limit 1;
```

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 4*

Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

---

### Ответ к заданию 4:


---

### Задание 5*

Найдите фильмы, которые ни разу не брали в аренду.

---

### Ответ к заданию 5:

```
SELECT i.inventory_id 
FROM inventory i
LEFT JOIN rental ON i.inventory_id = rental.inventory_id
WHERE rental.inventory_id IS NULL;
```
