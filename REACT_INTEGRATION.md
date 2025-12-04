# Подключение React фронтенда

1. Создай React проект с Vite: `npm create vite@latest my-react-frontend -- --template react`.
2. Перейди в папку проекта: `cd my-react-frontend`.
3. Установи зависимости: `npm install`.
4. Установи axios: `npm install axios`.
5. В App.jsx добавь код для fetch и post пользователей (как в примере выше).
6. В vite.config.js добавь proxy: `proxy: { '/api': 'http://localhost:3000' }` для разработки.
7. Для продакшена: собери `npm run build` и загрузи на сервер, настрой Nginx для статических файлов.