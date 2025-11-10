Лабораторная работа №1.

В рамках лабораторной работы требовалось:

Настроить сервер Nginx для работы по HTTPS с использованием сертификата.

Настроить принудительное перенаправление HTTP запросов на HTTPS

Использовать alias для создания псевдонимов путей к файлам или каталогам.

Настроить виртуальные хосты для обслуживания двух проектов на одном сервере.

Обеспечить корректную работу обоих проектов через HTTPS с разными доменными именами.


Установим nginx

brew install nginx


Определим путь конфигурации nginx

NGX_ETC="$(brew --prefix)/etc/nginx"

echo "nginx config dir: $NGX_ETC"


Создадим папки для проектов и общих ресурсов

mkdir -p /Users/user/sites/project1

mkdir -p /Users/user/sites/project2

mkdir -p /Users/user/shared_assets


Создадим простые пет-проекты


cat > /Users/user/sites/project1/index.html <<'EOF'

![](./image1.png)

EOF#

cat > /Users/user/sites/project2/index.html <<'EOF'


![](./image2.png)

EOF

Примитивные "статические" файлы, чтобы проверить alias

echo "LOGO1" > /Users/user/shared_assets/logo1.txt

echo "LOGO2" > /Users/user/shared_assets/logo2.txt


Выставим права (чтобы nginx читал)

chmod -R u+rwX,go+rX /Users/user/sites /Users/user/shared_assets


Добавим локальные домены:

sudo nano /etc/hosts

Добавим в конце файла строки:

127.0.0.1   project1.test

127.0.0.1   project2.test


Настроим nginx — создадим директорию servers и конфиги виртуальных хостов:

NGX_ETC="$(brew --prefix)/etc/nginx"

mkdir -p "$NGX_ETC/servers"


Резервная копия основного nginx.conf

cp "$NGX_ETC/nginx.conf" "$NGX_ETC/nginx.conf.bak"


Откроем основной nginx.conf, чтобы он включал servers 

В большинстве brew-установок уже есть include, но выполним безопасную замену: создадим минимальный nginx.conf, который включает стандартную секцию и наш includes.

cat > "$NGX_ETC/nginx.conf" <<'NGINXCONF'

user  _www;

worker_processes  auto;

error_log  logs/error.log notice;

pid        logs/nginx.pid;


events {
    worker_connections  1024;
    
}


http {
    include       mime.types;
        default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    SSL settings - базовые 
    Включаем конфиги сайтов
    include servers/ *. conf;
}

NGINXCONF

Создадим два файла: project1.conf и project2.conf. Они будут:

слушать 80 и делать 301 на https/

слушать 443 с SSL и обслуживать нужный каталог/

продемонстрируем alias на /static/ (shared_assets)/

project1:

cat > "$NGX_ETC/servers/project1.conf" <<'PROJECT1'

server {
    listen 80;
    server_name project1.test;
    Перенаправление на https (сохраняем урл)
    return 301 https: / /$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name project1.test;
    ssl_certificate     
    root /Users/user/sites/project1;
    index index.html index.htm;
    Пример alias: /static/ будет мапиться в /Users/user/shared_assets/
    location /static/ {
        alias /Users/user/shared_assets/;
        Пробуем отдать файл, иначе 404
        try_files $uri $uri/ =404;
    }
    location / {
        try_files $uri $uri/ =404;
    }
    access_log  /usr/local/var/log/nginx/project1.access.log;
    error_log   /usr/local/var/log/nginx/project1.error.log;
}
PROJECT1

project2:
cat > "$NGX_ETC/servers/project2.conf" <<'PROJECT2'

server {
    listen 80;
    server_name project2.test;
    return 301 https:/ /$host$request_uri;
}









