# JSON Maker UDF Development Guide

This guide walks through creating a Java UDF that generates JSON output from input parameters.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Project Setup](#project-setup)
- [JSON UDF Implementation](#json-udf-implementation)
- [Build and Deploy](#build-and-deploy)
- [Testing](#testing)

## Prerequisites
- Windows operating system
- Administrator privileges
- Internet connection

## Environment Setup

### Install Chocolatey
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### Install Java 8
```powershell
choco install jdk8 -y
```

### Install Maven
```powershell
choco install maven -y
```

## Project Setup

### Create Project Structure
```cmd
mkdir your-json-udf
cd your-json-udf
mvn archetype:generate -DgroupId=com.example -DartifactId=json-udf -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### Configure POM.xml
Navigate to `json-udf/pom.xml` and replace with:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>json-udf</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <!-- StarRocks UDF SDK -->
    <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>starrocks-udf-sdk</artifactId>
      <version>1.0</version>
      <scope>provided</scope>
    </dependency>
    
    <!-- JSON processing -->
    <dependency>
      <groupId>org.json</groupId>
      <artifactId>json</artifactId>
      <version>20230227</version>
    </dependency>
    
    <!-- Jackson for JSON -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.14.2</version>
    </dependency>
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

## JSON UDF Implementation

### Create the UDF Class
Navigate to `json-udf/src/main/java/com/example/` and create `JsonMakerUDF.java`:

```java
package com.example;

import com.starrocks.udf.UDF;
import org.json.JSONObject;

public class JsonMakerUDF extends UDF {
    
    /**
     * Creates a JSON object with key-value pairs.
     * Usage: json_make(key1, value1, key2, value2, ...)
     */
    public String evaluate(String... args) {
        if (args == null || args.length < 2 || args.length % 2 != 0) {
            return "{}"; // Return empty JSON if arguments are invalid
        }
        
        JSONObject jsonObject = new JSONObject();
        
        for (int i = 0; i < args.length; i += 2) {
            String key = args[i];
            String value = args[i + 1];
            
            if (key != null && value != null) {
                jsonObject.put(key, value);
            }
        }
        
        return jsonObject.toString();
    }
    
    /**
     * Specialized version for different data types
     */
    public String evaluate(String key1, String value1) {
        JSONObject jsonObject = new JSONObject();
        if (key1 != null && value1 != null) {
            jsonObject.put(key1, value1);
        }
        return jsonObject.toString();
    }
    
    public String evaluate(String key1, String value1, String key2, String value2) {
        JSONObject jsonObject = new JSONObject();
        if (key1 != null && value1 != null) {
            jsonObject.put(key1, value1);
        }
        if (key2 != null && value2 != null) {
            jsonObject.put(key2, value2);
        }
        return jsonObject.toString();
    }
    
    public String evaluate(String key1, String value1, String key2, String value2, 
                          String key3, String value3) {
        JSONObject jsonObject = new JSONObject();
        if (key1 != null && value1 != null) {
            jsonObject.put(key1, value1);
        }
        if (key2 != null && value2 != null) {
            jsonObject.put(key2, value2);
        }
        if (key3 != null && value3 != null) {
            jsonObject.put(key3, value3);
        }
        return jsonObject.toString();
    }
}
```

### Create a Nested JSON UDF (Optional)
Create `NestedJsonUDF.java`:

```java
package com.example;

import com.starrocks.udf.UDF;
import org.json.JSONObject;

public class NestedJsonUDF extends UDF {
    
    /**
     * Creates a nested JSON object.
     * Usage: nested_json(json1, json2, ...)
     */
    public String evaluate(String... jsonStrings) {
        if (jsonStrings == null || jsonStrings.length == 0) {
            return "{}";
        }
        
        JSONObject result = new JSONObject();
        
        for (int i = 0; i < jsonStrings.length; i++) {
            String jsonStr = jsonStrings[i];
            if (jsonStr != null && isValidJson(jsonStr)) {
                JSONObject json = new JSONObject(jsonStr);
                result.put("level" + (i+1), json);
            }
        }
        
        return result.toString();
    }
    
    private boolean isValidJson(String json) {
        try {
            new JSONObject(json);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

## Build and Deploy

### Build the Project
```cmd
cd json-udf
mvn clean package
```

Your JAR file will be available at: `json-udf/target/json-udf-1.0-SNAPSHOT-jar-with-dependencies.jar`

### Register UDF in StarRocks

1. Connect to StarRocks:
```cmd
mysql -P9030 -h127.0.0.1 -uroot
```

2. Create the function:
```sql
CREATE FUNCTION json_make(VARCHAR, VARCHAR, ...) RETURNS VARCHAR PROPERTIES (
    "symbol"="com.example.JsonMakerUDF",
    "file"="hdfs:///path/to/json-udf-1.0-SNAPSHOT-jar-with-dependencies.jar",
    "type"="JAVA_UDF"
);

CREATE FUNCTION nested_json(VARCHAR, VARCHAR, ...) RETURNS VARCHAR PROPERTIES (
    "symbol"="com.example.NestedJsonUDF",
    "file"="hdfs:///path/to/json-udf-1.0-SNAPSHOT-jar-with-dependencies.jar",
    "type"="JAVA_UDF"
);
```

## Testing

### Basic JSON Creation
```sql
SELECT json_make('name', 'John', 'age', '30');
```
Expected result: `{"name":"John","age":"30"}`

### Nested JSON
```sql
SELECT nested_json(
    json_make('firstName', 'John', 'lastName', 'Doe'),
    json_make('city', 'New York', 'state', 'NY')
);
```
Expected result: `{"level1":{"firstName":"John","lastName":"Doe"},"level2":{"city":"New York","state":"NY"}}`

### Complex Example
```sql
CREATE TABLE user_data (
  id INT,
  name VARCHAR(100),
  age INT,
  city VARCHAR(100)
) ENGINE=OLAP
DISTRIBUTED BY HASH(id) BUCKETS 1
PROPERTIES ("replication_num" = "1");

INSERT INTO user_data VALUES 
  (1, 'John', 25, 'New York'),
  (2, 'Alice', 30, 'Boston'),
  (3, 'Bob', 22, 'Chicago');

SELECT 
  id,
  json_make('name', name, 'age', CAST(age AS VARCHAR), 'city', city) AS user_json
FROM user_data;
```

## Optimizations and Extensions

1. **Add type support**:
   Implement additional methods for different data types (INT, DOUBLE, etc.)

2. **Add array support**:
   Create a separate UDF to handle array creation

3. **Improve error handling**:
   Add more robust validation and error messages

4. **Performance tuning**:
   Optimize for large datasets by minimizing object creation
