# Lab 6: Managing Docker Environment Variables Across Build and Runtime

## Overview

This lab demonstrates how to manage environment variables in Docker using different approaches during container execution.

The application is a Python Flask application running inside a Docker container. The same Docker image is used with different configurations for development, staging, and production environments.

The objectives of this lab are:

* Build a Docker image for a Python Flask application
* Install Flask inside the Docker image
* Expose application port 8080
* Run multiple containers with different environment variables
* Manage variables using command line, environment file, and Dockerfile

---

## Technologies Used

* Docker
* Python
* Flask

---

## Project Setup

Clone the repository:

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-3.git
```

Navigate to the project directory:

```bash
cd Docker-3
```

The project structure:

```
Docker-3
|
|-- app.py
|-- Dockerfile
```

---

## Dockerfile

Create a Dockerfile:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY . .

RUN pip install flask

ENV APP_MODE=production
ENV APP_REGION=canada-west

EXPOSE 8080

CMD ["python", "app.py"]
```

---

## Build Docker Image

Build the Docker image:

```bash
docker build -t app6 .
```

Check the created image:

```bash
docker images
```

---

# Running Containers with Different Environment Variables

The same image will be used with three different configurations.

---

## 1. Development Environment

Environment variables are passed directly in the docker run command.

Variables:

```
APP_MODE=development
APP_REGION=us-east
```

Run container:

```bash
docker run -d -p 8080:8080 \
-e APP_MODE=development \
-e APP_REGION=us-east \
--name container_dev app6
```

---

## 2. Staging Environment

Create an environment file:

```bash
vim env.list
```

Add variables:

```
APP_MODE=staging
APP_REGION=us-west
```

Run container using the environment file:

```bash
docker run -d -p 8081:8080 \
--env-file env.list \
--name container_stage app6
```

---

## 3. Production Environment

Production variables are defined inside the Dockerfile:

```dockerfile
ENV APP_MODE=production
ENV APP_REGION=canada-west
```

Run container:

```bash
docker run -d -p 8082:8080 \
--name container_prod app6
```

---

## Test the Application

Check running containers:

```bash
docker ps
```

Test each container:

```bash
curl http://localhost:8080
```

```bash
curl http://localhost:8081
```

```bash
curl http://localhost:8082
```

---

## Check Container Environment Variables

Development:

```bash
docker exec container_dev env
```

Staging:

```bash
docker exec container_stage env
```

Production:

```bash
docker exec container_prod env
```

---

## Stop Containers

```bash
docker stop container_dev container_stage container_prod
```

---

## Remove Containers

```bash
docker rm container_dev container_stage container_prod
```

---

## Remove Docker Image

```bash
docker rmi app6
```

---

## Environment Variable Priority

Docker applies environment variables according to the following priority:

1. Variables passed using `-e` in the docker run command
2. Variables passed using `--env-file`
3. Variables defined using `ENV` in the Dockerfile

Command line variables have the highest priority and can override other values.

---

## Conclusion

This lab demonstrates how Docker environment variables can be managed across different environments.

Using different methods for setting variables allows the same Docker image to be reused for development, staging, and production environments without changing the application code.
