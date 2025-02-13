# Внутренняя архитектура СУБД PostgreSQL 
##### Установка через докер
**(Всегда устанавливал под Windows разные версии ОС и PostgreSQL ранее, в рамках занятия опробовал варианты)
> осуществил на доманей РМ, но предпочёл работу на ВМ
```
sudo apt install docker.io
docker run --name my-postgres -e POSTGRES_PASSWORD='password' -d -p 5432:5432 postgres
```
```
psql -h localhost -U postgres
```

##### Установил на ВМ на домашнем ПК
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```
> переключился на пользователя postgres
```
cd */*/PostgreSQL/16/bin
pg_ctl start 
pg_ctl restart
pg_ctl stop
pg_ctl start
```
```
psql -U postgres
create database otus_project;
\c otus_project;
```
##### Подключимся к БД в командной строке ОС Windows, рестартанём, остановим, запустим сервер БД
![image](https://github.com/user-attachments/assets/a050d8ee-bf38-4468-afb1-fc8152a3d10f)
> Посмотрим , есть ли  БД, создадим БД, посмотрим
![image](https://github.com/user-attachments/assets/374eb6b8-b57a-4277-af97-3bf0ac39d1c5)
##### Создание пользователя из-под командной строки с ограничением действия по времени
![Screenshot 2025-02-13 112051](https://github.com/user-attachments/assets/d69d3165-3337-45f0-b59f-8a612a93ad05)
> войдём под user_otus и создfдим таблицу ttt с полями id и ttext
![Screenshot 2025-02-13 112334](https://github.com/user-attachments/assets/8da51624-61fe-48f6-be8c-4933137d1702)
> Посмотрим под каким пользователем работаем
![Screenshot 2025-02-13 112829](https://github.com/user-attachments/assets/ab3edc0a-b033-4489-9a4a-935f692ebf1a)

##### Подключение через DBeaver (видим бд , созданную таблицу)
![Screenshot 2025-02-13 113336](https://github.com/user-attachments/assets/9db8477c-b643-491f-bee4-c35de98c5475)







