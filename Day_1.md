#  Day 1 : Backend Foundations (Hands-On)

### What We Built
A Node.js + Express backend connected to a **cloud MySQL database on Aiven**.

### Project Structure

```
backend-auth-practice/
├── config/
│   └── db.js          # Database connection (pool)
├── routes/            # API endpoints (to be built)
├── .env               # Secrets — NEVER commit to GitHub
├── server.js          # Main entry point
├── package.json
```

### Packages Installed

```bash
npm install express mysql2 jsonwebtoken bcryptjs dotenv cors
```

| Package | Purpose |
|---|---|
| `express` | Web framework for creating API routes |
| `mysql2` | Connects Node.js to MySQL |
| `jsonwebtoken` | Generates JWT tokens for user auth |
| `bcryptjs` | Hashes passwords — never store plain text |
| `dotenv` | Loads `.env` variables into `process.env` |
| `cors` | Allows React frontend to talk to this backend |

---

## The .env File — Explained

```env
PORT=8000
DB_HOST=your_aiven_host_here
DB_USER=avnadmin
DB_PASSWORD=your_aiven_password_here
DB_NAME=defaultdb
DB_PORT=25321
JWT_SECRET=your_generated_secret_here
```

### What Each Variable Means

| Variable | What It Is | Where It Comes From |
|---|---|---|
| `PORT` | The door your **Node.js server** listens on. Your React app sends requests here. | **You choose** — typically `8000` or `5000` |
| `DB_PORT` | The door into your **cloud database**. | **Aiven provides** — usually `25321` for MySQL |
| `DB_HOST` | The address of your cloud database server. | **Aiven provides** (e.g., `mysql-xxx.aivencloud.com`) |
| `DB_USER` | Username for database access. | **Aiven provides** (usually `avnadmin`) |
| `DB_PASSWORD` | Password for database access. | **Aiven provides** |
| `DB_NAME` | Which database to use. | **Aiven provides** (defaults to `defaultdb`) |
| `JWT_SECRET` | A private key your server uses to **sign and verify** login tokens. Like a digital wax seal. | **You generate** — see below |

### How to Generate a Secure JWT_SECRET

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

This outputs a long random hex string. Paste it as your `JWT_SECRET`.

**Why is this important?** If anyone gets your `JWT_SECRET`, they can forge login tokens and impersonate any user — without needing a password.

---

## Key Code Files

### config/db.js — Database Connection Pool

```javascript
const mysql = require('mysql2');
require('dotenv').config();

const pool = mysql.createPool({
  host: process.env.DB_HOST,       // Where is the DB?
  user: process.env.DB_USER,       // Who is accessing it?
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,   // Which database?
  port: process.env.DB_PORT,       // Which door? (e.g., 25321)
  ssl: {
    rejectUnauthorized: true       // Encrypt the connection
  }
});

module.exports = pool.promise();
```

**Why `createPool` instead of `createConnection`?**

- `createConnection` = one phone line. One user at a time.
- `createPool` = a call center. Multiple connections ready to go, reused efficiently.

Pool keeps connections "warm" — no delay opening new ones for each request.

### server.js — Main Entry Point

```javascript
// 1. IMPORTS
const express = require('express');
const cors = require('cors');
require('dotenv').config();
const db = require('./config/db');

// 2. INITIALIZE
const app = express();

// 3. MIDDLEWARE
app.use(cors());           // Allow React (port 3000) to talk to this (port 8000)
app.use(express.json());   // Parse JSON from frontend requests

// 4. ROUTES
app.get('/', (req, res) => {
  res.send('Auth Backend is Running!');
});

// 5. START
const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### MySQL Table (Run in Aiven Console)

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Interview Must-Knows (Long-Term Memory)

### Q: Why use `bcrypt` for passwords?
**A:** We never store plain-text passwords. `bcrypt` hashes them into irreversible random strings. Even if the database is breached, attackers only see hashes they can't reverse.

### Q: Why do we need CORS?
**A:** Browsers enforce a Same-Origin Policy. If React runs on port 3000 and your API on port 8000, the browser blocks the request unless the server explicitly allows it via `cors()` middleware.

### Q: Difference between PORT and DB_PORT?
**A:** `PORT` is where the application listens for users. `DB_PORT` is the specific port to communicate with the database engine. They serve completely different purposes.

### Q: How do you prevent SQL Injection?
**A:** Use parameterized queries (the `?` placeholders). Never concatenate user input directly into SQL strings.

```javascript
// BAD (vulnerable)
db.query(`SELECT * FROM users WHERE id = ${userInput}`)

// GOOD (safe)
db.query('SELECT * FROM users WHERE id = ?', [userInput])
```

### Q: What is Connection Pooling?
**A:** A technique where the application maintains a set of reusable database connections instead of opening a new one for each request. Reduces latency and improves scalability.

### Q: What does `ssl: { rejectUnauthorized: true }` do?
**A:** Ensures the connection between your server and the cloud database is encrypted. Prevents attackers from intercepting data in transit.

---

## Troubleshooting Notes

| Problem | Solution |
|---|---|
| `403 Access Denied` on localhost:5000 | Port 5000 is often used by system services (AirPlay on Mac). Change to `PORT=8000`. |
| Can't connect to Aiven | Check `DB_HOST`, `DB_PORT`, `DB_PASSWORD` in `.env`. Make sure SSL is enabled. |
| `localhost` doesn't work | Try `http://127.0.0.1:8000` instead |

---

## Security Checklist

- [x] All secrets in `.env` file (not hardcoded)
- [x] `.env` added to `.gitignore`
- [x] Using parameterized SQL queries (`?` placeholders)
- [x] SSL enabled for cloud database connection
- [x] Connection pooling for efficient DB access
- [ ] Password hashing with bcrypt (next session)
- [ ] Register API endpoint (next session)
- [ ] Login API with JWT generation (next session)

---

## What's Next?

1. Build the **Register API** — hash passwords with bcrypt before saving
2. Build the **Login API** — validate credentials and return JWT
3. Connect **React frontend** to these APIs
4. Implement **token refresh logic** with Axios interceptors
