# Хранимые-процедуры-и-триггеры-в-MySQL

> Создание пользователя client
```
CREATE USER 'client'@'localhost' IDENTIFIED BY 'client_password';
```
>  Создание пользователя manager
```
CREATE USER 'manager'@'localhost' IDENTIFIED BY 'manager_password';
```
> Процедура выборки постов
> Эта процедура позволяет фильтровать посты по длине текста, наличию определённых слов (полный текстовый поиск), пользователю, а также поддерживает сортировку и постраничную выдачу.
```
CREATE PROCEDURE get_filtered_posts(
    IN p_min_len INT,               -- Минимальная длина текста
    IN p_max_len INT,               -- Максимальная длина текста
    IN p_words VARCHAR(500),      -- Слова для полнотекстового поиска
    IN p_login varchar(500), -- Фильтр по логину пользователя
    IN p_limit INT              -- Количество элементов на странице
)
BEGIN
    SELECT p.post_id, p.post_text, p.post_desc, p.post_len, i.inpost_id, i.inpost_dt, u.user_login, u.user_id 
      FROM inpost as i
     INNER JOIN post AS p using(post_id)
      LEFT JOIN users AS u using(user_id)
        WHERE 1=1
          AND (CASE WHEN p_min_len != 0 THEN p.post_len >= p_min_len ELSE 1=1 END)
          AND (CASE WHEN p_max_len != 0 THEN p.post_len <= p_max_len ELSE 1=1 END)
          AND (CASE WHEN p_login != 'NONE' THEN u.user_login = p_login ELSE 1=1 end)
          AND (CASE WHEN p_words != 'NONE' OR p_words != '' THEN MATCH(post_text) AGAINST(p_words IN NATURAL LANGUAGE MODE) ELSE 1=1 END)
        LIMIT p_limit;
END;
```
> Процедура выборки постов
```
CREATE PROCEDURE get_filtered_posts_more(
    IN p_min_len INT,               -- Минимальная длина текста
    IN p_max_len INT,               -- Максимальная длина текста
    IN p_words VARCHAR(500),        -- Слова для полнотекстового поиска
    IN p_login varchar(500),        -- Фильтр по логину пользователя
    IN p_field varchar(20),         -- Порядок полей через запятую 1-8, пример '1,3,4;
    IN p_sort VARCHAR(10),          -- Порядок сортировки (ASC/DESC)
    IN p_limit INT                  -- Количество элементов на странице
)
BEGIN
    SET @query = CONCAT(' 
    SELECT p.post_id, p.post_text, p.post_desc, p.post_len, i.inpost_id, i.inpost_dt, u.user_login, u.user_id 
      FROM inpost as i
     INNER JOIN post AS p using(post_id)
      LEFT JOIN users AS u using(user_id)
        WHERE 1=1
          AND ', IF(p_min_len != 0, ' p.post_len >= '||p_min_len, '1=1'),
        ' AND ', IF(p_max_len != 0, ' p.post_len <= '||p_max_len, '1=1'),
        ' AND ', IF(p_login != 'NONE', ' u.user_login = '||p_login, '1=1'),
        ' AND ', IF(p_words !=  'NONE' OR p_words !=  '', ' MATCH(post_text) AGAINST('||quote(p_words)||' IN NATURAL LANGUAGE MODE) ', '1 = 1'),
      ' ORDER BY ', IF(p_field !=  'NONE' OR p_field !=  '', p_field, ''), ' ',p_sort,        
        ' LIMIT ', p_limit);
    PREPARE stmt FROM @query;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END;
```
> Тестируем 
```
CALL get_filtered_posts_more(0,0,'','NONE','1,2,3','ASC',2);
CALL get_filtered_posts(3,15,'test','NONE',3);
```
> Процедура отчета по постам
> Эта процедура позволяет просматривать отчет по постам за определенный период с группировкой по длине текста, source_id или пользователю.
```
CREATE PROCEDURE get_posts(
    IN period VARCHAR(50),            -- Период (hour/day/week)
    IN group_by VARCHAR(50)           -- Группировка (post_len/source_id/user_id)
)
BEGIN
    IF period = 'hour' THEN
        SET @period_condition = 'AND HOUR(inpost_dt) = HOUR(NOW())';
    ELSEIF period = 'day' THEN
        SET @period_condition = 'AND DATE(inpost_dt) = CURDATE()';
    ELSEIF period = 'week' THEN
        SET @period_condition = 'AND WEEK(inpost_dt) = WEEK(NOW())';
    ELSEIF period = 'year' THEN 
        SET @period_condition = 'AND YEAR(inpost_dt) = YEAR(NOW())'; 
    ELSE
        SET @period_condition = '';
    END IF;

    SET @query = CONCAT(
        'SELECT p.post_len, 
                i.source_id, 
                i.inpost_dt,
                i.user_id, 
                u.user_login,
                COUNT(*) AS total_posts
           FROM inpost i 
          INNER JOIN post p ON i.post_id = p.post_id
          INNER JOIN users u ON u.user_id = i.user_id
          WHERE 1=1 ', @period_condition,
        ' GROUP BY 1,2,3,4,5
          ORDER BY ',group_by
    );
    PREPARE stmt FROM @query;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END;
```
> 
```
CALL get_posts('year', 'user_id');
```
> Даем права на выполнение процедуры get_filtered_posts
```
GRANT EXECUTE ON PROCEDURE otus.get_filtered_posts TO 'client'@'localhost';
```
> Даем права на выполнение процедуры get_posts
```
GRANT EXECUTE ON PROCEDURE otus.get_posts TO 'manager'@'localhost';
```
> Создание триггеров
> Триггеры могут использоваться для автоматического обновления данных в логах или других таблицах.
> Триггер для записи действий при создании нового поста
> Когда пользователь создает новый пост, записываем это действие в таблицу dct_log.
```
CREATE TRIGGER after_post_insert
AFTER INSERT ON inpost
FOR EACH ROW
BEGIN
    INSERT INTO dct_log (log_dt, log_status, log_info)
    VALUES (NOW(), 'INFO', CONCAT('User ', NEW.user_id, ' created a new post with ID ', NEW.inpost_id));
END;
```
>
```
SELECT * FROM dct_log;
CALL create_user_inpost('TEST2','Со мною вот , что происходит, ко мне на сайт никто не ходят, а ходят в праздной тишине всё баги разные не те',2);
```
