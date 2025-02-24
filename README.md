# DML: вставка, обновление, удаление, выборка данных в MySQL 
> Создадим БД
```
CREATE database otusdb;
USE otusdb;
CREATE TABLE info_user(
    userinf_id  int AUTO_INCREMENT PRIMARY KEY,
    user_id  int not NULL, 
    user_mail varchar(200), 
    user_tel varchar(20),  
    user_famaly  varchar(500), 
    user_grdate   datetime DEFAULT now() NOT NULL, 
    user_adress  varchar(500),  
    source_id int  NOT NULL default 0 
    ); 
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
CREATE TABLE users (
    user_id serial PRIMARY KEY ,
    user_login varchar(500) NOT NULL,
    user_dg timestamp DEFAULT now() NOT NULL,
    user_st varchar(100) DEFAULT 'ACTIVE'
);
```
> Произведём вставку данных
```
INSERT users (user_login) 
  VALUES 
    ('bambardia'), ('bambia'),('bumbardia'),('babardia'),('bardia'),('babdia'),('bardia'),
    ('shara'),('viagra'),('urna'),('kiakara'),('pipinsta'),('lualay'),('mara');
 INSERT INTO dct_source(source_name,source_url,source_desc)
   VALUES 
     ('telegraff','http://telegraff.com','очень интересно'),
     ('voknoklassniki','http://vocno.ru','взаимовыгодно'),
     ('tiki-toki','http://tiki_toki.ch','музыкально');
 INSERT INTO info_user(user_id,user_mail,user_tel,user_famaly,user_grdate,user_adress,source_id)	
   VALUES
     (2,'234@mail.ru','6382652287,3283228324,0023221','rer',now(),NULL,3),
     (3,'ba_tt@mail.ru','63826587',NULL ,'1975-12-12',NULL,1),
     (4,'otrtto@yandex.ru','10998',NULL,'1973-11-21 00:00:00',NULL,1);
INSERT INTO post(post_text,post_desc,post_len)
  VALUES 
    ('пост 1', 'о жизни', CHAR_LENGTH('пост 1')),
    ('пост 22', 'о красоте', CHAR_LENGTH('пост 22')),
    ('пост 333', 'о науке', CHAR_LENGTH('пост 333')),
    ('пост 444', 'программирование', CHAR_LENGTH('пост 444')),
    ('пост 5555', 'треш', CHAR_LENGTH('пост 5555')),
    ('пост6', 'беседа', CHAR_LENGTH('пост 6'));
INSERT INTO outpost (user_id,post_id,outpost_st,source_id)
  VALUES 
    (1,1,'active',1),
    (4,2,'delete',2),
    (5,3,'active',1),
    (2,4,'active',2),
    (1,5,'active',1),
    (7,6,'active',1),
    (8,2,'active',1),
    (6,4,'repost',1),
    (1,6,'active',1);
```
> Запросы с inner join, left join
```
SELECT user_login, post_text, post_desc, outpost_st 
  FROM post 
 INNER JOIN outpost USING(post_id)
 INNER JOIN users USING(user_id)
 ORDER BY user_login;
 
SELECT  user_id ,user_login,user_dg, user_mail, user_grdate
  FROM users u 
  LEFT JOIN info_user i using(user_id);
```
> Запросы с where
```
SELECT user_login, post_text, post_desc, outpost_st 
  FROM post 
 INNER JOIN outpost USING(post_id)
 INNER JOIN users USING(user_id)
 WHERE outpost_st != 'delete'
 ORDER BY user_login;
 
 SELECT user_login, post_text, post_desc, outpost_st 
  FROM post 
 INNER JOIN outpost USING(post_id)
 INNER JOIN users USING(user_id)
 WHERE post_len>7
 ORDER BY user_login;
 
 SELECT  user_id ,user_login,user_dg, user_mail, user_grdate
  FROM users u 
  LEFT JOIN info_user i using(user_id)
  WHERE user_mail IS NOT NULL;
  
SELECT  user_id ,user_login,user_dg, user_mail, user_grdate
  FROM users u 
  LEFT JOIN info_user i using(user_id)
  WHERE user_grdate > '1975-01-01';
  
 SELECT user_login, user_mail,source_name , source_url 
   FROM users u
   INNER JOIN info_user i ON i.user_id = u.user_id 
   INNER JOIN dct_source d ON d.source_id = i.source_id
   INNER JOIN ( SELECT count(1) count_post, user_id FROM outpost GROUP BY user_id) p ON p.user_id=u.user_id
   WHERE source_name LIKE '%telegraff'
     AND count_post > 1;
```  
