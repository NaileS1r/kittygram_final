Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

Описание проекта

Пользователи могут регистрироваться, загружать фотографии с описанием и смотреть питомцев других пользователей.

Установка

Склонируйте репозиторий на свой компьютер:
git clone git@github.com:NaileS1r/kittygram_final.git
Создайте файл .env и заполните его своими данными. Нужные переменные указаны в файле .env.example.

Создание Docker-образов

Замените DOCKERHUB_USERNAME на свой логин на DockerHub:

cd frontend
docker build -t DOCKERHUB_USERNAME/kittygram_frontend .
cd ../backend
docker build -t DOCKERHUB_USERNAME/kittygram_backend .
cd ../nginx
docker build -t DOCKERHUB_USERNAME/kittygram_gateway . 

Загрузите образы на DockerHub:

docker push DOCKERHUB_USERNAME/kittygram_frontend
docker push DOCKERHUB_USERNAME/kittygram_backend
docker push DOCKERHUB_USERNAME/kittygram_gateway

Деплой на сервере

Подключитесь к удаленному серверу

ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS 

Создайте на сервере директорию kittygram:

mkdir kittygram

Установите Docker Compose на сервер:

sudo apt update
sudo apt install curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose

Скопируйте файлы docker-compose.production.yml и .env в директорию kittygram/ на сервере:

scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml
Где:

PATH_TO_SSH_KEY - путь к файлу с вашим SSH-ключом
SSH_KEY_NAME - имя файла с вашим SSH-ключом
YOUR_USERNAME - ваше имя пользователя на сервере
SERVER_IP_ADDRESS - IP-адрес вашего сервера

Запустите Docker Compose в режиме демона:

sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml up -d

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py migrate
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker-compose -f /home/YOUR_USERNAME/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/

Откройте конфигурационный файл Nginx в редакторе nano:

sudo nano /etc/nginx/sites-enabled/default

Измените настройки location в секции server:

location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}
Проверьте правильность конфигурации Nginx:

sudo nginx -t

Перезапустите Nginx:

sudo service nginx reload

Настройка CI/CD

Файл workflow уже написан и находится в директории:

kittygram/.github/workflows/main.yml

Добавьте секреты в GitHub Actions:

DOCKER_USERNAME                # имя пользователя в DockerHub
DOCKER_PASSWORD                # пароль пользователя в DockerHub
HOST                           # IP-адрес сервера
USER                           # имя пользователя
SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
SSH_PASSPHRASE                 # пароль для SSH-ключа

TELEGRAM_TO                    # ID вашего телеграм-аккаунта
TELEGRAM_TOKEN                 # токен вашего бота

Технологии

Python 3.11.1, Django 3.2.3, djangorestframework==3.12.4, PostgreSQL 13.10

Автор

Naile Sirazitdinov - NaileS1r