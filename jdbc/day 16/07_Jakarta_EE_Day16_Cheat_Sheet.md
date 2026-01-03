# Jakarta EE Day 16 - Complete Cheat Sheet

## Quick Reference Guide

---

## 1. Internationalization (I18N)

### Key Classes
| Class | Package | Purpose |
|-------|---------|---------|
| `Locale` | `java.util` | Represents language + country |
| `ResourceBundle` | `java.util` | Load locale-specific properties |
| `DateFormat` | `java.text` | Format dates by locale |

### Locale Codes
| Code | Language | Country |
|------|----------|---------|
| `en_US` | English | United States |
| `de_DE` | German | Germany |
| `fr_FR` | French | France |
| `hi_IN` | Hindi | India |
| `ko_KR` | Korean | Korea |

### Java Standalone I18N
```java
// Load properties for specific locale
ResourceBundle bundle = ResourceBundle.getBundle("message", Locale.US);
String greeting = bundle.getString("greeting");

// Change default locale
Locale.setDefault(new Locale("de", "DE"));
bundle = ResourceBundle.getBundle("message");
```

### Spring Boot I18N
```java
// Main Application
@Bean
public LocaleResolver localeResolver() {
    AcceptHeaderLocaleResolver resolver = new AcceptHeaderLocaleResolver();
    resolver.setDefaultLocale(Locale.ENGLISH);
    return resolver;
}

@Bean
public MessageSource messageSource() {
    ReloadableResourceBundleMessageSource source = new ReloadableResourceBundleMessageSource();
    source.setBasename("classpath:messages");
    source.setDefaultEncoding("UTF-8");
    return source;
}

// Controller
@GetMapping("/greeting")
public String getGreeting(@RequestHeader(name = "Accept-Language", required = false) Locale locale) {
    return messageSource.getMessage("greeting", null, locale);
}
```

### Messages Properties Location
```
src/main/resources/
├── messages.properties         (Default)
├── messages_fr.properties      (French)
├── messages_de.properties      (German)
└── messages_hi.properties      (Hindi)
```

---

## 2. JUnit Testing

### Annotations Comparison
| JUnit 4 | JUnit 5 | Purpose |
|---------|---------|---------|
| `@Test` | `@Test` | Mark test method |
| `@Before` | `@BeforeEach` | Run before each test |
| `@After` | `@AfterEach` | Run after each test |
| `@BeforeClass` | `@BeforeAll` | Run once before all tests |
| `@AfterClass` | `@AfterAll` | Run once after all tests |
| `@Ignore` | `@Disabled` | Skip test |
| `@RunWith` | `@ExtendWith` | Custom runner/extension |

### Assert Methods
```java
// JUnit 4
import static org.junit.Assert.*;
assertEquals(expected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(object);
assertNotNull(object);

// AssertJ (Preferred)
import static org.assertj.core.api.Assertions.assertThat;
assertThat(actual).isEqualTo(expected);
assertThat(result).isTrue();
assertThat(list.size()).isGreaterThan(0);
```

### Exception Testing
```java
// JUnit 4
@Test(expected = ArithmeticException.class)
public void testDivideByZero() {
    int result = 10 / 0;
}

// JUnit 5
@Test
void testDivideByZero() {
    assertThrows(ArithmeticException.class, () -> {
        int result = 10 / 0;
    });
}
```

### Test Suites (JUnit 4)
```java
@RunWith(Suite.class)
@Suite.SuiteClasses({
    TestClass1.class,
    TestClass2.class
})
public class AllTests {}
```

### JUnitCore Programmatic Runner
```java
Result result = JUnitCore.runClasses(TestClass.class);
for (Failure failure : result.getFailures()) {
    System.out.println(failure.toString());
}
System.out.println(result.wasSuccessful());
```

### Maven Dependencies
```xml
<!-- JUnit 4 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

<!-- JUnit 5 -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.3</version>
</dependency>

<!-- Log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

---

## 3. Mockito Framework

### Core Methods
| Method | Purpose |
|--------|---------|
| `mock(Class)` | Create mock object |
| `when(mock.method()).thenReturn(value)` | Stub return value |
| `verify(mock).method()` | Verify method was called |
| `verify(mock, times(n)).method()` | Verify call count |
| `doThrow(ex).when(mock).method()` | Stub exception |

### Basic Mocking
```java
// Create mock
AuthenticatorInterface mockAuth = Mockito.mock(AuthenticatorInterface.class);

// Stub behavior
when(mockAuth.authenticateUser("admin", "pass")).thenReturn(true);

// Use mock
boolean result = mockAuth.authenticateUser("admin", "pass"); // returns true

// Verify
verify(mockAuth).authenticateUser("admin", "pass");
```

### With JUnit 5
```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    
    @Mock
    private Repository repository;
    
    private Service service;
    
    @BeforeEach
    void setUp() {
        service = new Service(repository);
    }
    
    @Test
    void testMethod() {
        service.doSomething();
        verify(repository).save(any());
    }
}
```

### Maven Dependency
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>4.5.1</version>
</dependency>
```

---

## 4. Spring Boot Unit Testing

### Testing Layers
| Layer | Annotation | Mock/Real |
|-------|------------|-----------|
| Repository | `@SpringBootTest` | Real DB |
| Service | `@Mock` + `@ExtendWith(MockitoExtension.class)` | Mock Repository |
| Controller | `@WebMvcTest` | Mock Service |

