# Lab 5: Multi-Stage Docker Build for Java Application

## Overview

This lab demonstrates how to use Docker Multi-Stage Builds to efficiently build and run a Java application using Maven and OpenJDK.

The main objectives:

* Reduce Docker image size
* Separate build environment from runtime
* Apply Docker best practices

---

## Technologies Used

* Docker
* Maven
* Java (OpenJDK 17)

---

## Project Setup

### Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

---

## Docker Multi-Stage Build

### Dockerfile

```dockerfile
# Stage 1: Build stage
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package -DskipTests


# Stage 2: Runtime stage
FROM eclipse-temurin:17-jdk

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

---

## Build Docker Image

```bash
docker build -t app3 .
```

### Check Image Size

```bash
docker images
```

Multi-stage build reduces the final image size by excluding build dependencies such as Maven.

---

## Run the Container

```bash
docker run -d -p 8080:8080 --name container3 app3
```

---

## Test the Application

Using browser:
http://localhost:8080

Or using curl:

```bash
curl http://localhost:8080
```

---

## Stop and Remove Container

```bash
docker stop container3
docker rm container3
```

---

## Remove Docker Image

```bash
docker rmi app3
```

---

## Key Concepts

### Multi-Stage Build

Uses multiple FROM statements to separate build and runtime environments, resulting in smaller and more secure images.

### Skip Tests

-DskipTests is used to speed up the build process inside Docker.

### Benefits

* Smaller image size
* Faster deployment
* Improved security

---

## Conclusion

This lab demonstrates how to build and run a Java application using Docker Multi-Stage Build, improving performance and following best practices for production environments.
