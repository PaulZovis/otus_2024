# DML: агрегация и сортировка, CTE, аналитические функции в PostgreSQL 
> Исходные данные
```
CREATE TABLE statistic(
player_name VARCHAR(100) NOT NULL,
    player_id INT NOT NULL,
    year_game SMALLINT NOT NULL CHECK (year_game > 0),
    points DECIMAL(12,2) CHECK (points >= 0),
    PRIMARY KEY (player_name,year_game));
INSERT INTO
  statistic(player_name, player_id, year_game, points)
VALUES
    ('Mike',1,2018,18),
    ('Jack',2,2018,14),
    ('Jackie',3,2018,30),
    ('Jet',4,2018,30),
    ('Luke',1,2019,16),
    ('Mike',2,2019,14),
    ('Jack',3,2019,15),
    ('Jackie',4,2019,28),
    ('Jet',5,2019,25),
    ('Luke',1,2020,19),
    ('Mike',2,2020,17),
    ('Jack',3,2020,18),
    ('Jackie',4,2020,29),
    ('Jet',5,2020,27);
```
> Запрос суммы очков с группировкой и сортировкой по годам
```
SELECT year_game , 
       sum(points) AS sumpo
  FROM STATISTIC
 GROUP BY 1
 ORDER BY 1;
```
![Результат](https://github.com/user-attachments/assets/5da4ceef-b6f7-44fd-84b8-151b7d64c808)
> cte показывающее тоже самое
```
WITH cte AS   
(SELECT year_game , 
        sum(points) AS sumpo
  FROM STATISTIC
  GROUP BY 1)
SELECT year_game as "год", sumpo as "сумма очков" 
  FROM cte
 ORDER BY 1;
```
![Результат](https://github.com/user-attachments/assets/83cc15ba-347d-48e8-a7db-958082340c66)
> Функция LAG вывести кол-во очков по всем игрокам за текущий код и за предыдущий.
```
WITH cte AS   
  (SELECT year_game , 
          sum(points) AS sumpo
     FROM STATISTIC
    GROUP BY 1)
   SELECT year_game AS "Год", 
          sumpo AS "очки текущего года", 
          lag(sumpo) OVER(order BY year_game) AS "очки предыдущего года"
     FROM cte
    ORDER BY 1;
```

>
```
WITH cte AS   
  (SELECT year_game , 
          sum(points) AS sumpo
     FROM STATISTIC
    GROUP BY 1)
   SELECT year_game AS "Год",
          lag(sumpo::varchar,'1','нет данных') OVER(order BY year_game) AS "очки предыдущего года",
          sumpo AS "очки текущего года", 
          lead(sumpo::varchar,'1','нет данных') OVER(order BY year_game) AS "очки следующего года"
     FROM cte
    ORDER BY 1;
```
>
```
SELECT player_name , 
       year_game, 
       sum(points) AS sumpo
  FROM STATISTIC
  GROUP BY ROLLUP(1,2)
  ORDER BY 1,2
```
> 
```
  SELECT player_name , 
       year_game, 
       sum(points) AS sumpo
  FROM STATISTIC
  GROUP BY cube(1,2)
  ORDER BY 1,2
```
