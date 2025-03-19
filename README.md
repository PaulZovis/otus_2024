# otus_2024
Установил Docker и Docker Compose.
Создайте конфигурацию Patroni. Пример docker-compose.yml для Patroni:
```
version: '3.8'

services:
  patroni1:
    image: registry.opensource.zalan.do/acid/spilo-14:2.1-p7
    container_name: patroni1
    environment:
      PATRONI_NAME: patroni1
      PATRONI_POSTGRESQL_DATA_DIR: /var/lib/postgresql/data/pgdata
      PATRONI_POSTGRESQL_PGPASS: /tmp/pgpass
      PATRONI_REPLICATION_USERNAME: replicator
      PATRONI_REPLICATION_PASSWORD: replicator_password
      PATRONI_SUPERUSER_USERNAME: postgres
      PATRONI_SUPERUSER_PASSWORD: postgres_password
      PATRONI_POSTGRESQL_LISTEN: 0.0.0.0:5432
      PATRONI_DCS_ETCD_HOST: etcd:2379
    ports:
      - "5432:5432"
    volumes:
      - ./patroni1/data:/var/lib/postgresql/data/pgdata
      - ./patroni1/patroni.yml:/etc/patroni.yml
    networks:
      - patroni_network

  patroni2:
    image: registry.opensource.zalan.do/acid/spilo-14:2.1-p7
    container_name: patroni2
    environment:
      PATRONI_NAME: patroni2
      PATRONI_POSTGRESQL_DATA_DIR: /var/lib/postgresql/data/pgdata
      PATRONI_POSTGRESQL_PGPASS: /tmp/pgpass
      PATRONI_REPLICATION_USERNAME: replicator
      PATRONI_REPLICATION_PASSWORD: replicator_password
      PATRONI_SUPERUSER_USERNAME: postgres
      PATRONI_SUPERUSER_PASSWORD: postgres_password
      PATRONI_POSTGRESQL_LISTEN: 0.0.0.0:5432
      PATRONI_DCS_ETCD_HOST: etcd:2379
    ports:
      - "5433:5432"
    volumes:
      - ./patroni2/data:/var/lib/postgresql/data/pgdata
      - ./patroni2/patroni.yml:/etc/patroni.yml
    networks:
      - patroni_network

  etcd:
    image: bitnami/etcd:latest
    container_name: etcd
    environment:
      ALLOW_NONE_AUTHENTICATION: "yes"
    ports:
      - "2379:2379"
    networks:
      - patroni_network

networks:
  patroni_network:
    driver: bridge
```
Запустил контейнеры:
```
docker-compose up -d
```
собралось
```
docker exec -it patroni2 curl http://patroni1:8008/patroni
docker exec -it patroni1 psql -U postgres
docker-compose logs patroni1
```
Тестирование репликации
a) Создайте тестовую таблицу на patroni1
Подключитесь к patroni1:
```
docker exec -it patroni1 psql -U postgres
CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO test (name) VALUES ('Test');
```
Проверьте данные на patroni2
Подключитесь к patroni2:
```
docker exec -it patroni2 psql -U postgres
SELECT * FROM test;
```
