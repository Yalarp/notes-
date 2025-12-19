# ⚡ ASP.NET Core vs ASP.NET Framework MVC

## 📚 Table of Contents
- [Overview](#-overview)
- [Key Differences](#-key-differences)
- [Code Comparison](#-code-comparison)
- [Migration Considerations](#-migration-considerations)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ASP.NET EVOLUTION                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ASP.NET Framework MVC (Legacy)                                   │
│   • Windows only                                                   │
│   • Full .NET Framework (4.x)                                      │
│   • IIS hosting required                                           │
│   • System.Web dependency                                          │
│   • Global.asax, Web.config                                        │
│                                                                     │
│   ASP.NET Core MVC (Modern)                                        │
│   • Cross-platform (Windows, Linux, macOS)                         │
│   • .NET Core / .NET 5+                                            │
│   • Any host (Kestrel, IIS, Docker)                                │
│   • Unified MVC and Web API                                        │
│   • Program.cs, appsettings.json                                   │
│   • Built-in DI                                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Key Differences

| Aspect | ASP.NET MVC (Framework) | ASP.NET Core MVC |
|--------|------------------------|------------------|
| **Platform** | Windows only | Cross-platform |
| **Performance** | Slower | Much faster |
| **Hosting** | IIS only | Kestrel, Docker, any |
| **Startup** | Global.asax | Program.cs |
| **Config** | Web.config (XML) | appsettings.json |
| **DI** | Third-party (Ninject, etc.) | Built-in |
| **HTTP Pipeline** | HttpModules/Handlers | Middleware |
| **Static Files** | Automatic | Must configure |
| **Side-by-side** | Not possible | Multiple versions |

---

## 🔶 Code Comparison

### Startup/Configuration

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET FRAMEWORK MVC
// ═══════════════════════════════════════════════════════════════════════════

// Global.asax.cs
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
        
        // Third-party DI (e.g., Ninject)
        var kernel = new StandardKernel();
        kernel.Bind<IRepository>().To<SqlRepository>();
        DependencyResolver.SetResolver(new NinjectResolver(kernel));
    }
}

// App_Start/RouteConfig.cs
public class RouteConfig
{
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
        );
    }
}

// Web.config
<configuration>
  <connectionStrings>
    <add name="DefaultConnection" connectionString="..." />
  </connectionStrings>
</configuration>

// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET CORE MVC
// ═══════════════════════════════════════════════════════════════════════════

// Program.cs (.NET 6+ minimal hosting)
var builder = WebApplication.CreateBuilder(args);

// Built-in DI
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddScoped<IRepository, SqlRepository>();

var app = builder.Build();

// Middleware pipeline
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();

// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyApp;Trusted_Connection=True;"
  }
}
```

### Controllers

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET FRAMEWORK MVC CONTROLLER
// ═══════════════════════════════════════════════════════════════════════════

public class ProductsController : Controller
{
    private IProductRepository _repo;
    
    public ProductsController()
    {
        // Manual or DI container creates
        _repo = DependencyResolver.Current.GetService<IProductRepository>();
    }
    
    public ActionResult Index()
    {
        var products = _repo.GetAll();
        return View(products);
    }
    
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create(Product product)
    {
        if (ModelState.IsValid)
        {
            _repo.Add(product);
            return RedirectToAction("Index");
        }
        return View(product);
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET CORE MVC CONTROLLER
// ═══════════════════════════════════════════════════════════════════════════

public class ProductsController : Controller
{
    private readonly IProductRepository _repo;
    
    // Constructor injection (built-in)
    public ProductsController(IProductRepository repo)
    {
        _repo = repo;
    }
    
    public async Task<IActionResult> Index()
    {
        var products = await _repo.GetAllAsync();
        return View(products);
    }
    
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Create(Product product)
    {
        if (ModelState.IsValid)
        {
            await _repo.AddAsync(product);
            return RedirectToAction(nameof(Index));
        }
        return View(product);
    }
}
```

### Dependency Injection

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET FRAMEWORK - THIRD PARTY DI (NINJECT EXAMPLE)
// ═══════════════════════════════════════════════════════════════════════════

// NinjectWebCommon.cs
private static void RegisterServices(IKernel kernel)
{
    kernel.Bind<IProductRepository>().To<SqlProductRepository>();
    kernel.Bind<IEmailService>().To<SmtpEmailService>().InRequestScope();
}

// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET CORE - BUILT-IN DI
// ═══════════════════════════════════════════════════════════════════════════

// Program.cs
builder.Services.AddTransient<IProductRepository, SqlProductRepository>();
builder.Services.AddScoped<IEmailService, SmtpEmailService>();
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

### Configuration

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET FRAMEWORK - WEB.CONFIG + CONFIGURATIONMANAGER
// ═══════════════════════════════════════════════════════════════════════════

// Web.config
<appSettings>
  <add key="ApiKey" value="abc123" />
</appSettings>

// Reading
string apiKey = ConfigurationManager.AppSettings["ApiKey"];
string connStr = ConfigurationManager.ConnectionStrings["Default"].ConnectionString;

// ═══════════════════════════════════════════════════════════════════════════
// ASP.NET CORE - ICONFIGURATION
// ═══════════════════════════════════════════════════════════════════════════

// appsettings.json
{
  "ApiSettings": {
    "ApiKey": "abc123"
  }
}

// Reading via DI
public class MyService
{
    private readonly IConfiguration _config;
    
    public MyService(IConfiguration config)
    {
        _config = config;
    }
    
    public void DoWork()
    {
        string apiKey = _config["ApiSettings:ApiKey"];
        string connStr = _config.GetConnectionString("Default");
    }
}

// Or strongly typed
public class ApiSettings { public string ApiKey { get; set; } }
builder.Services.Configure<ApiSettings>(builder.Configuration.GetSection("ApiSettings"));
```

---

## 🔵 Migration Considerations

### Key Migration Steps

1. **Project Structure**: Create new ASP.NET Core project
2. **NuGet Packages**: Replace Framework packages with Core equivalents
3. **Startup**: Move Global.asax logic to Program.cs
4. **Configuration**: Convert Web.config to appsettings.json
5. **DI**: Use built-in DI instead of third-party
6. **Controllers**: Update return types (ActionResult → IActionResult)
7. **Views**: Update Razor syntax if needed
8. **Static Files**: Configure middleware

---

## 🎤 Interview Questions

**Q1: Main difference between ASP.NET MVC and Core?**
> Core is cross-platform, faster, with built-in DI and middleware pipeline. MVC is Windows/IIS only.

**Q2: What replaced Global.asax?**
> Program.cs and middleware pipeline in ASP.NET Core.

**Q3: How is DI different?**
> Core has built-in DI container. Framework required third-party (Ninject, Unity).

**Q4: What is Kestrel?**
> Cross-platform web server built for ASP.NET Core. Can run standalone or behind IIS/Nginx.

**Q5: Why migrate to Core?**
> Performance, cross-platform, modern architecture, active development, built-in DI.

---

## 📝 Summary

| Feature | Framework MVC | Core MVC |
|---------|---------------|----------|
| **Platform** | Windows | Cross-platform |
| **Startup** | Global.asax | Program.cs |
| **Config** | Web.config | appsettings.json |
| **DI** | Third-party | Built-in |
| **Host** | IIS | Kestrel, any |
