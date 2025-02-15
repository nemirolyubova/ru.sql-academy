# Партиции в оконных функциях

В <a href="https://sql-academy.org/guide/windows-functions" target="_blank">прошлой статье</a> мы кратко уже упоминали, что
такое партиции и как их использовать в оконных функциях, пришло время разобраться в них поподробнее.

## Понятие партиции

> Партиции — подмножества строк, выделенные для оконной функции на основе одного или нескольких столбцов в таблице.

Они служат для сегментации данных, позволяя выполнить более детальный анализ и
расчёты вроде агрегации или ранжирования внутри каждой такой группы.

Применяя партиционирование, например, по типу жилья в таблице с данными о стоимости жилья,
мы можем рассчитать в отдельной колонке, скажем, среднюю цену для каждого типа жилья.

![Схема разбиения на партиции](https://sql-academy.org/static/guidePage/windows-functions/3.png "Схема разбиения на партиции")

## Применение партиций в SQL

Для того, чтобы использовать партицию вместе с оконной функцией необходимо придерживаться следующего
синтаксиса:

```sql
SELECT <оконная_функция>(<поле_таблицы>)
OVER (
    PARTITION BY <столбцы_для_разделения>
)
```

### Пример использования

А теперь давайте на простом примере рассмотрим использование партиции вместе с оконной функцией.

<ERD databaseName="Airbnb" />

Для этого рассмотрим таблицу `Rooms`, а именно поля `home_type` и `price`:

```sql-Airbnb-executable
SELECT home_type, price FROM Rooms;
```

Мы можем увидеть, что все жильё для аренды разделено на 3 категории: «Private room», «Entire home/apt» и «Shared room».

Каждая категория жилья имеет свои ценовые рамки.
Чтобы узнать среднюю стоимость в конкретной категории и сравнить её с текущей, как раз можно использовать оконные функции.

Для этого добавим к нашей результирующей таблице ещё одно поле `avg_price`, которое будет высчитывать среднюю стоимость по категориям. Это будет выглядеть следующим образом:

```sql-Airbnb-executable
SELECT
    home_type, price,
    AVG(price) OVER (PARTITION BY home_type) AS avg_price
    FROM Rooms
```

Что именно происходит в добавленной строке?

- `PARTITION BY home_type` делит все записи на разные партиции на основе уникальных значений столбца `home_type`
- затем, для каждой записи, `AVG(price)` вычисляет среднюю цену (`price`) в рамках её партиции (`home_type`)

Результатом выполнения этой части запроса будет столбец `avg_price`,
в котором для каждой записи будет указано среднее значение цены для её категории жилья (`home_type`).

## Партиции по нескольким колонками

Партиционирование также может быть выполнено по нескольким колонкам. Это позволяет создавать более сложные и точные сегменты для анализа.

Например, для нашей таблицы `Rooms` мы можем создать партиции на основании 2 колонок: категория жилья
`home_type` и наличие телевизора в жилье `has_tv`.

Пример запроса с партиционированием по двум столбцам:

```sql-Airbnb-executable
SELECT
    home_type, has_tv, price,
    AVG(price) OVER (PARTITION BY home_type, has_tv) AS avg_price
    FROM Rooms
```

Здесь `PARTITION BY home_type, has_tv` создаёт уникальные партиции для каждой комбинации `home_type` и `has_tv`,
позволяя вычислить среднюю стоимости жилья для текущей категории жилья с наличием или без наличия телевизора.

![Партиции по 2 колонками](https://sql-academy.org/static/guidePage/partitions/2-columns-partition.png "Партиции по 2 колонками")
