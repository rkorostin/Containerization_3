# Урок 3. Введение в Docker
## 1. Запустить контейнер с БД, отличной от mariaDB, используя инструкции на сайте: https://hub.docker.com/

**Сначала устанавливаем docker согласно инструкции с сайта**
1. Обновите apt индекс пакетов и установите пакеты, чтобы разрешить aptиспользование репозитория через HTTPS:
``````
 sudo apt-get update
 sudo apt-get install ca-certificates curl gnupg
```````
2. Добавьте официальный GPG-ключ Docker:
```
 sudo install -m 0755 -d /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 sudo chmod a+r /etc/apt/keyrings/docker.gpg
``````
3. Используйте следующую команду для настройки репозитория:
``````
 echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
``````
4. Обновите apt индекс пакета:
``````
 sudo apt-get update
``````
5. Установите Docker Engine, containerd и Docker Compose.
``````
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
``````
**Загружаем образ БД MySQL и образ phpMyAdmin:**
``````
docker pull mysql
docker pull phpmyadmin/phpmyadmin
``````
**Запускаем контейнер с БД:**
``````
docker run -d --name my_mysql -e MYSQL_ROOT_PASSWORD=mysecretpassword mysql
``````

**Убеждаемся что контейнер в апе**
```
docker ps -a
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                          NAMES
bddfde21fb19   mysql                   "docker-entrypoint.s…"   22 seconds ago   Up 21 seconds   3306/tcp, 33060/tcp            my_mysql
```
## 3. Заполнить БД данными через консоль
**Используем команду "mysql" внутри контейнера MySQL:**
```
docker exec -it my_mysql mysql -u root -p
```
**Вводим пароль**

**После ввода пароля можно выполнить SQL-запросы для создания схемы, таблиц и заполнения их данными:**
```
mysql> create schema gb_hw3;
Query OK, 1 row affected (0.01 sec)

mysql>
mysql>
mysql> use gb_hw3;
Database changed
mysql>
mysql>
mysql> CREATE TABLE users (id INT, name VARCHAR(255), email VARCHAR(255));
Query OK, 0 rows affected (0.03 sec)
mysql>
mysql>
mysql> INSERT INTO users (id, name, email) VALUES (1, 'Roman', 'roman@example.com');
Query OK, 1 row affected (0.00 sec)
```
## 4. Запустить phpmyadmin (в контейнере) и через веб проверить, что все введенные данные доступны
У меня поднята виртуалка на ноуте в режиме моста. VM с адресом 192.168.1.11

**Запускаем контейнер phpMyAdmin, связанный с контейнером MySQL:**
```
docker run -d --name my_phpmyadmin -p 8080:80 --link my_mysql:db phpmyadmin/phpmyadmin
```
**Убеждаемся что оба контейнера в апе**
```
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                   NAMES
047d63a7befa   phpmyadmin/phpmyadmin   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   my_phpmyadmin
0eda465e436c   mysql                   "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   3306/tcp, 33060/tcp                     my_mysql
```
**На ноуте запускаем браузер и переходим по адресу http://192.168.1.11:8080/, чтобы открыть phpMyAdmin**
![phpMyAdmin](https://github.com/rkorostin/Images/blob/main/container/hw3/phpMyAdmin.png)

**Входим в phpMyAdmin, используя учетные данные root/mysecretpassword, которые были заданы при запуске контейнера MySQL и смотрим созданную БД gb_hw3 с таблицей users, в которой есть один пользователь - Roman:**
![view_bd](https://github.com/rkorostin/Images/blob/main/container/hw3/user.PNG)

## 2. Добавить в контейнер hostname такой же, как hostname системы через переменную
**Выполняем команду, после которой контейнер с MySQL будет запущен и переменная MYSQL_HOSTNAME будет установлена в значение hostname системы, на которой запущен контейнер:**
```
docker run -d --name my_mysql2 -e MYSQL_ROOT_PASSWORD=123 -e MYSQL_HOSTNAME=$(hostname) mysql
```
**Заходим в bash:**
```
docker exec -it my_mysql2 bash
```
**В открывшемся терминале вводим команду:**
``````
echo $MYSQL_HOSTNAME
``````
![hostname](https://github.com/rkorostin/Images/blob/main/container/hw3/hostname.png)