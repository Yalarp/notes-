# Creating Executable JAR - Complete Guide

## Table of Contents
1. [What is an Executable JAR?](#what-is-an-executable-jar)
2. [Spring Boot JAR Structure](#spring-boot-jar-structure)
3. [Building with Maven](#building-with-maven)
4. [Running the JAR](#running-the-jar)
5. [Passing Arguments](#passing-arguments)
6. [Profiles](#profiles)
7. [Deployment](#deployment)
8. [Summary](#summary)

---

## What is an Executable JAR?

An **executable JAR** is a self-contained Java archive that includes:
- Your compiled code
- All dependencies (libraries)
- Embedded server (Tomcat)
- Configuration files

> [!IMPORTANT]
> Spring Boot creates "fat" JARs that contain **everything needed to run** the application - no external server required!

---

## Spring Boot JAR Structure

```
my-application-1.0.jar
├── BOOT-INF/
│   ├── classes/          ← Your compiled code
│   │   ├── com/example/
│   │   ├── application.properties
│   │   └── templates/
│   └── lib/              ← All dependencies
│       ├── spring-boot-2.7.0.jar
│       ├── tomcat-embed-core.jar
│       └── ... (hundreds of JARs)
├── META-INF/
│   └── MANIFEST.MF       ← Main class info
└── org/springframework/boot/loader/
    └── JarLauncher.class ← Spring Boot launcher
```

---

## Building with Maven

### Using Maven Command

From your course materials: **"How to create jar file from Spring Boot Project"**

```bash
# Navigate to project directory
cd my-spring-boot-project

# Clean and package
mvn clean package

# Skip tests (faster)
mvn clean package -DskipTests
```

### Output Location

```
my-spring-boot-project/
└── target/
    └── my-application-0.0.1-SNAPSHOT.jar  ← Executable JAR
```

### pom.xml Configuration

```xml
<project>
    <packaging>jar</packaging>  <!-- Ensure JAR packaging -->
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Running the JAR

### Basic Run

```bash
java -jar my-application-0.0.1-SNAPSHOT.jar
```

### With Custom Port

```bash
java -jar my-application.jar --server.port=9090
```

### With Memory Settings

```bash
java -Xms256m -Xmx512m -jar my-application.jar
```

### Background Execution (Linux/Mac)

```bash
nohup java -jar my-application.jar > app.log 2>&1 &
```

---

## Passing Arguments

### Command Line Arguments

```bash
# Override properties
java -jar app.jar --server.port=8081 --spring.datasource.url=jdbc:mysql://prod:3306/db

# Enable debug
java -jar app.jar --debug

# Set active profile
java -jar app.jar --spring.profiles.active=production
```

### System Properties (-D)

```bash
java -Dserver.port=8081 -Dspring.profiles.active=prod -jar app.jar
```

### Environment Variables

```bash
# Linux/Mac
export SERVER_PORT=8081
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar

# Windows
set SERVER_PORT=8081
set SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

---

## Profiles

### What are Profiles?

Profiles allow different configurations for different environments (dev, test, prod).

### Profile-Specific Properties

```
src/main/resources/
├── application.properties              ← Default
├── application-dev.properties           ← Development
├── application-test.properties          ← Testing
└── application-prod.properties          ← Production
```

### Example Properties

**application.properties** (default):
```properties
spring.application.name=MyApp
server.port=8080
```

**application-dev.properties**:
```properties
server.port=8081
spring.datasource.url=jdbc:h2:mem:devdb
spring.jpa.show-sql=true
logging.level.root=DEBUG
```

**application-prod.properties**:
```properties
server.port=80
spring.datasource.url=jdbc:mysql://prod-server:3306/proddb
spring.jpa.show-sql=false
logging.level.root=WARN
```

### Activating Profiles

```bash
# Via command line
java -jar app.jar --spring.profiles.active=prod

# Multiple profiles
java -jar app.jar --spring.profiles.active=prod,redis

# In application.properties
spring.profiles.active=dev
```

---

## Deployment

### Deployment Options

| Platform | Method |
|----------|--------|
| VPS/Server | Copy JAR + `java -jar` |
| Docker | Dockerfile + container |
| Cloud (AWS, Azure) | Elastic Beanstalk, App Service |
| Kubernetes | Pod deployment |

### Simple Deployment Script

```bash
#!/bin/bash
APP_NAME=my-application
JAR_FILE=target/${APP_NAME}-0.0.1-SNAPSHOT.jar

# Stop existing
pkill -f ${APP_NAME}

# Build
mvn clean package -DskipTests

# Run with production profile
nohup java -jar ${JAR_FILE} --spring.profiles.active=prod > app.log 2>&1 &

echo "Application started!"
```

### Docker Deployment

**Dockerfile**:
```dockerfile
FROM openjdk:17-jdk-slim
COPY target/my-application-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```bash
docker build -t my-app .
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod my-app
```

---

## Summary

### Key Commands

| Task | Command |
|------|---------|
| Build JAR | `mvn clean package` |
| Run JAR | `java -jar app.jar` |
| Custom port | `--server.port=9090` |
| Activate profile | `--spring.profiles.active=prod` |
| Skip tests | `mvn clean package -DskipTests` |

### Key Points

1. Spring Boot creates **fat JARs** with embedded server
2. Use `mvn clean package` to build
3. Use `java -jar` to run
4. Use **profiles** for environment-specific config
5. Override properties via command line or environment variables

---

## Practice Questions

1. What is an executable JAR in Spring Boot?
2. How do you build a JAR file using Maven?
3. How do you run a JAR with a custom port?
4. What are Spring profiles and how do you use them?
5. How do you pass arguments when running a JAR?
6. What is the difference between WAR and JAR deployment?

---

**End of Note 17: Creating Executable JAR**

*Previous: [16_JPQL_Custom_Queries.md](file:///c:/Users/2706p/Desktop/mcq/notes/16_JPQL_Custom_Queries.md)*  
*Next: [18_Exception_Handling.md](file:///c:/Users/2706p/Desktop/mcq/notes/18_Exception_Handling.md)*
