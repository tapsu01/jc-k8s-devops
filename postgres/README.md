# Intall Postgres with docker-compose

File: `docker-compose.yaml`

```yaml
version: '2'

services:
  postgres:
    container_name: postgres
    image: postgres:latest
    ports:
      # host port:container port
      - '5432:5432'
    restart: always
    volumes:
      #  You need to create /mnt/data/postgres folder manually or this will error
      - /mnt/data/postgres:/var/lib/postgresql/data
      # Need to mound passwd to use user option
      # - /etc/passwd:/etc/passwd:ro
    environment:
      # set password for superuser postgres
      POSTGRES_PASSWORD: example
```

Create and run container:

```none
# create and run container in the background
docker-compose up -d
```

[documents](https://github.com/docker-library/docs/blob/master/postgres/README.md)
