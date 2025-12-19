# JWT Authentication in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is JWT?](#what-is-jwt)
3. [JWT Structure](#jwt-structure)
4. [Configuring JWT Authentication](#configuring-jwt-authentication)
5. [Generating JWT Tokens](#generating-jwt-tokens)
6. [Protecting API Endpoints](#protecting-api-endpoints)
7. [Role-Based Authorization](#role-based-authorization)
8. [Refresh Tokens](#refresh-tokens)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
JWT is like a **concert wristband**:

```
Without JWT (Session-Based):
    Every request → Show ID → Guard checks database → Enter
    ❌ Guard must check database every time
    ❌ Doesn't work across multiple venues

With JWT (Token-Based):
    Login → Get wristband (token) with your info encoded
    Every request → Show wristband → Guard reads info from wristband
    ✅ No database lookup needed
    ✅ Works across multiple venues (microservices)
```

---

## What is JWT?

JSON Web Token (JWT) is a compact, self-contained way to securely transmit information between parties as a JSON object. It's digitally signed, so the information can be verified and trusted.

### When to Use JWT

| Use Case | JWT Suitable? |
|----------|---------------|
| Stateless authentication | ✅ Yes |
| Single Sign-On (SSO) | ✅ Yes |
| Microservices communication | ✅ Yes |
| Information exchange | ✅ Yes |
| Session replacement | ✅ Yes |
| Storing sensitive data | ❌ No (visible to anyone) |

### JWT vs Session Authentication

```
┌─────────────────────────────────────────────────────────────────┐
│                 SESSION VS JWT AUTHENTICATION                    │
├────────────────────────────┬────────────────────────────────────┤
│     SESSION-BASED          │           JWT-BASED                │
├────────────────────────────┼────────────────────────────────────┤
│ Server stores session      │ Token stored on client             │
│ Cookie sent with requests  │ Token sent in header               │
│ Stateful                   │ Stateless                          │
│ Server memory used         │ No server storage needed           │
│ Horizontal scaling harder  │ Easy horizontal scaling            │
│ CSRF protection needed     │ No CSRF (no cookies)               │
│ Works with browsers only   │ Works with any client              │
└────────────────────────────┴────────────────────────────────────┘
```

---

## JWT Structure

### Three Parts of JWT

A JWT consists of three parts separated by dots: `xxxxx.yyyyy.zzzzz`

```
┌─────────────────────────────────────────────────────────────────┐
│                       JWT STRUCTURE                              │
└─────────────────────────────────────────────────────────────────┘

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4iLCJpYXQiOjE1MTYyMzkwMjJ9.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

│                    │                              │
└──────┬─────────────┘──────────────┬───────────────┘───────┬─────
       │                            │                       │
    HEADER                       PAYLOAD                 SIGNATURE
  (Algorithm)                   (Claims)               (Verification)
```

### 1. Header

```json
{
  "alg": "HS256",    // Algorithm used for signing
  "typ": "JWT"       // Token type
}
```

### 2. Payload (Claims)

```json
{
  // Registered claims (standard)
  "sub": "1234567890",           // Subject (user ID)
  "iat": 1516239022,             // Issued at
  "exp": 1516242622,             // Expiration time
  "nbf": 1516239022,             // Not before
  
  // Public claims
  "name": "John Doe",
  "email": "john@example.com",
  
  // Private claims (custom)
  "role": "Admin",
  "department": "IT"
}
```

### 3. Signature

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

---

## Configuring JWT Authentication

### Step 1: Install NuGet Packages

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package System.IdentityModel.Tokens.Jwt
```

### Step 2: Add JWT Settings to appsettings.json

```json
{
  "JwtSettings": {
    "SecretKey": "YourSuperSecretKeyThatIsAtLeast32CharactersLong!",
    "Issuer": "YourApp",
    "Audience": "YourAppUsers",
    "ExpirationMinutes": 60
  }
}
```

### Step 3: Configure Services in Program.cs

```csharp
// File: Program.cs

using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// ═══════════════════════════════════════════════════════════════
// JWT AUTHENTICATION CONFIGURATION
// ═══════════════════════════════════════════════════════════════

// Line 1: Get JWT settings from configuration
var jwtSettings = builder.Configuration.GetSection("JwtSettings");
var secretKey = jwtSettings["SecretKey"];

// Line 2: Configure authentication
builder.Services.AddAuthentication(options =>
{
    // Line 3: Set default schemes
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
// Line 4: Add JWT Bearer authentication
.AddJwtBearer(options =>
{
    // Line 5: Token validation parameters
    options.TokenValidationParameters = new TokenValidationParameters
    {
        // Validate the issuer (who created the token)
        ValidateIssuer = true,
        ValidIssuer = jwtSettings["Issuer"],
        
        // Validate the audience (who the token is for)
        ValidateAudience = true,
        ValidAudience = jwtSettings["Audience"],
        
        // Validate the signing key
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(secretKey)),
        
        // Validate token expiration
        ValidateLifetime = true,
        
        // Allow clock skew (time difference between servers)
        ClockSkew = TimeSpan.Zero
    };
    
    // Line 6: Optional: Handle events
    options.Events = new JwtBearerEvents
    {
        OnAuthenticationFailed = context =>
        {
            if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
            {
                context.Response.Headers.Add("Token-Expired", "true");
            }
            return Task.CompletedTask;
        }
    };
});

// Line 7: Add authorization
builder.Services.AddAuthorization();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// MIDDLEWARE PIPELINE
// ═══════════════════════════════════════════════════════════════

app.UseHttpsRedirection();

// Line 8: Authentication MUST come before Authorization
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

## Generating JWT Tokens

### Token Service

```csharp
// File: Services/TokenService.cs

using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public interface ITokenService
{
    string GenerateToken(User user);
}

public class TokenService : ITokenService
{
    private readonly IConfiguration _configuration;
    
    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public string GenerateToken(User user)
    {
        var jwtSettings = _configuration.GetSection("JwtSettings");
        var secretKey = jwtSettings["SecretKey"];
        
        // ═══════════════════════════════════════════════════════
        // STEP 1: Create Security Key
        // ═══════════════════════════════════════════════════════
        
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(secretKey));
        
        // ═══════════════════════════════════════════════════════
        // STEP 2: Create Signing Credentials
        // ═══════════════════════════════════════════════════════
        
        var credentials = new SigningCredentials(
            key, 
            SecurityAlgorithms.HmacSha256);
        
        // ═══════════════════════════════════════════════════════
        // STEP 3: Create Claims
        // ═══════════════════════════════════════════════════════
        
        var claims = new[]
        {
            // Standard claims
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            
            // Custom claims
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim("Department", user.Department ?? "General")
        };
        
        // ═══════════════════════════════════════════════════════
        // STEP 4: Create Token
        // ═══════════════════════════════════════════════════════
        
        var expirationMinutes = int.Parse(jwtSettings["ExpirationMinutes"]);
        
        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(expirationMinutes),
            signingCredentials: credentials
        );
        
        // ═══════════════════════════════════════════════════════
        // STEP 5: Serialize Token to String
        // ═══════════════════════════════════════════════════════
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Authentication Controller

```csharp
// File: Controllers/AuthController.cs

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly ITokenService _tokenService;
    private readonly IUserService _userService;
    
    public AuthController(
        ITokenService tokenService, 
        IUserService userService)
    {
        _tokenService = tokenService;
        _userService = userService;
    }
    
    // ═══════════════════════════════════════════════════════════
    // LOGIN ENDPOINT
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        // Step 1: Validate credentials
        var user = await _userService.ValidateCredentialsAsync(
            request.Username, 
            request.Password);
        
        if (user == null)
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }
        
        // Step 2: Generate JWT token
        var token = _tokenService.GenerateToken(user);
        
        // Step 3: Return token
        return Ok(new LoginResponse
        {
            Token = token,
            ExpiresIn = 3600, // seconds
            Username = user.Username,
            Role = user.Role
        });
    }
    
    // ═══════════════════════════════════════════════════════════
    // REGISTER ENDPOINT
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost("register")]
    public async Task<IActionResult> Register([FromBody] RegisterRequest request)
    {
        // Check if user exists
        var existingUser = await _userService.GetByUsernameAsync(request.Username);
        if (existingUser != null)
        {
            return BadRequest(new { message = "Username already taken" });
        }
        
        // Create user
        var user = await _userService.CreateAsync(request);
        
        // Generate token
        var token = _tokenService.GenerateToken(user);
        
        return CreatedAtAction(nameof(Login), new LoginResponse
        {
            Token = token,
            ExpiresIn = 3600,
            Username = user.Username,
            Role = user.Role
        });
    }
}
```

### Request/Response Models

```csharp
// File: Models/AuthModels.cs

public class LoginRequest
{
    [Required]
    public string Username { get; set; }
    
    [Required]
    public string Password { get; set; }
}

public class LoginResponse
{
    public string Token { get; set; }
    public int ExpiresIn { get; set; }
    public string Username { get; set; }
    public string Role { get; set; }
}

public class RegisterRequest
{
    [Required]
    [StringLength(50, MinimumLength = 3)]
    public string Username { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    [Required]
    [StringLength(100, MinimumLength = 6)]
    public string Password { get; set; }
}
```

---

## Protecting API Endpoints

### Using [Authorize] Attribute

```csharp
// File: Controllers/ProductController.cs

[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    // ═══════════════════════════════════════════════════════════
    // PUBLIC ENDPOINT (No authentication required)
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet]
    [AllowAnonymous]
    public IActionResult GetAll()
    {
        // Anyone can access
        return Ok(products);
    }
    
    // ═══════════════════════════════════════════════════════════
    // PROTECTED ENDPOINT (Authentication required)
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("{id}")]
    [Authorize]
    public IActionResult GetById(int id)
    {
        // Only authenticated users can access
        return Ok(product);
    }
    
    // ═══════════════════════════════════════════════════════════
    // ROLE-BASED AUTHORIZATION
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost]
    [Authorize(Roles = "Admin")]
    public IActionResult Create(Product product)
    {
        // Only Admin role can access
        return CreatedAtAction(...);
    }
    
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin,Manager")]
    public IActionResult Delete(int id)
    {
        // Admin OR Manager role can access
        return NoContent();
    }
    
    // ═══════════════════════════════════════════════════════════
    // ACCESSING USER CLAIMS
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("profile")]
    [Authorize]
    public IActionResult GetProfile()
    {
        // Access claims from token
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var username = User.FindFirst(ClaimTypes.Name)?.Value;
        var email = User.FindFirst(ClaimTypes.Email)?.Value;
        var role = User.FindFirst(ClaimTypes.Role)?.Value;
        
        return Ok(new
        {
            UserId = userId,
            Username = username,
            Email = email,
            Role = role
        });
    }
}
```

### Controller-Level Authorization

```csharp
// All endpoints in this controller require authentication
[ApiController]
[Route("api/[controller]")]
[Authorize]  // Applied to all actions
public class SecureController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() { }  // Requires authentication
    
    [HttpPost]
    public IActionResult Post() { }  // Requires authentication
    
    [AllowAnonymous]  // Override for this action
    [HttpGet("public")]
    public IActionResult PublicEndpoint() { }  // No authentication needed
}
```

---

## Role-Based Authorization

### Adding Roles to Claims

```csharp
// In TokenService.GenerateToken()