### Repository Test
```java
@SpringBootTest
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class RepositoryTest {
    
    @Autowired
    private PersonRepository repository;
    
    @Test
    @Order(1)
    void testFindAll() {
        List<Person> list = repository.findAll();
        assertThat(list.size()).isGreaterThan(0);
    }
    
    @BeforeEach
    void setUp() {
        System.out.println("Setting up");
    }
}
```

### Service Test (With Mock)
```java
@SpringBootTest
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    
    @Mock
    private PersonRepository repository;
    
    private PersonService service;
    
    @BeforeEach
    void setUp() {
        service = new PersonService(repository);
    }
    
    @Test
    void testGetAll() {
        service.getAllPersons();
        verify(repository).findAll();
    }
}
```

### application.properties
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/hiber
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## 5. Excel Upload to MySQL

### Apache POI Classes
| Class | Purpose |
|-------|---------|
| `Workbook` | Represents Excel file |
| `XSSFWorkbook` | For .xlsx files |
| `Sheet` | Single sheet in workbook |
| `Row` | Single row in sheet |
| `Cell` | Single cell in row |
| `CellType` | NUMERIC, STRING, BOOLEAN, etc. |

### Reading Excel
```java
@Service
public class ExcelService {
    public List<Student> parseExcel(MultipartFile file) throws Exception {
        List<Student> students = new ArrayList<>();
        
        try (InputStream is = file.getInputStream();
             Workbook workbook = new XSSFWorkbook(is)) {
            
            Sheet sheet = workbook.getSheetAt(0);
            Iterator<Row> rows = sheet.iterator();
            
            if (rows.hasNext()) rows.next(); // Skip header
            
            while (rows.hasNext()) {
                Row row = rows.next();
                Cell nameCell = row.getCell(0);
                Cell ageCell = row.getCell(1);
                
                String name = nameCell.getStringCellValue();
                Integer age = (int) ageCell.getNumericCellValue();
                
                students.add(new Student(name, age));
            }
        }
        return students;
    }
}
```

### Upload Controller
```java
@RestController
@RequestMapping("/api/students")
public class StudentController {
    
    @Autowired
    private ExcelService excelService;
    
    @Autowired
    private StudentUploadService uploadService;
    
    @PostMapping("/upload")
    public ResponseEntity<?> upload(@RequestParam("file") MultipartFile file) {
        try {
            List<Student> students = excelService.parseExcel(file);
            uploadService.saveAll(students);
            return ResponseEntity.ok("Uploaded " + students.size() + " students");
        } catch (Exception e) {
            return ResponseEntity.status(500).body("Error: " + e.getMessage());
        }
    }
}
```

### Batch Save with @Transactional
```java
@Service
public class StudentUploadService {
    
    @Autowired
    private StudentRepository repo;
    
    @Transactional
    public void saveAll(List<Student> students) {
        repo.saveAll(students);
    }
}
```

### Serving Images
```java
@RestController
@RequestMapping("/photos")
public class PhotoController {
    
    private static final String DIR = "D:/photos/";
    
    @GetMapping("/{filename}")
    public ResponseEntity<?> getPhoto(@PathVariable String filename) {
        File file = new File(DIR + filename);
        byte[] image = Files.readAllBytes(file.toPath());
        return ResponseEntity.ok()
            .header("Content-Type", "image/jpeg")
            .body(image);
    }
}
```

### Maven Dependencies
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.3</version>
</dependency>
```

### Postman Upload
```
POST http://localhost:8080/api/students/upload
Body: form-data
  Key: file (Type: File)
  Value: students.xlsx
```

---

## 6. Quick Reference Tables

### All Annotations
| Annotation | Package | Purpose |
|------------|---------|---------|
| `@Test` | JUnit | Mark test method |
| `@BeforeEach` | JUnit 5 | Setup before each test |
| `@AfterEach` | JUnit 5 | Cleanup after each test |
| `@Order(n)` | JUnit 5 | Execution order |
| `@Mock` | Mockito | Create mock object |
| `@ExtendWith` | JUnit 5 | Register extension |
| `@SpringBootTest` | Spring | Load full context |
| `@Autowired` | Spring | Inject dependency |
| `@Service` | Spring | Service bean |
| `@RestController` | Spring | REST controller |
| `@Transactional` | Spring | Transaction boundary |
| `@Entity` | JPA | Database entity |
| `@Id` | JPA | Primary key |
| `@GeneratedValue` | JPA | Auto-generate ID |

### Common Test Patterns
```java
// Arrange
Object expected = ...;
when(mock.method()).thenReturn(expected);

// Act
Object actual = service.method();

// Assert
assertThat(actual).isEqualTo(expected);
verify(mock).method();
```

---

## 7. Interview Questions Quick Answers

### I18N
1. **I18N vs L10N**: I18N = designing for multiple languages; L10N = adapting to specific locale
2. **ResourceBundle**: Loads locale-specific properties files

### JUnit
1. **@Before vs @BeforeEach**: JUnit 4 vs JUnit 5 (same purpose)
2. **JUnitCore**: Programmatic test runner (Facade pattern)

### Mockito
1. **Mock vs Spy**: Mock = all fake; Spy = partial mock
2. **when().thenReturn()**: Defines mock behavior
3. **verify()**: Checks method was called

### Spring Boot Testing
1. **@SpringBootTest vs @DataJpaTest**: Full context vs JPA-only
2. **Why constructor injection**: Enables mock injection

### Excel Upload
1. **MultipartFile**: Spring interface for uploaded files
2. **@Transactional**: All saves in single transaction

---

**End of Cheat Sheet**
