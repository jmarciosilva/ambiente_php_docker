# Ambiente Docker PHP, Apache, MySql e PHP MyAdmin
### Como Gerar ambiente de estudos ou de desenvolvimento com PHP, MySql, Apache e PHP MyAdmin

Criar uma pasta com direitos administrativos (root ou similar)
- Dentro desta pasta criar o arquivo de configuração Dockerfile para configurar o PHP
### Arquido Dockerfile
```
  # Use a imagem base do PHP com Apache
  FROM php:8.2-apache
  
  # Instala dependências necessárias
  RUN apt-get update && apt-get install -y \
      libpng-dev \
      libjpeg-dev \
      libfreetype6-dev \
      zip \
      unzip \
      && docker-php-ext-configure gd --with-freetype --with-jpeg \
      && docker-php-ext-install gd \
      && docker-php-ext-install pdo pdo_mysql \
      && apt-get install -y curl git \
      && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
  
  # Ajuste as permissões da pasta html
  RUN chown -R www-data:www-data /var/www/html \
      && chmod -R 777 /var/www/html
  
  
  # Exponha a porta 8080
  EXPOSE 8080 # a porta do apache foi configurada para http://localhost:8080
  
  # Inicie o servidor Apache
  CMD ["apache2-foreground"]
```
### Criar o arquivo docker-compose.yml
```
version: '3.9'

services:
  # Serviço PHP
  php:
    build:
      context: .  # Diretório onde está o Dockerfile
      dockerfile: Dockerfile  # Nome do Dockerfile
    container_name: php-container_8_2 # nesta linha você define o nome do container PHP
    ports:
      - "8080:80" # a porta do apache foi configurada para http://localhost:8080
    volumes:
      - ./www:/var/www/html
    depends_on:
      - db
    environment:
      COMPOSER_ALLOW_SUPERUSER: 1

  # Serviço MySQL
  db:
    image: mysql:8.0
    container_name: mysql_8_0-container # nesta linha você define o nome do container MySQL
    environment:
      MYSQL_ROOT_PASSWORD: 12345678 # defina sua senha
      MYSQL_DATABASE: my_database # defina seu nome de banco de dados
      MYSQL_USER: user # usuario criado la no arquivo docker-compose.yml
      MYSQL_PASSWORD: 12345678 # defina a senha
    ports:
      - "3406:3306" # a porta do mySql no lado da máquina host (sua máquina ficou com 3406 - esta configuração foi feita pois já tenho mySql instalado na minha máquina
    volumes:
      - db_data:/var/lib/mysql

  # Serviço phpMyAdmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin-container_8_2 # nesta linha você define o nome do container phpMyAdmin
    depends_on:
      - db
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: 12345678 # sua senha definida
    ports:
      - "8081:80" # a porta do phpMyAdmin foi configurada para http://localhost:8081

# Definição dos volumes
volumes:
  db_data:
```




