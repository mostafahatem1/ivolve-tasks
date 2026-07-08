# 🐳 Lab 3: Run Java Spring Boot App in a Container

## 📌 Overview

This lab demonstrates how to containerize a **Java Spring Boot application** using Docker.

The application is built using **Maven with Java 17**, packaged into a JAR file, and then deployed inside a Docker container.

---

## 🔧 Prerequisites

* Java 17
* Maven
* Docker installed and running

Check installation:

```bash
java -version
mvn -version
docker --version
```

---

## 📥 Clone the Application Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

---

# 🐳 Docker Configuration

## Create Dockerfile

A multi-stage Docker build was used to separate the build environment from the runtime environment.

```dockerfile
# Stage 1: Build
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package -DskipTests


# Stage 2: Run
FROM eclipse-temurin:17-jdk

WORKDIR /app

COPY --from=builder /app/target/demo-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

---

# 🏗️ Build Docker Image

Create Docker image named `app1`:

```bash
docker build -t app1 .
```

Check the image size:

```bash
docker images
```

---

# 🚀 Run Container

Create and run a container from the image:

```bash
docker run -d -p 8080:8080 --name container1 app1
```

---

# 🧪 Test the Application

Open the application:

```
http://localhost:8080
```

Or test using:

```bash
curl http://localhost:8080
```

---

# 📋 Check Container Logs

To verify the application status:

```bash
docker logs container1
```

---

# 🛑 Stop and Remove Container

Stop the running container:

```bash
docker stop container1
```

Remove the container:

```bash
docker rm container1
```

---

# 🧠 Key Concepts Learned

* Creating Docker images
* Writing Dockerfiles
* Multi-stage Docker builds
* Building Spring Boot applications with Maven
* Running Java applications inside containers
* Port mapping between host and container