var claims = new List<Claim>
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Name, user.Username),
};

// Add multiple roles
foreach (var role in user.Roles)
{
    claims.Add(new Claim(ClaimTypes.Role, role));
}
```

### Policy-Based Authorization

```csharp
// File: Program.cs

// Configure authorization policies
builder.Services.AddAuthorization(options =>
{
    // Single role policy
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
    
    // Multiple roles policy
    options.AddPolicy("ManagerOrAdmin", policy =>
        policy.RequireRole("Admin", "Manager"));
    
    // Claim-based policy
    options.AddPolicy("ITDepartment", policy =>
        policy.RequireClaim("Department", "IT"));
    
    // Custom policy
    options.AddPolicy("MinimumAge", policy =>
        policy.RequireAssertion(context =>
        {
            var ageClaim = context.User.FindFirst("Age");
            return ageClaim != null && int.Parse(ageClaim.Value) >= 18;
        }));
});
```

```csharp
// Using policies in controller

[Authorize(Policy = "AdminOnly")]
[HttpPost]
public IActionResult AdminAction() { }

[Authorize(Policy = "ITDepartment")]
[HttpGet("it-resources")]
public IActionResult GetITResources() { }
```

---

## Refresh Tokens

### Refresh Token Implementation

```csharp
// File: Models/RefreshToken.cs

