# Node.js API with PostgreSQL

A simple RESTful API built with Node.js, Express, and PostgreSQL.

## Features

- RESTful API endpoints for user management
- PostgreSQL database integration
- CORS enabled for frontend integration
- Environment variable configuration

## Prerequisites

- Node.js (v14 or higher)
- PostgreSQL (v12 or higher)
- npm or yarn

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/ScriptVandal/nodejs-api.git
cd nodejs-api
```

### 2. Install dependencies

```bash
npm install
```

### 3. Configure environment variables

Copy the example environment file and update with your settings:

```bash
cp .env.example .env
```

Edit `.env` with your PostgreSQL credentials:

```
DB_USER=your_db_user
DB_HOST=localhost
DB_NAME=myapi_db
DB_PASSWORD=your_password
DB_PORT=5432
PORT=3000
```

### 4. Set up the database

Create the database and users table:

```sql
CREATE DATABASE myapi_db;

\c myapi_db

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 5. Start the server

```bash
# Development mode (with hot reload)
npm run dev

# Production mode
npm start
```

The API will be available at `http://localhost:3000`.

## API Endpoints

| Method | Endpoint     | Description         |
| ------ | ------------ | ------------------- |
| GET    | /api/users   | Get all users       |
| POST   | /api/users   | Create a new user   |

### Example Requests

**Get all users:**
```bash
curl http://localhost:3000/api/users
```

**Create a new user:**
```bash
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'
```

---

## Deployment Instructions for Virtual Server

This guide covers deploying the Node.js API to a virtual server (e.g., DigitalOcean, AWS EC2, Linode).

### 1. Server Setup

SSH into your server and install required software:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js (using NodeSource)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Install PM2 (process manager)
sudo npm install -g pm2
```

### 2. Configure PostgreSQL

```bash
# Switch to postgres user
sudo -u postgres psql

# Create database and user
CREATE USER api_user WITH PASSWORD 'secure_password';
CREATE DATABASE myapi_db OWNER api_user;
GRANT ALL PRIVILEGES ON DATABASE myapi_db TO api_user;
\q

# Connect to the database and create the users table
sudo -u postgres psql -d myapi_db -c "
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
GRANT ALL PRIVILEGES ON TABLE users TO api_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO api_user;
"
```

### 3. Deploy the Application

```bash
# Clone the repository
cd /var/www
sudo git clone https://github.com/ScriptVandal/nodejs-api.git
cd nodejs-api

# Install dependencies
sudo npm install --production

# Create environment file
sudo cp .env.example .env
sudo nano .env  # Update with production values
```

Update your `.env` file with production values:
```
DB_USER=api_user
DB_HOST=localhost
DB_NAME=myapi_db
DB_PASSWORD=secure_password
DB_PORT=5432
PORT=3000
```

### 4. Start with PM2

```bash
# Start the application
pm2 start server.js --name "nodejs-api"

# Save PM2 configuration
pm2 save

# Set PM2 to start on boot
pm2 startup
```

### 5. Configure Nginx (Reverse Proxy)

```bash
# Install Nginx
sudo apt install -y nginx

# Create Nginx configuration
sudo nano /etc/nginx/sites-available/nodejs-api
```

Add this configuration:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/nodejs-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 6. SSL Certificate (Optional but Recommended)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot --nginx -d your-domain.com
```

### 7. Firewall Configuration

```bash
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

---

## Connecting a React Frontend

This guide explains how to connect a React application to this API.

### 1. Create a React App (if you don't have one)

```bash
npx create-react-app my-frontend
cd my-frontend
```

### 2. Install Axios (HTTP Client)

```bash
npm install axios
```

### 3. Configure API Base URL

Create a file `src/api/config.js`:

```javascript
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000';

export default API_BASE_URL;
```

Add to your `.env` file in the React project:
```
REACT_APP_API_URL=http://localhost:3000
```

### 4. Create an API Service

Create a file `src/api/userService.js`:

```javascript
import axios from 'axios';
import API_BASE_URL from './config';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

export const getUsers = async () => {
  const response = await api.get('/api/users');
  return response.data;
};

export const createUser = async (userData) => {
  const response = await api.post('/api/users', userData);
  return response.data;
};

export default api;
```

### 5. Example React Component

Create a file `src/components/UserList.js`:

```javascript
import React, { useState, useEffect } from 'react';
import { getUsers, createUser } from '../api/userService';

function UserList() {
  const [users, setUsers] = useState([]);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    try {
      setLoading(true);
      const data = await getUsers();
      setUsers(data);
      setError(null);
    } catch (err) {
      setError('Failed to fetch users');
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await createUser({ name, email });
      setName('');
      setEmail('');
      fetchUsers(); // Refresh the list
    } catch (err) {
      setError('Failed to create user');
      console.error(err);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>Users</h1>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Name"
          value={name}
          onChange={(e) => setName(e.target.value)}
          required
        />
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
        <button type="submit">Add User</button>
      </form>

      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

### 6. Use the Component in App.js

```javascript
import React from 'react';
import UserList from './components/UserList';
import './App.css';

function App() {
  return (
    <div className="App">
      <UserList />
    </div>
  );
}

export default App;
```

### 7. Running Both Applications

**Start the API server:**
```bash
cd nodejs-api
npm run dev
```

**Start the React app (in a new terminal):**
```bash
cd my-frontend
npm start
```

The React app will run on `http://localhost:3000` by default. If the API is also on port 3000, update the React app's port:

```bash
# On Linux/Mac
PORT=3001 npm start

# On Windows
set PORT=3001 && npm start
```

### Production Deployment

For production, update the React app's environment variable to point to your deployed API:

```
REACT_APP_API_URL=https://your-api-domain.com
```

Build the React app:
```bash
npm run build
```

The `build` folder can be served by Nginx, Vercel, Netlify, or any static hosting service.

---

## License

MIT