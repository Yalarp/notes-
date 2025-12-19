# Session and State Management in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [State Management Options](#state-management-options)
3. [Session State](#session-state)
4. [Cookies](#cookies)
5. [Query Strings](#query-strings)
6. [Hidden Fields](#hidden-fields)
7. [Cache](#cache)
8. [Code Examples](#code-examples)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
State management is like **hotel guest tracking**:

```
Session State    → Room key card (identifies guest during stay)
Cookies          → Loyalty card (remembers preferences across visits)
Query Strings    → Room number on receipt (visible, temporary)
Hidden Fields    → Reservation details in file (hidden but accessible)
Cache            → Hotel's guest preference database (shared info)
```

### Why State Management?

HTTP is **stateless** - each request is independent. State management enables:
- User authentication tracking
- Shopping cart persistence
- User preferences
- Form data across pages

---

## State Management Options

### Comparison Table

| Method | Storage | Lifetime | Security | Use Case |
|--------|---------|----------|----------|----------|
| **Session** | Server | Until timeout/close | Secure | Shopping cart, user data |
| **Cookies** | Client | Configurable | Less secure | Remember me, preferences |
| **TempData** | Session/Cookie | One redirect | Secure | Flash messages |
| **Query String** | URL | Request only | Visible | Search filters, page number |
| **Hidden Fields** | Form | Form submit | Less secure | Form state |
| **Cache** | Server | Configurable | Secure | Shared data, performance |

### State Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                  STATE MANAGEMENT FLOW                           │
└─────────────────────────────────────────────────────────────────┘

CLIENT SIDE                          SERVER SIDE
─────────────────────────────────────────────────────────────────
┌─────────────┐                    ┌─────────────────────────────┐
│  Browser    │                    │       ASP.NET Core          │
│             │                    │                             │
│ ┌─────────┐ │    Request +       │ ┌─────────────────────────┐ │
│ │ Cookies │─┼───Session ID─────▶ │ │     Session Store       │ │
│ └─────────┘ │                    │ │  (Memory/Redis/SQL)     │ │
│             │                    │ └─────────────────────────┘ │
│ ┌─────────┐ │                    │                             │
│ │ Local   │ │                    │ ┌─────────────────────────┐ │
│ │ Storage │ │                    │ │        Cache            │ │
│ └─────────┘ │                    │ │   (In-Memory/Dist.)     │ │
│             │◀────Response─────┼─│ └─────────────────────────┘ │
└─────────────┘                    └─────────────────────────────┘
```

---

## Session State

### What is Session State?

Session stores user data on the **server** and uses a cookie to track the session ID.

### Configuring Session

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// STEP 1: Add Session Services
// ═══════════════════════════════════════════════════════════════

builder.Services.AddDistributedMemoryCache(); // Required for session

builder.Services.AddSession(options =>
{
    // Line 1: Session timeout (default: 20 minutes)
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    
    // Line 2: Cookie is essential (GDPR compliance)
    options.Cookie.IsEssential = true;
    
    // Line 3: Cookie accessible only via HTTP (not JavaScript)
    options.Cookie.HttpOnly = true;
    
    // Line 4: Cookie name
    options.Cookie.Name = ".MyApp.Session";
    
    // Line 5: Secure cookie (HTTPS only)
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

builder.Services.AddControllersWithViews();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// STEP 2: Use Session Middleware
// ═══════════════════════════════════════════════════════════════

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

// Line 6: Add session middleware (MUST be before endpoints)
app.UseSession();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Using Session in Controller

```csharp
// File: Controllers/CartController.cs

using Microsoft.AspNetCore.Http;
using System.Text.Json;

public class CartController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // STORING VALUES IN SESSION
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult AddToCart(int productId)
    {
        // Line 1: Store string
        HttpContext.Session.SetString("LastProduct", "Product " + productId);
        
        // Line 2: Store integer
        HttpContext.Session.SetInt32("CartCount", 5);
        
        // Line 3: Store complex object (serialize to JSON)
        var cart = new ShoppingCart
        {
            Items = new List<CartItem>
            {
                new CartItem { ProductId = productId, Quantity = 1 }
            }
        };
        
        var cartJson = JsonSerializer.Serialize(cart);
        HttpContext.Session.SetString("Cart", cartJson);
        
        return RedirectToAction("Index");
    }
    
    // ═══════════════════════════════════════════════════════════
    // READING VALUES FROM SESSION
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult Index()
    {
        // Line 4: Read string (null if not found)
        var lastProduct = HttpContext.Session.GetString("LastProduct");
        
        // Line 5: Read integer (null if not found)
        var cartCount = HttpContext.Session.GetInt32("CartCount");
        
        // Line 6: Read complex object
        var cartJson = HttpContext.Session.GetString("Cart");
        ShoppingCart cart = null;
        
        if (!string.IsNullOrEmpty(cartJson))
        {
            cart = JsonSerializer.Deserialize<ShoppingCart>(cartJson);
        }
        
        // Line 7: Always check for null before using
        ViewBag.LastProduct = lastProduct ?? "No product selected";
        ViewBag.CartCount = cartCount ?? 0;
        ViewBag.Cart = cart;
        
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // REMOVING SESSION VALUES
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult ClearCart()
    {
        // Line 8: Remove specific key
        HttpContext.Session.Remove("Cart");
        
        // Line 9: Clear all session data
        HttpContext.Session.Clear();
        
        return RedirectToAction("Index");
    }
}
```

### Session Extension Methods

```csharp
// File: Extensions/SessionExtensions.cs

public static class SessionExtensions
{
    // ═══════════════════════════════════════════════════════════
    // GENERIC SET/GET FOR COMPLEX OBJECTS
    // ═══════════════════════════════════════════════════════════
    
    public static void Set<T>(this ISession session, string key, T value)
    {
        session.SetString(key, JsonSerializer.Serialize(value));
    }
    
    public static T? Get<T>(this ISession session, string key)
    {
        var value = session.GetString(key);
        return value == null ? default : JsonSerializer.Deserialize<T>(value);
    }
}

// Usage:
// HttpContext.Session.Set("Cart", cartObject);
// var cart = HttpContext.Session.Get<ShoppingCart>("Cart");
```

### Session Lifetime

```
┌─────────────────────────────────────────────────────────────────┐
│                    SESSION LIFECYCLE                             │
└─────────────────────────────────────────────────────────────────┘

User visits site
     │
     ▼
┌───────────────────────────────────────────────────────────────┐
│  Session Created                                              │
│  - Session ID generated                                       │
│  - Session cookie sent to browser                             │
│  - Server-side storage initialized                            │
└───────────────────────────────────────────────────────────────┘
     │
     │ User activity resets timeout
     ▼
┌───────────────────────────────────────────────────────────────┐
│  Session Active                                               │
│  - IdleTimeout countdown restarts                             │
│  - Data persists across requests                              │
└───────────────────────────────────────────────────────────────┘
     │
     │ No activity for IdleTimeout (default: 20 min)
     ▼
┌───────────────────────────────────────────────────────────────┐
│  Session Expired                                              │
│  - Data deleted from server                                   │
│  - Cookie invalidated                                         │
│  - Next request gets new session                              │
└───────────────────────────────────────────────────────────────┘
```

---

## Cookies

### What are Cookies?

Cookies store small pieces of data on the **client browser**.

### Creating and Reading Cookies

```csharp
// File: Controllers/PreferencesController.cs

public class PreferencesController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // CREATING COOKIES
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult SetPreferences(string theme, string language)
    {
        // Line 1: Simple cookie
        Response.Cookies.Append("Theme", theme);
        
        // Line 2: Cookie with options
        var cookieOptions = new CookieOptions
        {
            // Expires in 30 days
            Expires = DateTimeOffset.Now.AddDays(30),
            
            // Only accessible via HTTP (not JavaScript)
            HttpOnly = true,
            
            // HTTPS only
            Secure = true,
            
            // Cookie scope
            Path = "/",
            
            // SameSite policy (Strict, Lax, None)
            SameSite = SameSiteMode.Lax,
            
            // Essential for GDPR
            IsEssential = false
        };
        
        Response.Cookies.Append("Language", language, cookieOptions);
        
        // Line 3: Serialize complex object
        var preferences = new UserPreferences
        {
            Theme = theme,
            Language = language,
            FontSize = 14
        };
        
        var prefJson = JsonSerializer.Serialize(preferences);
        Response.Cookies.Append("UserPrefs", prefJson, cookieOptions);
        
        return RedirectToAction("Index", "Home");
    }
    
    // ═══════════════════════════════════════════════════════════
    // READING COOKIES
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult GetPreferences()
    {
        // Line 4: Read simple cookie
        var theme = Request.Cookies["Theme"];
        var language = Request.Cookies["Language"];
        
        // Line 5: Read complex object
        var prefJson = Request.Cookies["UserPrefs"];
        UserPreferences preferences = null;
        
        if (!string.IsNullOrEmpty(prefJson))
        {
            preferences = JsonSerializer.Deserialize<UserPreferences>(prefJson);
        }
        
        ViewBag.Theme = theme ?? "light";
        ViewBag.Language = language ?? "en";
        ViewBag.Preferences = preferences;
        
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // DELETING COOKIES
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult ClearPreferences()
    {
        // Line 6: Delete cookie by setting expired date
        Response.Cookies.Delete("Theme");
        Response.Cookies.Delete("Language");
        Response.Cookies.Delete("UserPrefs");
        
        return RedirectToAction("Index", "Home");
    }
}
```

### Cookie Policy

```csharp
// File: Program.cs

// Configure cookie policy (GDPR compliance)
builder.Services.Configure<CookiePolicyOptions>(options =>
{
    // Line 1: Require consent for non-essential cookies
    options.CheckConsentNeeded = context => true;
    
    // Line 2: Minimum SameSite mode
    options.MinimumSameSitePolicy = SameSiteMode.Lax;
});

var app = builder.Build();

// Line 3: Use cookie policy middleware
app.UseCookiePolicy();
```

---

## Query Strings

### Using Query Strings

```csharp
// File: Controllers/SearchController.cs

public class SearchController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // READING QUERY STRINGS
    // ═══════════════════════════════════════════════════════════
    
    // URL: /Search?term=laptop&category=electronics&page=2
    public IActionResult Index(string term, string category, int page = 1)
    {
        // Line 1: Parameters bound from query string automatically
        ViewBag.SearchTerm = term;
        ViewBag.Category = category;
        ViewBag.Page = page;
        
        // Line 2: Manual access via Request.Query
        var sortBy = Request.Query["sortBy"].ToString();
        var sortOrder = Request.Query["order"].ToString();
        
        // Perform search...
        var results = SearchProducts(term, category, page);
        
        return View(results);
    }
    
    // ═══════════════════════════════════════════════════════════
    // GENERATING QUERY STRINGS
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult NextPage(string term, string category, int currentPage)
    {
        // Line 3: Redirect with query string parameters
        return RedirectToAction("Index", new
        {
            term = term,
            category = category,
            page = currentPage + 1
        });
        // Generates: /Search?term=laptop&category=electronics&page=3
    }
}
```

### Query Strings in Views

```html
<!-- Generating query string links in view -->

<!-- Line 1: Using Tag Helper with route values -->
<a asp-action="Index" 
   asp-route-term="@Model.SearchTerm"
   asp-route-category="@Model.Category"
   asp-route-page="@(Model.CurrentPage + 1)">
    Next Page
</a>

<!-- Line 2: Using Url.Action -->
<a href="@Url.Action("Index", new { term = "laptop", page = 2 })">
    Search Laptops
</a>
```

---

## Hidden Fields

### Using Hidden Fields

```html
<!-- File: Views/Order/Create.cshtml -->

@model OrderViewModel

<form asp-action="Create" method="post">
    <!-- Line 1: Hidden field for order ID -->
    <input type="hidden" asp-for="OrderId" />
    
    <!-- Line 2: Hidden field for anti-forgery token (automatic) -->
    @Html.AntiForgeryToken()
    
    <!-- Line 3: Hidden field for product ID -->
    <input type="hidden" name="ProductId" value="@Model.ProductId" />
    
    <!-- Line 4: Hidden field with ViewData -->
    <input type="hidden" name="UserId" value="@ViewData["UserId"]" />
    
    <!-- Visible form fields -->
    <div class="form-group">
        <label asp-for="Quantity"></label>
        <input asp-for="Quantity" class="form-control" />
    </div>
    
    <button type="submit" class="btn btn-primary">Place Order</button>
</form>
```

```csharp
// Controller receiving hidden field values
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(OrderViewModel model)
{
    // OrderId, ProductId, and UserId are bound from hidden fields
    if (ModelState.IsValid)
    {
        // Process order...
    }
    return View(model);
}
```

---

## Cache

### In-Memory Cache

```csharp
// File: Program.cs

// Register memory cache
builder.Services.AddMemoryCache();
```

```csharp
// File: Controllers/ProductController.cs

using Microsoft.Extensions.Caching.Memory;

public class ProductController : Controller
{
    private readonly IMemoryCache _cache;
    private readonly IProductService _productService;
    
    public ProductController(IMemoryCache cache, IProductService productService)
    {
        _cache = cache;
        _productService = productService;
    }
    
    public IActionResult Index()
    {
        // Line 1: Define cache key
        const string cacheKey = "AllProducts";
        
        // Line 2: Try to get from cache
        if (!_cache.TryGetValue(cacheKey, out List<Product> products))
        {
            // Line 3: Not in cache, get from database
            products = _productService.GetAllProducts();
            
            // Line 4: Set cache options
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5))   // Reset on access
                .SetAbsoluteExpiration(TimeSpan.FromHours(1))    // Max lifetime
                .SetPriority(CacheItemPriority.Normal);
            
            // Line 5: Store in cache
            _cache.Set(cacheKey, products, cacheOptions);
        }
        
        return View(products);
    }
    
    // ═══════════════════════════════════════════════════════════
    // USING GetOrCreate (SIMPLER)
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult Featured()
    {
        var featuredProducts = _cache.GetOrCreate("FeaturedProducts", entry =>
        {
            entry.SlidingExpiration = TimeSpan.FromMinutes(10);
            return _productService.GetFeaturedProducts();
        });
        
        return View(featuredProducts);
    }
    
    // ═══════════════════════════════════════════════════════════
    // INVALIDATING CACHE
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost]
    public IActionResult Create(Product product)
    {
        _productService.Add(product);
        
        // Line 6: Remove cached data when data changes
        _cache.Remove("AllProducts");
        _cache.Remove("FeaturedProducts");
        
        return RedirectToAction("Index");
    }
}
```

### Distributed Cache (Redis)

```csharp
// File: Program.cs

// For production with multiple servers
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp_";
});
```

---

## Code Examples

### Complete Shopping Cart Example

```csharp
// File: Controllers/CartController.cs

