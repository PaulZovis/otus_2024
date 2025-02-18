# DDL: создание, изменение и удаление объектов в PostgreSQL 

> создаём бд
```
DROP DATABASE IF exists psy_project;
CREATE DATABASE psy_project;
```

>SELECT current_database()


> предварительно создал папки для табличных пространств
> создание табличных пространств 
```
CREATE TABLESPACE calc_space LOCATION 'c:\PostgreSQL\16\data\calc_space';
```
> создание бд со своим табличным простанством
```
DROP DATABASE IF EXISTS psy_calc;
CREATE DATABASE psy_calc WITH TABLESPACE calc_space;
```
> создание табличных пространств 
```
CREATE TABLESPACE info_space  LOCATION 'c:\PostgreSQL\16\data\info_space';
```
> создание пользователей,ролей
```
DROP USER IF EXISTS user_calc;
DROP USER IF EXISTS user_info;
DROP USER IF EXISTS user_sapu;
DROP USER IF EXISTS user_admin;
```
> создание пользователей c разными характеристиками
```
CREATE ROLE  user_calc WITH login NOSUPERUSER NOCREATEDB NOCREATEROLE PASSWORD '111' VALID UNTIL '2025-02-25' ;
CREATE USER user_info WITH login NOSUPERUSER NOCREATEDB PASSWORD '222';
CREATE USER user_sapu WITH login NOSUPERUSER NOCREATEDB PASSWORD '333';
CREATE USER user_admin WITH SUPERUSER PASSWORD '444';
```

> создание информационной схемы
```
DROP schema IF EXISTS info;
CREATE SCHEMA info AUTHORIZATION user_info;
```
> создание целевой схемы
```
DROP schema IF EXISTS calc;
CREATE SCHEMA calc AUTHORIZATION user_admin;
```
> создание целевой схемы
```
DROP schema IF EXISTS sapu;
CREATE SCHEMA sapu AUTHORIZATION user_sapu;
```
> создание схемы расширений
```
CREATE SCHEMA extensions;
```
> создание таблицы в нужном нам табличном пространстве
```
DROP TABLE IF EXISTS info.info_users ;
CREATE TABLE info.info_users(users_id serial PRIMARY KEY, users_text jsonb) TABLESPACE info_space;
```
> создание последовательность для таблицы user
```
CREATE SEQUENCE sapu.seq_user_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```    
> создание таблицы пользователей
```
CREATE TABLE sapu.users (
    user_id bigint DEFAULT nextval('sapu.seq_user_id'::regclass) NOT NULL PRIMARY key,  -- уникальный идентификатор
    user_login character varying(500) NOT NULL,                                         -- логин
    user_rdate timestamp without time zone DEFAULT now() NOT NULL,                      -- дата регистрации
    user_st varchar(50) DEFAULT 'ACTIVE' NOT NULL                                       -- статус пользователя
    );        
```
> создание последовательности для идентифиатора таблицs информации о пользователях
```
CREATE SEQUENCE sapu.seq_userinf_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> создание таблицы информации о пользователях
```
CREATE TABLE sapu.info_user(
    userinf_id  int8 DEFAULT nextval('sapu.seq_userinf_id'::regclass) NOT NULL PRIMARY KEY ,  --уникальный идентификатор
    user_id  int8 not NULL, -- идентификатор пользователя
    user_mail varchar NULL, -- адрес эл.почты
    user_tel varchar NULL,  -- телефон
    user_famaly  varchar NULL, -- семейное положение
    user_grdate   date NULL, -- дата рождения
    user_adress  varchar NULL,  -- адрес
    source_id int8 default 0 not NULL  --идентификатор ресурса
    ); 
