\# otus\_2024

Обучение в ОТУС

/\*

\-- создание схемы

create schema sapu

\--создание таблиц

CREATE TABLE sapu.duser (

user\_id int8 DEFAULT nextval('sapu.seq\_users\_id'::regclass) NOT NULL,

user\_fio varchar(300) DEFAULT 'некто'::character varying NOT NULL,

user\_login varchar(500) NOT NULL,

user\_rgdate timestamp DEFAULT now() NOT NULL,

user\_gr timestamp NULL,

user\_mr varchar(1000) NULL,

user\_adress\_id int8 NULL,

CONSTRAINT users\_pkey PRIMARY KEY (user\_id)

);

CREATE TABLE sapu.inpost (

inpost\_id int8 DEFAULT nextval('sapu.seq\_inpost\_id'::regclass) NOT NULL,

outpost\_id int8 NOT NULL,

user\_id int8 NOT NULL,

inpost\_reac\_id int8 NOT NULL,

snet\_id int8 NOT NULL,

inpost\_date timestamp DEFAULT now() NOT NULL,

CONSTRAINT inpost\_pkey PRIMARY KEY (inpost\_id)

);

CREATE TABLE sapu.outpost (

outpost\_id int8 DEFAULT nextval('sapu.seq\_outpost\_id'::regclass) NOT NULL,

outpost\_text text DEFAULT 'пусто'::text NOT NULL,

user\_id int8 NOT NULL,

snet\_id int8 NOT NULL,

outpost\_date timestamp DEFAULT now() NOT NULL,

CONSTRAINT outpost\_pkey PRIMARY KEY (outpost\_id)

);

\--наполнение таблиц

INSERT INTO sapu.duser(user\_fio , user\_login, user\_rgdate, user\_gr, user\_mr)

SELECT md5(CAST(i AS varchar)),

substr(md5(random()::text), 1, 6),

timestamp '2022-01-10 20:00:00' +

random()\*i\*(timestamp '2022-09-20 20:00:00' -

timestamp '2022-09-10 10:00:00'),

timestamp '1980-01-01 09:00:00' +

random()\*i\*(timestamp '1981-03-01 10:00:00'-

timestamp '1979-05-01 20:00:00'),

(SELECT \* FROM unnest(ARRAY\['Тамбов','Москва','Волгоград','Курск','Екатеринбунг','Новосибирск','Тобольск'\]) AS t ORDER BY random()\*i LIMIT 1)

FROM generate\_series(1,100) i;

\*/

\--text1 = 'В этой серии статей речь пойдет об индексах в PostgreSQL.Любой вопрос можно рассматривать с разных точек зрения. Мы будем говорить о том, что должно интересовать прикладного разработчика, использующего СУБД: какие индексы существуют, почему в PostgreSQL их так много разных, и как их использовать для ускорения запросов. Пожалуй, тему можно было бы раскрыть и меньшим числом слов, но мы втайне надеемся на любознательного разработчика, которому также интересны и подробности внутреннего устройства, тем более, что понимание таких подробностей позволяет не только прислушиваться к чужому мнению, но и делать собственные выводы.За скобками обсуждения останутся вопросы разработки новых типов индексов. Это требует знания языка Си и относится скорее к компетенции системного программиста, а не прикладного разработчика. По этой же причине мы практически не будем рассматривать программные интерфейсы, а остановимся только на том, что имеет значение для использования уже готовых к употреблению индексов.В этой части мы поговорим про разделение сфер ответственности между общим механизмом индексирования, относящимся к ядру СУБД, и отдельными методами индексного доступа, которые в PostgreSQL можно добавлять как расширения. В следующей части мы рассмотрим интерфейс метода доступа и такие важные понятия, как классы и семейства операторов. После такого длинного, но необходимого введения мы подробно рассмотрим устройство и применение различных типов индексов: Hash, B-tree, GiST, SP-GiST, GIN и RUM, BRIN и Bloom.'