public class CartController : Controller
{
    private const string CartSessionKey = "ShoppingCart";
    
    // Get cart from session
    private ShoppingCart GetCart()
    {
        var cart = HttpContext.Session.Get<ShoppingCart>(CartSessionKey);
        return cart ?? new ShoppingCart { Items = new List<CartItem>() };
    }
    
    // Save cart to session
    private void SaveCart(ShoppingCart cart)
    {
        HttpContext.Session.Set(CartSessionKey, cart);
    }
    
    public IActionResult Index()
    {
        var cart = GetCart();
        return View(cart);
    }
    
    [HttpPost]
    public IActionResult AddItem(int productId, string productName, decimal price)
    {
        var cart = GetCart();
        
        var existingItem = cart.Items.FirstOrDefault(i => i.ProductId == productId);
        
        if (existingItem != null)
        {
            existingItem.Quantity++;
        }
        else
        {
            cart.Items.Add(new CartItem
            {
                ProductId = productId,
                ProductName = productName,
                Price = price,
                Quantity = 1
            });
        }
        
        SaveCart(cart);
        
        TempData["Message"] = $"{productName} added to cart!";
        return RedirectToAction("Index");
    }
    
    [HttpPost]
    public IActionResult RemoveItem(int productId)
    {
        var cart = GetCart();
        var item = cart.Items.FirstOrDefault(i => i.ProductId == productId);
        
        if (item != null)
        {
            cart.Items.Remove(item);
            SaveCart(cart);
        }
        
        return RedirectToAction("Index");
    }
    