public class RefreshToken
{
    public string Token { get; set; }
    public DateTime Expires { get; set; }
    public bool IsExpired => DateTime.UtcNow >= Expires;
    public DateTime Created { get; set; }
    public string CreatedByIp { get; set; }
    public DateTime? Revoked { get; set; }
    public bool IsActive => Revoked == null && !IsExpired;
}
```

```csharp
// File: Services/TokenService.cs

public RefreshToken GenerateRefreshToken(string ipAddress)
{
    var randomBytes = new byte[64];
    using var rng = RandomNumberGenerator.Create();
    rng.GetBytes(randomBytes);
    
    return new RefreshToken
    {
        Token = Convert.ToBase64String(randomBytes),
        Expires = DateTime.UtcNow.AddDays(7),
        Created = DateTime.UtcNow,
        CreatedByIp = ipAddress
    };
}
```

```csharp
// File: Controllers/AuthController.cs

[HttpPost("refresh-token")]
public async Task<IActionResult> RefreshToken([FromBody] RefreshTokenRequest request)
{
    var user = await _userService.GetByRefreshTokenAsync(request.RefreshToken);
    
    if (user == null || user.RefreshToken.IsExpired)
    {
        return Unauthorized(new { message = "Invalid or expired refresh token" });
    }
    
    // Generate new tokens
    var newAccessToken = _tokenService.GenerateToken(user);
    var newRefreshToken = _tokenService.GenerateRefreshToken(GetIpAddress());
    
    // Update user's refresh token
    await _userService.UpdateRefreshTokenAsync(user.Id, newRefreshToken);
    
    return Ok(new
    {
        AccessToken = newAccessToken,
        RefreshToken = newRefreshToken.Token
    });
}
```

---

## Code Examples

### Complete JWT Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    JWT AUTHENTICATION FLOW                       │
└─────────────────────────────────────────────────────────────────┘

1. USER LOGIN
   ─────────────────────────────────────────────────────────────
   
   Client                              Server
   ──────                              ──────
   POST /api/auth/login
   { username, password }  ───────────────▶  Validate credentials
                                             Generate JWT
                          ◀───────────────  { token, expiresIn }
   Store token locally

2. ACCESSING PROTECTED RESOURCES
   ─────────────────────────────────────────────────────────────
   
   Client                              Server
   ──────                              ──────
   GET /api/products
   Header: Authorization: 
   Bearer eyJhbGciOiJIUzI...  ────────────▶  Validate JWT
                                             Check expiration
                                             Extract claims
                             ◀───────────────  { data }

3. TOKEN EXPIRED
   ─────────────────────────────────────────────────────────────
   
   Client                              Server
   ──────                              ──────
   GET /api/products
   Header: Authorization: 
   Bearer [expired_token]  ──────────────▶  Token expired!
                          ◀──────────────  401 Unauthorized
                                           Header: Token-Expired: true
   
   POST /api/auth/refresh-token
   { refreshToken }  ────────────────────▶  Validate refresh token
                                            Generate new JWT
                     ◀────────────────────  { newToken, newRefreshToken }
```