\--text2 = 'Индексы в PostgreSQL — специальные объекты базы данных, предназначенные в основном для ускорения доступа к данным. Это вспомогательные структуры: любой индекс можно удалить и восстановить заново по информации в таблице. Иногда приходится слышать, что СУБД может работать и без индексов, просто медленно. Однако это не так, ведь индексы служат также для поддержки некоторых ограничений целостности.В настоящее время в PostgreSQL 9.6 встроены шесть разных видов индексов, и еще один доступен как расширение — это стало возможным благодаря важным изменениям в версии 9.6. Так что в ближайшее время стоит ожидать появления и других типов индексов.Несмотря на все различия между типами индексов (называемыми также методами доступа), в конечном счете любой из них устанавливает соответствие между ключом (например, значением проиндексированного столбца) и строками таблицы, в которых этот ключ встречается. Строки идентифицируются с помощью TID (tuple id), который состоит из номера блока файла и позиции строки внутри блока. Тогда, зная ключ или некоторую информацию о нем, можно быстро прочитать те строки, в которых может находиться интересующая нас информация, не просматривая всю таблицу полностью.Важно понимать, что индекс, ускоряя доступ к данным, взамен требует определенных затрат на свое поддержание. При любой операции над проиндексированными данными — будь то вставка, удаление или обновление строк таблицы, — индексы, созданные для этой таблицы, должны быть перестроены, причем в рамках той же транзакции. Заметим, что обновление полей таблицы, по которым не создавались индексы, не приводит к перестроению индексов; этот механизм называется HOT (Heap-Only Tuples).Расширяемость влечет некоторые следствия. Чтобы новый метод доступа можно было легко встроить в систему, в PostgreSQL выделен общий механизм индексирования. Его основной задачей является получение TID от метода доступа и работа с ними:'

/\*

INSERT INTO sapu.outpost(outpost\_text,outpost\_date,user\_id,snet\_id)

SELECT substr('Индексы в PostgreSQL — специальные объекты базы данных, предназначенные в основном для ускорения доступа к данным. Это вспомогательные структуры: любой индекс можно удалить и восстановить заново по информации в таблице. Иногда приходится слышать, что СУБД может работать и без индексов, просто медленно. Однако это не так, ведь индексы служат также для поддержки некоторых ограничений целостности.В настоящее время в PostgreSQL 9.6 встроены шесть разных видов индексов, и еще один доступен как расширение — это стало возможным благодаря важным изменениям в версии 9.6. Так что в ближайшее время стоит ожидать появления и других типов индексов.Несмотря на все различия между типами индексов (называемыми также методами доступа), в конечном счете любой из них устанавливает соответствие между ключом (например, значением проиндексированного столбца) и строками таблицы, в которых этот ключ встречается. Строки идентифицируются с помощью TID (tuple id), который состоит из номера блока файла и позиции строки внутри блока. Тогда, зная ключ или некоторую информацию о нем, можно быстро прочитать те строки, в которых может находиться интересующая нас информация, не просматривая всю таблицу полностью.Важно понимать, что индекс, ускоряя доступ к данным, взамен требует определенных затрат на свое поддержание. При любой операции над проиндексированными данными — будь то вставка, удаление или обновление строк таблицы, — индексы, созданные для этой таблицы, должны быть перестроены, причем в рамках той же транзакции. Заметим, что обновление полей таблицы, по которым не создавались индексы, не приводит к перестроению индексов; этот механизм называется HOT (Heap-Only Tuples).Расширяемость влечет некоторые следствия. Чтобы новый метод доступа можно было легко встроить в систему, в PostgreSQL выделен общий механизм индексирования. Его основной задачей является получение TID от метода доступа и работа с ними:'

,(random()\*10)::integer,(random()\*10+ random()\*100)::integer),

(SELECT now() - INTERVAL '1 day' \* round(random()\*i)),

(SELECT user\_id FROM sapu.duser ORDER BY random()\*i LIMIT 1),

0

FROM generate\_series(1,150000) i;

\*/

/\*

\--чем больше данных , тем интереснее результаты

INSERT INTO sapu.inpost(outpost\_id,user\_id,inpost\_reac\_id,snet\_id,inpost\_date)

SELECT (SELECT outpost\_id FROM sapu.outpost ORDER BY random()\*i LIMIT 1),

(SELECT user\_id FROM sapu.duser ORDER BY random()\*i LIMIT 1),

(SELECT outpost\_id FROM sapu.outpost where outpost\_id>10000 ORDER BY random()\*i LIMIT 1),

0,

now() - INTERVAL '1 day'

FROM generate\_series(1,150000) i

\*/

\-- получим план выполнения на чистой таблице

