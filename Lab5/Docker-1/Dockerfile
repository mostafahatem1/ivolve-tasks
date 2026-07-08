# =====================
# Stage 1: Build stage
# =====================
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

# copy code
COPY . .

# build jar
RUN mvn clean package -DskipTests


# =====================
# Stage 2: Run stage
# =====================
FROM eclipse-temurin:17-jdk

WORKDIR /app

# copy jar from stage 1
COPY --from=builder /app/target/*.jar app.jar

# expose port
EXPOSE 8080

# run app
CMD ["java", "-jar", "app.jar"]
