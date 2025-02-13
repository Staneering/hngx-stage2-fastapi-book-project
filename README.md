# FastAPI Book Management API with CI/CD

## 📌 Overview

This project is a FastAPI-based book management system, deployed with a **CI/CD pipeline** using **GitHub Actions, AWS EC2, and Nginx as a reverse proxy**. It includes a missing endpoint implementation and automated deployment for continuous integration.

## 🚀 Features

- 📚 Retrieve book details by ID
- ✅ Continuous Integration with **pytest**
- 🚀 Automatic Deployment via **GitHub Actions**
- 🌍 Deployed on AWS EC2 with Nginx

## 📂 Project Structure

```
fastapi-book-project/
├── api/
│   ├── db/
│   │   ├── __init__.py
│   │   └── schemas.py      # Data models and in-memory database
│   ├── routes/
│   │   ├── __init__.py
│   │   └── books.py        # Book route handlers
│   └── router.py           # API router configuration
├── core/
│   ├── __init__.py
│   └── config.py           # Application settings
├── tests/
│   ├── __init__.py
│   └── test_books.py       # API endpoint tests
├── main.py                 # Application entry point
├── requirements.txt        # Project dependencies
├── .github/workflows/      # CI/CD pipeline configurations
└── README.md
```

## 🔧 Setup & Installation

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/Staneering/hngx-stage2-fastapi-book-project.git
cd hngx-stage2-fastapi-book-project
```

### 2️⃣ Create and Activate Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3️⃣ Install Dependencies
```bash
pip install -r requirements.txt
```

### 4️⃣ Set Up the Database
- Ensure **SQLite** or your preferred database is installed.
- Run migrations:
```bash
alembic upgrade head
```

### 5️⃣ Run the Application Locally
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```
- Visit `http://127.0.0.1:8000/docs` to access Swagger UI.

## 📌 API Endpoints

### Books
- `GET /api/v1/books/{book_id}` - Retrieve book details by ID

**Example Response (Success - 200 OK):**
```json
{
    "id": 1,
    "title": "The Alchemist",
    "author": "Paulo Coelho",
    "year": 1988
}
```

**Example Response (Book Not Found - 404):**
```json
{
    "detail": "Book not found"
}
```

## 🛠 CI/CD Setup

### ✅ CI Pipeline
- Runs on **pull requests** to the `main` branch.
- Executes **pytest** to ensure all tests pass.

### 🚀 Deployment Pipeline
- Triggers **on merging a PR** to `main`.
- Deploys the latest code on AWS EC2.

#### GitHub Actions Workflow (`deploy.yml`)
```yaml
name: Deploy
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v3

      - name: Deploy to Server
        run: |
          ssh ubuntu@your-ec2-ip "cd /home/ubuntu/fastapi-app && git pull origin main && sudo systemctl restart fastapi"
```

## 🌍 Deployment Setup

### 1️⃣ Server Configuration (AWS EC2)
- **OS**: Ubuntu 22.04
- **Firewall**: Open ports `22`, `80`, and `443`.
- **Install dependencies**:
```bash
sudo apt update && sudo apt install python3-pip nginx -y
```

### 2️⃣ Running FastAPI as a Systemd Service
Create `/etc/systemd/system/fastapi.service`:
```ini
[Unit]
Description=FastAPI Application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/fastapi-app
ExecStart=/home/ubuntu/fastapi-env/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```
Reload and start service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
```

### 3️⃣ Nginx Configuration
Create `/etc/nginx/sites-available/fastapi`:
```nginx
server {
    listen 80;
    server_name your-ec2-ip;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Enable config and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## ✅ Testing the Deployment
- Run:
```bash
curl http://your-ec2-ip/api/v1/books/1
```
- If it returns **book details**, deployment is successful!



