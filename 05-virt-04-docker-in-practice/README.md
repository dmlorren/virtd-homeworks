# Домашнее задание к занятию 5. «Практическое применение Docker» - Иванов Дмитрий (fops-13)

---
## Задача 0
1. Убедитесь что у вас НЕ(!) установлен ```docker-compose```, для этого получите следующую ошибку от команды ```docker-compose --version```
2. Убедитесь что у вас УСТАНОВЛЕН ```docker compose```(без тире) версии не менее v2.24.X, для это выполните команду ```docker compose version``` 


## Решение задачи 0
<img src="img/docker_version.png">
---


## Задача 1
1. Сделайте в своем github пространстве fork репозитория ```https://github.com/netology-code/shvirtd-example-python/blob/main/README.md```.   
2. Создайте файл с именем ```Dockerfile.python``` для сборки данного проекта(для 3 задания изучите https://docs.docker.com/compose/compose-file/build/ ). Используйте базовый образ ```python:3.9-slim```. Протестируйте корректность сборки. Не забудьте dockerignore. 



## Решение задачи 1

```
root@ubuntu-2004:/home/dmlorren/docker# docker build -t my-app . -f /home/dmlorren/docker/Dockerfile.python 
```
<img src="img/docker_build.png">

<a href="https://github.com/dmlorren/shvirtd-example-python">Ссылка на репозиторий в котором выполнялось ДЗ</a>

---


## Задача 3
1. Изучите файл "proxy.yaml"
2. Создайте в репозитории с проектом файл ```compose.yaml```. С помощью директивы "include" подключите к нему файл "proxy.yaml".
3. Опишите в файле ```compose.yaml``` следующие сервисы: 

- ```web```. Образ приложения должен ИЛИ собираться при запуске compose из файла ```Dockerfile.python``` ИЛИ скачиваться из yandex cloud container registry(из задание №2 со *). Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.5```. Сервис должен всегда перезапускаться в случае ошибок.
Передайте необходимые ENV-переменные для подключения к Mysql базе данных по сетевому имени сервиса ```web``` 

- ```db```. image=mysql:8. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.10```. Явно перезапуск сервиса в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.Обязательно используйте уже существующий .env file для назначения секретных ENV-переменных!

2. Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда ```curl -L http://127.0.0.1:8090``` должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: ```docker ps -a ``` и ```docker logs <container_name>``` 

5. Подключитесь к БД mysql с помощью команды ```docker exec <имя_контейнера> mysql -uroot -p<пароль root-пользователя>``` . Введите последовательно команды (не забываем в конце символ ; ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;```.

6. Остановите проект. В качестве ответа приложите скриншот sql-запроса.


## Решение задачи 3

<img src="img/docker_mysql.png">

---

## Задача 4
1. Запустите в Yandex Cloud ВМ (вам хватит 2 Гб Ram).
2. Подключитесь к Вм по ssh и установите docker.
3. Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4. Зайдите на сайт проверки http подключений, например(или аналогичный): ```https://check-host.net/check-http``` и запустите проверку вашего сервиса ```http://<внешний_IP-адрес_вашей_ВМ>:8090```. Таким образом трафик будет направлен в ingress-proxy.
5. (Необязательная часть) Дополнительно настройте remote ssh context к вашему серверу. Отобразите список контекстов и результат удаленного выполнения ```docker ps -a```
6. В качестве ответа повторите  sql-запрос и приложите скриншот с данного сервера, bash-скрипт и ссылку на fork-репозиторий.


## Решение задачи 4
1. В облаке яндекса развёрнута vm ubunty 20.04
2. Написан bash-скрипт:

dmlorren@ubunty2004:~$ cat download_repo.sh 
```bash
#! /bin/bash

cd /opt
git clone https://github.com/dmlorren/shvirtd-example-python.git
cd shvirtd-example-python/

sudo docker compose -f compose.yaml -f proxy.yaml up -d
```

<img src="img/yandex_docker1.png">
<img src="img/yandex_docker2.png">

```
В данном задании я столкнулся с проблемой перезапуска контейнеров web и db в виртуальной машине yandex cloud.
На stackoverflow и других ресурсах обозначили, что проблема может заключаться в совместимости ubunty 20.04 и mysql:8 
Попробовал запустить с mariaDB - всё запустилось.
```
<img src="img/docker_ps.png">
<img src="img/check_host.png">
<img src="img/mariadb.png">

---

## Задача 6
Скачайте docker образ ```hashicorp/terraform:latest``` и скопируйте бинарный файл ```/bin/terraform``` на свою локальную машину, используя dive и docker save.
Предоставьте скриншоты  действий .


## Решение задачи 6

1. Ставим dive по инструкции из: https://github.com/wagoodman/dive
```
sudo snap install docker
sudo snap install dive
sudo snap connect dive:docker-executables docker:docker-executables
sudo snap connect dive:docker-daemon docker:docker-daemon
```

2. Посмотрим слои образа через dive:
```
root@ubuntu-2004:/home/dmlorren/docker# dive hashicorp/terraform:latest
```
<img src="img/dive.png">


3. Сохраняем докер образ на локальную машину с помощью docker save:
```
root@ubuntu-2004:/home/dmlorren/docker# docker save hashicorp/terraform:latest -o /opt/myapp.tar
```

4. Через mc захожу в архив, выбираю нужный слой и копирую к себе на локальную машину в директорию home.
<img src="img/bin.png">

---

## Задача 6.1
Добейтесь аналогичного результата, используя docker cp.  
Предоставьте скриншоты  действий .

## Решение задачи 6.1

1. Здесь как минимум нужно собрать контейнер на основе образа (не обязательно запускать) для проведение каких-либо операций с ним:

Выполняем:
```
docker create --name my_container_dmlorren hashicorp/terraform:latest
docker cp 540cb6a28e112f1de87325310c6e82d15d43e7eb9dee7621fa9cadf9ec93b3aa:/bin/terraform /opt/
```

<img src="img/docker_cp.png">

---

### README.md по умолчанию

# shvirtd-example-python

Example Flask-application for docker compose training.
## Installation
First, you need to clone this repository:

```bash
git clone https://github.com/netology-code/shvirtd-example-python.git
```

Now, we will need to create a virtual environment and install all the dependencies:

```bash
python3 -m venv venv  # on Windows, use "python -m venv venv" instead
. venv/bin/activate   # on Windows, use "venv\Scripts\activate" instead
pip install -r requirements.txt
python main.py
```
You need to run Mysql database and provide following ENV-variables for connection:  
- DB_HOST (default: '127.0.0.1')
- DB_USER (default: 'app')
- DB_PASSWORD (default: 'very_strong')
- DB_NAME (default: 'example')

The applications will always running on http://localhost:5000.  
To exit venv just type ```deactivate```

## License

This project is licensed under the MIT License (see the `LICENSE` file for details).

---