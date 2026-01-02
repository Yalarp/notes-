# REST API Fundamentals - Complete Guide

## Table of Contents
1. [What is REST?](#what-is-rest)
2. [REST Principles](#rest-principles)
3. [HTTP Methods in Detail](#http-methods-in-detail)
4. [Safe, Unsafe, Idempotent Methods](#safe-unsafe-idempotent-methods)
5. [HTTP Status Codes](#http-status-codes)
6. [Resource Naming Conventions](#resource-naming-conventions)
7. [Request and Response Formats](#request-and-response-formats)
8. [Statelessness Explained](#statelessness-explained)
9. [Summary](#summary)

---

## What is REST?

### Definition

**REST (Representational State Transfer)** is an architectural style for designing networked applications. It was defined by Roy Fielding in his 2000 PhD dissertation.

> [!IMPORTANT]
> **REST is NOT a protocol** - it's an architectural style that uses existing protocols (HTTP) and defines guidelines for creating web services.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Resource** | Any data that can be named (user, book, order) |
| **Representation** | How the resource is formatted (JSON, XML) |
| **State Transfer** | Exchanging representations between client and server |
| **Uniform Interface** | Standard HTTP methods for operations |

---

## REST Principles

### The 6 Constraints

1. **Client-Server Architecture**
   - Separation of concerns
   - Client handles UI, server handles data
   
2. **Statelessness**
   - Each request is independent
   - Server doesn't store client session
   
3. **Cacheability**
   - Responses can be cached
   - Improves performance
   
4. **Layered System**
   - Client doesn't know if talking to final server
   - Allows load balancers, proxies
   
5. **Uniform Interface**
   - Standard HTTP methods
   - Consistent URL patterns
   
6. **Code on Demand (Optional)**
   - Server can send executable code to client

---

## HTTP Methods in Detail

### CRUD Mapping

| HTTP Method | CRUD | SQL | Description |
|-------------|------|-----|-------------|
| **GET** | Read | SELECT | Retrieve resource |
| **POST** | Create | INSERT | Create new resource |
| **PUT** | Update | UPDATE | Replace entire resource |
| **PATCH** | Update | UPDATE | Modify part of resource |
| **DELETE** | Delete | DELETE | Remove resource |

### GET - Retrieve Resource

```
GET /api/books         → Get all books
GET /api/books/1       → Get book with ID 1
GET /api/books?author=John → Get books by author
```

### POST - Create Resource

```
POST /api/books
Content-Type: application/json

{
    "title": "Spring Boot Guide",
    "author": "John Doe",
    "price": 500
}

Response: 201 Created
```

### PUT - Replace Resource

```
PUT /api/books/1
Content-Type: application/json

{
    "title": "Updated Title",
    "author": "John Doe",
    "price": 600
}

Response: 200 OK
```

### DELETE - Remove Resource

```
DELETE /api/books/1

Response: 204 No Content
```

---

## Safe, Unsafe, Idempotent Methods

From your course materials: **"Safe_Unsafe_Idempotent_Non-Idempotent methods info.txt"**

### Definitions

> **Safe methods**: Methods that do not change any state on the server.
> Example: "GET" method
>
> **Unsafe methods**: Methods that do change state on the server.
> Example: "POST", "PUT" and "DELETE"
>
> **Idempotent methods**: First time they change the state on the server, but if you call them again and again they don't change anything.
> Example: "PUT" and "DELETE"
>
> **Non-Idempotent methods**: Every time you call them they will change the state on the server.
> Example: "POST"

### Method Properties Table

| Method | Safe | Idempotent | Description |
|--------|------|------------|-------------|
| **GET** | ✅ Yes | ✅ Yes | Read-only, no state change |
| **POST** | ❌ No | ❌ No | Creates new resource each time |
| **PUT** | ❌ No | ✅ Yes | Same result if called multiple times |
| **PATCH** | ❌ No | ✅ Yes | Same result if called multiple times |
| **DELETE** | ❌ No | ✅ Yes | Resource deleted, subsequent calls return 404 |

### Example: POST is Non-Idempotent

From source:
```json
{
    "name": "Sachin",
    "age": 48
}
```

> "With this JSON body if you click POST again and again, it will insert new record every time with the same data."

```
POST /api/students → Creates student ID 1
POST /api/students → Creates student ID 2 (duplicate data, new record!)
POST /api/students → Creates student ID 3 (another duplicate!)
```

---

## HTTP Status Codes

### Categories

| Range | Category | Examples |
|-------|----------|----------|
| **1xx** | Informational | 100 Continue |
| **2xx** | Success | 200 OK, 201 Created |
| **3xx** | Redirection | 301 Moved, 304 Not Modified |
| **4xx** | Client Error | 400 Bad Request, 404 Not Found |
| **5xx** | Server Error | 500 Internal Server Error |

### Common Status Codes

| Code | Name | When to Use |
|------|------|-------------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST (new resource created) |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Invalid input, validation error |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Authenticated but not authorized |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method |
| **409** | Conflict | Resource conflict (duplicate) |
| **500** | Internal Server Error | Server exception |

### When to Use Each

```mermaid
graph TD
    A[API Request] --> B{Request Valid?}
    B -->|No| C[400 Bad Request]
    B -->|Yes| D{Authenticated?}
    D -->|No| E[401 Unauthorized]
    D -->|Yes| F{Authorized?}
    F -->|No| G[403 Forbidden]
    F -->|Yes| H{Resource Exists?}
    H -->|No| I[404 Not Found]
    H -->|Yes| J{Operation Success?}
    J -->|No| K[500 Internal Error]
    J -->|Yes| L[200/201/204 Success]
    
    style L fill:#4CAF50,color:#fff
    style C fill:#f44336,color:#fff
    style K fill:#f44336,color:#fff
```

---

## Resource Naming Conventions

### Best Practices

| Practice | Good ✅ | Bad ❌ |
|----------|---------|--------|
| Use nouns | `/books` | `/getBooks` |
| Use plurals | `/books` | `/book` |
| Use lowercase | `/books` | `/Books` |
| Use hyphens | `/book-categories` | `/book_categories` |
| Use hierarchy | `/authors/1/books` | `/getBooksByAuthor` |

### URL Structure Examples

```
# Collection Resources
GET    /api/books              → Get all books
POST   /api/books              → Create new book

# Singleton Resources
GET    /api/books/1            → Get book with ID 1
PUT    /api/books/1            → Update book 1
DELETE /api/books/1            → Delete book 1

# Nested Resources
GET    /api/authors/1/books    → Get all books by author 1
POST   /api/authors/1/books    → Create book for author 1

# Filtering
GET    /api/books?category=fiction
GET    /api/books?price_min=100&price_max=500

# Pagination
GET    /api/books?page=1&size=10

# Sorting
GET    /api/books?sort=price&order=desc
```

---

## Request and Response Formats

### JSON Format (Preferred)

**Request**:
```json
POST /api/books
Content-Type: application/json

{
    "title": "Spring Boot in Action",
    "author": "Craig Walls",
    "price": 599,
    "isbn": "978-1617292545"
}
```

**Response**:
```json
HTTP/1.1 201 Created
Content-Type: application/json

{
    "id": 1,
    "title": "Spring Boot in Action",
    "author": "Craig Walls",
    "price": 599,
    "isbn": "978-1617292545",
    "createdAt": "2024-01-02T10:30:00Z"
}
```

### Collection Response with Pagination

```json
{
    "content": [
        {"id": 1, "title": "Book 1", "price": 100},
        {"id": 2, "title": "Book 2", "price": 200}
    ],
    "page": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10,
    "first": true,
    "last": false
}
```

### Error Response

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "timestamp": "2024-01-02T10:30:00Z",
    "status": 400,
    "error": "Bad Request",
    "message": "Validation failed",
    "errors": [
        {
            "field": "title",
            "message": "Title is required"
        },
        {
            "field": "price",
            "message": "Price must be positive"
        }
    ]
}
```

---

## Statelessness Explained

### What is Statelessness?

Each REST request must contain ALL information needed to process it. The server does NOT store any client state between requests.

### Stateless Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server
    
    Client->>Server: Request 1 (with all context)
    Note right of Server: Process, forget client state
    Server-->>Client: Response 1
    
    Client->>Server: Request 2 (with all context again)
    Note right of Server: No memory of Request 1
    Server-->>Client: Response 2
```

### Authentication in Stateless REST

Since server doesn't store sessions, authentication uses:
- **JWT (JSON Web Tokens)**: Token contains user info
- **API Keys**: Passed in headers
- **OAuth 2.0**: Token-based authorization

```
GET /api/books
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Summary

### Key Takeaways

1. **REST** is an architectural style, not a protocol
2. **HTTP Methods**: GET (read), POST (create), PUT (update), DELETE (delete)
3. **Safe methods** don't change state (GET)
4. **Idempotent methods** give same result on multiple calls (GET, PUT, DELETE)
5. **Status codes** indicate success (2xx), client error (4xx), server error (5xx)
6. **Statelessness** means each request is independent
7. Use **nouns** and **plurals** for resource URLs

### Quick Reference

| Method | Safe | Idempotent | Success Code |
|--------|------|------------|--------------|
| GET | ✅ | ✅ | 200 |
| POST | ❌ | ❌ | 201 |
| PUT | ❌ | ✅ | 200 |
| DELETE | ❌ | ✅ | 204 |

---

## Practice Questions

1. What is REST and how is it different from SOAP?
2. Explain the difference between safe and idempotent methods.
3. Why is POST considered non-idempotent?
4. List 5 HTTP status codes and when to use each.
5. What are the best practices for REST URL naming?
6. What is statelessness and how does authentication work without sessions?
7. Write example URLs for a book management system.

---

**End of Note 12: REST API Fundamentals**

*Previous: [11_Web_Services_Introduction.md](file:///c:/Users/2706p/Desktop/mcq/notes/11_Web_Services_Introduction.md)*  
*Next: [13_Building_REST_APIs_Spring_Boot.md](file:///c:/Users/2706p/Desktop/mcq/notes/13_Building_REST_APIs_Spring_Boot.md)*
