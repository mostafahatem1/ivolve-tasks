# Lab 2: Building and Packaging Java Application with Maven

## Overview

This lab demonstrates how to build, test, and package a Java application using Apache Maven.

The objectives of this lab are:

* Install and configure Maven
* Clone a Java Maven project
* Run unit tests
* Build the application
* Generate a JAR artifact
* Run and verify the application

---

## Technologies Used

* Java
* Apache Maven
* Git

---

## Project Setup

Clone the source code repository:

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-maven.git
```

Navigate to the project directory:

```bash
cd calculator-maven
```

Check project files:

```bash
ls
```

Project structure:

```
calculator-maven
|
|-- pom.xml
|
|-- src
    |
    |-- main
    |
    |-- test
```

---

## Install Maven

Check Java installation:

```bash
java -version
```

Install Maven:

```bash
sudo apt update
sudo apt install maven -y
```

Verify Maven installation:

```bash
mvn -version
```

---

## Run Unit Tests

Execute the project tests:

```bash
mvn test
```

Successful execution should display:

```
BUILD SUCCESS
```

Unit tests verify that the calculator functions work correctly before creating the application package.

---

## Build Application

Package the application using Maven:

```bash
mvn package
```

Maven performs the following tasks:

* Compile Java source code
* Run unit tests
* Package the application
* Generate the JAR artifact

After a successful build, check the target directory:

```bash
ls target
```

The generated artifact:

```
target/calculator.jar
```

---

## Run Application

Execute the generated JAR file:

```bash
java -jar target/calculator.jar
```

The calculator application will start and accept user input.

---

## Verify Application

Test the application by providing input values and checking the output.

Example:

```
Enter first number:
10

Enter second number:
5

Result:
15
```

---

## Maven Commands Summary

| Command        | Description                  |
| -------------- | ---------------------------- |
| `mvn test`     | Run unit tests               |
| `mvn package`  | Build and generate JAR file  |
| `mvn clean`    | Remove generated build files |
| `mvn -version` | Check Maven installation     |

---

## Clean Build Files

To remove generated files:

```bash
mvn clean
```

---

## Conclusion

This lab demonstrates how Maven can be used to manage a Java project lifecycle, including testing, building, packaging, and running a Java application using a generated JAR artifact.
