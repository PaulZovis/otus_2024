![image](https://github.com/user-attachments/assets/7700af1f-1f9b-4f61-8926-d6b195718373)# Внутренняя архитектура СУБД PostgreSQL 
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
![]![image](https://github.com/user-attachments/assets/05b0bc8e-2ee5-4f44-a879-6f2dad5602a9))
