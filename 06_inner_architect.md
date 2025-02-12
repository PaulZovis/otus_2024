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
##### Демонстрирую с РМ из под Windows на этой же ОС
![image]((https://github.com/user-attachments/assets/a050d8ee-bf38-4468-afb1-fc8152a3d10f))
![image](https://github.com/user-attachments/assets/374eb6b8-b57a-4277-af97-3bf0ac39d1c5)
![image](https://github.com/user-attachments/assets/5a944175-6844-45d3-b4f6-b9abbb90d4c4)



