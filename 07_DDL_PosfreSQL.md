-- создаём бд
--DROP DATABASE IF exists psy_project;
--CREATE DATABASE psy_project;

--создание схемы для расчётов
CREATE SCHEMA calc;
--создание информационной схемы
CREATE SCHEMA info;
--создание целевой схемы
CREATE SCHEMA sapu;
--создание схемы расширений
CREATE SCHEMA extensions;

--создание табличных пространств 
--CREATE TABLESPACE calc_space;

--CREATE TABLESPACE info_space;

-- создание пользователей
CREATE USER user_calc;
CREATE USER user_info;
CREATE USER user_sapu;
CREATE USER user_admin;

--создание ролей
CREATE ROLE role_calc;
CREATE ROLE role_info;
CREATE ROLE role_sapu;
--CREATE ROLE role_admin INHERITS WITH postgres;

--создание последовательность для таблицы user
CREATE SEQUENCE sapu.seq_user_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
--создание таблицы пользователей
CREATE TABLE sapu.user (
    user_id bigint DEFAULT nextval('sapu.seq_user_id'::regclass) NOT NULL PRIMARY key,  -- уникальный идентификатор
    user_login character varying(500) NOT NULL,                                         -- логин
    user_rdate timestamp without time zone DEFAULT now() NOT NULL,                      -- дата регистрации
    user_st varchar(50) DEFAULT 'ACTIVE' NOT NULL                                       -- статус пользователя
    );        

--создание последовательности для идентифиатора таблицs информации о пользователях
CREATE SEQUENCE sapu.seq_userinf_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
--создание таблицы информации о пользователях
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

--создание последовательности для идентифиатора таблицы сообщений
CREATE SEQUENCE sapu.seq_post_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
-- таблица сообщений (словарь, архив) возможно вынести в колоночную субд
CREATE TABLE post(
    post_id  int8 DEFAULT nextval('sapu.seq_post_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный идентификатор 
    post_text  TEXT not NULL, -- само текстовое сообщение
    post_desc varchar NULL,   -- описание
    post_len int8 default 1 not NULL, -- длина сообщения lenght()
    post_dt  timestamp(0) default now() not NULL  -- дата помещения в бд
    ); 
--создание последовательности для идентифиатора таблицы объектов, которые могут присоединять к постам
CREATE SEQUENCE sapu.seq_object_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
-- таблица объектов в сообщениях (фото, видео, ....)
CREATE TABLE dct_object(
    object_id int8 DEFAULT nextval('sapu.seq_object_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный  идентификатор
    object bytea NULL, -- объект
    object_desc varchar NULL, -- описание объекта
    object_path varchar NULL, -- путь к хранилищу или признак хранения
    object_st varchar DEFAULT 'active' NOT NULL  -- статус состояния объектов
    );

--создание последовательности для идентифиатора таблицы постов
CREATE SEQUENCE sapu.seq_inpost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
-- таблица непосредственно самих постов (сообщений)
CREATE TABLE inpost(
    inpost_id int8 DEFAULT nextval('sapu.seq_inpost_id'::regclass) NOT NULL PRIMARY KEY,  -- уникальный идентификатор
    user_id int8 not NULL, -- пользователь
    post_id int8 not NULL, -- сообщение
    inpost_dt timestamp(0) default now() not NULL,  -- дата поста
    object_id int8 default 0 not NULL, -- присоединённый объект
    source_id  int8 default 0 not NULL -- ресурс, откуда взято сообщение
    ); 

--создание последовательности для идентифиатора таблицы ответов на посты
CREATE SEQUENCE sapu.seq_outpost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
-- таблица ответов на посты
CREATE TABLE outpost(
    outpost_id int8 DEFAULT nextval('sapu.seq_outpost_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    inpost_id int8 NOT NULL,  -- на какой пост отвечаем
    user_id int8 NOT NULL, -- пользователь
    post_id int8 not NULL, -- сообщение
    outpost_dt timestamp(0) default now() not NULL,  -- дата ответа
    outpost_st varchar default 'active' not NULL, --статус ответа
    object_id int8 default 0 not NULL, -- объект
    source_id int8 default 0  not NULL -- ресурс
    );

--создание последовательности для идентифиатора таблицы репостов
CREATE SEQUENCE sapu.seq_repost_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
--таблица репостов
CREATE TABLE repost(
    repost_id int8 DEFAULT nextval('sapu.seq_repost_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    user_id int8 not NULL, -- пользователь
    post_id int8 not NULL,  -- сообщения
    repost_dt timestamp(0) default now() not NULL,  -- дата репоста
    repost_st varchar default 'active' not NULL,  -- статус репоста
    object_id int8 default 0 not NULL,  -- объект
    source_id int8 default 0 not NULL  -- ресурс
    );

--создание последовательности для идентифиатора таблицы ресурса постов(сообщений)
CREATE SEQUENCE sapu.seq_source_id
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
-- таблица ресурса, беседы, откуда взяты тексты
CREATE TABLE dct_source(
    source_id int8 DEFAULT nextval('sapu.seq_source_id'::regclass) NOT NULL PRIMARY KEY, -- уникальный идентификатор
    source_name varchar not NULL, -- наименование ресурса
    source_url varchar NULL,  --ссылка, откуда ведём беседу
    source_desc varchar NULL -- описание ресурса
    );
