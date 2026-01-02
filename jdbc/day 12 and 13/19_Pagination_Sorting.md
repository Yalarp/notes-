# Pagination and Sorting - Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Pageable Interface](#pageable-interface)
3. [PageRequest](#pagerequest)
4. [Page vs Slice](#page-vs-slice)
5. [Sorting](#sorting)
6. [Combined Pagination and Sorting](#combined-pagination-and-sorting)
7. [REST API with Pagination](#rest-api-with-pagination)
8. [Summary](#summary)

---

## Introduction

Spring Data JPA provides built-in support for pagination and sorting through the `Pageable` and `Sort` interfaces. This is essential for large datasets.

> [!IMPORTANT]
> **Pagination** splits large result sets into smaller chunks (pages), improving performance and user experience.

---

## Pageable Interface

### What is Pageable?

`Pageable` is an interface that contains pagination information:
- Page number (0-indexed)
- Page size
- Sort information

### Repository Support

```java
@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Built-in methods accept Pageable
    Page<Book> findAll(Pageable pageable);
    
    // Custom methods with Pageable
    Page<Book> findByAuthor(String author, Pageable pageable);
    
    Page<Book> findByPriceGreaterThan(double price, Pageable pageable);
}
```

---

## PageRequest

### Creating Pageable Objects

```java
// Basic pagination (page 0, size 10)
Pageable pageable = PageRequest.of(0, 10);

// With sorting
Pageable pageable = PageRequest.of(0, 10, Sort.by("price"));

// With direction
Pageable pageable = PageRequest.of(0, 10, Sort.by(Sort.Direction.DESC, "price"));

// Multiple sort fields
Pageable pageable = PageRequest.of(0, 10, 
    Sort.by("author").ascending()
        .and(Sort.by("price").descending())
);
```

### Service Layer Usage

```java
@Service
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    public Page<Book> getBooks(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return bookRepository.findAll(pageable);
    }
    
    public Page<Book> getBooksSortedByPrice(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("price").descending());
        return bookRepository.findAll(pageable);
    }
}
```

---

## Page vs Slice

### Page<T>

`Page<T>` includes total count (requires extra COUNT query):

```java
Page<Book> page = bookRepository.findAll(pageable);

// Available methods
page.getContent();        // List<Book> - actual data
page.getTotalElements();  // Total count in database
page.getTotalPages();     // Total number of pages
page.getNumber();         // Current page number (0-indexed)
page.getSize();           // Page size
page.getNumberOfElements(); // Elements in current page
page.hasContent();        // Has any content?
page.hasNext();           // Has next page?
page.hasPrevious();       // Has previous page?
page.isFirst();           // Is first page?
page.isLast();            // Is last page?
```

### Slice<T>

`Slice<T>` doesn't include total count (faster, one less query):

```java
Slice<Book> slice = bookRepository.findAll(pageable);

slice.getContent();       // List<Book>
slice.getNumber();        // Current page
slice.getSize();          // Page size
slice.hasNext();          // Has next page?
slice.hasPrevious();      // Has previous?

// NOT AVAILABLE in Slice:
// slice.getTotalElements() - NO!
// slice.getTotalPages() - NO!
```

### When to Use Which?

| Use Page<T> | Use Slice<T> |
|-------------|--------------|
| Need total count | Don't need total |
| Classic pagination with page numbers | Infinite scroll |
| Smaller datasets | Very large datasets |
| User needs "Page X of Y" | User just clicks "Load More" |

---

## Sorting

### Sort Class

```java
// Single field (ascending by default)
Sort sort = Sort.by("price");

// Descending
Sort sort = Sort.by(Sort.Direction.DESC, "price");

// Multiple fields
Sort sort = Sort.by("author").ascending()
                .and(Sort.by("price").descending());

// Shorthand
Sort sort = Sort.by("author", "price");  // Both ascending
```

### Using Sort Directly

```java
// Repository method
List<Book> findAllByOrderByPriceAsc();
List<Book> findAllByOrderByPriceDesc();

// Or with Sort parameter
List<Book> findAll(Sort sort);
```

### In Repository

```java
@Query("SELECT b FROM Book b")
List<Book> findAllSorted(Sort sort);

// Usage
List<Book> books = bookRepository.findAllSorted(Sort.by("price").descending());
```

---

## Combined Pagination and Sorting

### Controller Implementation

```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    @Autowired
    private BookRepository bookRepository;
    
    @GetMapping
    public Page<Book> getBooks(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String direction) {
        
        Sort sort = direction.equalsIgnoreCase("desc") 
            ? Sort.by(sortBy).descending() 
            : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        
        return bookRepository.findAll(pageable);
    }
}
```

### Request Examples

```
GET /api/books                          → Default: page 0, size 10
GET /api/books?page=0&size=20           → First 20 items
GET /api/books?page=1&size=10           → Items 11-20
GET /api/books?sortBy=price             → Sort by price ascending
GET /api/books?sortBy=price&direction=desc → Sort by price descending
GET /api/books?page=0&size=10&sortBy=title → Paginated and sorted
```

---

## REST API with Pagination

### Response Structure

```json
{
    "content": [
        {"id": 1, "title": "Book 1", "price": 100},
        {"id": 2, "title": "Book 2", "price": 200}
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 10,
        "sort": {
            "sorted": true,
            "unsorted": false
        },
        "offset": 0,
        "paged": true
    },
    "totalElements": 100,
    "totalPages": 10,
    "size": 10,
    "number": 0,
    "first": true,
    "last": false,
    "numberOfElements": 10,
    "empty": false
}
```

### Custom Response DTO

```java
public class PageResponse<T> {
    private List<T> content;
    private int pageNumber;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
    
    public static <T> PageResponse<T> from(Page<T> page) {
        PageResponse<T> response = new PageResponse<>();
        response.content = page.getContent();
        response.pageNumber = page.getNumber();
        response.pageSize = page.getSize();
        response.totalElements = page.getTotalElements();
        response.totalPages = page.getTotalPages();
        response.first = page.isFirst();
        response.last = page.isLast();
        return response;
    }
}
```

### Usage

```java
@GetMapping
public PageResponse<Book> getBooks(@RequestParam int page, @RequestParam int size) {
    Page<Book> bookPage = bookRepository.findAll(PageRequest.of(page, size));
    return PageResponse.from(bookPage);
}
```

---

## Summary

### Key Takeaways

1. Use **Pageable** for pagination information
2. Use **PageRequest.of()** to create Pageable
3. **Page<T>** includes total count
4. **Slice<T>** is faster (no count query)
5. Use **Sort** for ordering results
6. Combine **pagination and sorting** as needed

### Quick Reference

```java
// Basic pagination
PageRequest.of(page, size)

// With sorting
PageRequest.of(page, size, Sort.by("field"))

// Descending
PageRequest.of(page, size, Sort.by("field").descending())

// Multiple sorts
PageRequest.of(page, size, 
    Sort.by("field1").ascending()
        .and(Sort.by("field2").descending()))
```

### Page Methods Summary

| Method | Returns |
|--------|---------|
| `getContent()` | List of items |
| `getTotalElements()` | Total count |
| `getTotalPages()` | Total pages |
| `getNumber()` | Current page (0-indexed) |
| `hasNext()` | Has next page? |
| `hasPrevious()` | Has previous page? |

---

## Practice Questions

1. What is the difference between Page and Slice?
2. How do you create a Pageable with sorting?
3. Write a controller that supports pagination and sorting.
4. When would you use Slice instead of Page?
5. How do you sort by multiple fields?
6. What methods are available on Page<T>?

---

**End of Note 19: Pagination and Sorting**

*Previous: [18_Exception_Handling.md](file:///c:/Users/2706p/Desktop/mcq/notes/18_Exception_Handling.md)*  
*Next: [20_Interview_Questions.md](file:///c:/Users/2706p/Desktop/mcq/notes/20_Interview_Questions.md)*
