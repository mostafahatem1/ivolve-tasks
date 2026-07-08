# 🐳 Lab 4: Run Java Spring Boot App in a Container

## 📌 Overview

This lab demonstrates how to run a **Java Spring Boot application inside a Docker container** by building the application first, then copying the generated JAR file into a Docker image based on **Java 17**.

The main goal is to create a Docker image (`app2`) and run the Spring Boot application inside a container (`container2`).

---

# 🔧 Prerequisites

* Java 17 installed
* Maven installed
* Docker installed and running

Verify installations:

```bash
java -version
mvn -version
docker --version
```

---

# 📥 Clone Application Code

Clone the Spring Boot application repository:

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git

cd Docker-1
```

---

# 🏗️ Build the Application

Build the Spring Boot application using Maven:

```bash
mvn clean package -DskipTests
```

After a successful build, the JAR file will be generated:

```
target/demo-0.0.1-SNAPSHOT.jar
```

Verify:

```bash
ls target
```

---

# 🐳 Create Dockerfile

Create a file named:

```
Dockerfile
```

Add the following configuration:

```dockerfile
FROM eclipse-temurin:17-jdk

WORKDIR /app

COPY target/demo-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

---

# 🏗️ Build Docker Image

Create Docker image with the name `app2`:

```bash
docker build -t app2 .
```

---

# 📊 Check Image Size

Display available Docker images:

```bash
docker images
```

Check the size of the generated `app2` image.

---

# 🚀 Run Docker Container

Create and run a container from the `app2` image:

```bash
docker run -d -p 8080:8080 --name container2 app2
```

---

# 🔍 Verify Container Status

Check running containers:

```bash
docker ps
```

View application logs:

```bash
docker logs container2
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

# 🛑 Stop and Remove Container

Stop the container:

```bash
docker stop container2
```

Remove the container:

```bash
docker rm container2
```

---

# 🧠 Key Concepts Learned

* Building Spring Boot applications using Maven
* Creating Docker images from JAR files
* Using Java 17 base images
* Containerizing Java applications
* Exposing application ports
* Managing Docker containers

---

# 🔄 Difference Between Lab 3 and Lab 4

| Lab   | Approach                                                                               |
| ----- | -------------------------------------------------------------------------------------- |
| Lab 3 | Application is built inside Docker using Maven multi-stage build                       |
| Lab 4 | Application is built outside Docker, then the JAR file is copied into the Docker image |

