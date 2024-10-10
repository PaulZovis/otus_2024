-- DML: вставка, обновление, удаление, выборка данных в PostgreSQL  -- создание последовательности


-- создадим табицы для выполнения дом.задания
create sequence seq_outpost_id start 1;                                              -- создание последовательности для идентификатора
create table outpost (                                                               -- таблица постов пользователя
  outpost_id int8 default nextval('seq_outpost_id'::regclass) not null primary key,  -- идентификатор
  outpost_text text default 'пусто' not null,                                        -- текст поста 
  users_id int8 not null,                                                            -- идентификатор пользователя
  snet_id int8 not null,                                                             -- идентификатор соц.сети
  outpost_date timestamp default now() not null);                                    -- дата поста



create sequence seq_users_id start 1;                                               -- создание последовательности для идентификатора
create table users (                                                                -- таблица пользователей
  users_id int8 default nextval('seq_users_id'::regclass) not null primary key,     -- идентификатор
  users_fio varchar(300) default 'некто' not null,                                  -- фио пользователя
  users_login varchar(500) not null,                                                -- логин пользователя
  users_rgdate timestamp default now() not null,                                    -- дата регистрации
  users_gr timestamp,                                                               -- дата рождения
  users_mr varchar(1000),                                                           -- место рождения
  users_adress_id int8,                                                             -- идентификатор адреса
  snet_id int8 not null);                                                           -- идентификтор сети


create sequence seq_inpost_id start 1;                                             -- создание последовательности для идентификатора
create table inpost (                                                              -- таблица постов, где отметился пользователь
  inpost_id int8 default nextval('seq_inpost_id'::regclass) not null primary key,  -- идентификатор
  outpost_id int8 not null,                                                        -- идентификатор поста из таблицы постов пользователей
  users_id int8 not null,                                                          -- идентификатор пользователя
  inpost_reac_id int8 not null,                                                    -- идентификатор реакции
  snet_id int8 not null,                                                           -- идентификатор сети
  inpost_date timestamp default now() not null,                                    -- дата реакции 
  inpostout_id int8 default 'no' not null);                                        -- ответ на пост (репост)


--1.
-- во скольких постах у каких пользователей стречается слово, частью которого является РЕГУЛЯР%
select us.users_fio, 						              		-- фио пользователя
       count(oup.outpost_text) as kpost        		-- количество постов
  from  outpost oup                    	        	-- таблица постов , написанных пользователем 
 inner join users us                   		        -- таблица пользователей
    on (oup.users_id = us.users_id)         	  	-- соединяем по идентификатору пользователя данного поста   
 where upper(oup.outpost_text) like '% РЕГУЛЯР%'  -- проверяем , что слово с составляющей РЕГУЛЯР% встречается в тексте
    or upper(oup.outpost_text) like 'РЕГУЛЯР%'    -- или пост начинается с РЕГУЛЯР*
 group by us.users_fio     						          	-- группируем по us.users_fio
 order by us.users_fio;								            -- сортируем по фио пользователя

select us.users_fio, 								              -- фио пользователя
       count(oup.outpost_text) as kpost        		-- количество постов
  from  outpost oup                    		        -- таблица постов , написанных пользователем 
 inner join users us                   		        -- таблица пользователей
    on (oup.users_id = us.users_id)         	   	-- соединяем по идентификатору пользователя данного поста   
 where oup.outpost_text ilike '% РЕГУЛЯР%'       	-- проверяем , что слово с составляющей РЕГУЛЯР% встречается в тексте
    or oup.outpost_text ilike 'РЕГУЛЯР%'    	  	-- или пост начинается с РЕГУЛЯР*
 group by us.users_fio     							          -- группируем по us.users_fio
 order by us.users_fio;	 							            -- сортируем по фио пользователя

select us.users_fio, 							              	-- фио пользователя
       count(oup.outpost_text) as kpost        		-- количество постов
  from  outpost oup                    	        	-- таблица постов , написанных пользователем 
 inner join users us                   	        	-- таблица пользователей
    on (oup.users_id = us.users_id)         	   	-- соединяем по идентификатору пользователя данного поста   
 where upper(oup.outpost_text) ~~ '% РЕГУЛЯР%'   	-- проверяем , что слово с составляющей РЕГУЛЯР% встречается в тексте
    or upper(oup.outpost_text) ~~ 'РЕГУЛЯР%'     	-- или пост начинается с РЕГУЛЯР*
 group by us.users_fio     					          		-- группируем по us.users_fio
 order by us.users_fio	;	 				            		-- сортируем по фио пользователя
 
 select us.users_fio, 							            	-- фио пользователя
       count(oup.outpost_text) as kpost        		-- количество постов
  from  outpost oup                           		-- таблица постов , написанных пользователем 
 inner join users us                   		        -- таблица пользователей
    on (oup.users_id = us.users_id)         	  	-- соединяем по идентификатору пользователя данного поста   
 where oup.outpost_text ~~* '% РЕГУЛЯР%'     	  	-- проверяем , что слово с составляющей РЕГУЛЯР% встречается в тексте
    or oup.outpost_text ~~* 'РЕГУЛЯР%'    		  	-- или пост начинается с РЕГУЛЯР*
 group by us.users_fio     						          	-- группируем по us.users_fio
 order by us.users_fio;								            -- сортируем по фио пользователя
 
 
 
 
 --2.
