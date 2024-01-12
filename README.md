## Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## Установка
1. Клонируйте репозиторий на свой компьютер:

```
  git clone git@github.com:mdflamingo/taski_docker.git
```
2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.

## Создание Docker-образов
1. Замените username на ваш логин на Dockerhub:
```
  cd frontend
  docker build -t username/taski_frontend .
  cd ../backend
  docker build -t username/taski_backend .
  cd ../nginx
  docker build -t username/taski_gateway . 
```
2. Загрузите образы на DockerHub:
```
  docker push username/taski_frontend
  docker push username/taski_backend
  docker push username/taski_gateway
```
## Деплой на сервере
1. Подключитесь к удаленному серверу
```
  ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
```

3. Создайте на сервере директорию taski через терминал
```
  mkdir taski
```
3. Установка docker compose на сервер:
```
  sudo apt update
  sudo apt install curl
  curl -fSL https://get.docker.com -o get-docker.sh
  sudo sh ./get-docker.sh
  sudo apt-get install docker-compose-plugin
```
4. В директорию taski/ скопируйте файлы docker-compose.production.yml и .env:
```
  scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/taski/docker-compose.production.yml
  * ath_to_SSH — путь к файлу с SSH-ключом;
  * SSH_name — имя файла с SSH-ключом (без расширения);
  * username — ваше имя пользователя на сервере;
  * server_ip — IP вашего сервера.
```
5. Запустите docker compose в режиме демона:
```
  sudo docker compose -f docker-compose.production.yml up -d
```
6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
```
  sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
  sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
  sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```
7. На сервере в редакторе nano откройте конфиг Nginx:
```
  sudo nano /etc/nginx/sites-enabled/default
```
8. Измените настройки location в секции server:
```
  location / {
      proxy_set_header Host $http_host;
      proxy_pass http://127.0.0.1:8000;
  }
```
9. Проверьте работоспособность конфига Nginx:
```
  sudo nginx -t
```
Если ответ в терминале такой, значит, ошибок нет:
```
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
```
10. Перезапускаем Nginx
```
  sudo service nginx reload
```

## Автор
Анастасия Вольнова
