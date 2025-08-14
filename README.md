# Documentation

# Python Project - Math Service

## Team members:

- Marcos Dimitris Fetcu (CLD- Team Xception)
- Bogdan Nicolae Cotta (CLD- Team Xception)


##  Overview

A full-stack containerized microservice application offering mathematical operations through a FastAPI backend and a static HTML/CSS/JavaScript frontend. Both services are decoupled and run in separate Docker containers.

---

This project supports:
- Exponentiation (Power)
- Factorial
- Fibonacci
- Operation History (saved in SQLite DB)

Frontend and backend run independently and communicate via HTTP over `localhost`.

---

##  Repository Structure
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
│ ├── requirements.txt
│ └── .github/workflows/docker-build.yml
├── Frontend/
│ ├── index.html
│ ├── script.js
│ ├── style.css
│ ├── Dockerfile
│ └── .github/workflows/docker-build.yml
└── docker-compose.yml

```
---

##  Backend (FastAPI)

###  Features

- FastAPI REST endpoints
- SQLite DB for persistence
- Async background queue for factorial and fibonacci
- Swagger API documentation at `/docs`
- CORS enabled for external frontend access
- Request data models defined using Pydantic for serialization/deserialization
- Code linting enforced using Flake8

###  How to Run Backend

```bash
cd Backend
docker build -t math-backend .
docker run --name backend -p 8000:8000 math-backend
```

| Method | Endpoint     | Body Example                   | Description |
| ------ | ------------ | ------------------------------ | ----------- |
| POST   | `/pow`       | `{ "base": 2, "exponent": 3 }` | Returns 8   |
| POST   | `/factorial` | `{ "number": 5 }`              | Returns 120 |
| POST   | `/fibonacci` | `{ "index": 6 }`               | Returns 8   |
               

###  Code Quality & Style

- This project follows PEP8 coding conventions.
- Static analysis and linting are performed using flake8. Example run:
```bash
flake8 app/
```

##  Frontend (HTML + CSS + JS)

###  Features

- Clean UI for power, factorial, and fibonacci calculations  
- Responsive layout styled with **CSS**
- Communicates with backend via `fetch()` using JSON
- Dockerized and served using Nginx

  
###  How to Run Frontend
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
```js
const BASE_URL = "http://localhost:8000";
```
### All opreations are sent as POST requests. The backend acceptsd them because CORSE is enabled:
```python
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

```yaml
version: '3.8'

services:
  backend:
    build:
      dockerfile: Dockerfile
    container_name: math_microservice
    ports:
      - "8000:8000"
    volumes:
      - ../math_operations.db:/math_operations.db
    restart: always

  frontend:
    build:
      context: ../Frontend
      dockerfile: Dockerfile
    container_name: frontend_static
    ports:
      - "80:80"                    
    depends_on:
      - fastapi                    
    restart: always
```

###  Run both services with:
```bash
docker-compose up --build
```

---

##  Azure Deployment with CI/CD

This project includes a full CI/CD pipeline with GitHub Actions to automate build and deployment to **Azure Container Registry (ACR)** and **Azure Container Apps**, enabling secure public HTTPS endpoints.

### Azure Setup

- Backend and frontend images are built and pushed to **Azure Container Registry (ACR)**.
- The containers are deployed to **Azure Container Apps** with custom domain bindings and HTTPS.
- All updates to `main` and `master` trigger a rebuild and redeployment.

### CI/CD with GitHub Actions

- One `docker-build.yml` workflow exists in both frontend and backend repos.
- It builds the image, pushes it to ACR, and triggers deployment to Container Apps.
- Secrets like `ACR_LOGIN_SERVER`, `ACR_USERNAME`, `ACR_PASSWORD`, and `AZURE_CREDENTIALS` are used securely.


### Workflow Template

```yaml
name: Build and Push Frontend to ACR

on:
  push:
    branches:
      - main
      - master

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Azure Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ secrets.ACR_LOGIN_SERVER }}
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.ACR_LOGIN_SERVER }}/math-frontend:latest

      - name: Log in to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy to Azure Container App
        run: |
          az containerapp update \
            --name math-frontend-app \
            --resource-group pythonmathservice \
            --image ${{ secrets.ACR_LOGIN_SERVER }}/math-frontend:latest \
            --revision-suffix auto-${{ github.run_number }}
```

---

##  Live URLs

- **Frontend**:  
  [https://math-frontend-app.salmonstone-a26147bd.eastus.azurecontainerapps.io/](https://math-frontend-app.salmonstone-a26147bd.eastus.azurecontainerapps.io/)

- **Backend**:  
  [https://math-backend-app.salmonstone-a26147bd.eastus.azurecontainerapps.io/](https://math-backend-app.salmonstone-a26147bd.eastus.azurecontainerapps.io/)

---

 This improves developer velocity and ensures a secure and scalable deployment using modern DevOps best practices.
