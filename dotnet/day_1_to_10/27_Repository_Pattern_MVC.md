# 🏗️ Repository Pattern and MVC in ASP.NET Core

## 📚 Table of Contents
- [Repository Pattern](#-repository-pattern)
- [Unit of Work](#-unit-of-work)
- [MVC Architecture](#-mvc-architecture)
- [ASP.NET Core MVC](#-aspnet-core-mvc)
- [Interview Questions](#-interview-questions)

---

## 🎯 Repository Pattern

**Repository** abstracts data access logic, making code testable and maintainable.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REPOSITORY PATTERN                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Controller ────▶ Service ────▶ Repository ────▶ DbContext        │
│                        │              │              │              │
│                        │              │              ▼              │
│                    Business       Data Access    Database          │
│                    Logic          Abstraction                      │
│                                                                     │
│   Benefits:                                                         │
│   • Testable (mock repository)                                     │
│   • Consistent data access                                         │
│   • Single place to change DB logic                                │
│   • Swappable data sources                                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Generic Repository

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// GENERIC REPOSITORY INTERFACE
// ═══════════════════════════════════════════════════════════════════════════

public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
}

// ═══════════════════════════════════════════════════════════════════════════
// GENERIC REPOSITORY IMPLEMENTATION
// ═══════════════════════════════════════════════════════════════════════════

public class Repository<T> : IRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }
    
    public void Update(T entity)
    {
        _dbSet.Update(entity);
    }
    
    public void Remove(T entity)
    {
        _dbSet.Remove(entity);
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// SPECIFIC REPOSITORY
// ═══════════════════════════════════════════════════════════════════════════

public interface IEmployeeRepository : IRepository<Employee>
{
    Task<IEnumerable<Employee>> GetByDepartmentAsync(int deptId);
    Task<Employee> GetWithDetailsAsync(int id);
}

public class EmployeeRepository : Repository<Employee>, IEmployeeRepository
{
    public EmployeeRepository(AppDbContext context) : base(context) { }
    
    public async Task<IEnumerable<Employee>> GetByDepartmentAsync(int deptId)
    {
        return await _dbSet.Where(e => e.DepartmentId == deptId).ToListAsync();
    }
    
    public async Task<Employee> GetWithDetailsAsync(int id)
    {
        return await _dbSet
            .Include(e => e.Department)
            .FirstOrDefaultAsync(e => e.Id == id);
    }
}
```

---

## 🔶 Unit of Work

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// UNIT OF WORK - SINGLE TRANSACTION ACROSS REPOSITORIES
// ═══════════════════════════════════════════════════════════════════════════

public interface IUnitOfWork : IDisposable
{
    IEmployeeRepository Employees { get; }
    IDepartmentRepository Departments { get; }
    Task<int> SaveChangesAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    
    private IEmployeeRepository _employees;
    private IDepartmentRepository _departments;
    
    public UnitOfWork(AppDbContext context)
    {
        _context = context;
    }
    
    public IEmployeeRepository Employees =>
        _employees ??= new EmployeeRepository(_context);
    
    public IDepartmentRepository Departments =>
        _departments ??= new DepartmentRepository(_context);
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    public void Dispose()
    {
        _context.Dispose();
    }
}

// Usage
public class EmployeeService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public EmployeeService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task TransferEmployee(int empId, int newDeptId)
    {
        var emp = await _unitOfWork.Employees.GetByIdAsync(empId);
        var dept = await _unitOfWork.Departments.GetByIdAsync(newDeptId);
        
        emp.DepartmentId = newDeptId;
        _unitOfWork.Employees.Update(emp);
        
        // Single transaction for all changes
        await _unitOfWork.SaveChangesAsync();
    }
}

// Register in DI
services.AddScoped<IUnitOfWork, UnitOfWork>();
services.AddScoped<IEmployeeRepository, EmployeeRepository>();
```

---

## 🔵 MVC Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MVC PATTERN                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   USER ────▶ CONTROLLER ────▶ MODEL                                │
│     ▲            │              │                                   │
│     │            │              │                                   │
│     └─── VIEW ◀──┴──────────────┘                                  │
│                                                                     │
│   MODEL:      Data and business logic                              │
│   VIEW:       UI presentation (HTML, Razor)                        │
│   CONTROLLER: Handles requests, coordinates Model and View        │
│                                                                     │
│   Request Flow:                                                     │
│   1. User request → Controller                                     │
│   2. Controller → Get/Modify Model                                 │
│   3. Controller → Select View                                      │
│   4. View → Render with Model data                                 │
│   5. Response → User                                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🟢 ASP.NET Core MVC

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CONTROLLER
// ═══════════════════════════════════════════════════════════════════════════

[Route("api/[controller]")]
[ApiController]
public class EmployeesController : ControllerBase
{
    private readonly IEmployeeRepository _repo;
    
    public EmployeesController(IEmployeeRepository repo)
    {
        _repo = repo;
    }
    
    // GET api/employees
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Employee>>> GetAll()
    {
        var employees = await _repo.GetAllAsync();
        return Ok(employees);
    }
    
    // GET api/employees/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Employee>> GetById(int id)
    {
        var employee = await _repo.GetByIdAsync(id);
        if (employee == null)
            return NotFound();
        return Ok(employee);
    }
    
    // POST api/employees
    [HttpPost]
    public async Task<ActionResult<Employee>> Create(Employee employee)
    {
        await _repo.AddAsync(employee);
        await _repo.SaveChangesAsync();
        
        return CreatedAtAction(nameof(GetById), new { id = employee.Id }, employee);
    }
    
    // PUT api/employees/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, Employee employee)
    {
        if (id != employee.Id)
            return BadRequest();
        
        _repo.Update(employee);
        await _repo.SaveChangesAsync();
        
        return NoContent();
    }
    
    // DELETE api/employees/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var employee = await _repo.GetByIdAsync(id);
        if (employee == null)
            return NotFound();
        
        _repo.Remove(employee);
        await _repo.SaveChangesAsync();
        
        return NoContent();
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// PROGRAM.CS SETUP
// ═══════════════════════════════════════════════════════════════════════════

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();

var app = builder.Build();

// Middleware pipeline
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### MVC with Views (Razor)

```csharp
// Controller returning View
public class HomeController : Controller
{
    private readonly IEmployeeRepository _repo;
    
    public HomeController(IEmployeeRepository repo)
    {
        _repo = repo;
    }
    
    // GET /Home/Index
    public async Task<IActionResult> Index()
    {
        var employees = await _repo.GetAllAsync();
        return View(employees);  // Pass model to view
    }
    
    // GET /Home/Details/5
    public async Task<IActionResult> Details(int id)
    {
        var employee = await _repo.GetByIdAsync(id);
        if (employee == null)
            return NotFound();
        return View(employee);
    }
}
```

```html
<!-- Views/Home/Index.cshtml -->
@model IEnumerable<Employee>

<h1>Employees</h1>
<table>
    <thead>
        <tr><th>Name</th><th>Email</th></tr>
    </thead>
    <tbody>
        @foreach (var emp in Model)
        {
            <tr>
                <td>@emp.Name</td>
                <td>@emp.Email</td>
            </tr>
        }
    </tbody>
</table>
```

---

## 🎤 Interview Questions

**Q1: What is Repository Pattern?**
> Abstraction layer between business logic and data access. Provides consistent interface for data operations.

**Q2: Why use Repository Pattern?**
> Testability (mock repos), single source for data logic, swappable data sources, cleaner code.

**Q3: What is Unit of Work?**
> Maintains list of objects affected by a business transaction and coordinates saving changes as single transaction.

**Q4: Controller vs ControllerBase?**
> Controller includes View support (for MVC). ControllerBase is for API-only (no View methods).

**Q5: Common action result types?**
> Ok(), NotFound(), BadRequest(), CreatedAtAction(), NoContent(), View()

---

## 📝 Summary

| Pattern/Concept | Purpose |
|-----------------|---------|
| **Repository** | Abstract data access |
| **Unit of Work** | Single transaction scope |
| **Controller** | Handle HTTP requests |
| **Model** | Data and logic |
| **View** | UI presentation |
