# Индексы в MySQL 
> Схема всё та же, таблицы те же
```
CREATE database IF NOT EXISTS otusdb;
USE otus;
CREATE TABLE users (
  user_id int(11) NOT NULL AUTO_INCREMENT,
  user_login varchar(500) NOT NULL,
  user_rdate timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  user_st varchar(200) NOT NULL DEFAULT 'ACTIVE',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB; 
CREATE TABLE info_user(
    userinf_id  int AUTO_INCREMENT PRIMARY KEY,
    user_id  int not NULL, 
    user_mail varchar(200), 
    user_tel varchar(20),  
    user_fio  varchar(500), 
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
    ) ENGINE=InnoDB;  
CREATE TABLE dct_object(
    object_id int AUTO_INCREMENT PRIMARY KEY,
    object blob NULL, 
    object_desc varchar(200) NULL, 
    object_path varchar(500) NULL, 
    object_st varchar(100) DEFAULT 'active' NOT NULL 
    ) ENGINE=InnoDB; 
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
> В оригинальной  схеме есть несколько таблиц, но для реализации полнотекстового поиска нам нужно выбрать те таблицы, которые содержат текстовые поля для поиска. Потенциально подходящие таблицы:
```
post (содержит post_text и post_desc)
dct_object (содержит object_desc)
dct_source (содержит source_name и source_desc)
users (содкржит user_login, но тут высокая кардинальность и уникальность, uniq однозначно)
info_user (содержит несколько полей с типом данных varchar, но эти поля с высокой степенью вероятности будут null, надо смотреть по ситуации и дальнейшей статистике наполнения)
```
> Накидаем скрипты для создания индексов
```
ALTER TABLE post ADD FULLTEXT(post_text);  -- возможен полнотекстовый поиск
ALTER TABLE post ADD FULLTEXT(post_desc);  -- возможен полнотекстовый поиск
ALTER TABLE dct_object ADD FULLTEXT(object_desc); --возможен полнотекстовый поиск
ALTER TABLE dct_source ADD FULLTEXT(post_desc); --возможен полнотекстовый поиск
CREATE UNIQUE INDEX users_user_login_IDX USING HASH ON otus.users (user_login); -- поиск по равенству логина, тут мысль мелькнула , а не заменить ли user_id на user_login, но стоит опробовать на данных
```
> Пример: при построении индексов видим построение плана с учётом индекса (в командной строке)
```
explain select * from post where post_text = 'test';
explain  select u.user_login, user_id, i.user_fio    from users u    left join info_user i using (user_id)   where user_login = 'mIkoLa';
```
![in1](https://github.com/user-attachments/assets/10146086-5266-47eb-91b7-8dc9d8406b7a)
![in2](https://github.com/user-attachments/assets/5bc22a90-8e25-449b-b2dd-1fa424fa9f73)
> в DBeaver
```
explain 
SELECT * 
FROM post 
WHERE MATCH(post_text) AGAINST ('парниша' IN NATURAL LANGUAGE MODE);
```
![in3](https://github.com/user-attachments/assets/3991ea31-d555-4380-ac40-059bc0634902)