EXPLAIN ANALYZE

SELECT user\_id , count(1)

FROM sapu.outpost oup

GROUP BY user\_id

ORDER BY 2 DESC

LIMIT 10;

\-- получим план выполнения на чистой таблице

Limit (cost=5101.22..5101.24 rows=10 width=16) (actual time=55.030..58.394 rows=10 loops=1)

\-> Sort (cost=5101.22..5101.48 rows=104 width=16) (actual time=55.029..58.393 rows=10 loops=1)

Sort Key: (count(1)) DESC

Sort Method: top-N heapsort Memory: 25kB

\-> Finalize GroupAggregate (cost=5072.62..5098.97 rows=104 width=16) (actual time=54.988..58.376 rows=104 loops=1)

Group Key: user\_id

\-> Gather Merge (cost=5072.62..5096.89 rows=208 width=16) (actual time=54.981..58.351 rows=104 loops=1)

Workers Planned: 2

Workers Launched: 2

\-> Sort (cost=4072.60..4072.86 rows=104 width=16) (actual time=9.361..9.363 rows=35 loops=3)

Sort Key: user\_id

Sort Method: quicksort Memory: 29kB

Worker 0: Sort Method: quicksort Memory: 25kB

Worker 1: Sort Method: quicksort Memory: 25kB

\-> Partial HashAggregate (cost=4068.07..4069.11 rows=104 width=16) (actual time=9.338..9.342 rows=35 loops=3)

Group Key: user\_id

Batches: 1 Memory Usage: 24kB

Worker 0: Batches: 1 Memory Usage: 24kB

Worker 1: Batches: 1 Memory Usage: 24kB

\-> Parallel Seq Scan on outpost oup (cost=0.00..3818.05 rows=50005 width=8) (actual time=0.002..2.289 rows=40004 loops=3)

Planning Time: 0.474 ms

Execution Time: 58.427 ms

\--построим btree индекс по полю user\_id

CREATE INDEX IF NOT EXISTS idx\_oupost\_user\_id ON sapu.outpost(user\_id) ; -- простой btree индекс по полю, проблем не было

\--поссмотрим план выполнения запроса

Limit (cost=2827.82..2827.84 rows=10 width=16) (actual time=16.083..16.085 rows=10 loops=1)

\-> Sort (cost=2827.82..2828.08 rows=104 width=16) (actual time=16.082..16.082 rows=10 loops=1)

Sort Key: (count(1)) DESC

Sort Method: top-N heapsort Memory: 25kB

\-> GroupAggregate (cost=0.29..2825.57 rows=104 width=16) (actual time=0.445..16.056 rows=104 loops=1)

Group Key: user\_id

\-> Index Only Scan using idx\_oupost\_user\_id on outpost oup (cost=0.29..2224.47 rows=120012 width=8) (actual time=0.304..8.594 rows=120012 loops=1)

Heap Fetches: 0

Planning Time: 0.866 ms

Execution Time: 16.105 ms

\-- заметим , что поиск идёт по индексу, который мы построили,

\-- заметно снизились используемые ресурсы

\-- вдвое уменьшилось количество "попугаев"

\----------------------------------------------------------------------------------------------------

EXPLAIN ANALYZE

select us.user\_fio , -- фио пользователя

count(oup.outpost\_text) as kpost -- количество постов

from sapu.duser us -- таблица постов , написанных пользователем

inner join sapu.outpost oup -- таблица пользователей

on (oup.user\_id = us.user\_id) -- соединяем по идентификатору пользователя данного поста

where lower(oup.outpost\_text) like '%insert%' -- и пост содержащий слово или часть "индекс"

group by us.user\_fio -- группируем по us.users\_fio

order by us.user\_fio;

Finalize GroupAggregate (cost=5072.80..5077.31 rows=38 width=41) (actual time=227.209..231.325 rows=2 loops=1)

Group Key: us.user\_fio

\-> Gather Merge (cost=5072.80..5076.77 rows=32 width=41) (actual time=227.205..231.320 rows=2 loops=1)

Workers Planned: 2

Workers Launched: 2

\-> Partial GroupAggregate (cost=4072.78..4073.06 rows=16 width=41) (actual time=191.004..191.005 rows=1 loops=3)

Group Key: us.user\_fio

