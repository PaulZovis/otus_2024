-- DML: вставка, обновление, удаление, выборка данных в PostgreSQL  -- создание последовательности
-- что-то пока не разобрался с форматированием,  а дата сдачи оказалась не по готовности ,  а до 10.10.2024 (надеюсь включительно)


-- простенький вариант, схему не создаю, но в оригинале создал схемку под данное ДЗ
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

-- не пригодилась, но задумка была
create sequence seq_inpost_id start 1;                                             -- создание последовательности для идентификатора
create table inpost (                                                              -- таблица постов, где отметился пользователь
  inpost_id int8 default nextval('seq_inpost_id'::regclass) not null primary key,  -- идентификатор
  outpost_id int8 not null,                                                        -- идентификатор поста из таблицы постов пользователей
  users_id int8 not null,                                                          -- идентификатор пользователя
  inpost_reac_id int8 not null,                                                    -- идентификатор реакции
  snet_id int8 not null,                                                           -- идентификатор сети
  inpost_date timestamp default now() not null,                                    -- дата реакции 
  inpostout_id int8 default 'no' not null);                                        -- ответ на пост (репост)

-- из "бобра" экспортнул данные для теста под каждую строку свой инсерт 
 -- вставка тестовых данных
INSERT INTO sapu.users (users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) VALUES ('Шакал Василь Джонович','Джон','2024-10-01 00:00:00.000','1935-03-25 00:00:00.000','Тунгуска',2);
INSERT INTO sapu.users (users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) VALUES ('Иванов Иван Попович','Поп','2024-10-09 14:53:16.590',NULL,NULL,NULL);
INSERT INTO sapu.users (users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) VALUES ('Веселов Ирраклий Борзанович','Борзый','2024-01-09 14:23:29.629','1994-05-03 00:00:00.000','Шаинск',1);
--INSERT INTO sapu.users (users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) VALUES ('Забавный Пётр Сергеевич',' Забава','2024-04-23 00:00:00.000','1999-01-01 00:00:00.000','Bonn',3); -- этого пользоателя импортил через copy из users.txt , созданный а notepad 
 --вставка тестовых данных
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('регуляр регулярный регулировал регулярки',2,1,'2024-10-09 14:29:37.282');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('регулярно регулировщик читал нам стихи на регулируемом перекрётске',1,1,'2024-10-09 14:29:37.280');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('приём пищи регулярно , не даст вам умереть от истощения',2,1,'2024-10-06 00:00:00.000');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('Ругулярка встала поперёк запроса урегулирования',2,1,'2024-10-05 00:00:00.000');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('Регулярка решает всё, будт как всё',1,1,'2024-10-04 00:00:00.000');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('Было время, был я беден, а теперь никак',1,1,'2024-10-03 00:00:00.000');
INSERT INTO sapu.outpost (outpost_text,users_id,snet_id,outpost_date) VALUES ('Регулярный обход здания регулярно поражал нас всех',1,1,'2024-10-03 00:00:00.000');



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
-- удаление повторных постов по пользователю, тексту и идентификатору соц.сети (изменил форматирование в dbeaver под заглавные буквы с лужебных словах, поэтому отличается от предыдущих вариантов)
DELETE 
  from outpost oup
  USING (SELECT outpost_text,users_id, snet_id, max(ctid) sctid FROM outpost GROUP BY 1,2,3) dd
 where oup.outpost_text = dd.outpost_text
   AND oup.users_id = dd.users_id
   AND oup.snet_id = dd.snet_id
   AND oup.ctid <> dd.sctid
   returning oup.users_id, oup.outpost_text, outpost_date; 
   
--6.
-- имеем выгруженный в *.* файл с данными по *, где поля совпадают (в частном случае или не совпадают, но тогда просто создадим временную таблицу с количеством полей совпадающим с файлом *.*)
-- необходимо вставить данные в целевую таблицу посгресс из файла *.* 
copy users(users_fio,users_login,users_rgdate,users_gr,users_mr,users_adress_id) from 'c:/users.txt' (delimiter ';' , header 'OFF', encoding 'utf8')  -- не успел на убунте поиграться, но был бы путь прописан чуть иначе
-- для ченр нужно?
-- для отчёта (простейшего без соответствующего интерфейса)) или надо перенести на физическом носителе или через сеть справочник/таблицу
-- выгружаем в файл
copy  (select * from outpost o where outpost_text ~~* '%тестовый вариант%') to 'c:/outpost.csv';    -- не успел на убунте поиграться, но был бы путь прописан чуть иначе

-- copy работает как из командной строки , так и из ide, но copy - не является командой sql (поэтому всегда писал скрипты из-под командной строки)
-- и только на этом занятии опрбовал в dbeaver (необходимости не было, обычно в командной строке работал), вердикт - работает в "бобре" (ide-шке)
