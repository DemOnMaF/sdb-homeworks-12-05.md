# Домашнее задание к занятию «Индексы» Федоров Дмитрий

### Инструкция по выполнению домашнего задания

<details>

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

</details>

Желаем успехов в выполнении домашнего задания.

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

<details>
<summary>Ответ</summary>

```sql
SELECT
 ROUND(SUM(index_length) / SUM(data_length + index_length) * 100, 2) AS "процентное отношение размера индексов к общему размеру (данные + индексы)",
 CONCAT(ROUND(SUM(data_length) / (1024 * 1024), 2), ' MB') AS "Общий размер таблиц",
 CONCAT(ROUND(SUM(index_length) / (1024 * 1024), 2), ' MB') AS "Общий размер индексов"
FROM
 information_schema.TABLES
WHERE
 table_schema = 'sakila';

```

![image](img/01.png)

</details>

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

<details>
<summary>Ответ</summary>

После анализа исходного запроса было выявляено, что наиболее узким местом в предлагаемом запросе является то, что оконная функция обрабатывает излишние таблицы (inventory, rental и film). Исходя из того, что нужно посчитать сумму платежей покупателей за конкретную дату, обработка и присоединение этих таблиц не имеет смысла т.к. дальше данные не используются. Все необходимые данные есть в таблицах payment и customer, соответственно, остальные таблицы можно исключить.

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id ;


```

![image](img/02.png)

После доработки:

```sql
explain analyze
select distinct concat(c.last_name, ' ', c.first_name) AS full_name, SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE payment_date >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY) ;



```

</details>

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