\-> Sort (cost=4072.78..4072.82 rows=16 width=128) (actual time=191.000..191.002 rows=1 loops=3)

Sort Key: us.user\_fio

Sort Method: quicksort Memory: 25kB

Worker 0: Sort Method: quicksort Memory: 25kB

Worker 1: Sort Method: quicksort Memory: 25kB

\-> Hash Join (cost=4.34..4072.46 rows=16 width=128) (actual time=115.399..190.948 rows=1 loops=3)

Hash Cond: (oup.user\_id = us.user\_id)

\-> Parallel Seq Scan on outpost oup (cost=0.00..4068.07 rows=16 width=103) (actual time=115.383..190.930 rows=1 loops=3)

Filter: (upper(outpost\_text) ~~ '%INSERT%'::text)

Rows Removed by Filter: 40003

\-> Hash (cost=3.04..3.04 rows=104 width=41) (actual time=0.037..0.037 rows=104 loops=1)

Buckets: 1024 Batches: 1 Memory Usage: 17kB

\-> Seq Scan on duser us (cost=0.00..3.04 rows=104 width=41) (actual time=0.012..0.021 rows=104 loops=1)

Planning Time: 0.162 ms

Execution Time: 231.388 ms

set default\_text\_search\_config = russian;

ALTER TABLE sapu.outpost ADD COLUMN outpost\_tsv tsvector;

update sapu.outpost set outpost\_tsv = to\_tsvector(outpost\_text);

create index outpost\_text\_gin on sapu.outpost using gin(outpost\_tsv); -- самый сложный поиск варианта типа индекса, долгий поиск сначала на стандартных вариантах, затем переход к gin-индексам, перестроение запроса, анализ

\-- остались вопросы применения в различных ситуациях, но значительный прирост в скорост очевиден. Провёл сравнение типов индексов на реальных данных