---

## Best Practices

### 1. Keep Secret Key Secure

```csharp
// ✅ GOOD: Use environment variables or Key Vault
var secretKey = Environment.GetEnvironmentVariable("JWT_SECRET");

// ❌ BAD: Hardcoded in source code
var secretKey = "MySecretKey123";
```

### 2. Use Short Expiration for Access Tokens

```csharp
// ✅ GOOD: Short-lived access token with refresh token
var token = new JwtSecurityToken(
    expires: DateTime.UtcNow.AddMinutes(15),  // Short expiration
    // ...
);

// ❌ BAD: Long-lived access token
var token = new JwtSecurityToken(
    expires: DateTime.UtcNow.AddDays(30),  // Too long!
    // ...
);
```

### 3. Always Use HTTPS

```csharp
// ✅ GOOD: Force HTTPS
app.UseHttpsRedirection();
```

### 4. Don't Store Sensitive Data in JWT

```csharp
// ✅ GOOD: Store only necessary claims
new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
new Claim(ClaimTypes.Role, user.Role)

// ❌ BAD: Storing sensitive data
new Claim("Password", user.Password),  // Never!
new Claim("SSN", user.SSN)             // Never!
```

---

## Interview Questions

### Q1: What is JWT and what are its three parts?
**Answer:** JWT (JSON Web Token) is a compact, self-contained token for securely transmitting information. Three parts:
1. **Header**: Algorithm and token type
2. **Payload**: Claims (user data)
3. **Signature**: Verification using secret key

