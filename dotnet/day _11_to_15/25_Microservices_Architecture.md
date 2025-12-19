# Microservices Architecture in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What are Microservices?](#what-are-microservices)
3. [Monolithic vs Microservices](#monolithic-vs-microservices)
4. [Key Characteristics](#key-characteristics)
5. [Design Principles](#design-principles)
6. [Communication Patterns](#communication-patterns)
7. [API Gateway](#api-gateway)
8. [Service Discovery](#service-discovery)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Microservices are like a **food court vs a single restaurant**:

```
Monolithic (Single Restaurant):
    One kitchen → Makes all types of food
    ❌ If kitchen breaks, entire restaurant closes
    ❌ Can't scale pizza making without scaling everything
    ❌ All chefs must coordinate

Microservices (Food Court):
    Pizza Shop → Makes pizza only
    Burger Shop → Makes burgers only
    Chinese Shop → Makes Chinese food only
    ✅ Each shop independent
    ✅ Can scale pizza shop during peak hours
    ✅ If burger shop closes, others continue
```

---

## What are Microservices?

Microservices is an architectural style where an application is built as a collection of small, independent services that:
- Run in their own process
- Communicate over network protocols (HTTP, gRPC)
- Are deployed independently
- Have their own database

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    MICROSERVICES ARCHITECTURE                    │
└─────────────────────────────────────────────────────────────────┘

                         ┌─────────────┐
                         │   Client    │
                         │ (Web/Mobile)│
                         └──────┬──────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │     API Gateway       │
                    │   (Ocelot/YARP)       │
                    └───────────┬───────────┘
                                │
         ┌──────────────────────┼──────────────────────┐
         │                      │                      │
         ▼                      ▼                      ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  User Service   │  │ Product Service │  │  Order Service  │
│   (.NET Core)   │  │   (.NET Core)   │  │   (.NET Core)   │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│   User API      │  │  Product API    │  │   Order API     │
│   User Logic    │  │  Product Logic  │  │   Order Logic   │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│   User DB       │  │  Product DB     │  │   Order DB      │
│   (SQL Server)  │  │   (MongoDB)     │  │  (PostgreSQL)   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

## Monolithic vs Microservices

### Comparison Table

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| **Deployment** | Single unit | Independent services |
| **Scaling** | Scale entire app | Scale individual services |
| **Technology** | Same stack | Polyglot (different stacks) |
| **Database** | Shared database | Database per service |
| **Team Structure** | One large team | Small focused teams |
| **Complexity** | Simpler initially | More complex infrastructure |
| **Failure Impact** | Entire app affected | Isolated to service |
| **Development Speed** | Slower at scale | Faster for large teams |

### Visual Comparison

```
MONOLITHIC APPLICATION:
┌─────────────────────────────────────────────────────┐
│                   Single Application                 │
├─────────────────────────────────────────────────────┤
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │   User   │ │ Product  │ │  Order   │  ...       │
│  │  Module  │ │  Module  │ │  Module  │            │
│  └──────────┘ └──────────┘ └──────────┘            │
├─────────────────────────────────────────────────────┤
│                   Shared Database                    │
└─────────────────────────────────────────────────────┘


MICROSERVICES APPLICATION:
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ User Service │ │Product Service│ │Order Service │
├──────────────┤ ├──────────────┤ ├──────────────┤
│   User DB    │ │  Product DB  │ │   Order DB   │
└──────────────┘ └──────────────┘ └──────────────┘
       ↑                ↑                ↑
       └────────────────┼────────────────┘
                        │
                  API Gateway
                        │
                     Client
```

---

## Key Characteristics

### 1. Single Responsibility

Each service does one thing well.

```csharp
// ✅ User Service - Only handles user operations
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id) { }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(UserDto dto) { }
}

// ✅ Product Service - Only handles product operations
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetProducts() { }
}
```

### 2. Independent Deployment

```yaml
# docker-compose.yml
version: '3.8'
services:
  user-service:
    build: ./UserService
    ports:
      - "5001:80"
  
  product-service:
    build: ./ProductService
    ports:
      - "5002:80"
  
  order-service:
    build: ./OrderService
    ports:
      - "5003:80"
```

### 3. Database Per Service

```
User Service ──────> User Database (SQL Server)
Product Service ────> Product Database (MongoDB)
Order Service ──────> Order Database (PostgreSQL)

Each service owns its data!
```

### 4. Resilience & Fault Tolerance

```
If Product Service fails:
    ├── User Service continues working ✅
    ├── Order Service continues (with degraded functionality) ✅
    └── Only Product features affected
```

---

## Design Principles

### 1. Domain-Driven Design (DDD)

Align services with business domains:

```
E-Commerce Platform
├── User Domain      → User Service
├── Product Domain   → Product Service  
├── Order Domain     → Order Service
├── Payment Domain   → Payment Service
└── Shipping Domain  → Shipping Service
```

### 2. Bounded Context

Each service has its own understanding of entities:

```csharp
// User Service - Full user details
public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }
    public string Address { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Order Service - Only needs basic user info
public class OrderUser
{
    public int UserId { get; set; }
    public string Name { get; set; }
    public string ShippingAddress { get; set; }
}
```

### 3. API First Design

Design APIs before implementation:

```yaml
# OpenAPI Specification
openapi: 3.0.0
info:
  title: User Service API
  version: 1.0.0
paths:
  /api/users/{id}:
    get:
      summary: Get user by ID
      responses:
        '200':
          description: User found
```

---

## Communication Patterns

### 1. Synchronous (HTTP/REST)

```csharp
// Order Service calling User Service
public class UserServiceClient
{
    private readonly HttpClient _httpClient;
    
    public UserServiceClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
        _httpClient.BaseAddress = new Uri("http://user-service:5001");
    }
    
    public async Task<User> GetUserAsync(int userId)
    {
        var response = await _httpClient.GetAsync($"/api/users/{userId}");
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadFromJsonAsync<User>();
    }
}
```

### 2. Synchronous (gRPC)

```protobuf
// user.proto
syntax = "proto3";

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  int32 user_id = 1;
}

message UserResponse {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

### 3. Asynchronous (Message Queue)

```csharp
// Order Service - Publishes event
public class OrderService
{
    private readonly IMessageBus _messageBus;
    
    public async Task CreateOrderAsync(Order order)
    {
        // Save order
        await _orderRepository.AddAsync(order);
        
        // Publish event (doesn't wait for response)
        await _messageBus.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            UserId = order.UserId,
            Total = order.Total
        });
    }
}

// Notification Service - Subscribes to event
public class OrderEventHandler
{
    public async Task HandleOrderCreated(OrderCreatedEvent @event)
    {
        // Send email notification
        await _emailService.SendOrderConfirmationAsync(@event.UserId);
    }
}
```

### Communication Pattern Comparison

| Pattern | Use Case | Pros | Cons |
|---------|----------|------|------|
| **REST** | Simple CRUD | Easy, Well-known | Higher latency |
| **gRPC** | High performance | Fast, Streaming | Complex setup |
| **Message Queue** | Event-driven | Decoupled, Resilient | Eventually consistent |

---

## API Gateway

### What is API Gateway?

Single entry point for all client requests. Routes requests to appropriate services.

```
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY PATTERN                         │
└─────────────────────────────────────────────────────────────────┘

               Client Request
                    │
                    ▼
           ┌───────────────────┐
           │    API Gateway    │
           │                   │
           │ • Authentication  │
           │ • Rate Limiting   │
           │ • Load Balancing  │
           │ • Request Routing │
           │ • Response Caching│
           └─────────┬─────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
┌───────┐       ┌───────┐       ┌───────┐
│ User  │       │Product│       │ Order │
│Service│       │Service│       │Service│
└───────┘       └───────┘       └───────┘
```

### Ocelot API Gateway (Popular for .NET)

```csharp
// File: Program.cs (Gateway Project)

using Ocelot.DependencyInjection;
using Ocelot.Middleware;

var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddJsonFile("ocelot.json");
builder.Services.AddOcelot();

var app = builder.Build();

await app.UseOcelot();

app.Run();
```

```json
// ocelot.json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "user-service", "Port": 5001 }
      ],
      "UpstreamPathTemplate": "/api/users/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Put", "Delete" ]
    },
    {
      "DownstreamPathTemplate": "/api/products/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "product-service", "Port": 5002 }
      ],
      "UpstreamPathTemplate": "/api/products/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post", "Put", "Delete" ]
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "http://localhost:5000"
  }
}
```

---

## Service Discovery

### What is Service Discovery?

Automatic detection of services in a network. Essential when services scale dynamically.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVICE DISCOVERY                             │
└─────────────────────────────────────────────────────────────────┘

1. Service A starts
   └──> Registers with Discovery Server (e.g., Consul)
        "I'm Service A at 192.168.1.10:5001"

2. Service B needs to call Service A
   └──> Queries Discovery Server
        "Where is Service A?"
   └──> Gets response: "192.168.1.10:5001"

3. Service A scales to 3 instances
   └──> All register themselves
        Instance 1: 192.168.1.10:5001
        Instance 2: 192.168.1.11:5001
        Instance 3: 192.168.1.12:5001

4. Service B calls Service A
   └──> Load balancer distributes to any instance
```

