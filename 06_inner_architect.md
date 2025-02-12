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
![](![Uploading image.png…])
