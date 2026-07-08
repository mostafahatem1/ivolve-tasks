# 🧪 Lab 1: Building and Packaging Java Application with Gradle

## 📌 Overview

This lab demonstrates how to build, test, and run a Java application using **Gradle**.
The application is a simple calculator project, and the goal is to generate a runnable `.jar` file.

---

## 🔧 Prerequisites

* Java JDK installed (Java 17 or compatible)
* Gradle installed

Check installation:

```bash
java -version
gradle -v
```

---

## 📥 Clone the Project

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-gradle.git
cd calculator-gradle
```

---

## 🧪 Run Unit Tests

Run all unit tests using:

```bash
gradle test
```

✅ This ensures the application logic works correctly before building.

---

## 🏗️ Build the Application

```bash
gradle build
```

📦 After a successful build, the generated JAR file will be located at:

```
build/libs/calculator.jar
```

---

## ▶️ Run the Application

```bash
java -jar build/libs/calculator.jar
```

---

## ✅ Verify the Application

* The application should run without errors.
* Output should appear in the terminal.
* Ensure calculations or functionality behave correctly.

---

## 🧠 Key Concepts

* **Gradle Build Lifecycle** (init → compile → test → package)
* **Unit Testing** in Java projects
* **JAR Packaging** for Java applications
* **Automation of build process**

---

## 💡 Notes

* Running tests before building helps catch errors early.
* The `build` command automatically runs tests unless skipped.
* Gradle simplifies dependency management and build automation.