\-- размером от 500 ГБ таблица и , понимаю, что нужен анализ стоимости индекса (скорость, ресурсы, время подготовки, так как все индексы надо создавать,

\-- а это время, место на сервере и нужна отдача.

EXPLAIN ANALYZE

select us.user\_fio , -- фио пользователя

count(oup.outpost\_text) as kpost -- количество постов

from sapu.duser us -- таблица постов , написанных пользователем

inner join sapu.outpost oup -- таблица пользователей

on (oup.user\_id = us.user\_id) -- соединяем по идентификатору пользователя данного поста

where outpost\_tsv @@ to\_tsquery('insert') -- и пост содержащий слово или часть "индекс"

group by us.user\_fio -- группируем по us.users\_fio

order by us.user\_fio;

GroupAggregate (cost=103.72..104.11 rows=22 width=41) (actual time=0.099..0.101 rows=2 loops=1)

Group Key: us.user\_fio

\-> Sort (cost=103.72..103.78 rows=22 width=128) (actual time=0.095..0.096 rows=3 loops=1)

Sort Key: us.user\_fio

Sort Method: quicksort Memory: 25kB

\-> Hash Join (cost=13.25..103.23 rows=22 width=128) (actual time=0.082..0.085 rows=3 loops=1)

Hash Cond: (oup.user\_id = us.user\_id)

\-> Bitmap Heap Scan on outpost oup (cost=8.91..98.84 rows=22 width=103) (actual time=0.027..0.029 rows=3 loops=1)

Recheck Cond: (outpost\_tsv @@ to\_tsquery('insert'::text))

Heap Blocks: exact=2

\-> Bitmap Index Scan on outpost\_text\_gin (cost=0.00..8.90 rows=22 width=0) (actual time=0.022..0.022 rows=3 loops=1)

Index Cond: (outpost\_tsv @@ to\_tsquery('insert'::text))

\-> Hash (cost=3.04..3.04 rows=104 width=41) (actual time=0.044..0.044 rows=104 loops=1)

Buckets: 1024 Batches: 1 Memory Usage: 17kB

\-> Seq Scan on duser us (cost=0.00..3.04 rows=104 width=41) (actual time=0.012..0.024 rows=104 loops=1)

Planning Time: 0.240 ms

Execution Time: 0.144 ms

\------------------------------------------------------------------------------------------

DROP INDEX IF EXISTS idx\_inpost\_idout;

EXPLAIN analyze

SELECT user\_id, inpost\_reac\_id, outpost\_id

FROM sapu.inpost i

WHERE USER\_id = 16

Seq Scan on inpost i (cost=0.00..2731.50 rows=1190 width=24) (actual time=0.012..7.937 rows=1169 loops=1)

Filter: (user\_id = 16)

Rows Removed by Filter: 123831

Planning Time: 0.475 ms

Execution Time: 7.973 ms

CREATE INDEX IF NOT EXISTS idx\_inpost\_idout ON sapu.inpost(user\_id, outpost\_id) ; -- составной индекс

Recheck Cond: (user\_id = 16)

Heap Blocks: exact=735

\-> Bitmap Index Scan on idx\_inpost\_idout (cost=0.00..29.34 rows=1190 width=0) (actual time=0.143..0.143 rows=1169 loops=1)

Index Cond: (user\_id = 16)

Planning Time: 0.893 ms

Execution Time: 0.786 ms

DROP INDEX IF EXISTS idx\_inpost\_idout;

EXPLAIN analyze

SELECT user\_id, inpost\_reac\_id, outpost\_id

FROM sapu.inpost i

WHERE USER\_id = 16

AND outpost\_id = inpost\_reac\_id

Seq Scan on inpost i (cost=0.00..3044.00 rows=6 width=24) (actual time=8.893..8.893 rows=0 loops=1)

Filter: ((user\_id = 16) AND (outpost\_id = inpost\_reac\_id))

Rows Removed by Filter: 125000

Planning Time: 0.068 ms

Execution Time: 8.907 ms

CREATE INDEX IF NOT EXISTS idx\_inpost\_idout ON sapu.inpost(user\_id, outpost\_id, inpost\_reac\_id) ; -- составной индекс

Bitmap Heap Scan on inpost i (cost=29.34..1258.60 rows=6 width=24) (actual time=2.586..2.586 rows=0 loops=1)

Recheck Cond: (user\_id = 16)

Filter: (outpost\_id = inpost\_reac\_id)

Rows Removed by Filter: 1169

Heap Blocks: exact=735

\-> Bitmap Index Scan on idx\_inpost\_idout (cost=0.00..29.34 rows=1190 width=0) (actual time=0.138..0.139 rows=1169 loops=1)

Index Cond: (user\_id = 16)

Planning Time: 1.185 ms

Execution Time: 2.616 ms

DROP INDEX IF EXISTS idx\_inpost\_idout;

CREATE INDEX IF NOT EXISTS idx\_inpost\_idout ON sapu.inpost(user\_id, inpost\_reac\_id) ;

Bitmap Heap Scan on inpost i (cost=29.34..1258.60 rows=6 width=24) (actual time=0.771..0.772 rows=0 loops=1)

Recheck Cond: (user\_id = 16)

Filter: (outpost\_id = inpost\_reac\_id)

Rows Removed by Filter: 1169

Heap Blocks: exact=735

\-> Bitmap Index Scan on idx\_inpost\_idout (cost=0.00..29.34 rows=1190 width=0) (actual time=0.157..0.157 rows=1169 loops=1)

Index Cond: (user\_id = 16)

Planning Time: 3.366 ms

Execution Time: 0.801 ms

DROP INDEX IF EXISTS idx\_inpost\_idout;

CREATE INDEX IF NOT EXISTS idx\_inpost\_idout ON sapu.inpost(user\_id, outpost\_id) ;

Bitmap Heap Scan on inpost i (cost=29.34..1258.60 rows=6 width=24) (actual time=0.700..0.700 rows=0 loops=1)

Recheck Cond: (user\_id = 16)

Filter: (outpost\_id = inpost\_reac\_id)

Rows Removed by Filter: 1169

Heap Blocks: exact=735

\-> Bitmap Index Scan on idx\_inpost\_idout (cost=0.00..29.34 rows=1190 width=0) (actual time=0.091..0.091 rows=1169 loops=1)

Index Cond: (user\_id = 16)

Planning Time: 0.079 ms

Execution Time: 0.722 ms

\-- стоит отметить , что для меня мстал сложным полнотекстовый поиск, вплоть до того, что есть мысли введения каких-то спецполей, подготовки их преобразованием кастомной функции и построению индекса

\-- пространственные данные не анализировал, есть набор данных, но надо загружать и сейчас не актуально в текущем проекте
