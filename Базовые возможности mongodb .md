# Базовые возможности mongodb 
> Установил сначала в docker, пройдя путь:
> docker-compose.yml
```
version: '3.8'

services:
  mongo:
    image: mongo:latest  # Используйте последнюю версию MongoDB
    container_name: mongodb  # Имя контейнера
    ports:
      - "27017:27017"  # Проброс порта MongoDB на хост
    volumes:
      - mongodb_data:/data/db  # Сохранение данных MongoDB в volume
    environment:
      MONGO_INITDB_ROOT_USERNAME: root  # Имя пользователя для root
      MONGO_INITDB_ROOT_PASSWORD: 030201  # Пароль для root

volumes:
  mongodb_data:  # Создание volume для хранения данных MongoDB
```
> запустив
```docker-compose up -d```
> подключился
```docker exec -it mongodb mongosh -u root -p example```
> Прошло время и решил снова запустить и куча ошибок, ???? ,время потратил, но не решил ,
> задание надо уже сдавать и пошёл другим путём
>
> Всё почистил
> затем
> Скачивание образа
> Команда docker вызывало утилиту docker, запустило ее подкоманду для запуска контейнера, опция –name помогает обозначить контейнер так, как нам нужно, а опция -d указывает на образ из репозитория. Проверил список запущенных командой контейнеров:
```
docker run --name my-mongo -d mongo
docker ps
docker exec -it my-mongo bash
mongosh
```
![m1](https://github.com/user-attachments/assets/b5a9e670-8bac-47bb-971e-928921dada19)
![m2](https://github.com/user-attachments/assets/fc1d7af4-49fc-455f-b4ee-24dd14f6acbd)
#### Заполнение базы данных тестовыми данными
> Создал базу данных и коллекцию:
```
use testdb
db.createCollection("users")
```
> Добавил тестовые данные:
```
db.users.insertMany([
  { name: "Alice", age: 25, city: "New York" },
  { name: "Bob", age: 30, city: "San Francisco" },
  { name: "Charlie", age: 35, city: "Los Angeles" },
  { name: "David", age: 40, city: "Chicago" }
])
```
![m3](https://github.com/user-attachments/assets/48069ab0-2db3-4f27-aa76-86d256d55aad)
#### Запросы на выборку данных
> Выбрал пользователей
```
db.users.find()
```
> Выбрал пользователей старше 30 лет:
```
db.users.find({ age: { $gt: 30 } })
```
> Выбрал пользователей из определенного города:
```
db.users.find({ city: "New York" })
```
> Выбрал имена и возраст пользователей:
```
db.users.find({}, { name: 1, age: 1, _id: 0 })
```
#### Запросы на обновление данных
> Обновил возраст пользователя:
```
db.users.updateOne({ name: "Alice" }, { $set: { age: 26 } })
```
> Обновил город для всех пользователей старше 30 лет:
```
db.users.updateMany({ age: { $gt: 30 } }, { $set: { city: "Miami" } })
```
> Добавил новое поле "status":
```
db.users.updateMany({}, { $set: { status: "active" } })
```
> Удалил поле "status" у всех пользователей:
```
db.users.updateMany({}, { $unset: { status: "" } })
```
![m4](https://github.com/user-attachments/assets/e4e90e24-a29d-4ffb-bc59-36a2adcd6f79)
![m5](https://github.com/user-attachments/assets/6767131d-ec8c-41a8-8bca-b8c569a75829)
