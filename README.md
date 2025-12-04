# nodejs-api

Простой API на Node.js с базой данных PostgreSQL.

## Установка и запуск

1. Клонируйте репозиторий: `git clone https://github.com/ScriptVandal/nodejs-api.git && cd nodejs-api`
2. Установите зависимости: `npm install`
3. Настройте базу данных PostgreSQL и обновите .env файл с вашими данными.
4. Запустите сервер: `npm start` или `npm run dev` для разработки.

## API роуты

- GET /api/users: Получить всех пользователей
- POST /api/users: Добавить нового пользователя (тело: {name, email})

## Деплой на виртуальный сервер

См. файл DEPLOYMENT.md

## Подключение React фронтенда

См. файл REACT_INTEGRATION.md