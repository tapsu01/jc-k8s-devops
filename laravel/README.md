# Deploy Laravel with Apache to Kubernetes

> [Refenrences](https://learnk8s.io/blog/kubernetes-deploy-laravel-the-easy-way)

1. Create `Dockerfile` and `vhost.conf` in root project

`Dockerfile`

```Dockerfile
FROM composer:1.6.5 as build
WORKDIR /app
COPY . /app
RUN composer install

FROM php:7.1.8-apache
EXPOSE 80
COPY --from=build /app /app
COPY vhost.conf /etc/apache2/sites-available/000-default.conf
RUN chown -R www-data:www-data /app a2enmod rewrite
```

`vhost.conf`

```config
<VirtualHost *:80>
  DocumentRoot /app/public

  <Directory "/app/public">
    AllowOverride all
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

2. Build docker image

```sh
docker build -t laravel-app .
```

- `-t` `laravel-app` defines the name ("tag") of your container — in this case, your container is just called `laravel-app`
- `.` is the location of the Dockerfile and application code — in this case, it's the current directory