select us.users_fio, 							              	-- фио пользователя
       oup.outpost_text ,  			        		      -- пост пользователя
       oup.outpost_date								            -- дата публикации 
  from outpost oup                     		        -- таблица постов , написанных пользователем 
  left join users us                   		        -- таблица пользователей
    on (oup.users_id = us.users_id)         	  	-- соединяем по айди пользователя данного поста   
 where us.users_rgdate between '2024-05-01'::timestamp and '2024-10-10'::timestamp -- отбираем посты тех людей, которые зарегистрировались в период с 2024-05-01 по 2024-10-10
 order by 3		;									                  -- сортируем по фио пользователя
 --будут заполнены все поля, так как мы ищем посты и уже к ним приклеиваем пользователя
 
 select us.users_fio, 								            -- фио пользователя
       oup.outpost_text ,  			        	       	-- пост пользователя
       oup.outpost_date								            -- дата публикации 
  from users us                        		        -- таблица пользователей
  left join outpost oup                   		    -- таблица постов , написанных пользователем 
    on (oup.users_id = us.users_id)         		  -- соединяем по айди пользователя данного поста   
 where us.users_rgdate between '2024-05-01'::timestamp and '2024-10-10'::timestamp -- отбираем тех людей, которые зарегистрировались в период с 2024-05-01 по 2024-10-10 и их посты
 order by 3;										                	-- сортируем по фио пользователя
-- находим сначала пользователей, а потом их посты, т.е. пользователь будет, а постов у него не было
-- для оператора left или right join порядок таблиц важен

 
select us.users_fio, 							              	-- фио пользователя
       oup.outpost_text ,  			        		      -- пост пользователя
       oup.outpost_date								            -- дата публикации 
  from outpost oup                     	        	-- таблица постов , написанных пользователем 
 inner join users us                   		        -- таблица пользователей
    on (oup.users_id = us.users_id)         	  	-- соединяем по айди пользователя данного поста   
 where us.users_rgdate between '2024-05-01'::timestamp and '2024-10-10'::timestamp -- отбираем посты тех людей, которые зарегистрировались в период с 2024-05-01 по 2024-10-10
 order by 3	 ;

select us.users_fio, 								              -- фио пользователя
       oup.outpost_text ,  			              		-- пост пользователя
       oup.outpost_date								            -- дата публикации 
  from users us,                    	          	-- таблица постов , написанных пользователем 
       outpost oup                   			        -- таблица пользователей
 where oup.users_id = us.users_id					        -- соединяем по айди пользователя данного поста 
   and us.users_rgdate between '2024-05-01'::timestamp and '2024-10-10'::timestamp -- отбираем посты тех людей, которые зарегистрировались в период с 2024-05-01 по 2024-10-10
 order by 3	 ;
-- А вот порядок таблиц для оператора inner join неважен, поскольку оператор является коммутативным
 
 --3.
 -- вставка(добавление) поста в таблицу вручную с отображением вставленных строк
 insert into outpost (outpost_text, users_id , snet_id, outpost_date)
   values ('INSERT — добавить строки в таблицу', 1,1, '2024-06-23'::timestamp),
          ('INSERT добавляет строки в таблицу. Эта команда может добавить одну или несколько строк, сформированных выражениями значений, либо ноль или более строк, выданных дополнительным запросом.', 1,1, '2024-08-05'::timestamp),
          ('Чтобы добавлять строки в таблицу, необходимо иметь право INSERT для неё. Если присутствует предложение ON CONFLICT DO UPDATE, также требуется иметь право UPDATE для этой таблицы.',2,1,'2024-03-25'::timestamp)
     returning users_id, outpost_date ;
 
--4. 
--обновление поста в таблице постов по условию отсутствия вхождения в пост "insert" вне зависимости от регистра, с отображением обновлённых строк , где отображается дата и идентификатор соц.сети
update outpost
   set outpost_text = outpost_text || ' (тестовый вариант)'
 where outpost_text !~~* '%insert%' 
 returning outpost_date, snet_id;
 
--5.
-- удаление постов которые были созданы до 1  сентября с оборажением удалённых строк: дата, айди пользака, текст поста 
delete 
  from outpost oup
 where oup.outpost_date < '2024-09-01'::timestamp
   returning outpost_date, users_id, outpost_text;
   
--6.
-- имеем выгруженный в *.* файл с данными по *, где поля совпадают (в частном случае или не совпадают, но тогда просто создадим временную таблицу с количеством полей совпадающим с файлом *.*)
-- необходимо вставить данные в целевую таблицу посгресс из файла *.* 
copy users(users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) from 'c:/docs/users.txt' (delimiter ';' , header 'OFF', encoding 'utf8')
-- для ченр нужно?
-- для отчёта (простейшего без соответствующего интерфейса)) или надо перенести на физическом носителе или через сеть справочник/таблицу
-- выгружаем в файл
copy  (select * from outpost o where outpost_text ~~* '%тестовый вариант%') to 'c:/docs/outpost.csv';   

-- copy работает как из командной строки , так и из ide, но copy - не является командой sql (поэтому всегда писал скрипты из-под командной строки)
-- и только на этом занятии опрбовал в dbeaver (необходимости не было, обычно в командной строке работал), вердикт - работает в "бобре" (ide-шке)