### Q2: Why use JWT instead of session-based authentication?
**Answer:**
- Stateless (no server storage)
- Scalable (works across multiple servers)
- Works with any client (mobile, web, IoT)
- Self-contained (no database lookup needed)

### Q3: What is the difference between authentication and authorization?
**Answer:**
- **Authentication**: Verifying WHO you are (login)
- **Authorization**: Verifying WHAT you can access (permissions)

### Q4: How do you handle token expiration?
**Answer:** Use refresh tokens:
1. Access token: Short-lived (15-60 minutes)
2. Refresh token: Longer-lived (7-30 days)
3. When access token expires, use refresh token to get new access token

### Q5: Where should JWT be stored on the client?
**Answer:**
- **HttpOnly Cookie**: Most secure for web apps (prevents XSS)
- **localStorage**: Vulnerable to XSS but convenient
- **Memory**: Most secure but lost on refresh

### Q6: What is the [Authorize] attribute?
**Answer:** An attribute that restricts access to controllers or actions to authenticated users. Can be combined with roles or policies for more granular control.

---

## Quick Reference

### JWT Configuration

```csharp
// appsettings.json
"JwtSettings": {
  "SecretKey": "minimum32character...",
  "Issuer": "YourApp",
  "Audience": "YourUsers",
  "ExpirationMinutes": 60
}

// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { ... });

app.UseAuthentication();
app.UseAuthorization();
```

### Authorization Attributes

```csharp
[AllowAnonymous]              // No auth needed
[Authorize]                   // Auth required
[Authorize(Roles = "Admin")]  // Role required
[Authorize(Policy = "...")]   // Policy required
```

### Token Generation

```csharp
var token = new JwtSecurityToken(
    issuer: issuer,
    audience: audience,
    claims: claims,
    expires: DateTime.UtcNow.AddMinutes(60),
    signingCredentials: credentials
);

return new JwtSecurityTokenHandler().WriteToken(token);
```