---

## Code Examples

### Sample Microservice Structure

```
Solution
├── ApiGateway/
│   ├── Program.cs
│   └── ocelot.json
├── UserService/
│   ├── Controllers/
│   │   └── UserController.cs
│   ├── Services/
│   │   └── UserService.cs
│   ├── Data/
│   │   └── UserDbContext.cs
│   └── Program.cs
├── ProductService/
│   ├── Controllers/
│   │   └── ProductController.cs
│   └── Program.cs
└── OrderService/
    ├── Controllers/
    │   └── OrderController.cs
    └── Program.cs
```

### Inter-Service Communication

```csharp
// File: OrderService/Services/UserServiceClient.cs

public interface IUserServiceClient
{
    Task<UserDto> GetUserAsync(int userId);
}

public class UserServiceClient : IUserServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<UserServiceClient> _logger;

    public UserServiceClient(
        HttpClient httpClient,
        ILogger<UserServiceClient> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    public async Task<UserDto> GetUserAsync(int userId)
    {
        try
        {
            var response = await _httpClient.GetAsync($"/api/users/{userId}");
            
            if (response.IsSuccessStatusCode)
            {
                return await response.Content.ReadFromJsonAsync<UserDto>();
            }
            
            _logger.LogWarning("User {UserId} not found", userId);
            return null;
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "Error calling User Service");
            throw;
        }
    }
}

// Register HttpClient with resilience
builder.Services.AddHttpClient<IUserServiceClient, UserServiceClient>(client =>
{
    client.BaseAddress = new Uri("http://user-service:5001");
    client.Timeout = TimeSpan.FromSeconds(30);
})
.AddTransientHttpErrorPolicy(policy =>
    policy.WaitAndRetryAsync(3, _ => TimeSpan.FromSeconds(2)));
```

