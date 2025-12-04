# Подключение React фронтенда

1. Создай React проект: `npx create-react-app my-react-frontend`.
2. Установи axios: `npm install axios`.
3. В App.js добавь код для fetch и post пользователей (как в примере выше).
4. В package.json добавь proxy: "proxy": "http://localhost:3000" для разработки.
5. Для продакшена: собери `npm run build` и загрузи на сервер, настрой Nginx для статических файлов.