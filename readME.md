# Generic JSON UDF Development Guide

This guide walks through creating a Java UDF for StarRocks that converts string key-value pairs into nested JSON.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Project Setup](#project-setup)
- [UDF Implementation](#udf-implementation)
- [Build Process](#build-process)
- [Deployment](#deployment)
- [Testing](#testing)
- [Test Results](#test-results)

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
mkdir your-starrocks-udf
cd your-starrocks-udf
mvn archetype:generate -DgroupId=com.example -DartifactId=udf -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Your project structure will look like:
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
Replace the contents of `udf/pom.xml` with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
<artifactId>udf</artifactId>
<version>1.0-SNAPSHOT</version>

    <properties>
<maven.compiler.source>8</maven.compiler.source>
<maven.compiler.target>8</maven.compiler.target>
</properties>

 <dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.76</version>
    </dependency>
   
    <!-- Add this dependency -->
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>3.1.2</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

    <build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-dependency-plugin</artifactId>
<version>2.10</version>
<executions>
<execution>
<id>copy-dependencies</id>
<phase>package</phase>
<goals>
<goal>copy-dependencies</goal>
</goals>
<configuration>
<outputDirectory>${project.build.directory}/lib</outputDirectory>
</configuration>
</execution>
</executions>
</plugin>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-assembly-plugin</artifactId>
<version>3.3.0</version>
<executions>
<execution>
<id>make-assembly</id>
<phase>package</phase>
<goals>
<goal>single</goal>
</goals>
</execution>
</executions>
<configuration>
<descriptorRefs>
<descriptorRef>jar-with-dependencies</descriptorRef>
</descriptorRefs>
</configuration>
</plugin>
</plugins>
</build>
</project>
```

## UDF Implementation

### Create the UDF Class
Navigate to `udf/src/main/java/com/example/` and delete the existing `App.java` file.

Create a new file `newGenericToJsonUDF.java` with the following content:

```java
package com.example;
 
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;
 
public class newGenericToJsonUDF {
    public String evaluate(String keyValueString) {
        if (keyValueString == null || keyValueString.trim().isEmpty()) {
            return "{}";
        }
 
        // Use ORDERED feature to preserve key order
        JSONObject json = new JSONObject(true); // true enables LinkedHashMap internally
        String[] pairs = keyValueString.split(",");
        for (String pair : pairs) {
            if (pair == null || pair.trim().isEmpty()) {
                continue;
            }
            String[] keyValue = pair.split("=", 2);
            if (keyValue.length == 2) {
                String key = keyValue[0].trim();
                String value = keyValue[1].trim();
                if (key.isEmpty()) {
                    continue;
                }
 
                // Handle nested keys (e.g., Result1.YearNumber_CY)
                String[] keyParts = key.split("\\.");
                JSONObject current = json;
                for (int i = 0; i < keyParts.length - 1; i++) {
                    String part = keyParts[i];
                    if (!current.containsKey(part)) {
                        // Create nested object with order preservation
                        current.put(part, new JSONObject(true));
                    }
                    current = (JSONObject) current.get(part);
                }
                current.put(keyParts[keyParts.length - 1], value == null ? "" : value);
            }
        }
        // Serialize with PrettyFormat for readability
        return json.toJSONString();
    }
}
```

## Build Process

### Compile and Package
1. Navigate to your project directory:
```cmd
cd your-starrocks-udf/udf
```

2. Build the project:
```cmd
mvn clean package
```

3. Verify the JAR file:
```cmd
dir target\udf-1.0-SNAPSHOT-jar-with-dependencies.jar
```

## Deployment

### Setup HTTP Server for JAR Files
1. Create a simple HTTP server to serve the JAR file:
```cmd
python -m http.server 8000
```
Or for Python 2:
```cmd
python -m SimpleHTTPServer 8000
```

2. Make sure your JAR file is in the directory where the HTTP server is running.

### Register UDF in StarRocks
Connect to your StarRocks instance and run:

```sql
CREATE FUNCTION generic_to_json(VARCHAR(65533)) RETURNS VARCHAR(65533) 
PROPERTIES (
    "symbol" = "com.example.newGenericToJsonUDF",
    "type" = "StarrocksJar",
    "file" = "http://192.168.1.76:8000/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

Note: Replace `192.168.1.76` with your actual server IP address.

## Testing

### Basic Testing
```sql
SELECT generic_to_json('key1=value1,key2=value2');
```

Expected output:
```json
{"key1":"value1","key2":"value2"}
```

### Nested JSON Testing
```sql
SELECT generic_to_json('Result1.YearNumber_CY=2025,Result1.FullDate_CY=2025-03-18,Result2.StoreNumber=1027');
```

Expected output:
```json
{"Result1":{"YearNumber_CY":"2025","FullDate_CY":"2025-03-18"},"Result2":{"StoreNumber":"1027"}}
```

### Advanced Testing with CTEs
```sql
WITH 
cte1 AS (
    SELECT StoreNumber FROM vwStoresActive WHERE StoreNumber = 1027
),
cte2 AS (
    SELECT YearNumber_CY, FullDate_CY FROM vwCalendar WHERE FullDate_CY = '2025-03-18'
)
SELECT 
    JSON_PARSE(generic_to_json(
        CONCAT_WS(',', 
            CONCAT('Result1.YearNumber_CY=', YearNumber_CY),
            CONCAT('Result1.FullDate_CY=', DATE_FORMAT(FullDate_CY, '%Y-%m-%d')),
            CONCAT('Result2.StoreNumber=', StoreNumber)
        )
    )) AS combined_result
FROM cte1, cte2;
```

## Test Results

### CTE Test Case Results

The CTE test query returns a parsed JSON object with the following structure:

```json
{
  "Result1": {
    "YearNumber_CY": "2025",
    "FullDate_CY": "2025-03-18"
  },
  "Result2": {
    "StoreNumber": "1027"
  }
}
```

This demonstrates:
1. Nested JSON objects are properly created
2. Data from different CTEs is correctly combined
3. Dot notation in keys creates proper object hierarchy
4. The UDF works correctly with database views and functions

These results validate that our UDF works correctly for combining data from multiple sources into a structured JSON object that preserves nested relationships.

## Troubleshooting

### UDF Not Found
If StarRocks cannot find the UDF, check:
1. The JAR file is accessible via the HTTP URL
2. The package and class name match exactly in the CREATE FUNCTION statement
3. The HTTP server is running and accessible from StarRocks nodes

### Syntax Errors
If queries using the UDF return syntax errors:
1. Verify the input format matches what the UDF expects
2. Check for any special characters in input strings
3. Make sure the SQL query properly escapes quotes

### Performance Issues
If the UDF is slow:
1. Consider limiting the size of input strings
2. Use targeted CTEs to reduce data volume
3. Consider implementing a more efficient parsing algorithm for very large inputs

## Additional Resources
- [StarRocks Documentation](https://docs.starrocks.io/)
- [FastJSON Documentation](https://github.com/alibaba/fastjson/wiki)
- [Maven Documentation](https://maven.apache.org/guides/index.html)
