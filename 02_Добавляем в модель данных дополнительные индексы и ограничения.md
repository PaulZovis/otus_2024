# Компоненты современной СУБД 
##### table user
```
user_id     int8 'sequence' not null          --высокая кардинальность
user_login  varchar not null                  --высокая кардинальность
user_rdate   date (timestamp) null            --высокая кардинальность
user_st  статус varchar default 'active' null --низкая кардинальность
```
> индексы
* уникальный индекс по user_id  *(возможен поиск и запросы, связь с другими таблицами)*
* индекс по user_login  *(поиск по логину)*
* индекс по user_st  *(поиск по статусу)*

##### table info_user
```
userinf_id  int8 'sequence' not null --высокая кардинальность
user_id  int8 not null               --высокая кардинальность
user_mail varchar (маска) null       --средняя кардинальность из-за null
user_tel varchar (маска) null        --высокая кардинальность
user_famaly  varchar null            --средняя кардинальность
user_grdate   date null              --средняя кардинальность
user_adress  varchar null            --высокая кардинальность
source_id int8 default 0 not null    --низкая кардинальность
```
> индексы
* уникальный индекс по userinf_id  *(поиск по идентификатору)*
* уникальный индекс по user_id, source_id *(поиск по ресурсу с пользователем)*

##### table post
```
post_id  int8 'sequence' not null  --высокая кардинальность
post_text  text not null           --высокая кардинальность
post_desc varchar null             --низкая кардинальность
post_len int8 default 1 not null   --высокая кардинальность
post_dt  date (timestamp) default now() not null --высокая кардинальность
```
> индексы
* уникальный индекс по userinf_id  *(поиск по идентификатору (ключу , если переносить в колоночную субд, ключ-значение))*
* индекс post_dt *(возможен поиск по дате)*
* индекс по post_desc, post_len  *(возможен поиск по описанию и длине)*
* индекс по тексту сообщения *(возможен поиск текстовых паттернов, полнотекстовый поиск)*

##### table dct_object
```
object_id int8 'sequence' not null   --высокая кардинальность
object null                          --высокая кардинальность
object_desc varchar null             --низкая кардинальность
object_path null                     --высокая кардинальность
```
> индексы
* уникальный индекс по object_id  *(поиск по идентификатору, для поиска и связей с др, таблицами при построении отчётов, сортировки)*

##### table inpost
```
inpost_id int8 'sequence' not null   --высокая кардинальность
user_id int8 not null                --высокая кардинальность
post_id int8 not null                --высокая кардинальность
inpost_dt date (timestamp) default now() not null  --высокая кардинальность
object_id int8 default 0 not null    --невысокая кардинальность
source_id  int8 not default 0 null   --низкая кардинальность
```
> индексы
* уникальный индекс по идентификатору
* индекс по пользователю и посту
* индекс по дате

##### table outpost
```
outpost_id int8 'sequence' not null  --высокая кардинальность
inpost_id int8   not null                     --высокая кардинальность
user_id int8  not null                       --высокая кардинальность
post_id int8 not null                --высокая кардинальность
outpost_dt date(timestamp) default now() not null  --высокая кардинальность
outpost_st varchar default 'active' not null --низкая кардинальность
object_id int8 default 0 not null    --низкая кардинальность
source_id int8 default 0  not null   --низкая кардинальность
```
> индексы
* уникальный индекс по мдентификатору outpost_id для поиска и связей с др, таблицами при построении отчётов, сортировки
* индекс по user_id, post_id для поиска и построения отчётов и дальнейших расчётов
* индекс по дате поста

##### table repost
```
repost_id int8 'sequence' not null   --высокая кардинальность
user_id int8 not null                --высокая кардинальность
post_id int8 not null                --высокая кардинальность
repost_dt date (timestamp) default now() not null  --высокая кардинальность
repost_st varchar default 'active' not null --низкая кардинальность
object_id int8 default 0 not null    --низкая кардинальность
source_id int8 default 0 not null    --низкая кардинальность
```
> индексы
* уникальный индекс по repost_id для поиска и связей с др, таблицами при построении отчётов, сортировки
* индекс по user_id, post_id для поиска и построения отчётов и дальнейших расчётов
* индекс по дате поста

##### table source
```
source_id int8 'sequence' not null   --высокая кардинальность
source_name varchar not null         --низкая кардинальность
source_url varchar not null          --низкая кардинальность
source_desc varchar null             --низкая кардинальность
```
> индексы
* уникальный индекс по source_id для поиска и связей с др, таблицами при построении отчётов, сортировки

##### Ограничения
- **(Возможно поставить ограничение по дате рождения не менее гггг-мм-дд)**
- **(Возможно поставить ограничение по дате поста не менее гггг-мм-дд и не более текущей)**
- **(Статус может принимать строго определённые значения, к примеру: актив, не актив, на паузе и т.д.)**
- **(not null указал в таблицах)**
- **(создать внешние ключи (в задании схема БД описал данные варианты))**
