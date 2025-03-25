# DML-агрегация-и-сортировка-в-MySQL
> группировки с ипользованием CASE, HAVING, ROLLUP, GROUPING()
> сделать rollup с количеством товаров по категориям
```
SELECT count(p.post_text) cnt, ifnull(u.user_login, 'не определено') 
  FROM post p
  LEFT JOIN inpost i USING(post_id)
  LEFT JOIN users u USING(user_id)
 GROUP BY 2
 order BY 1   

 SELECT count(p.post_text) cnt, ifnull(u.user_login, 'не определено') 
  FROM post p
  LEFT JOIN inpost i USING(post_id)
  LEFT JOIN users u USING(user_id)
 GROUP BY 2
 HAVING count(p.post_text) > 1
 order BY 1 
 
 SELECT (CASE 
           WHEN i.source_id = 1 THEN 'источник 1'
           WHEN i.source_id = 2 then 'источник 2'
           ELSE 'пока не понятно'
         END) source_name,
        p.post_text
   FROM inpost i
   INNER JOIN post p using(post_id);
   
 SELECT sum(p.post_len) len_text,
        u.user_login,
        i.source_id 
   FROM inpost i
   LEFT JOIN post p using(post_id)  
   LEFT JOIN users u using(user_id)
   GROUP BY 2,3
   WITH ROLLUP ;
  
 SELECT sum(p.post_len) len_text,
        u.user_login,
        i.source_id,
        grouping(u.user_login),
        grouping(i.source_id)
   FROM inpost i
   LEFT JOIN post p using(post_id)  
   LEFT JOIN users u using(user_id)
   GROUP BY 2,3
  WITH ROLLUP ;
```
> Скрины
![a1](https://github.com/user-attachments/assets/b72ff263-bc41-40c3-a05b-8df84263e9c4)
![a2](https://github.com/user-attachments/assets/3fb8d2bd-c707-41d6-93fb-57f3477eda8c)
![a3](https://github.com/user-attachments/assets/e40277fe-277b-4de2-9d26-900455ac240b)
> для магазина к предыдущему списку продуктов добавить максимальную и минимальную цену и кол-во предложений
> сделать выборку показывающую самый дорогой и самый дешевый товар в каждой категории
```
 SELECT max(p.post_len) max_post,
         min(p.post_len) min_post , 
         sum(p.post_len) sum_len, 
         AVG(p.post_len) avg_post,
         u.user_login
   FROM inpost i
   LEFT JOIN post p using(post_id)  
   LEFT JOIN users u using(user_id)
  GROUP BY u.user_login
```
![a4](https://github.com/user-attachments/assets/e10f7ad7-dfb3-4a93-8545-a1c6001fbb25)
