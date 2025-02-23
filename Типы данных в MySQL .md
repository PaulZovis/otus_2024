# Типы данных в MySQL 
## Изменения типов данных
- меняем, так как возможно несколько телефонов 
```
ALTER TABLE info_user MODIFY user_mail VARCHAR(200);4
```
- 4 GB проттив 64 КВ , текст поста может быть и "войной и миром"
```
ALTER TABLE post MODIFY post_text longtext;
```
- расширяем возможности описания поста
```
ALTER TABLE post MODIFY post_desc varchar(1000);
```
- расширяем возможности описания поста
```
ALTER TABLE dct_object MODIFY object_desc varchar(1000);
```
- расширяем возможности описания ресурса
```
ALTER TABLE dct_source MODIFY source_desc varchar(1000);
```
- расширяем возможности описания поста
```
ALTER TABLE dct_object MODIFY object_desc varchar(1000);
```

## Добавление типа JSON
- Добавлено поле `info_preferences` с типом `JSON` для хранения дополнительных атрибутов пользователя.
```
ALTER TABLE info_user ADD COLUMN info_preferences JSON;
```
- Добавлено поле `log_preferences` с типом `JSON` для хранения дополнительных атрибутов пользователя.
```
ALTER TABLE dct_log ADD COLUMN log_preferences JSON;
```

## Анализ JSON
> В данное поле `log_preferences` можно дублировать всю информацию о пользователе, дополненную и заточенную под задачу
> В поле `log_preferences` можно складывать  информацию, не предусмотренную основными полями таблицы, более расширенные логи

![image](https://github.com/user-attachments/assets/f0b5f145-7861-422c-8ebc-459b1213bfd7)
