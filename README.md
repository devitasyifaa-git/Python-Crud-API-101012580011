
# Python CRUD API

A simple REST API built with Python 3.11 using the built-in HTTP server module.

This project is part of a CI/CD practical assignment using Jenkins, GitHub, and Docker.

## Features

- Health check endpoint
- Create a new message
- Read all messages
- Delete a message by index
- JSON-based data storage
- Docker volume for data persistence
- Automated deployment using Jenkins Pipeline

## Project Structure

```text
python-crud-api/
│
├── app.py
├── Dockerfile
├── .dockerignore
├── Jenkinsfile
└── README.md
```

## Requirements

- Python 3.11
- Docker
- Git
- Jenkins
- GitHub account

## Build Docker Image

```bash
docker build -t python-crud-api:manual .
```

## Create Docker Volume

```bash
docker volume create production-api-data
```

## Run Application

```bash
docker run -d \
  --name production-api-container \
  -p 5000:5000 \
  -v production-api-data:/var/lib/backend_data \
  python-crud-api:manual
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /health | Check API status |
| GET | /messages | Retrieve all messages |
| POST | /messages | Create a new message |
| DELETE | /messages?index=0 | Delete a message by index |

## API Testing

### 1. Health Check

```bash
curl -i http://localhost:5000/health
```

Expected response:

```json
{
  "status": "active",
  "service": "Backend API"
}
```

### 2. Read Messages

```bash
curl -i http://localhost:5000/messages
```

### 3. Create Message

```bash
curl -i -X POST http://localhost:5000/messages \
  -H 'Content-Type: application/json' \
  -d '{"message":"Belajar CI/CD menggunakan Jenkins"}'
```

### 4. Delete Message

```bash
curl -i -X DELETE 'http://localhost:5000/messages?index=0'
```

## Data Persistence

Messages are stored in:

`/var/lib/backend_data/messages.json`

Docker named volume:

`production-api-data`

The named volume keeps application data available even when the application container is removed and recreated.

## Jenkins CI/CD Pipeline

The Jenkins Pipeline automates the following processes:

1. Checkout source code from GitHub.
2. Verify Python syntax.
3. Build Docker image.
4. Deploy the updated application container.
5. Perform an API health check.

## Security Notes

- Never commit passwords or access tokens.
- Use Jenkins Credentials to store GitHub Personal Access Tokens.
- Use minimum required permissions for repository access.
- Keep sensitive credentials out of source code and documentation.

## Project Information

Project: Python CRUD API

Practical Topic: CI/CD Pipeline

Tools: Python, Docker, Jenkins, and GitHub

Webhook test 2026-11-02 19:36:20
