# Java UDF Development Guide

This guide walks through the complete process of setting up and developing a Java UDF (User Defined Function), from environment setup to deployment.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
  - [Install Chocolatey](#install-chocolatey)
  - [Install Java 8](#install-java-8)
  - [Install Maven](#install-maven)
- [Project Setup](#project-setup)
  - [Create Maven Project](#create-maven-project)
  - [Project Structure](#project-structure)
  - [Configure POM.xml](#configure-pomxml)
- [UDF Development](#udf-development)
  - [Writing Java UDF](#writing-java-udf)
  - [Building the Project](#building-the-project)
- [Deployment](#deployment)
  - [Docker Setup](#docker-setup)
  - [MySQL Integration](#mysql-integration)
  - [StarRocks Integration](#starrocks-integration)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- Windows operating system
- Administrator privileges
- Internet connection

## Environment Setup

### Install Chocolatey

1. Open PowerShell as Administrator
2. Run the following command:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

3. Verify installation:

```powershell
choco --version
```

### Install Java 8

1. Use Chocolatey to install JDK 8:

```powershell
choco install jdk8 -y
```

2. Verify Java installation:

```powershell
java -version
```

3. Verify Java compiler:

```powershell
javac -version
```

### Install Maven

1. Use Chocolatey to install Maven:

```powershell
choco install maven -y
```

2. Verify Maven installation:

```powershell
mvn --version
```

## Project Setup

### Create Maven Project

1. Navigate to your desired workspace directory:

```cmd
cd C:\workspace
```

2. Create a parent directory for your project:

```cmd
mkdir your-starrocks-udf
cd your-starrocks-udf
```

3. Generate a Maven project:

```cmd
mvn archetype:generate -DgroupId=com.example -DartifactId=udf -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### Project Structure

Your project structure should look like this:

```
your-starrocks-udf/
└── udf/
    ├── pom.xml
    ├── src/
    │   ├── main/
    │   │   ├── java/
    │   │   │   └── com/
    │   │   │       └── example/
    │   │   │           └── App.java
    │   │   └── resources/
    └── target/    (created after building)
```

### Configure POM.xml

Open `udf/pom.xml` and replace its contents with the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>udf</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>udf</name>
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <!-- Add StarRocks UDF dependencies -->
    <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>starrocks-udf-sdk</artifactId>
      <version>1.0</version>
      <scope>provided</scope>
    </dependency>
    
    <!-- Add other dependencies as needed -->
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
        </configuration>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

Note: You may need to update the StarRocks UDF SDK version based on your requirements.

## UDF Development

### Writing Java UDF

1. Navigate to the source directory:

```cmd
cd udf/src/main/java/com/example/
```

2. Remove the default App.java file:

```cmd
del App.java
```

3. Create a new UDF class (e.g., `StringLengthUDF.java`):

```java
package com.example;

// Import StarRocks UDF SDK classes
import com.starrocks.udf.UDF;

public class StringLengthUDF extends UDF {
    
    public Integer evaluate(String str) {
        if (str == null) {
            return null;
        }
        return str.length();
    }
}
```

### Building the Project

1. Navigate back to the project directory:

```cmd
cd C:\workspace\your-starrocks-udf\udf
```

2. Clean and build the project:

```cmd
mvn clean package
```

3. Verify the JAR file was created:

```cmd
dir target\udf-1.0-SNAPSHOT-jar-with-dependencies.jar
```

After building, your JAR file will be available at: `C:\workspace\your-starrocks-udf\udf\target\udf-1.0-SNAPSHOT-jar-with-dependencies.jar`

## Deployment

### Docker Setup

1. Install Docker Desktop for Windows:

```powershell
choco install docker-desktop -y
```

2. Start Docker Desktop and wait for it to initialize

3. Create a Dockerfile in your project root:

```cmd
cd C:\workspace\your-starrocks-udf
notepad Dockerfile
```

4. Add the following content to the Dockerfile:

```Dockerfile
FROM openjdk:8-jdk-slim

WORKDIR /app

# Copy the JAR file
COPY udf/target/udf-1.0-SNAPSHOT-jar-with-dependencies.jar /app/

# Set environment variables if needed
ENV JAVA_OPTS=""

# Command to run when the container starts
CMD ["java", "-jar", "udf-1.0-SNAPSHOT-jar-with-dependencies.jar"]
```

5. Build the Docker image:

```cmd
docker build -t starrocks-udf:1.0 .
```

6. Run the Docker container:

```cmd
docker run -it --name starrocks-udf starrocks-udf:1.0
```

### MySQL Integration

1. Pull the MySQL Docker image:

```cmd
docker pull mysql:5.7
```

2. Start a MySQL container:

```cmd
docker run --name mysql-db -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 -d mysql:5.7
```

3. Connect to MySQL:

```cmd
docker exec -it mysql-db mysql -uroot -ppassword
```

4. Create a test database:

```sql
CREATE DATABASE testdb;
USE testdb;
```

5. Create a test table:

```sql
CREATE TABLE test_data (
  id INT PRIMARY KEY AUTO_INCREMENT,
  text_data VARCHAR(255)
);
```

6. Insert sample data:

```sql
INSERT INTO test_data (text_data) VALUES 
  ('Hello World'),
  ('StarRocks UDF'),
  ('Testing 123');
```

### StarRocks Integration

1. Pull the StarRocks Docker image:

```cmd
docker pull starrocks/starrocks-server:latest
```

2. Start a StarRocks container:

```cmd
docker run -d --name starrocks-server -p 9030:9030 -p 8040:8040 starrocks/starrocks-server:latest
```

3. Connect to StarRocks:

```cmd
docker exec -it starrocks-server mysql -P9030 -h127.0.0.1 -uroot
```

4. Create a function using your UDF:

```sql
CREATE FUNCTION string_length(VARCHAR) RETURNS INT PROPERTIES (
    "symbol"="com.example.StringLengthUDF",
    "file"="hdfs:///path/to/udf-1.0-SNAPSHOT-jar-with-dependencies.jar",
    "type"="JAVA_UDF"
);
```

Note: Replace `hdfs:///path/to/` with the actual path where you store the JAR file.

5. Test the function:

```sql
SELECT string_length('Hello StarRocks');
```

## Testing

1. Connect to StarRocks:

```cmd
docker exec -it starrocks-server mysql -P9030 -h127.0.0.1 -uroot
```

2. Create a test database and table:

```sql
CREATE DATABASE test_udf;
USE test_udf;

CREATE TABLE test (
  id INT,
  text_col VARCHAR(100)
) ENGINE=OLAP
DISTRIBUTED BY HASH(id) BUCKETS 1
PROPERTIES (
  "replication_num" = "1"
);
```

3. Insert test data:

```sql
INSERT INTO test VALUES (1, 'test'), (2, 'longer test string'), (3, NULL);
```

4. Test your UDF:

```sql
SELECT id, text_col, string_length(text_col) AS length FROM test;
```

## Troubleshooting

### Maven Build Issues

If `mvn clean package` fails:

1. Verify Java installation:
```cmd
java -version
javac -version
```

2. Check Maven settings:
```cmd
mvn -version
```

3. Ensure POM.xml is correctly formatted

### Docker Issues

1. Check Docker service is running:
```cmd
docker ps
```

2. Restart Docker if needed

### UDF Registration Issues

1. Ensure JAR file is accessible to StarRocks
2. Check logs for class loading errors:
```cmd
docker logs starrocks-server
```

## Additional Resources

- [StarRocks Documentation](https://docs.starrocks.io/)
- [Maven Documentation](https://maven.apache.org/guides/index.html)
- [Java UDF Development Guide](https://docs.starrocks.io/docs/developer-guide/java-udf/)
