# Spring Boot Excel Upload to MySQL

## Table of Contents
1. [Introduction](#introduction)
2. [MultipartFile Interface](#multipartfile-interface)
3. [Project Setup](#project-setup)
4. [Basic Excel Upload](#basic-excel-upload)
5. [Excel Upload with Photo Path](#excel-upload-with-photo-path)
6. [Serving Images from Server](#serving-images-from-server)
7. [Consuming List APIs with RestTemplate](#consuming-list-apis-with-resttemplate)
8. [Execution Flow](#execution-flow)
9. [Interview Questions](#interview-questions)

---

## Introduction

### What We'll Build

This guide covers building a **Spring Boot application that uploads Excel files and stores data in MySQL**. We'll learn:

1. Uploading Excel (`.xlsx`) files via REST API
2. Parsing Excel using Apache POI library
3. Storing parsed data in MySQL database
4. Handling photo file paths in Excel
5. Serving images from the server

---

## MultipartFile Interface

### What is MultipartFile?

**Definition:** In Java (especially in Spring/Spring Boot web applications), `MultipartFile` is used to handle **file uploads** sent from a client (browser, Postman, frontend app) using `multipart/form-data` requests.

> MultipartFile is an interface provided by Spring Framework (`org.springframework.web.multipart.MultipartFile`) that represents an uploaded file received in an HTTP request.
>
> When a user uploads a file via a form or API, **Spring automatically converts that file into a MultipartFile object**.

### Key Methods

| Method | Description |
|--------|-------------|
| `getInputStream()` | Get file content as InputStream |
| `getOriginalFilename()` | Get uploaded file's name |
| `getSize()` | Get file size in bytes |
| `getBytes()` | Get file content as byte array |
| `isEmpty()` | Check if file is empty |
| `getContentType()` | Get MIME type (e.g., application/vnd.ms-excel) |

---

## Project Setup

### Dependencies (Spring Initializr)

```
✓ Spring Boot DevTools
✓ Spring Data JPA
✓ MySQL Driver
✓ Spring Web
```

### Additional Dependency: Apache POI

Add to `pom.xml`:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.3</version>
</dependency>
```

> This is required for reading `.xlsx` Excel files.

### application.properties

```properties
spring.application.name=Excel_To_MySQL

spring.datasource.url=jdbc:mysql://localhost:3306/hiber
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=update
```

### Project Structure
```
src/main/java/com/example/demo/
├── ExcelToMySqlApplication.java    (Main class)
├── Student.java                     (Entity)
├── StudentRepository.java           (Repository)
├── ExcelService.java                (Excel parsing)
├── StudentUploadService.java        (Database operations)
└── StudentController.java           (REST controller)
```

---

## Basic Excel Upload

### Student.java (Entity)

```java
package com.example.demo;

import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)   // MySQL Auto Increment
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

    public Student() {
    }

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `@Entity` | JPA entity (maps to database table) |
| 6 | `@Table(name = "students")` | Table name in database |
| 9-10 | `@Id @GeneratedValue(strategy = GenerationType.IDENTITY)` | Auto-increment primary key |
| 13-14 | `@Column(nullable = false)` | Column cannot be null |
| 19-20 | Default constructor | Required by JPA |
| 22-25 | Parameterized constructor | For creating new students |

### StudentRepository.java

```java
package com.example.demo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

### ExcelService.java (Excel Parsing)

```java
package com.example.demo;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

@Service
public class ExcelService {

    public List<Student> parseExcel(MultipartFile file) throws Exception {
        List<Student> students = new ArrayList<>();

        try (InputStream is = file.getInputStream();
             Workbook workbook = new XSSFWorkbook(is)) {

            Sheet sheet = workbook.getSheetAt(0);
            Iterator<Row> rows = sheet.iterator();

            if (rows.hasNext()) rows.next(); // skip header row

            while (rows.hasNext()) {
                Row row = rows.next();

                Cell nameCell = row.getCell(0);
                Cell ageCell = row.getCell(1);

                String name = nameCell != null ? nameCell.getStringCellValue() : null;
                Integer age = null;

                if (ageCell != null) {
                    if (ageCell.getCellType() == CellType.NUMERIC) {
                        age = (int) ageCell.getNumericCellValue();
                    } else {
                        age = Integer.parseInt(ageCell.getStringCellValue());
                    }
                }

                if (name != null && !name.isBlank()) {
                    students.add(new Student(name, age));
                }
            }
        }

        return students;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3-4 | POI imports | Apache POI classes for Excel |
| 13 | `@Service` | Spring service bean |
| 16 | `parseExcel(MultipartFile file)` | Parses uploaded Excel file |
| 17 | `List<Student> students` | Result list |
| 19-20 | `try-with-resources` | Auto-closes InputStream and Workbook |
| 20 | `new XSSFWorkbook(is)` | Creates workbook from .xlsx file |
| 22 | `workbook.getSheetAt(0)` | Gets first sheet |
| 23 | `sheet.iterator()` | Iterator over all rows |
| 25 | `if (rows.hasNext()) rows.next()` | **Skip header row** |
| 30-31 | `row.getCell(0)`, `row.getCell(1)` | Get cells by index (0=Name, 1=Age) |
| 33 | `nameCell.getStringCellValue()` | Read string from cell |
| 36-41 | Age handling | Handle NUMERIC vs TEXT cell types |

### Why Check Cell Type for Age?

```
┌─────────────────────────────────────────────────────────────────────┐
│                 EXCEL CELL TYPE HANDLING                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Excel does NOT enforce data types!                                  │
│                                                                      │
│  Age stored as NUMBER:           Age stored as TEXT:                 │
│  ┌─────┐                         ┌─────┐                             │
│  │ 25  │ (Numeric cell)          │"25" │ (String cell)              │
│  └─────┘                         └─────┘                             │
│     ↓                               ↓                                │
│  CellType.NUMERIC               CellType.STRING                      │
│     ↓                               ↓                                │
│  getNumericCellValue()          getStringCellValue()                 │
│  Returns: 25.0 (double)          Returns: "25"                       │
│     ↓                               ↓                                │
│  Cast to int: (int)25.0          Integer.parseInt("25")              │
│  Result: 25                       Result: 25                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### StudentUploadService.java

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class StudentUploadService 
{
    @Autowired
    private StudentRepository repo;

    // @Transactional is needed here, otherwise individual save will happen 
    // in different transactions which is not recommended.

    @Transactional
    public void saveAll(List<Student> students) {
        repo.saveAll(students);   // Hibernate batching will apply
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 9 | `@Service` | Spring service bean |
| 12-13 | `@Autowired StudentRepository` | Inject repository |
| 18 | `@Transactional` | Wrap in single database transaction |
| 19-20 | `saveAll(List<Student>)` | Saves all students in batch |

### Why @Transactional?

Without `@Transactional`, each `save()` operation would be a **separate transaction**. With `@Transactional`:
- All operations in single transaction
- Either all succeed or all rollback
- Better performance with Hibernate batching

### StudentController.java

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.util.List;

@RestController
@RequestMapping("/api/students")
public class StudentController 
{
    @Autowired
    private ExcelService excelService;
    
    @Autowired
    private StudentUploadService uploadService;

    @PostMapping("/upload")
    public ResponseEntity<?> uploadExcel(@RequestParam("file") MultipartFile file) {
        try {
            List<Student> students = excelService.parseExcel(file);

            uploadService.saveAll(students);

            return ResponseEntity.ok("Uploaded " + students.size() + " students successfully.");
        } catch (Exception ex) {
            ex.printStackTrace();
            return ResponseEntity.status(500).body("Error: " + ex.getMessage());
        }
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 9 | `@RestController` | REST controller |
| 10 | `@RequestMapping("/api/students")` | Base path for all endpoints |
| 19 | `@PostMapping("/upload")` | POST endpoint |
| 20 | `@RequestParam("file") MultipartFile file` | Extract file from form-data |
| 22 | `excelService.parseExcel(file)` | Parse Excel to Student list |
| 24 | `uploadService.saveAll(students)` | Save to database |
| 26 | `ResponseEntity.ok(...)` | 200 OK response |
| 29 | `ResponseEntity.status(500)` | 500 Internal Server Error |

### Testing with Postman

**Steps to upload a file using Postman (multipart/form-data):**

1. **Open Postman**

2. **Select POST method**

3. **Enter URL:**
   ```
   http://localhost:8080/api/students/upload
   ```

4. **Click on Body**

5. **Select form-data**

6. **In the key-value table:**
   - Under Key, type: `file`
   - In that row, under Type, change from **Text** to **File**
   - In the Value column, attach your file: `students.xlsx`

7. **Click Send**

Postman will automatically send:
```
Content-Type: multipart/form-data
```

### Expected Response
```
Uploaded 5 students successfully.
```

---

## Excel Upload with Photo Path

### Extended Student Entity

```java
package com.example.demo;

import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

    @Column(nullable = true)
    private String photoPath;

    public Student() {
    }

    public Student(String name, Integer age, String photoPath) {
        this.name = name;
        this.age = age;
        this.photoPath = photoPath;
    }

    public String getPhotoPath() {
        return photoPath;
    }

    public void setPhotoPath(String photoPath) {
        this.photoPath = photoPath;
    }

    // ... other getters/setters
}
```

### Extended ExcelService

```java
package com.example.demo;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

@Service
public class ExcelService {

    public List<Student> parseExcel(MultipartFile file) throws Exception {
        List<Student> students = new ArrayList<>();

        try (InputStream is = file.getInputStream();
             Workbook workbook = new XSSFWorkbook(is)) {

            Sheet sheet = workbook.getSheetAt(0);
            Iterator<Row> rows = sheet.iterator();

            if (rows.hasNext()) rows.next(); // skip header row

            while (rows.hasNext()) {
                Row row = rows.next();

                Cell nameCell = row.getCell(0);
                Cell ageCell = row.getCell(1);
                Cell photoCell = row.getCell(2);
                
                String photoFileName = photoCell != null ? photoCell.getStringCellValue() : null;
                String basePhotoFolder = "D:/student_photos/";

                String fullPhotoPath = null;
                if (photoFileName != null && !photoFileName.isBlank()) {
                    fullPhotoPath = basePhotoFolder + photoFileName;
                }
                
                String name = nameCell != null ? nameCell.getStringCellValue() : null;
                Integer age = null;

                if (ageCell != null) {
                    if (ageCell.getCellType() == CellType.NUMERIC) {
                        age = (int) ageCell.getNumericCellValue();
                    } else {
                        age = Integer.parseInt(ageCell.getStringCellValue());
                    }
                }

                if (name != null && !name.isBlank()) {
                    students.add(new Student(name, age, fullPhotoPath));
                }
            }
        }

        return students;
    }
}
```

#### Key Additions

| Line | Code | Explanation |
|------|------|-------------|
| 32 | `Cell photoCell = row.getCell(2)` | Third column for photo filename |
| 34 | `photoCell.getStringCellValue()` | Get filename (e.g., "john.jpg") |
| 35 | `basePhotoFolder = "D:/student_photos/"` | Photo storage directory |
| 38-40 | Build full path | `D:/student_photos/john.jpg` |
| 54 | `new Student(name, age, fullPhotoPath)` | Include photo path in entity |

### Excel File Format
| Name | Age | Photo |
|------|-----|-------|
| John | 25 | john.jpg |
| Alice | 22 | alice.jpg |

---

## Serving Images from Server

### PhotoController.java

```java
package com.example.demo;

import java.io.File;
import java.nio.file.Files;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/photos")
public class PhotoController {

    private static final String PHOTO_DIR = "D:/student_photos/";

    @GetMapping("/{filename}")
    public ResponseEntity<?> getPhoto(@PathVariable String filename) {
        try {
            File file = new File(PHOTO_DIR + filename);
            if (!file.exists()) {
                return ResponseEntity.notFound().build();
            }

            byte[] image = Files.readAllBytes(file.toPath());
            return ResponseEntity.ok()
                    .header("Content-Type", "image/jpeg")
                    .body(image);

        } catch (Exception e) {
            return ResponseEntity.status(500).body("Error: " + e.getMessage());
        }
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 16 | `PHOTO_DIR = "D:/student_photos/"` | Directory where photos are stored |
| 18 | `@GetMapping("/{filename}")` | URL pattern: /photos/john.jpg |
| 19 | `@PathVariable String filename` | Extract filename from URL |
| 21 | `new File(PHOTO_DIR + filename)` | Build file path |
| 22-24 | `if (!file.exists())` | Check file exists |
| 26 | `Files.readAllBytes(file.toPath())` | Read entire file as byte array |
| 27-29 | `ResponseEntity.ok()` | Return image with proper headers |
| 28 | `.header("Content-Type", "image/jpeg")` | Set MIME type for images |

### How to Access Photos
```
GET http://localhost:8080/photos/john.jpg
```

Returns the actual image that can be displayed in browser or frontend.

---

## Consuming List APIs with RestTemplate

### How to Read List from Other API

```java
ResponseEntity<List<Student>> response =
        template.exchange(
                url,
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<Student>>() {}
        );

List<Student> students = response.getBody();
```

#### Explanation

| Component | Explanation |
|-----------|-------------|
| `template` | RestTemplate instance |
| `exchange()` | Generic HTTP method |
| `url` | API endpoint URL |
| `HttpMethod.GET` | HTTP GET method |
| `null` | No request body |
| `ParameterizedTypeReference<List<Student>>` | Preserves generic type at runtime |

### Why ParameterizedTypeReference?

Java type erasure removes generic information at runtime. `ParameterizedTypeReference` preserves the `List<Student>` type so Jackson can deserialize correctly.

---

## Execution Flow

### Complete Upload Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EXCEL UPLOAD EXECUTION FLOW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Client sends POST /api/students/upload                           │
│       ├─ Content-Type: multipart/form-data                           │
│       └─ Body: students.xlsx file                                    │
│                                                                      │
│  2. Spring converts to MultipartFile                                 │
│       └─ StudentController.uploadExcel(MultipartFile file)          │
│                                                                      │
│  3. ExcelService.parseExcel(file)                                    │
│       ├─ file.getInputStream() → Get file bytes                      │
│       ├─ new XSSFWorkbook(is) → Parse as Excel                       │
│       ├─ workbook.getSheetAt(0) → Get first sheet                    │
│       ├─ Skip header row                                             │
│       ├─ For each row:                                               │
│       │     ├─ Read cell 0 → name                                    │
│       │     ├─ Read cell 1 → age (handle NUMERIC/STRING)             │
│       │     ├─ Read cell 2 → photoFileName (optional)                │
│       │     └─ Create Student object                                 │
│       └─ Return List<Student>                                        │
│                                                                      │
│  4. StudentUploadService.saveAll(students)                           │
│       ├─ @Transactional begins                                       │
│       ├─ repo.saveAll(students)                                      │
│       │     └─ Hibernate batch insert                                │
│       └─ @Transactional commits                                      │
│                                                                      │
│  5. Return Response                                                  │
│       └─ "Uploaded 5 students successfully."                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Photo Serving Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHOTO SERVING FLOW                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Client sends GET /photos/john.jpg                                │
│                                                                      │
│  2. PhotoController.getPhoto("john.jpg")                             │
│       ├─ Build path: D:/student_photos/john.jpg                      │
│       ├─ Check file exists                                           │
│       ├─ Files.readAllBytes() → byte[]                               │
│       └─ Return with Content-Type: image/jpeg                        │
│                                                                      │
│  3. Client receives image bytes                                      │
│       └─ Displays image in browser/app                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q1: What is MultipartFile in Spring Boot?
**Answer:** MultipartFile is an interface representing uploaded files in HTTP requests. Spring automatically converts files from `multipart/form-data` requests into MultipartFile objects.

### Q2: Why use Apache POI instead of reading Excel directly?
**Answer:** Excel files (.xlsx) are complex ZIP archives with XML content. Apache POI provides an API to read/write Excel files without dealing with this complexity.

### Q3: Why is @Transactional important for saveAll()?
**Answer:** Without @Transactional, each save would be a separate transaction. With @Transactional:
- All operations in single transaction
- Rollback if any fails
- Better performance (batch operations)

### Q4: How do you handle different cell types in Excel?
**Answer:** Check `cell.getCellType()`:
- `CellType.NUMERIC` → use `getNumericCellValue()`
- `CellType.STRING` → use `getStringCellValue()`
- `CellType.BOOLEAN` → use `getBooleanCellValue()`

### Q5: What's the difference between poi and poi-ooxml?
**Answer:**
- `poi`: For older .xls files (HSSF)
- `poi-ooxml`: For newer .xlsx files (XSSF)

---

## Summary

| Component | Purpose |
|-----------|---------|
| **MultipartFile** | Handle uploaded files |
| **Apache POI** | Parse Excel files |
| **XSSFWorkbook** | .xlsx file handler |
| **Sheet, Row, Cell** | Excel structure |
| **@Transactional** | Batch database operations |
| **@RequestParam("file")** | Receive uploaded file |
| **Files.readAllBytes()** | Serve files as bytes |
| **Content-Type: image/jpeg** | MIME type for images |

| Key Classes | Description |
|-------------|-------------|
| `ExcelService` | Parses Excel, creates entities |
| `StudentUploadService` | Saves to database with @Transactional |
| `StudentController` | REST endpoint for upload |
| `PhotoController` | Serves photos from filesystem |

---

**Previous:** [05_Spring_Boot_Unit_Testing_Complete.md](./05_Spring_Boot_Unit_Testing_Complete.md)  
**Next:** [07_Jakarta_EE_Day16_Cheat_Sheet.md](./07_Jakarta_EE_Day16_Cheat_Sheet.md)