```
> создание последовательности для идентифиатора таблицы сообщений
```
CREATE SEQUENCE sapu.seq_post_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица сообщений (словарь, архив) возможно вынести в колоночную субд
```
CREATE TABLE sapu.post(
    post_id  int8 DEFAULT nextval('sapu.seq_post_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный идентификатор 
    post_text  TEXT not NULL, -- само текстовое сообщение
    post_desc varchar NULL,   -- описание
    post_len int8 default 1 not NULL, -- длина сообщения lenght()
    post_dt  timestamp(0) default now() not NULL  -- дата помещения в бд
    ); 
```
> создание последовательности для идентифиатора таблицы объектов, которые могут присоединять к постам
```
CREATE SEQUENCE sapu.seq_object_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица объектов в сообщениях (фото, видео, ....)
```
CREATE TABLE sapu.dct_object(
    object_id int8 DEFAULT nextval('sapu.seq_object_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный  идентификатор
    object bytea NULL, -- объект
    object_desc varchar NULL, -- описание объекта
    object_path varchar NULL, -- путь к хранилищу или признак хранения
    object_st varchar DEFAULT 'active' NOT NULL  -- статус состояния объектов
    );
```
> создание последовательности для идентифиатора таблицы постов
```
CREATE SEQUENCE sapu.seq_inpost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица непосредственно самих постов (сообщений)
```
CREATE TABLE sapu.inpost(
    inpost_id int8 DEFAULT nextval('sapu.seq_inpost_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный идентификатор
    user_id int8 not NULL, -- пользователь
    post_id int8 not NULL, -- сообщение
    inpost_dt timestamp(0) default now() not NULL,  -- дата поста
    object_id int8 default 0 not NULL, -- присоединённый объект
    source_id  int8 default 0 not NULL -- ресурс, откуда взято сообщение
    ); 
```
> создание последовательности для идентифиатора таблицы ответов на посты
```
CREATE SEQUENCE sapu.seq_outpost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица ответов на посты
```
CREATE TABLE sapu.outpost(
    outpost_id int8 DEFAULT nextval('sapu.seq_outpost_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    inpost_id int8 NOT NULL,  -- на какой пост отвечаем
    user_id int8 NOT NULL, -- пользователь
    post_id int8 not NULL, -- сообщение
    outpost_dt timestamp(0) default now() not NULL,  -- дата ответа
    outpost_st varchar default 'active' not NULL, --статус ответа
    object_id int8 default 0 not NULL, -- объект
    source_id int8 default 0  not NULL -- ресурс
    );
```
> создание последовательности для идентифиатора таблицы репостов
```
CREATE SEQUENCE sapu.seq_repost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица репостов
```
CREATE TABLE sapu.repost(
    repost_id int8 DEFAULT nextval('sapu.seq_repost_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    user_id int8 not NULL, -- пользователь
    post_id int8 not NULL,  -- сообщения
    repost_dt timestamp(0) default now() not NULL,  -- дата репоста
    repost_st varchar default 'active' not NULL,  -- статус репоста
    object_id int8 default 0 not NULL,  -- объект
    source_id int8 default 0 not NULL  -- ресурс
    );
```
> создание последовательности для идентифиатора таблицы ресурса постов(сообщений)
```
CREATE SEQUENCE sapu.seq_source_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> таблица ресурса, беседы, откуда взяты тексты
```
CREATE TABLE sapu.dct_source(
    source_id int8 DEFAULT nextval('sapu.seq_source_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    source_name varchar not NULL, -- наименование ресурса
    source_url varchar NULL,  --ссылка, откуда ведём беседу
    source_desc varchar NULL -- описание ресурса
    );
```
> создание последовательности для идентифиатора таблицы логирования
```
CREATE SEQUENCE info.seq_log_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```
> создание таблицы логирования
```
CREATE TABLE info.dct_log(
    log_id int8 DEFAULT nextval('info.seq_log_id'::regclass) NOT NULL PRIMARY KEY,
    log_dt timestamp(8) DEFAULT now() NOT NULL,
    log_status varchar(200) DEFAULT 'INFO' NOT NULL,
    log_info TEXT NULL);
```
> создание внешних ключей
```
ALTER TABLE sapu.info_user ADD CONSTRAINT fk_info_user_id  
  FOREIGN KEY (user_id) REFERENCES sapu.users (user_id);
  
ALTER TABLE sapu.inpost ADD CONSTRAINT fk_inpost_id  
  FOREIGN KEY (post_id) REFERENCES sapu.post (post_id);
    
ALTER TABLE sapu.outpost ADD CONSTRAINT fk_outpost_id  
  FOREIGN KEY (post_id) REFERENCES sapu.post (post_id);
    
ALTER TABLE sapu.repost ADD CONSTRAINT fk_repost_id  
  FOREIGN KEY (post_id) REFERENCES sapu.post (post_id);
  
ALTER TABLE sapu.inpost ADD CONSTRAINT fk_inpost_user_id  
  FOREIGN KEY (user_id) REFERENCES sapu.users (user_id);
    
ALTER TABLE sapu.outpost ADD CONSTRAINT fk_outpost_user_id  
  FOREIGN KEY (user_id) REFERENCES sapu.users (user_id);
  
ALTER TABLE sapu.repost ADD CONSTRAINT fk_repost_user_id  
  FOREIGN KEY (user_id) REFERENCES sapu.users (user_id);
  
ALTER TABLE sapu.inpost ADD CONSTRAINT fk_inpost_object_id  
  FOREIGN KEY (object_id) REFERENCES sapu.dct_object (object_id);
    
ALTER TABLE sapu.outpost ADD CONSTRAINT fk_outpost_object_id  
  FOREIGN KEY (object_id) REFERENCES sapu.dct_object (object_id);
    
ALTER TABLE sapu.repost ADD CONSTRAINT fk_repost_object_id  
  FOREIGN KEY (object_id) REFERENCES sapu.dct_object (object_id);  
  
ALTER TABLE sapu.inpost ADD CONSTRAINT fk_inpost_source_id  
  FOREIGN KEY (source_id) REFERENCES sapu.dct_source (source_id);
    
ALTER TABLE sapu.outpost ADD CONSTRAINT fk_outpost_source_id  
  FOREIGN KEY (source_id) REFERENCES sapu.dct_source (source_id);
    
ALTER TABLE sapu.repost ADD CONSTRAINT fk_repost_source_id  
  FOREIGN KEY (source_id) REFERENCES sapu.dct_source (source_id);    
```
