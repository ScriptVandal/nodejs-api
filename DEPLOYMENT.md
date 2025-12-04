# Инструкции по деплою на виртуальный сервер

## Шаг 1: Подготовка сервера (пример для DigitalOcean VPS с Ubuntu)

1. Зарегистрируйся на DigitalOcean, создай Droplet (VPS) с Ubuntu 22.04.
2. Подключись по SSH: `ssh root@your_ip`.

## Шаг 2: Установка зависимостей

1. Обнови систему: `sudo apt update && sudo apt upgrade -y`.
2. Установи Node.js: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - && sudo apt-get install -y nodejs`.
3. Установи PostgreSQL: `sudo apt install postgresql postgresql-contrib -y`.
4. Создай базу данных и пользователя:
   - `sudo -u postgres createdb myapi_db`.
   - `sudo -u postgres createuser myuser`.
   - `sudo -u postgres psql -c "ALTER USER myuser PASSWORD 'your_password';"`.
5. Обнови .env с данными сервера (DB_HOST=localhost, etc.).

## Шаг 3: Деплой API

1. Клонируй репозиторий: `git clone https://github.com/ScriptVandal/nodejs-api.git && cd nodejs-api`.
2. Установи зависимости: `npm install`.
3. Запусти API: `npm start` (или используй PM2: `npm install pm2 -g && pm2 start server.js --name myapi`).

## Шаг 4: Настройка Nginx (опционально для прокси)

1. Установи Nginx: `sudo apt install nginx -y`.
2. Создай конфиг: `sudo nano /etc/nginx/sites-available/myapp`.
   Добавь:
   ```
   server {
       listen 80;
       server_name your_ip;
       location /api/ {
           proxy_pass http://localhost:3000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```
3. Включи сайт: `sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/ && sudo systemctl reload nginx`.

API будет доступен по IP/api/.