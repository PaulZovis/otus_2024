# Репликация MySQL 
##### Используем ранее созданную базу изначальную БД 
```
CREATE database IF NOT EXISTS otusdb;
USE otus;
CREATE TABLE info_user(
    userinf_id  int AUTO_INCREMENT PRIMARY KEY,
    user_id  int not NULL, 
    user_mail varchar(200), 
    user_tel varchar(20),  
    user_famaly  varchar(500), 
    user_grdate   datetime DEFAULT now() NOT NULL, 
    user_adress  varchar(500),  
    source_id int  NOT NULL default 0 
    ) ENGINE=InnoDB; 
CREATE TABLE post(
    post_id  int AUTO_INCREMENT PRIMARY KEY,
    post_text  TEXT not NULL, 
    post_desc varchar(200) NULL,   
    post_len int default 1 not NULL, 
    post_dt  datetime default now() not NULL
    ); 
CREATE TABLE dct_object(
    object_id int AUTO_INCREMENT PRIMARY KEY,
    object blob NULL, 
    object_desc varchar(200) NULL, 
    object_path varchar(500) NULL, 
    object_st varchar(100) DEFAULT 'active' NOT NULL 
    );
CREATE TABLE inpost(
    inpost_id int AUTO_INCREMENT PRIMARY KEY,
    user_id int not NULL, 
    post_id int not NULL, 
    inpost_dt datetime default now() not NULL,  
    object_id int default 0 not NULL, 
    source_id  int default 0 not NULL 
    ); 
CREATE TABLE outpost(
    outpost_id int AUTO_INCREMENT PRIMARY KEY,
    user_id int NOT NULL, 
    post_id int not NULL, 
    outpost_dt datetime default now() not NULL,  
    outpost_st varchar(100) default 'active' not NULL, 
    object_id int default 0 not NULL, 
    source_id int default 0  not NULL 
    );
CREATE TABLE repost(
    repost_id int AUTO_INCREMENT PRIMARY KEY,
    user_id int not NULL, 
    post_id int not NULL,  
    repost_dt datetime default now() not NULL,  
    repost_st varchar(100) default 'active' not NULL,  
    object_id int default 0 not NULL, 
    source_id int default 0 not NULL 
    );
CREATE TABLE dct_source(
    source_id int AUTO_INCREMENT PRIMARY KEY,
    source_name varchar(200) not NULL, 
    source_url varchar(400) NULL,  
    source_desc varchar(200) NULL 
    );
CREATE TABLE dct_log(
    log_id int AUTO_INCREMENT PRIMARY KEY,
    log_dt datetime DEFAULT now() NOT NULL,
    log_status varchar(100) DEFAULT 'INFO' NOT NULL,
    log_info varchar(500) NULL);
```
> Прогноз по возможному росту базы
```
**Таблица info_user: Рост зависит от количества пользователей. Если ожидается 100000 пользователей, то размер таблицы будет примерно:**
- user_id: 4 байта
- user_mail: 200 байт
- user_tel: 20 байт
- user_famaly: 500 байт
- user_grdate: 8 байт
- user_adress: 500 байт
- source_id: 4 байта
Итого: около 1.2 КБ на запись. Для 100000 пользователей: ~120 МБ.

**Таблица post: Рост зависит от активности пользователей. Если каждый пользователь создаёт по 10 постов (допустим):**
- post_text: TEXT (до 64 КБ)
- post_desc: 200 байт
- post_len: 4 байта
- post_dt: 8 байт
Итого: около 65 КБ на запись. Для 1 000 000 постов:  около 64 ГБ.

**Таблица dct_object:** 
- Рост зависит от количества объектов (например, изображений). Если объекты хранятся в базе (BLOB), то размер может быть значительным (например, 1 МБ на объект). Для 10000 объектов:  +-10 ГБ.

**Таблицы inpost, outpost, repost: Рост зависит от активности пользователей. Если каждый пользователь создаёт 10 записей:**
- +-100 байт на запись. Для 1000000 записей: +-100 МБ.

**Таблица dct_source:**
- Рост незначительный, так как источников обычно немного.

**Таблица dct_log:**
- Рост зависит от активности системы. Если система генерирует 1000 логов в день, то за год: более 365000 записей. Размер одной записи: от 100 байт. Итого: более 40 МБ.
```
> Рост количества пользователей
> Ожидается рост до 100000 пользователей в первый год.
> Активность пользователей: 10 постов на пользователя, 10 действий (inpost/outpost/repost) на пользователя.
> Всплески одновременных соединений
> Пиковые нагрузки: до 1000 одновременных соединений (например, во время маркетинговых акций или событий).
> Средняя нагрузка: 100-200 одновременных соединений.

##### Возможные угрозы и методы защиты
> Угрозы
> SQL-инъекции и решения:
```
- Использование подготовленных выражений (prepared statements) и ORM.
- Валидация входных данных.
- Потеря данных:
- Регулярное резервное копирование.
- Использование транзакций для критических операций.
```
> Недоступность сервиса и решение:
```
- Репликация и кластеризация для обеспечения отказоустойчивости.
- Мониторинг и автоматическое восстановление.
```
> Несанкционированный доступ и решение:
``` 
- Шифрование данных (например, паролей).
- Использование ролевой модели доступа.
```
> Перегрузка сервера:
```
- Оптимизация запросов (индексы, кэширование).
- Ограничение количества запросов от одного пользователя (rate limiting).
```
##### Стратегии бэкапа
> Регулярное резервное копирование
> Полный бэкап: Еженедельно.

> Инкрементальный бэкап: Ежедневно.

> Хранение бэкапов: На отдельном сервере или в облаке (например, AWS S3).

> Автоматизация
> Использование cron-заданий или инструментов вроде mysqldump для автоматического создания бэкапов.

> Тестирование восстановления
> Регулярно проверять, что бэкапы можно восстановить.
##### Репликация
> Настройка репликации
> Master-Slave репликация:
```
- Master для записи.
- Один или несколько Slave для чтения.
```
> Преимущества:
```
- Повышение отказоустойчивости.
-  Распределение нагрузки.
```
> Инструменты
```
- Встроенные механизмы MySQL/MariaDB для репликации.
```
##### Кластеризация
> Использование кластеров
```
- InnoDB, MySQL Cluster или Galera Cluster для горизонтального масштабирования.
```
> Преимущества:
```
- Высокая доступность.
- Распределение нагрузки между узлами.
```
##### Оптимизация производительности
> Индексы
```
Добавить индексы на часто используемые поля:
- user_id в таблицах info_user, inpost, outpost, repost.
- post_id в таблицах post, inpost, outpost, repost.
```
> Кэширование
```
- Использование Redis для кэширования часто запрашиваемых данных.
- Использование Clickhouse для аналитики
```
> Партиционирование
```
Партиционирование больших таблиц (например, post) по дате или другому критерию.
```

#### Итоговый документ
> Прогноз роста базы
> Ожидаемый размер базы через год: более 75 ГБ. 
> Количество пользователей: 100 000.
> Пиковые нагрузки: 1000 одновременных соединений.

> Угрозы и защита
> Защита от SQL-инъекций, потери данных, несанкционированного доступа и перегрузки сервера.

> Стратегии бэкапа
> Регулярное резервное копирование (полное и инкрементальное).

> Хранение бэкапов в облаке.

> Репликация и кластеризация
> Настройка Master-Slave репликации.

> Использование кластеров для высокой доступности.

> Оптимизация
> Добавление индексов, кэширование и партиционирование.
	