---

## Best Practices

### 1. Design for Failure

```csharp
// Circuit breaker pattern with Polly
builder.Services.AddHttpClient<IUserServiceClient, UserServiceClient>()
    .AddTransientHttpErrorPolicy(policy =>
        policy.CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 3,
            durationOfBreak: TimeSpan.FromSeconds(30)));
```

### 2. Implement Health Checks

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString)
    .AddCheck<UserServiceHealthCheck>("UserService");

app.MapHealthChecks("/health");
```

### 3. Centralized Logging

```csharp
// Use correlation IDs across services
public class CorrelationIdMiddleware
{
    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers["X-Correlation-ID"]
            .FirstOrDefault() ?? Guid.NewGuid().ToString();
        
        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        await _next(context);
    }
}
```

---

## Interview Questions

### Q1: What are Microservices?
**Answer:** Microservices is an architectural style where an application is built as a collection of small, independent services that run in their own process, communicate over network protocols, and are deployed independently.

### Q2: What are the advantages of Microservices?
**Answer:**
- Independent deployment and scaling
- Technology diversity (polyglot)
- Fault isolation
- Better team organization
- Easier to understand and maintain individual services

### Q3: What are the challenges of Microservices?
**Answer:**
- Distributed system complexity
- Inter-service communication
- Data consistency
- Service discovery and orchestration
- Monitoring and debugging

### Q4: How do Microservices communicate?
**Answer:**
- Synchronous: HTTP/REST, gRPC
- Asynchronous: Message queues (RabbitMQ, Kafka)

### Q5: What is an API Gateway?
**Answer:** A single entry point for all client requests that handles routing, authentication, rate limiting, and load balancing. Examples: Ocelot, YARP, Kong.

### Q6: What is the Circuit Breaker pattern?
**Answer:** A resilience pattern that prevents cascading failures by "breaking" the circuit when a service repeatedly fails, returning a fallback response instead of continuing to call the failing service.

---

## Quick Reference

### When to Use Microservices

| Use Microservices | Use Monolith |
|-------------------|--------------|
| Large teams | Small teams |
| Complex domains | Simple domains |
| Need independent scaling | Uniform load |
| Different tech needs | Same tech stack |

### Key Patterns

```
API Gateway         → Single entry point
Service Discovery   → Find services dynamically
Circuit Breaker     → Handle failures gracefully
Event Sourcing      → Event-based communication
CQRS               → Separate read/write models
```
