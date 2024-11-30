1. create file docker-compose.yml

```yaml
version: "3.1"
services:
    oracle-db:
        image: epiclabs/docker-oracle-xe-11g
        container_name: "oracle-db"
        environment:
            - ORACLE_CHARACTERSET=AL32UTF8
            - ORACLE_PASSWORD=secret
        ports:
            - 1521:1521
        volumes:
            - ./data:/opt/oracle/oradata
```

2. `mkdir data`

3. `chown -R 54321:54321 data`

4. `docker compose up`

reference:
https://github.com/oracle/docker-images/issues/1190#issuecomment-480576521

database client oracle :

- https://dbeaver.io/

referensi lain:

- https://hpeixoto.me/2020-10-23-oracle-enterprise-docker/

- https://dev.to/docker/how-to-run-oracle-database-in-a-docker-container-using-docker-compose-1c9b
