# Documentation

# 🧮 Python Project Math Service

A full-stack containerized microservice application offering mathematical operations through a FastAPI backend and a static HTML/CSS/JavaScript frontend. Both services are decoupled and run in separate Docker containers.

---

## 📌 Overview

This project supports:
- Exponentiation (Power)
- Factorial
- Fibonacci
- Operation History (saved in SQLite DB)

Frontend and backend run independently and communicate via HTTP over `localhost`.

---

## 📂 Repository Structure
```
Python-Project-Math-Service/
├── Backend/
│ ├── app/
│ │ ├── api/routes.py
│ │ ├── db/database.py
│ │ ├── models/
│ │ │ ├── operation.py
│ │ │ └── request_models.py
│ │ ├── services/calculator.py
│ │ └── workers/queue_worker.py
│ ├── main.py
│ ├── Dockerfile
│ └── requirements.txt
├── Frontend/
│ ├── index.html
│ ├── script.js
│ ├── style.css
│ └── Dockerfile
└── docker-compose.yml
```
---

## 🐍 Backend (FastAPI)

### 🔧 Features

- FastAPI REST endpoints
- SQLite DB for persistent history
- Async background queue for factorial and fibonacci
- Swagger API documentation at `/docs`
- CORS enabled for external frontend access

### ▶️ How to Run Backend

```bash
cd Backend
docker build -t math-backend .
docker run --name backend -p 8000:8000 math-backend
```

| Method | Endpoint     | Body Example                   | Description                 |
| ------ | ------------ | ------------------------------ | --------------------------- |
| POST   | `/pow`       | `{ "base": 2, "exponent": 3 }` | Returns 8                   |
| POST   | `/factorial` | `{ "number": 5 }`              | Returns 120                 |
| POST   | `/fibonacci` | `{ "index": 6 }`               | Returns 8                   |
| GET    | `/history`   | *none*                         | Returns all past operations |
| GET    | `/health`    | *none*                         | Returns server status       |

## 🌐 Frontend (HTML + CSS + JS)

### 💡 Features

- Clean UI for power, factorial, and fibonacci calculations  
- Responsive layout styled with **CSS**
- Communicates with backend via `fetch()` using JSON
- Dockerized and served using Nginx

  
### ▶️ How to Run Frontend
```bash
cd Frontend
docker build -t math-frontend .
docker run --name frontend -p 8080:80 math-frontend
```
### Then open:
```
http://localhost:8080
```

## Frontend <--> Backend Communication
```
const BASE_URL = "http://localhost:8000";
```
### All opreations are sent as POST requests. The backend acceptsd them because CORSE is enabled:
```
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Testing the Application
```
Use Swagger UI:
http://localhost:8000/docs
Use the web UI:
http://localhost:8080
```

### Docker Compose
```
version: '3.8'

services:
  backend:
    build: ./Backend
    ports:
      - "8000:8000"
    container_name: math-backend

  frontend:
    build: ./Frontend
    ports:
      - "8080:80"
    container_name: math-frontend
    depends_on:
      - backend
```

### ▶️ Run both services with:
```
docker-compose up --build
```


