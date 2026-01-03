# Internationalization (I18N) Complete Guide

## Table of Contents
1. [Introduction to Internationalization](#introduction-to-internationalization)
2. [Locale Class](#locale-class)
3. [ResourceBundle Class](#resourcebundle-class)
4. [Standalone Java I18N Application](#standalone-java-i18n-application)
5. [Spring Boot I18N Implementation](#spring-boot-i18n-implementation)
6. [Execution Flow](#execution-flow)
7. [Interview Questions](#interview-questions)

---

## Introduction to Internationalization

### What is Internationalization (I18N)?

**Definition:** Internationalization (abbreviated as I18N because there are 18 letters between 'I' and 'N') is the process of designing and developing software applications that can be adapted to different languages, regions, and cultures without requiring engineering changes.

### Purpose
- Display messages, dates, numbers, and currencies in user's preferred language
- Support multiple locales without code modification
- Separate language-specific content from application logic

### Key Terminology
| Term | Description |
|------|-------------|
| **I18N** | Internationalization - designing for multiple languages |
| **L10N** | Localization - adapting to a specific locale |
| **Locale** | Represents a specific geographical, political, or cultural region |

---

## Locale Class

### What is Locale Class?

**Definition:** Locale class represents Language and Country. It is found in `java.util` package.

### Structure of Locale Class

A Locale object is typically constructed using two main components:

| Component | Description | Examples |
|-----------|-------------|----------|
| **Language** | A two-letter code representing the language | `en` (English), `fr` (French), `de` (German), `hi` (Hindi) |
| **Country** | A two-letter code representing the country/region | `US` (United States), `FR` (France), `DE` (Germany), `IN` (India) |

### Code Example: Displaying All Available Locales

```java
import java.util.*;
import java.text.*;

public class DisplayAllLocales
{
    public static void main(String args[])
    {
        // Get array of all available locales
        Locale locales[] = DateFormat.getAvailableLocales();
        
        // Iterate and print each locale's details
        for(Locale loc : locales)
        {
            System.out.println(
                loc.getDisplayName() + "\t" +   // Human-readable name (e.g., "English (United States)")
                loc.getLanguage() + "\t" +       // Language code (e.g., "en")
                loc.getCountry()                 // Country code (e.g., "US")
            );
        }
    }
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 1 | `import java.util.*;` | Imports Locale class from java.util package |
| 2 | `import java.text.*;` | Imports DateFormat class for getting available locales |
| 5-6 | `public static void main(...)` | Program entry point |
| 8 | `Locale locales[] = DateFormat.getAvailableLocales();` | Gets array of all locales supported by JVM |
| 9-12 | `for(Locale loc : locales)` | Enhanced for loop to iterate each locale |
| 11 | `loc.getDisplayName()` | Returns human-readable locale name |
| 11 | `loc.getLanguage()` | Returns ISO 639 language code (2 letters) |
| 11 | `loc.getCountry()` | Returns ISO 3166 country code (2 letters) |

### Execution Flow
```
┌─────────────────────────────────────────────────────────┐
│ Program Start                                           │
├─────────────────────────────────────────────────────────┤
│ 1. DateFormat.getAvailableLocales() called              │
│    ├─ JVM queries all installed locale providers        │
│    └─ Returns Locale[] array with 100+ locales          │
├─────────────────────────────────────────────────────────┤
│ 2. For loop iterates each Locale object                 │
│    ├─ getDisplayName() → "English (United States)"      │
│    ├─ getLanguage() → "en"                              │
│    └─ getCountry() → "US"                               │
├─────────────────────────────────────────────────────────┤
│ 3. Prints: "English (United States)  en  US"            │
└─────────────────────────────────────────────────────────┘
```

---

## ResourceBundle Class

### What is ResourceBundle Class?

**Definition:** ResourceBundle class in Java is used to store locale-specific objects such as strings, numbers, and other objects. It provides a way to separate locale-specific data from the main application code.

### How ResourceBundle Works

```
┌─────────────────────────────────────────────────────────┐
│           ResourceBundle.getBundle("message", Locale)   │
├─────────────────────────────────────────────────────────┤
│  Locale = en_US  →  Searches for:                       │
│    1. message_en_US.properties  (exact match)           │
│    2. message_en.properties     (language match)        │
│    3. message.properties        (default)               │
└─────────────────────────────────────────────────────────┘
```

### Key Methods
| Method | Description |
|--------|-------------|
| `getBundle(String baseName)` | Loads bundle for default locale |
| `getBundle(String baseName, Locale locale)` | Loads bundle for specified locale |
| `getString(String key)` | Returns value for the given key |

---

## Standalone Java I18N Application

### Project Structure
```
src/
├── InternationalizationDemo.java
├── message_en_US.properties
├── message_de_DE.properties
├── message_in_ID.properties
└── message_da.properties
```

### Properties Files

#### message_en_US.properties (English - United States)
```properties
greeting=Hello, how are you?
```

#### message_de_DE.properties (German - Germany)
```properties
greeting=Hallo wie geht es dir
```

#### message_in_ID.properties (Indonesian)
```properties
greeting=Halo apa kabar?
```

#### message_da.properties (Danish)
```properties
greeting=Hej, hvordan går det?
```

### Main Application Code

```java
import java.util.Locale;  
import java.util.ResourceBundle;  

public class InternationalizationDemo {  
    public static void main(String[] args) {  
   
        // ========== ENGLISH (US) ==========
        System.out.println();
        System.out.println("Greeting in English");
        
        // Load resource bundle for US English locale
        ResourceBundle bundle = ResourceBundle.getBundle("message", Locale.US);  
        
        // Get greeting message for this locale
        System.out.println("Message in " + Locale.US + ":" + "\t" + bundle.getString("greeting"));  

        // ========== INDONESIAN ==========
        System.out.println();
        System.out.println("Greeting in indonasian");
        
        // Change default locale to Indonesian   
        Locale.setDefault(new Locale("in", "ID"));  
        
        // Load resource bundle (uses default locale now)
        bundle = ResourceBundle.getBundle("message");  
        
        System.out.println("Message in " + Locale.getDefault() + ":" + "\t" + bundle.getString("greeting")); 
 
        // ========== GERMAN ==========
        System.out.println();
        System.out.println("Greeting in German");
        
        // Change default locale to German 
        Locale.setDefault(new Locale("de", "DE"));  
        bundle = ResourceBundle.getBundle("message");  
        
        System.out.println("Message in " + Locale.getDefault() + ":" + "\t" + bundle.getString("greeting"));

        // ========== DANISH ==========
        System.out.println();
        System.out.println("Greeting in Danish");
        
        // Change default locale to Danish 
        Locale.setDefault(new Locale("da"));  
        bundle = ResourceBundle.getBundle("message");  
        
        System.out.println("Message in " + Locale.getDefault() + ":" + "\t\t" + bundle.getString("greeting"));
   
    }  
}  
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 1-2 | `import java.util.*` | Imports Locale and ResourceBundle classes |
| 3 | `public class InternationalizationDemo` | Main class declaration |
| 8 | `ResourceBundle.getBundle("message", Locale.US)` | Loads `message_en_US.properties` file |
| 9 | `bundle.getString("greeting")` | Gets value of "greeting" key |
| 14 | `Locale.setDefault(new Locale("in", "ID"))` | Changes JVM's default locale to Indonesian |
| 15 | `ResourceBundle.getBundle("message")` | Loads `message_in_ID.properties` (uses default) |
| 21 | `new Locale("de", "DE")` | Creates German-Germany locale |
| 27 | `new Locale("da")` | Creates Danish locale (country omitted) |

### Expected Output
```
Greeting in English
Message in en_US:	Hello, how are you?

Greeting in indonasian
Message in in_ID:	Halo apa kabar?

Greeting in German
Message in de_DE:	Hallo wie geht es dir

Greeting in Danish
Message in da:		Hej, hvordan går det?
```

### Execution Flow Diagram
```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROGRAM EXECUTION FLOW                           │
├─────────────────────────────────────────────────────────────────────┤
│ Step 1: English Greeting                                            │
│    ├─ Locale.US → en_US                                             │
│    ├─ ResourceBundle searches: message_en_US.properties             │
│    ├─ Found! Loads file                                             │
│    └─ getString("greeting") → "Hello, how are you?"                 │
├─────────────────────────────────────────────────────────────────────┤
│ Step 2: Indonesian Greeting                                         │
│    ├─ Locale.setDefault(new Locale("in", "ID"))                     │
│    ├─ Default locale changed to in_ID                               │
│    ├─ ResourceBundle searches: message_in_ID.properties             │
│    └─ getString("greeting") → "Halo apa kabar?"                     │
├─────────────────────────────────────────────────────────────────────┤
│ Step 3: German Greeting                                             │
│    ├─ Locale.setDefault(new Locale("de", "DE"))                     │
│    ├─ ResourceBundle searches: message_de_DE.properties             │
│    └─ getString("greeting") → "Hallo wie geht es dir"               │
├─────────────────────────────────────────────────────────────────────┤
│ Step 4: Danish Greeting                                             │
│    ├─ Locale.setDefault(new Locale("da"))                           │
│    ├─ ResourceBundle searches: message_da.properties                │
│    └─ getString("greeting") → "Hej, hvordan går det?"               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Spring Boot I18N Implementation

### Project Setup

#### Dependencies (Spring Initializr)
```
✓ Spring Web
✓ Spring Boot DevTools
```

#### Project Structure
```
src/main/
├── java/com/example/demo/
│   └── I18NApplication.java
├── java/com/example/i18n/
│   └── I18nController.java
└── resources/
    ├── messages.properties         (Default - English)
    ├── messages_fr.properties      (French)
    ├── messages_de.properties      (German)
    ├── messages_es.properties      (Spanish)
    ├── messages_hi.properties      (Hindi)
    ├── messages_mr.properties      (Marathi)
    ├── messages_ko.properties      (Korean)
    ├── messages_ru.properties      (Russian)
    └── messages_bih.properties     (Bihari)
```

### Key Concepts

#### LocaleResolver
**Purpose:** Determines the current locale based on the incoming HTTP request.

> LocaleResolver is responsible for determining and setting the current locale based on the incoming request (using headers like Accept-Language, cookies, or session data). It creates an instance of Locale and passes it to your REST controller method.

#### MessageSource
**Purpose:** Retrieves and resolves messages based on the current locale.

> The MessageSource is used to resolve and retrieve messages based on the locale that was set by the LocaleResolver. In the code, you don't need to directly interact with the LocaleResolver in your controller; it's handled automatically by Spring.

### Application Configuration

#### I18NApplication.java

```java
package com.example.demo;

import java.util.Locale;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

@SpringBootApplication
@ComponentScan(basePackages="com.example.*")
public class I18NApplication {

    public static void main(String[] args) {
        SpringApplication.run(I18NApplication.class, args);
    }
    
    // Configure the LocaleResolver to accept the language from the HTTP header
    @Bean
    public LocaleResolver localeResolver() {
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(Locale.ENGLISH); // Default language
        return localeResolver;
    }

    // Configure the MessageSource for i18n
    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:messages");  // Base name of the messages files
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 14 | `@SpringBootApplication` | Enables auto-configuration, component scanning |
| 15 | `@ComponentScan(basePackages="com.example.*")` | Scans all sub-packages for @Component, @Controller, etc. |
| 22-26 | `localeResolver()` | Creates bean for resolving locale from HTTP request |
| 23 | `AcceptHeaderLocaleResolver` | Reads `Accept-Language` header from HTTP request |
| 24 | `setDefaultLocale(Locale.ENGLISH)` | Fallback locale if header not present |
| 29-33 | `messageSource()` | Creates bean for loading message properties |
| 30 | `ReloadableResourceBundleMessageSource` | Can reload properties without restart |
| 31 | `setBasename("classpath:messages")` | Base filename (looks for messages_*.properties) |
| 32 | `setDefaultEncoding("UTF-8")` | Supports non-ASCII characters (Hindi, Korean, etc.) |

### REST Controller

#### I18nController.java

```java
package com.example.i18n;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

import java.util.Locale;

@RestController
public class I18nController {

    @Autowired
    private MessageSource messageSource;
   
    // REST endpoint for greeting
    @GetMapping("/greeting")
    public String getGreeting(@RequestHeader(name = "Accept-Language", required = false) Locale locale) {
        // If no Accept-Language header is provided, default to English
        if (locale == null) {
            locale = Locale.ENGLISH;
        }

        // Retrieve the message based on the locale
        return messageSource.getMessage("greeting", null, locale);
    }
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 11 | `@RestController` | Marks class as REST controller, returns JSON/String directly |
| 14-15 | `@Autowired MessageSource` | Injects the configured MessageSource bean |
| 18 | `@GetMapping("/greeting")` | Maps GET requests to /greeting endpoint |
| 19 | `@RequestHeader(name = "Accept-Language", required = false)` | Extracts Accept-Language header; optional |
| 19 | `Locale locale` | Spring auto-converts header to Locale object |
| 21-23 | `if (locale == null)` | Fallback to English if no header provided |
| 26 | `messageSource.getMessage("greeting", null, locale)` | Looks up "greeting" key for given locale |

### Message Properties Files

All files must be in `src/main/resources/`:

#### messages.properties (Default - English)
```properties
greeting=Hello, welcome to our API!
```

#### messages_fr.properties (French)
```properties
greeting=Bonjour, bienvenue dans notre API!
```

#### messages_de.properties (German)
```properties
greeting=Hallo, willkommen in unserer API!
```

#### messages_es.properties (Spanish)
```properties
greeting=¡Hola, bienvenido a nuestra API!
```

#### messages_hi.properties (Hindi)
```properties
greeting=\u0928\u092E\u0938\u094D\u0924\u0947, \u0939\u092E\u093E\u0930\u0940 API \u092E\u0947\u0902 \u0906\u092A\u0915\u093E \u0938\u094D\u0935\u093E\u0917\u0924 \u0939\u0948!
```
*(Displays: नमस्ते, हमारी API में आपका स्वागत है!)*

#### messages_mr.properties (Marathi)
```properties
greeting=\u0928\u092E\u0938\u094D\u0915\u093E\u0930, \u0906\u092E\u091A\u094D\u092F\u093E API \u092E\u0927\u094D\u092F\u0947 \u0906\u092A\u0932\u0947 \u0938\u094D\u0935\u093E\u0917\u0924 \u0906\u0939\u0947!
```

#### messages_ko.properties (Korean)
```properties
greeting=\uC548\uB155\uD558\uC138\uC694, \uC800\uD76C API\uC5D0 \uC624\uC2E0 \uAC83\uC744 \uD658\uC601\uD569\uB2C8\uB2E4!
```

#### messages_ru.properties (Russian)
```properties
greeting=\u0417\u0434\u0440\u0430\u0432\u0441\u0442\u0432\u0443\u0439\u0442\u0435, \u0434\u043E\u0431\u0440\u043E \u043F\u043E\u0436\u0430\u043B\u043E\u0432\u0430\u0442\u044C \u0432 \u043D\u0430\u0448\u0435 API!
```

#### messages_bih.properties (Bihari)
```properties
greeting=\u0928\u092E\u0938\u094D\u0915\u093E\u0930, \u0939\u092E\u0930\u093E API \u092E\u0947\u0902 \u0930\u0909\u0906 \u0938\u094D\u0935\u093E\u0917\u0924 \u092C\u093E!
```

### Testing with Postman

#### Step 1: Start Application
```
Run I18NApplication.java
```

#### Step 2: Default Language Test (English)
```
Method: GET
URL: http://localhost:8080/greeting
Headers: (none)

Expected Response:
Hello, welcome to our API!
```

#### Step 3: French Language Test
```
Method: GET
URL: http://localhost:8080/greeting
Headers:
  Key: Accept-Language
  Value: fr

Expected Response:
Bonjour, bienvenue dans notre API!
```

#### Step 4: German Language Test
```
Method: GET
URL: http://localhost:8080/greeting
Headers:
  Key: Accept-Language
  Value: de

Expected Response:
Hallo, willkommen in unserer API!
```

### Complete Execution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                 SPRING BOOT I18N FLOW                               │
├─────────────────────────────────────────────────────────────────────┤
│ 1. Client sends GET /greeting with Accept-Language: fr              │
├─────────────────────────────────────────────────────────────────────┤
│ 2. DispatcherServlet receives request                               │
├─────────────────────────────────────────────────────────────────────┤
│ 3. AcceptHeaderLocaleResolver reads Accept-Language header          │
│    └─ Converts "fr" → Locale.FRENCH                                 │
├─────────────────────────────────────────────────────────────────────┤
│ 4. I18nController.getGreeting(Locale locale) invoked                │
│    └─ locale = Locale.FRENCH                                        │
├─────────────────────────────────────────────────────────────────────┤
│ 5. messageSource.getMessage("greeting", null, Locale.FRENCH)        │
│    └─ Searches: messages_fr.properties                              │
│    └─ Finds key "greeting"                                          │
│    └─ Returns: "Bonjour, bienvenue dans notre API!"                 │
├─────────────────────────────────────────────────────────────────────┤
│ 6. Response sent to client                                          │
│    └─ Body: "Bonjour, bienvenue dans notre API!"                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q1: What is the difference between I18N and L10N?
**Answer:** 
- **I18N (Internationalization):** Designing software to support multiple languages/locales without code changes
- **L10N (Localization):** Adapting the software for a specific language/locale (translating content, formatting dates/numbers)

### Q2: How does ResourceBundle locate property files?
**Answer:** ResourceBundle follows a fallback mechanism:
1. `baseName_language_country_variant.properties` (most specific)
2. `baseName_language_country.properties`
3. `baseName_language.properties`
4. `baseName.properties` (default fallback)

### Q3: What is AcceptHeaderLocaleResolver?
**Answer:** It's a LocaleResolver implementation that reads the `Accept-Language` HTTP header to determine the client's preferred locale. If the header is missing, it uses the configured default locale.

### Q4: Why use ReloadableResourceBundleMessageSource?
**Answer:** It can reload property files at runtime without restarting the application. Use `setCacheSeconds()` to control refresh interval.

### Q5: How to add a new language in Spring Boot I18N?
**Answer:**
1. Create `messages_<language_code>.properties` in `src/main/resources`
2. Add key-value pairs with translated messages
3. No code changes required - Spring automatically picks up new files

---

## Summary

| Concept | Java Standard | Spring Boot |
|---------|---------------|-------------|
| Locale handling | `Locale` class | `LocaleResolver` bean |
| Message loading | `ResourceBundle` | `MessageSource` bean |
| Default locale | `Locale.setDefault()` | `resolver.setDefaultLocale()` |
| Message retrieval | `bundle.getString()` | `messageSource.getMessage()` |
| File location | Classpath root | `src/main/resources` |
| File naming | `baseName_lang_COUNTRY.properties` | Same convention |

---

**Next:** [02_JUnit_Fundamentals_and_Annotations.md](./02_JUnit_Fundamentals_and_Annotations.md)