    public IActionResult Clear()
    {
        HttpContext.Session.Remove(CartSessionKey);
        return RedirectToAction("Index");
    }
}
```

---

## Best Practices

### 1. Always Check for Null

```csharp
// ✅ GOOD: Null check before use
var name = HttpContext.Session.GetString("Name");
if (!string.IsNullOrEmpty(name))
{
    // Use name
}

// ❌ BAD: No null check
var name = HttpContext.Session.GetString("Name");
var greeting = "Hello, " + name; // Might be null!
```

### 2. Set Session Timeout Appropriately

```csharp
// ✅ GOOD: Reasonable timeout
options.IdleTimeout = TimeSpan.FromMinutes(30);

// ❌ BAD: Too long (wastes server memory)
options.IdleTimeout = TimeSpan.FromHours(24);
```

### 3. Don't Store Sensitive Data in Cookies

```csharp
// ✅ GOOD: Store in session (server-side)
HttpContext.Session.SetString("UserId", userId);

// ❌ BAD: Store in cookie (client-side, can be tampered)
Response.Cookies.Append("UserId", userId);
```

### 4. Use HttpOnly for Security

```csharp
// ✅ GOOD: HttpOnly prevents JavaScript access
options.Cookie.HttpOnly = true;

// ❌ BAD: Cookie accessible to JavaScript (XSS risk)
options.Cookie.HttpOnly = false;
```

---

## Interview Questions

### Q1: What is the difference between Session and Cookies?
**Answer:**
- **Session**: Stored on server, more secure, larger capacity, expires on timeout/close
- **Cookies**: Stored on client browser, less secure, limited size (4KB), configurable expiration

### Q2: What is the default session timeout in ASP.NET Core?
**Answer:** 20 minutes of idle time. Can be configured using `options.IdleTimeout`.

### Q3: How do you store complex objects in Session?
**Answer:** Serialize to JSON using `JsonSerializer.Serialize()`, then store with `SetString()`. Retrieve with `GetString()` and deserialize.

### Q4: What is HttpOnly cookie flag?
**Answer:** When set to `true`, the cookie cannot be accessed by JavaScript (`document.cookie`), protecting against XSS attacks.

### Q5: What is the difference between SlidingExpiration and AbsoluteExpiration in cache?
**Answer:**
- **SlidingExpiration**: Resets countdown on each access
- **AbsoluteExpiration**: Fixed expiration time regardless of access

### Q6: Why is `UseSession()` middleware position important?
**Answer:** It must be placed after `UseRouting()` and before `MapControllerRoute()` to ensure session is available in controllers.

---

## Quick Reference

### Session Methods

```csharp
// Store
HttpContext.Session.SetString("Key", "value");
HttpContext.Session.SetInt32("Key", 123);

// Retrieve
var str = HttpContext.Session.GetString("Key");
var num = HttpContext.Session.GetInt32("Key");

// Remove
HttpContext.Session.Remove("Key");
HttpContext.Session.Clear();
```

### Cookie Methods

```csharp
// Create
Response.Cookies.Append("Key", "value", options);

// Read
var value = Request.Cookies["Key"];

// Delete
Response.Cookies.Delete("Key");
```

### Cache Methods

```csharp
// Store
_cache.Set("Key", value, options);

// Retrieve
var value = _cache.Get<Type>("Key");
var value = _cache.GetOrCreate("Key", entry => GetValue());

// Remove
_cache.Remove("Key");
```
