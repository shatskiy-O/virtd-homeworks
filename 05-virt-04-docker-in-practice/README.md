# Домашнее задание к занятию 5. «Практическое применение Docker»

### Инструкция к выполнению

1. Для выполнения заданий обязательно ознакомьтесь с [инструкцией](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD) по экономии облачных ресурсов. Это нужно, чтобы не расходовать средства, полученные в результате использования промокода.
3. Своё решение к задачам оформите в вашем GitHub репозитории.
4. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

---

## Задача 1
1. Сделайте fork репозитория ```https://github.com/netology-code/shvirtd-example-python/blob/main/README.md```.   
2. **Необязательно.** Изучите инструкцию проекта и запустите приложение без использования docker в venv. (Mysql БД можно запустить в docker run).   
3. Создайте файл ```Dockerfile.python``` для сборки данного проекта. Используйте базовый образ ```python:3.9-slim``` Протестируйте корректность сборки. Не забудьте dockerignore.    

## Ответ 1

### Dockerfile.python
```
# Используем базовый образ python:3.9-slim
FROM python:3.9-slim

# Установим рабочую директорию в контейнере
WORKDIR /app

# Копируем файлы проекта в рабочую директорию
COPY . /app

# Установим зависимости из файла requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Определяем переменные среды для подключения к базе данных
ENV DB_HOST=new-mysql-db \
    DB_USER=app \
    DB_PASSWORD=very_strong \
    DB_NAME=example

# Открываем порт 5000
EXPOSE 5000

# Запускаем приложение
CMD ["python", "main.py"]
```

### dockerignore

```
venv
*.pyc
*.pyo
*.pyd
__pycache__
```
![alt text](https://github.com/shatskiy-O/virtd-homeworks/blob/shvirtd-1/05-virt-04-docker-in-practice/img/1.png)

## Задача 2
1. Создайте файл ```compose.yaml```. Опишите в нем следующие сервисы: 

- ```web```. Образ должен собираться при запуске compose из файла ```Dockerfile.python```. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.5```. Сервис должен всегда перезапускаться в случае ошибок.
Передайте необходимые ENV-переменные для подключения к Mysql базе данных по имени сервиса ```web```

- ```db```. image=mysql:8. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.10```. Сервис должен всегда перезапускаться в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.

2. Одной командой запустите c помощью docker compose ```compose.yaml``` и ```proxy.yaml```, добейтесь его стабильной работы.

3. Протестируйте локально приложение с помощью команд ```curl -L http://127.0.0.1:8080``` и ```curl -L http://127.0.0.1:8090```.

4. Подключитесь к БД mysql с помощью команды ```docker exec <имя_контейнера> mysql -uroot -p<пароль root-пользователя>``` . Введите последовательно команды (не забываем ```;``` ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; select * from requests;```.

5. Остановите проект и закомитьте все изменения в свой fork-репозиторий. В качестве ответа приложите скриншот таблицы ```requests```.

## Ответ 2

### compose.yaml

```
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.python
    networks:
      backend:
        ipv4_address: 172.20.0.5
    restart: always
    environment:
      DB_HOST: db
      DB_USER: app
      DB_PASSWORD: very_strong
      DB_NAME: example

  db:
    image: mysql:8
    networks:
      backend:
        ipv4_address: 172.20.0.10
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: very_strong
      MYSQL_DATABASE: example
      MYSQL_USER: app
      MYSQL_PASSWORD: very_strong

networks:
  backend:
    driver: bridge
    ipam:
     config:
       - subnet: 172.20.0.0/16
```
![alt text](https://github.com/shatskiy-O/virtd-homeworks/blob/shvirtd-1/05-virt-04-docker-in-practice/img/2.png)

![alt text](https://github.com/shatskiy-O/virtd-homeworks/blob/shvirtd-1/05-virt-04-docker-in-practice/img/3.png)


## Задача 3
1. Запустите в Yandex Cloud ВМ согласно инструкциям (вам хватит 2 Гб Ram).
2. Установите стек docker приложений.
3. Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4. Зайдите на сайт проверки http подключений, например(или аналогичный): ```https://check-host.net/check-http``` и проверьте ваш адрес ``http://<внешний_IP-адрес_вашей_ВМ>:5000```.
5. В качестве ответа приложите скриншот таблицы ```requests``` с данного сервера и bash-скрипт.

## Ответ 3

### bash-скрипт

```
#!/bin/bash

# Устанавливаем Docker, если он еще не установлен
if ! command -v docker &> /dev/null
then
    echo "Установка Docker..."
    sudo apt-get update
    sudo apt-get install -y docker.io
fi

# Устанавливаем Docker Compose, если он еще не установлен
if ! command -v docker-compose &> /dev/null
then
    echo "Установка Docker Compose..."
    sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
fi

# Клонируем репозиторий, если он еще не клонирован
if [ ! -d "/opt/shvirtd-example-python" ] ; then
    echo "Клонирование репозитория..."
    sudo git clone https://github.com/shatskiy-O/shvirtd-example-python /opt/shvirtd-example-python
else
    echo "Репозиторий уже клонирован."
    cd /opt/shvirtd-example-python
    sudo git pull
fi

# Переходим в директорию проекта
cd /opt/shvirtd-example-python

# Добавляем проброс порта для веб-сервиса в compose.yaml
sudo sed -i '/web:/a \    ports:\n      - "5000:5000"' compose.yaml

# Запускаем проект
echo "Запуск проекта..."
sudo docker-compose -f compose.yaml -f proxy.yaml up -d
```

![alt text](https://github.com/shatskiy-O/virtd-homeworks/blob/shvirtd-1/05-virt-04-docker-in-practice/img/4.png)


### Правила приема

Домашнее задание выполните в файле readme.md в GitHub-репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.
