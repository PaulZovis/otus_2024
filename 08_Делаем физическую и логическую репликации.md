# Репликация в PostgreSQL

### Создание реплаики с задержкой в 5 минут
> Создал YML
```
version: '3.8'

services:
  postgres-master:
    image: postgres:14
    container_name: postgres-master
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
      POSTGRES_DB: master_db
    ports:
      - "5432:5432"
    volumes:
      - postgres-master-data:/var/lib/postgresql/data
    networks:
      - postgres-network
    command: >
      -c wal_level=replica
      -c max_wal_senders=10
      -c hot_standby=on

  postgres-replica:
    image: postgres:14
    container_name: postgres-replica
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
      POSTGRES_DB: replica_db
    ports:
      - "5433:5432"
    volumes:
      - postgres-replica-data:/var/lib/postgresql/data
    networks:
      - postgres-network
    depends_on:
      - postgres-master
    command: >
      -c primary_conninfo='host=postgres-master port=5432 user=postgres password=password'
      -c primary_slot_name='replica_slot'
      -c recovery_min_apply_delay='5min'
      -c hot_standby=on

  postgres-logical:
    image: postgres:14
    container_name: postgres-logical
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
      POSTGRES_DB: logical_db
    ports:
      - "5434:5432"
    volumes:
      - postgres-logical-data:/var/lib/postgresql/data
    networks:
      - postgres-network
    depends_on:
      - postgres-master

volumes:
  postgres-master-data:
  postgres-replica-data:
  postgres-logical-data:

networks:
  postgres-network:
```
> Запустил
```
docker-compose up -d
```
> Подключился к контейнеру мастера
```
docker exec -it postgres-master psql -U postgres
```
> Создал слот репликации
```
SELECT * FROM pg_create_physical_replication_slot('replica_slot');
```
> На мастере создал таблицу и добавил данные
```
CREATE TABLE test_table (id SERIAL PRIMARY KEY, data TEXT);
INSERT INTO test_table (data) VALUES ('test data');
```
> Подключился к реплике и проверил, что данные появились с задержкой в 5 минут. Не появились
```
docker exec -it postgres-replica psql -U postgres -d replica_db -c "SELECT * FROM test_table;"
```
**Прбовал различные варианты реализации и на разных ПК, где-то косячу. Надо изучать детальнее**

### Настройка логической репликации
> Подключился к мастеру
```
docker exec -it postgres-master psql -U postgres
```
> Создал базу данных и таблицу
```
CREATE DATABASE logical_db;
\c logical_db
CREATE TABLE logical_table (id SERIAL PRIMARY KEY, data TEXT);
INSERT INTO logical_table (data) VALUES ('initial data');
```
> Создал публикацию
```
CREATE PUBLICATION logical_pub FOR TABLE logical_table;
```
> Подключился к логической реплике:
```
docker exec -it postgres-logical psql -U postgres
```
> Создайте базу данных:
```
CREATE DATABASE logical_db;
\c logical_db
```
> Создал таблицу с такой же структурой, как на мастере:
```
CREATE TABLE logical_table (id SERIAL PRIMARY KEY, data TEXT);
```
> Создайте подписку:
```
CREATE SUBSCRIPTION logical_sub
CONNECTION 'host=postgres-master port=5432 dbname=logical_db user=postgres password=password'
PUBLICATION logical_pub;
```
**Тоже неудачка**
> Скрины работы
![rep1](https://github.com/user-attachments/assets/6e302328-ec2f-40e7-875c-1cd5b50d2cfa)
![rep2](https://github.com/user-attachments/assets/da613044-d645-4c1c-be64-ef7246c5ce48)
![rep3](https://github.com/user-attachments/assets/a8eb4d11-2274-4345-86d6-38659ab4facc)
![rep4](https://github.com/user-attachments/assets/8765086b-db83-432b-82fd-ebb104d9d4b5)
![rep5](https://github.com/user-attachments/assets/b0470b22-dfc3-4041-90a6-899e4c24803f)
