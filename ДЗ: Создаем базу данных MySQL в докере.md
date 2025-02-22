# ДЗ: Создаем базу данных MySQL в докере

> Сделал согласно инструкции
```
Описание/Пошаговая инструкция выполнения домашнего задания:
забрать стартовый репозиторий https://github.com/aeuge/otus-mysql-docker
прописать sql скрипт для создания своей БД в init.sql
проверить запуск и работу контейнера следую описанию в репозитории
```
![image](https://github.com/user-attachments/assets/8e99f030-e1d2-4d82-83df-91f728ecddd4)
![image](https://github.com/user-attachments/assets/0c7e481e-087d-4f63-bc63-cc9ee1fbafec)
![image](https://github.com/user-attachments/assets/fb52d88d-d808-44e1-9f4d-0f10441b11ec)

> *.yml в данном варианте , как родной, если не ошибаюсь , на домашнем возникли сложности и времени ушло много,
> простой вариант делается быстро, а вот сложность вызвала проблемы, не связанные со сложностью задачи, но... 
```
version: '3.1'

services:
  otusdb:
    image: mysql:8.0.15
    environment:
      - MYSQL_ROOT_PASSWORD=12345
    command: 
      --init-file /init.sql
    volumes:
      - data:/var/lib/mysql
      - ./init.sql:/init.sql
      - ./custom.conf:/conf.d/my.cnf
    expose:
      - "3306"
    ports:
      - "3306:3306"

volumes:
  data:

```
> init.sql
```
DROP DATABASE IF EXISTS otus;
CREATE database otus;
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
> Для сложного создал конфиг
```
[mysqld]
innodb_buffer_pool_size = 1G
max_connections = 200
query_cache_size = 64M
```
