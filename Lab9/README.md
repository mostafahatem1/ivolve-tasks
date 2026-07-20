# Lab 9: Containerized Node.js and MySQL Stack Using Docker Compose

## Overview

This lab demonstrates how to containerize a Node.js application and connect it to a MySQL database using Docker Compose.

The application:

- Runs inside a Docker container.
- Connects to a MySQL container.
- Uses a database named `ivolve`.
- Is accessible from the host on port `3001`.
- Provides `/health` and `/ready` endpoints.
- Stores access logs inside `/app/logs`.
- Uses a persistent Docker volume for MySQL data.
- Is published to Docker Hub.

## Source Code

Original application repository:

```text
https://github.com/Ibrahim-Adel15/kubernets-app.git
```

## Project Structure

```text
Lab9/
├── frontend/
├── logs/
├── .dockerignore
├── .env
├── .gitignore
├── db.js
├── docker-compose.yml
├── Dockerfile
├── package.json
└── server.js
```

## Requirements

Make sure the following tools are installed and running:

- Docker
- Docker Compose
- Git

Verify the installation:

```bash
docker --version
docker compose version
git --version
```

## Environment Variables

The application uses the following variables:

```env
DB_HOST=db
DB_USER=ivolve_user
DB_PASSWORD=ivolve_password
```

The MySQL container uses:

```env
MYSQL_ROOT_PASSWORD=root_password
MYSQL_DATABASE=ivolve
MYSQL_USER=ivolve_user
MYSQL_PASSWORD=ivolve_password
```

> Do not commit real production passwords to Git.

## Docker Compose Services

### App Service

The `app` service:

- Is built from the local `Dockerfile`.
- Runs the Node.js application on container port `3000`.
- Is exposed on host port `3001`.
- Connects to MySQL using the service name `db`.
- Mounts the local `logs` directory to `/app/logs`.

Port mapping:

```yaml
ports:
  - "3001:3000"
```

### Database Service

The `db` service:

- Uses the `mysql:8.0` image.
- Creates a database named `ivolve`.
- Uses a health check to confirm that MySQL is ready.
- Stores database files in the `db_data` Docker volume.

## Start the Stack

From inside the `Lab9` directory, run:

```bash
docker compose up -d --build
```

Check the containers:

```bash
docker compose ps
```

Expected result:

```text
app container: running
db container: running and healthy
```

View container logs:

```bash
docker compose logs -f
```

Press `Ctrl + C` to exit the logs view without stopping the containers.

## Verify the Application

Open the application in a browser:

```text
http://localhost:3001
```

Or test it with `curl`:

```bash
curl -i http://localhost:3001
```

## Health Check

```bash
curl -i http://localhost:3001/health
```

Expected status:

```text
HTTP/1.1 200 OK
```

## Readiness Check

```bash
curl -i http://localhost:3001/ready
```

Expected status:

```text
HTTP/1.1 200 OK
```

## Verify the MySQL Database

Confirm that the `ivolve` database exists:

```bash
docker compose exec db mysql \
  -uivolve_user \
  -pivolve_password \
  -e "SHOW DATABASES LIKE 'ivolve';"
```

Test the database connection:

```bash
docker compose exec db mysql \
  -uivolve_user \
  -pivolve_password \
  -Divolve \
  -e "SELECT 1 AS database_connection;"
```

## Verify Access Logs

Generate some requests:

```bash
curl -s http://localhost:3001 > /dev/null
curl -s http://localhost:3001/health > /dev/null
curl -s http://localhost:3001/ready > /dev/null
```

Check the logs inside the application container:

```bash
docker compose exec app ls -lah /app/logs
docker compose exec app tail -n 20 /app/logs/access.log
```

Check the same logs on the host:

```bash
ls -lah logs
tail -n 20 logs/access.log
```

## Docker Hub Image

Docker Hub image:

```text
mostafa760/ivolve-node-mysql:lab9
```

Push the image:

```bash
docker login -u mostafa760
docker push mostafa760/ivolve-node-mysql:lab9
```

Pull the image for verification:

```bash
docker pull mostafa760/ivolve-node-mysql:lab9
```

## Stop the Stack

Stop and remove the containers:

```bash
docker compose down
```

Stop the containers and delete the MySQL volume:

```bash
docker compose down -v
```

> `docker compose down -v` deletes the stored MySQL data.

## Useful Commands

Show running containers:

```bash
docker ps
```

Show local images:

```bash
docker image ls
```

Restart the stack:

```bash
docker compose restart
```

Rebuild the application:

```bash
docker compose up -d --build
```

Follow application logs:

```bash
docker compose logs -f app
```

Follow database logs:

```bash
docker compose logs -f db
```

## Troubleshooting

### Port 3000 Is Already in Use

The application is mapped to host port `3001` to avoid conflicts:

```yaml
ports:
  - "3001:3000"
```

Use:

```text
http://localhost:3001
```

### Database Changes Are Not Applied

MySQL initialization variables are applied only when the data volume is created for the first time.

Reset the database volume:

```bash
docker compose down -v
docker compose up -d --build
```

### Docker Hub Authentication Error

Log in again and retry:

```bash
docker logout
docker login -u mostafa760
docker push mostafa760/ivolve-node-mysql:lab9
```

## Lab Verification Checklist

- [x] Node.js application containerized.
- [x] MySQL service configured.
- [x] `ivolve` database created.
- [x] Docker Compose used to run both services.
- [x] MySQL data persisted using `db_data`.
- [x] Application accessible on port `3001`.
- [x] `/health` endpoint verified.
- [x] `/ready` endpoint verified.
- [x] Access logs available at `/app/logs`.
- [x] Docker image pushed to Docker Hub.