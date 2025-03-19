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
    ports:
      - "5432:5432"
    volumes:
      - ./patroni1/data:/var/lib/postgresql/data/pgdata
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
    ports:
      - "5433:5432"
    volumes:
      - ./patroni2/data:/var/lib/postgresql/data/pgdata
    networks:
      - patroni_network

networks:
  patroni_network:
    driver: bridge
image: registry.opensource.zalan.do/acid/spilo-14:2.1-p7
```
Запустил контейнеры:
```
docker-compose up -d
```
