# 📁 File Handling in C# - Complete Deep Dive

## 📚 Table of Contents
- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Core Concepts](#-core-concepts)
- [File and Directory Classes](#-file-and-directory-classes)
- [Stream Classes](#-stream-classes)
- [StreamReader and StreamWriter](#-streamreader-and-streamwriter)
- [BinaryReader and BinaryWriter](#-binaryreader-and-binarywriter)
- [FileStream Operations](#-filestream-operations)
- [Code Examples with Explanations](#-code-examples-with-explanations)
- [Execution Flow Diagrams](#-execution-flow-diagrams)
- [Best Practices](#-best-practices)
- [Common Pitfalls](#-common-pitfalls)
- [Interview Questions](#-interview-questions)
- [Summary](#-summary)

---

## 🎯 Overview

**File Handling** in C# refers to the process of reading from and writing to files on the disk. The .NET Framework provides comprehensive classes in the `System.IO` namespace to perform various file operations including:

- Creating, reading, writing, and deleting files
- Reading and writing text and binary data
- Working with directories and file paths
- Streaming data for efficient memory usage

### Why File Handling is Important
1. **Data Persistence**: Store data permanently beyond application runtime
2. **Configuration Management**: Read/write application settings
3. **Logging**: Record application events and errors
4. **Data Exchange**: Import/export data between systems

---

## 📋 Prerequisites

Before learning file handling, you should understand:
- C# basic syntax and data types
- Exception handling (try-catch-finally)
- Object-Oriented Programming concepts
- `using` statement for resource management (IDisposable)

---

## 🔷 Core Concepts

### System.IO Namespace

The `System.IO` namespace contains all file and stream-related classes:

```csharp
using System.IO;  // Import for file operations
```

### Key Class Categories

| Category | Classes | Purpose |
|----------|---------|---------|
| **File Operations** | `File`, `FileInfo` | Create, copy, delete, move files |
| **Directory Operations** | `Directory`, `DirectoryInfo` | Create, enumerate, delete directories |
| **Text Streams** | `StreamReader`, `StreamWriter` | Read/write text files |
| **Binary Streams** | `BinaryReader`, `BinaryWriter` | Read/write binary data |
| **Base Stream** | `FileStream` | Low-level byte stream operations |
| **Path Utilities** | `Path` | Manipulate file paths |

---

## 📂 File and Directory Classes

### File Class (Static Methods)

The `File` class provides **static methods** for file operations:

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// FILE CLASS - STATIC METHODS FOR FILE OPERATIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;          // Line 1: Import System namespace for Console
using System.IO;       // Line 2: Import System.IO for File class

class FileOperationsDemo
{
    static void Main()
    {
        string filePath = @"C:\Demo\example.txt";  // Line 3: Define file path
                                                    // @ symbol = verbatim string (no escape chars)

        // ─────────────────────────────────────────────────────────────────────
        // CREATE AND WRITE
        // ─────────────────────────────────────────────────────────────────────
        
        // WriteAllText - Creates file (overwrites if exists) and writes text
        File.WriteAllText(filePath, "Hello World!");  
        // Line 4: Creates new file if not exists
        //         Writes "Hello World!" as content
        //         Closes file automatically
        //         OVERWRITES existing content if file exists

        // AppendAllText - Adds text to end of file
        File.AppendAllText(filePath, "\nSecond Line");
        // Line 5: Opens file in append mode
        //         Adds newline + "Second Line" at end
        //         Creates file if doesn't exist
        //         Does NOT overwrite existing content

        // ─────────────────────────────────────────────────────────────────────
        // READ OPERATIONS
        // ─────────────────────────────────────────────────────────────────────
        
        // ReadAllText - Reads entire file as single string
        string content = File.ReadAllText(filePath);
        // Line 6: Opens file, reads ALL content
        //         Returns content as single string
        //         Includes newlines as \n or \r\n
        //         Closes file automatically
        
        Console.WriteLine(content);
        // Output:
        // Hello World!
        // Second Line

        // ReadAllLines - Reads file as array of lines
        string[] lines = File.ReadAllLines(filePath);
        // Line 7: Opens file, reads line by line
        //         Returns string[] where each element = one line
        //         Removes line endings from each line
        //         Useful for processing line by line
        
        foreach (string line in lines)
        {
            Console.WriteLine($"Line: {line}");
        }
        // Output:
        // Line: Hello World!
        // Line: Second Line

        // ─────────────────────────────────────────────────────────────────────
        // FILE INFORMATION & CHECKS
        // ─────────────────────────────────────────────────────────────────────
        
        // Exists - Check if file exists
        bool exists = File.Exists(filePath);
        // Line 8: Returns true if file exists at path
        //         Returns false if file doesn't exist
        //         Returns false if path is directory
        //         Does NOT throw exception
        
        Console.WriteLine($"File exists: {exists}");  // Output: File exists: True

        // ─────────────────────────────────────────────────────────────────────
        // COPY, MOVE, DELETE
        // ─────────────────────────────────────────────────────────────────────
        
        // Copy - Create a copy of file
        string copyPath = @"C:\Demo\example_copy.txt";
        File.Copy(filePath, copyPath, overwrite: true);
        // Line 9: Copies file from source to destination
        //         overwrite: true = replace if exists
        //         overwrite: false = throw exception if exists

        // Move - Rename or move file
        string newPath = @"C:\Demo\renamed.txt";
        File.Move(copyPath, newPath);
        // Line 10: Moves file to new location
        //          Can also rename file (same directory, new name)
        //          Throws exception if destination exists

        // Delete - Remove file
        File.Delete(newPath);
        // Line 11: Permanently deletes file
        //          Does NOT move to Recycle Bin
        //          Does NOT throw if file doesn't exist
    }
}
```

### Execution Flow for File.WriteAllText

```
┌─────────────────────────────────────────────────────────────────────┐
│                    File.WriteAllText() Flow                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐                                                    │
│  │ Start Call  │                                                    │
│  └──────┬──────┘                                                    │
│         │                                                           │
│         ▼                                                           │
│  ┌─────────────────┐                                                │
│  │ Check if path   │                                                │
│  │ is valid        │                                                │
│  └────────┬────────┘                                                │
│           │                                                         │
│           ▼                                                         │
│  ┌─────────────────┐     ┌─────────────────┐                       │
│  │ File exists?    │────▶│ Delete existing │                       │
│  │                 │ Yes │ content         │                       │
│  └────────┬────────┘     └────────┬────────┘                       │
│           │ No                    │                                 │
│           ▼                       ▼                                 │
│  ┌─────────────────┐     ┌─────────────────┐                       │
│  │ Create new file │     │ Open file for   │                       │
│  │                 │────▶│ writing         │                       │
│  └─────────────────┘     └────────┬────────┘                       │
│                                   │                                 │
│                                   ▼                                 │
│                          ┌─────────────────┐                       │
│                          │ Write content   │                       │
│                          │ to file         │                       │
│                          └────────┬────────┘                       │
│                                   │                                 │
│                                   ▼                                 │
│                          ┌─────────────────┐                       │
│                          │ Flush & Close   │                       │
│                          │ file handle     │                       │
│                          └────────┬────────┘                       │
│                                   │                                 │
│                                   ▼                                 │
│                          ┌─────────────────┐                       │
│                          │    Complete     │                       │
│                          └─────────────────┘                       │
└─────────────────────────────────────────────────────────────────────┘
```

### FileInfo Class (Instance Methods)

Unlike `File`, `FileInfo` creates an **object** representing a file:

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// FILEINFO CLASS - INSTANCE-BASED FILE OPERATIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class FileInfoDemo
{
    static void Main()
    {
        // Create FileInfo object - does NOT create actual file
        FileInfo fileInfo = new FileInfo(@"C:\Demo\sample.txt");
        // Line 1: Creates FileInfo object representing file at path
        //         Does NOT create the physical file
        //         Does NOT check if file exists yet
        //         Just stores path information

        // Create the actual file if it doesn't exist
        if (!fileInfo.Exists)
        // Line 2: Exists property checks if physical file exists
        //         Returns bool (true/false)
        //         Different from static File.Exists()
        {
            using (StreamWriter sw = fileInfo.CreateText())
            // Line 3: CreateText() creates file and returns StreamWriter
            //         using statement ensures proper disposal
            //         StreamWriter is for writing text data
            {
                sw.WriteLine("Created via FileInfo");
                // Line 4: Write a line to the file
            }
        }
        // After using block, StreamWriter is automatically closed/disposed

        // ─────────────────────────────────────────────────────────────────────
        // FILEINFO PROPERTIES - Get File Metadata
        // ─────────────────────────────────────────────────────────────────────
        
        Console.WriteLine($"Full Name: {fileInfo.FullName}");
        // FullName: Complete absolute path (C:\Demo\sample.txt)
        
        Console.WriteLine($"Name: {fileInfo.Name}");
        // Name: Just filename with extension (sample.txt)
        
        Console.WriteLine($"Extension: {fileInfo.Extension}");
        // Extension: File extension including dot (.txt)
        
        Console.WriteLine($"Directory: {fileInfo.DirectoryName}");
        // DirectoryName: Parent directory path (C:\Demo)
        
        Console.WriteLine($"Size: {fileInfo.Length} bytes");
        // Length: File size in bytes
        // Throws exception if file doesn't exist
        
        Console.WriteLine($"Created: {fileInfo.CreationTime}");
        // CreationTime: When file was created
        
        Console.WriteLine($"Modified: {fileInfo.LastWriteTime}");
        // LastWriteTime: When file was last modified
        
        Console.WriteLine($"Accessed: {fileInfo.LastAccessTime}");
        // LastAccessTime: When file was last opened/read
        
        Console.WriteLine($"ReadOnly: {fileInfo.IsReadOnly}");
        // IsReadOnly: Returns true if file has read-only attribute

        // ─────────────────────────────────────────────────────────────────────
        // FILEINFO INSTANCE METHODS
        // ─────────────────────────────────────────────────────────────────────
        
        // Copy file
        FileInfo copiedFile = fileInfo.CopyTo(@"C:\Demo\copy.txt", overwrite: true);
        // Line 5: CopyTo returns NEW FileInfo for copied file
        //         overwrite: true = replace if exists
        //         Original FileInfo still points to original file

        // Move/Rename file
        fileInfo.MoveTo(@"C:\Demo\moved.txt");
        // Line 6: Physically moves file to new location
        //         FileInfo object now points to NEW location
        //         Unlike File.Move, this updates the object

        // Delete file
        fileInfo.Delete();
        // Line 7: Permanently deletes the file
        //         FileInfo object becomes invalid
        //         Exists property will return false
    }
}
```

### File vs FileInfo - When to Use Which?

| Aspect | File (Static) | FileInfo (Instance) |
|--------|---------------|---------------------|
| **Single Operation** | ✅ Best choice | ❌ Overhead |
| **Multiple Operations** | ❌ Security check each time | ✅ Single security check |
| **Get Properties** | Need multiple calls | Single object, multiple properties |
| **Memory** | No object created | Object created in memory |
| **Performance** | Slower for multiple ops | Faster for multiple ops |

```csharp
// COMPARISON EXAMPLE

// ❌ BAD: Multiple static calls = multiple security checks
if (File.Exists(path))
{
    string content = File.ReadAllText(path);
    DateTime created = File.GetCreationTime(path);
    long size = new FileInfo(path).Length;
}
// Each call performs security check on path

// ✅ GOOD: Single FileInfo instance
FileInfo fi = new FileInfo(path);  // One security check
if (fi.Exists)
{
    using (StreamReader sr = fi.OpenText())
    {
        string content = sr.ReadToEnd();
    }
    DateTime created = fi.CreationTime;
    long size = fi.Length;
}
// Properties cached, no additional security checks
```

---

## 📂 Directory Operations

### Directory Class (Static Methods)

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DIRECTORY CLASS - STATIC METHODS FOR DIRECTORY OPERATIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class DirectoryDemo
{
    static void Main()
    {
        string dirPath = @"C:\Demo\MyFolder";

        // ─────────────────────────────────────────────────────────────────────
        // CREATE DIRECTORY
        // ─────────────────────────────────────────────────────────────────────
        
        Directory.CreateDirectory(dirPath);
        // Line 1: Creates directory at specified path
        //         Creates ALL parent directories if they don't exist
        //         Example: C:\Demo\MyFolder\Sub1\Sub2 creates Demo, MyFolder, Sub1, Sub2
        //         Does NOT throw exception if directory exists
        //         Returns DirectoryInfo object

        // ─────────────────────────────────────────────────────────────────────
        // CHECK EXISTENCE
        // ─────────────────────────────────────────────────────────────────────
        
        bool exists = Directory.Exists(dirPath);
        // Line 2: Returns true if directory exists
        //         Returns false if path is a file or doesn't exist

        // ─────────────────────────────────────────────────────────────────────
        // GET CONTENTS
        // ─────────────────────────────────────────────────────────────────────
        
        // Get all files in directory
        string[] files = Directory.GetFiles(dirPath);
        // Line 3: Returns array of full file paths
        //         Only immediate files, not subdirectory files
        
        // Get files with filter
        string[] txtFiles = Directory.GetFiles(dirPath, "*.txt");
        // Line 4: Filter by pattern
        //         *.txt = all text files
        //         report*.csv = files starting with "report" ending with .csv

        // Get files including subdirectories
        string[] allFiles = Directory.GetFiles(dirPath, "*.*", SearchOption.AllDirectories);
        // Line 5: SearchOption.AllDirectories = recursive search
        //         SearchOption.TopDirectoryOnly = only current directory (default)

        // Get subdirectories
        string[] subDirs = Directory.GetDirectories(dirPath);
        // Line 6: Returns array of subdirectory paths
        //         Only immediate subdirectories

        // ─────────────────────────────────────────────────────────────────────
        // ENUMERATE (Better for large directories)
        // ─────────────────────────────────────────────────────────────────────
        
        // EnumerateFiles - Returns IEnumerable (lazy loading)
        foreach (string file in Directory.EnumerateFiles(dirPath))
        // Line 7: EnumerateFiles is more memory efficient
        //         Returns files one at a time (streaming)
        //         Better for directories with many files
        //         GetFiles loads ALL into array first
        {
            Console.WriteLine(file);
        }

        // ─────────────────────────────────────────────────────────────────────
        // MOVE AND DELETE
        // ─────────────────────────────────────────────────────────────────────
        
        // Move directory
        Directory.Move(dirPath, @"C:\Demo\RenamedFolder");
        // Line 8: Moves/renames directory
        //         Cannot move across drives
        //         Throws exception if destination exists

        // Delete directory
        Directory.Delete(@"C:\Demo\RenamedFolder", recursive: true);
        // Line 9: Deletes directory
        //         recursive: true = delete even if not empty
        //         recursive: false = throw exception if not empty
    }
}
```

---

## 🌊 Stream Classes

### What is a Stream?

A **Stream** is an abstract representation of a sequence of bytes. Think of it like water flowing through a pipe:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         STREAM CONCEPT                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Data Source              Stream                    Application    │
│  ┌──────────┐         ┌─────────────┐              ┌──────────┐    │
│  │   File   │ ═══════▶│   Bytes     │═════════════▶│ Program  │    │
│  │   Disk   │◀═══════ │   Flow      │◀═════════════│   Code   │    │
│  └──────────┘         └─────────────┘              └──────────┘    │
│                                                                     │
│  Types of Streams:                                                  │
│  • FileStream     - Read/Write files                               │
│  • MemoryStream   - Read/Write memory buffer                       │
│  • NetworkStream  - Read/Write network sockets                     │
│  • BufferedStream - Adds buffering to another stream               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Stream Class Hierarchy

```
                          Stream (Abstract Base)
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
          FileStream       MemoryStream      NetworkStream
                │
        ┌───────┴───────┐
        │               │
   StreamReader    StreamWriter
   (Text Input)    (Text Output)
        │               │
   BinaryReader    BinaryWriter
   (Binary Input)  (Binary Output)
```

---

## 📄 StreamReader and StreamWriter

### StreamWriter - Writing Text to Files

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// STREAMWRITER - WRITING TEXT DATA TO FILES
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class StreamWriterDemo
{
    static void Main()
    {
        string filePath = @"C:\Demo\output.txt";

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 1: Using statement (RECOMMENDED)
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamWriter writer = new StreamWriter(filePath))
        // Line 1: Creates StreamWriter connected to file
        //         using statement ensures Dispose() is called
        //         Dispose() flushes buffer and closes file
        //         File is created if doesn't exist
        //         File is OVERWRITTEN if exists
        {
            writer.WriteLine("First Line");
            // Line 2: Writes text + newline character
            //         Automatic line ending based on OS (\r\n Windows, \n Unix)
            
            writer.WriteLine("Second Line");
            // Line 3: Writes another line
            
            writer.Write("No newline here");
            // Line 4: Writes text WITHOUT newline at end
            //         Next write continues on same line
            
            writer.Write(" - continued");
            // Line 5: Continues on same line
            
            writer.WriteLine();
            // Line 6: Writes just a newline (empty line)
            
            // Write formatted data
            int age = 25;
            string name = "John";
            writer.WriteLine($"Name: {name}, Age: {age}");
            // Line 7: String interpolation for formatting
            
            writer.WriteLine("Name: {0}, Age: {1}", name, age);
            // Line 8: Alternative format using placeholders
        }
        // Line 9: End of using block
        //         writer.Dispose() called automatically
        //         Buffer flushed, file handle closed
        //         File is now safe for other programs to access

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 2: Append Mode
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamWriter appendWriter = new StreamWriter(filePath, append: true))
        // Line 10: append: true = ADD to existing content
        //          append: false (default) = OVERWRITE
        {
            appendWriter.WriteLine("This line is appended");
            // Line 11: Added at end of file
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 3: Specify Encoding
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamWriter utf8Writer = new StreamWriter(filePath, false, System.Text.Encoding.UTF8))
        // Line 12: Third parameter specifies encoding
        //          UTF8 - Universal encoding for all characters
        //          ASCII - Only English characters (0-127)
        //          Unicode - UTF-16 encoding
        {
            utf8Writer.WriteLine("日本語テキスト");  // Japanese text
            // Line 13: UTF8 can handle international characters
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 4: Auto-Flush
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamWriter autoFlush = new StreamWriter(filePath))
        {
            autoFlush.AutoFlush = true;
            // Line 14: AutoFlush = true means write immediately to disk
            //          Default is false (buffered)
            //          Buffered is faster but data may be lost on crash
            //          AutoFlush is slower but safer
            
            autoFlush.WriteLine("Immediately written to disk");
        }
    }
}
```

### Execution Flow - StreamWriter with `using`

```
┌─────────────────────────────────────────────────────────────────────┐
│              StreamWriter using Statement Flow                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────┐                                               │
│  │ using block      │                                               │
│  │ starts           │                                               │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐     ┌──────────────────┐                     │
│  │ new StreamWriter │────▶│ Open file handle │                     │
│  │ (constructor)    │     │ Create buffer    │                     │
│  └────────┬─────────┘     └──────────────────┘                     │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │ WriteLine()      │                                               │
│  │ calls inside     │                                               │
│  │ using block      │                                               │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │ Data written to  │   (Not yet on disk!)                         │
│  │ internal buffer  │                                               │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │ using block ends │                                               │
│  │ } reached        │                                               │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐     ┌──────────────────┐                     │
│  │ Dispose() called │────▶│ Flush() - write  │                     │
│  │ automatically    │     │ buffer to disk   │                     │
│  └──────────────────┘     └────────┬─────────┘                     │
│                                    │                                │
│                                    ▼                                │
│                           ┌──────────────────┐                     │
│                           │ Close file       │                     │
│                           │ handle           │                     │
│                           └────────┬─────────┘                     │
│                                    │                                │
│                                    ▼                                │
│                           ┌──────────────────┐                     │
│                           │ StreamWriter     │                     │
│                           │ object becomes   │                     │
│                           │ unusable         │                     │
│                           └──────────────────┘                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### StreamReader - Reading Text from Files

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// STREAMREADER - READING TEXT DATA FROM FILES
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class StreamReaderDemo
{
    static void Main()
    {
        string filePath = @"C:\Demo\input.txt";

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 1: Read Entire File at Once
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath))
        // Line 1: Opens file for reading
        //         using ensures file is closed properly
        {
            string allContent = reader.ReadToEnd();
            // Line 2: Reads ENTIRE file into single string
            //         Includes all newlines
            //         Good for small files
            //         ⚠️ BAD for large files (loads all into memory)
            
            Console.WriteLine(allContent);
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 2: Read Line by Line (RECOMMENDED for large files)
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath))
        {
            string line;
            int lineNumber = 0;
            
            while ((line = reader.ReadLine()) != null)
            // Line 3: ReadLine() returns one line at a time
            //         Returns null when end of file reached
            //         Removes newline character from returned string
            //         Memory efficient - only one line in memory
            {
                lineNumber++;
                Console.WriteLine($"Line {lineNumber}: {line}");
            }
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 3: Read Character by Character
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath))
        {
            int charCode;
            while ((charCode = reader.Read()) != -1)
            // Line 4: Read() returns one character as int
            //         Returns -1 at end of file
            //         Cast to char to get actual character
            {
                char c = (char)charCode;
                // Line 5: Convert int to char
                
                Console.Write(c);
                // Line 6: Print without newline
            }
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 4: Check End of File
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath))
        {
            while (!reader.EndOfStream)
            // Line 7: EndOfStream property checks if more data available
            //         true = no more data to read
            //         false = more data available
            {
                Console.WriteLine(reader.ReadLine());
            }
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 5: Peek Without Reading
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath))
        {
            // Peek looks at next character without consuming it
            int nextChar = reader.Peek();
            // Line 8: Peek() returns next char as int
            //         Does NOT advance position
            //         Returns -1 if at end of file
            
            if (nextChar != -1)
            {
                Console.WriteLine($"Next character is: {(char)nextChar}");
                
                // Actually read the character
                reader.Read();  // Now position advances
            }
        }

        // ─────────────────────────────────────────────────────────────────────
        // METHOD 6: Specify Encoding
        // ─────────────────────────────────────────────────────────────────────
        
        using (StreamReader reader = new StreamReader(filePath, System.Text.Encoding.UTF8))
        // Line 9: Second parameter specifies encoding
        //         Must match encoding used when writing
        //         UTF8 is safe default for most cases
        {
            string content = reader.ReadToEnd();
            Console.WriteLine(content);
        }
    }
}
```

---

## 🔢 BinaryReader and BinaryWriter

For reading/writing **primitive data types** in binary format:

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// BINARYWRITER AND BINARYREADER - BINARY DATA OPERATIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class BinaryOperationsDemo
{
    static void Main()
    {
        string filePath = @"C:\Demo\data.bin";

        // ─────────────────────────────────────────────────────────────────────
        // WRITE BINARY DATA
        // ─────────────────────────────────────────────────────────────────────
        
        using (FileStream fs = new FileStream(filePath, FileMode.Create))
        // Line 1: FileStream opens file for binary access
        //         FileMode.Create = create new or overwrite
        using (BinaryWriter writer = new BinaryWriter(fs))
        // Line 2: BinaryWriter wraps FileStream
        //         Writes primitive types in binary format
        {
            // Write different data types
            writer.Write(42);
            // Line 3: Writes int (4 bytes in binary)
            //         Stored as: 2A 00 00 00 (little-endian)
            
            writer.Write(3.14159);
            // Line 4: Writes double (8 bytes in binary)
            
            writer.Write("Hello");
            // Line 5: Writes string
            //         First writes length as 7-bit encoded int
            //         Then writes UTF-8 bytes of string
            
            writer.Write(true);
            // Line 6: Writes bool (1 byte)
            //         true = 01, false = 00
            
            writer.Write('A');
            // Line 7: Writes char (2 bytes - UTF-16)
            
            writer.Write((byte)255);
            // Line 8: Writes byte (1 byte)
            
            byte[] data = { 1, 2, 3, 4, 5 };
            writer.Write(data);
            // Line 9: Writes byte array
            //         Does NOT write length - you must track it
        }

        // ─────────────────────────────────────────────────────────────────────
        // READ BINARY DATA
        // ─────────────────────────────────────────────────────────────────────
        
        using (FileStream fs = new FileStream(filePath, FileMode.Open))
        // Line 10: Open existing file for reading
        using (BinaryReader reader = new BinaryReader(fs))
        // Line 11: BinaryReader for reading primitives
        {
            // ⚠️ IMPORTANT: Must read in SAME ORDER as written!
            
            int number = reader.ReadInt32();
            // Line 12: Read 4 bytes as int
            //          Other methods: ReadInt16(), ReadInt64()
            Console.WriteLine($"Int: {number}");  // Output: 42
            
            double pi = reader.ReadDouble();
            // Line 13: Read 8 bytes as double
            //          Other: ReadSingle() for float
            Console.WriteLine($"Double: {pi}");  // Output: 3.14159
            
            string text = reader.ReadString();
            // Line 14: Reads length-prefixed string
            Console.WriteLine($"String: {text}");  // Output: Hello
            
            bool flag = reader.ReadBoolean();
            // Line 15: Read 1 byte as bool
            Console.WriteLine($"Bool: {flag}");  // Output: True
            
            char character = reader.ReadChar();
            // Line 16: Read 2 bytes as char
            Console.WriteLine($"Char: {character}");  // Output: A
            
            byte b = reader.ReadByte();
            // Line 17: Read 1 byte
            Console.WriteLine($"Byte: {b}");  // Output: 255
            
            byte[] readData = reader.ReadBytes(5);
            // Line 18: Read 5 bytes into array
            //          Parameter = number of bytes to read
            Console.WriteLine($"Bytes: {string.Join(", ", readData)}");
            // Output: Bytes: 1, 2, 3, 4, 5
        }
    }
}
```

### Binary vs Text Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Binary vs Text File Storage                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Storing integer 12345:                                             │
│                                                                     │
│  TEXT FILE (StreamWriter):                                          │
│  ┌─────────────────────────────┐                                   │
│  │ '1' '2' '3' '4' '5'         │  = 5 bytes (characters)           │
│  │  31  32  33  34  35         │  (ASCII/UTF-8 codes)              │
│  └─────────────────────────────┘                                   │
│                                                                     │
│  BINARY FILE (BinaryWriter):                                        │
│  ┌─────────────────────────────┐                                   │
│  │ 39 30 00 00                 │  = 4 bytes (int32)                │
│  │ (little-endian binary)      │                                   │
│  └─────────────────────────────┘                                   │
│                                                                     │
│  Advantages of Binary:                                              │
│  • Smaller file size for numbers                                    │
│  • Faster read/write (no parsing)                                   │
│  • Exact precision for floats                                       │
│                                                                     │
│  Advantages of Text:                                                │
│  • Human readable                                                   │
│  • Editable in any text editor                                      │
│  • Cross-platform compatible                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📀 FileStream Operations

FileStream provides low-level byte-based file access:

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// FILESTREAM - LOW-LEVEL BYTE OPERATIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class FileStreamDemo
{
    static void Main()
    {
        string filePath = @"C:\Demo\stream.dat";

        // ─────────────────────────────────────────────────────────────────────
        // FILEMODE OPTIONS
        // ─────────────────────────────────────────────────────────────────────
        
        /*
        FileMode.Create      - Create new, overwrite if exists
        FileMode.CreateNew   - Create new, exception if exists
        FileMode.Open        - Open existing, exception if not exists
        FileMode.OpenOrCreate- Open if exists, create if not
        FileMode.Append      - Open and seek to end, create if not
        FileMode.Truncate    - Open existing and clear content
        */

        // ─────────────────────────────────────────────────────────────────────
        // FILEACCESS OPTIONS
        // ─────────────────────────────────────────────────────────────────────
        
        /*
        FileAccess.Read      - Read only
        FileAccess.Write     - Write only
        FileAccess.ReadWrite - Both read and write
        */

        // ─────────────────────────────────────────────────────────────────────
        // WRITE BYTES
        // ─────────────────────────────────────────────────────────────────────
        
        using (FileStream fs = new FileStream(
            filePath,                    // File path
            FileMode.Create,             // Create or overwrite
            FileAccess.Write))           // Write access only
        // Line 1: Create FileStream with specific mode and access
        {
            byte[] data = { 65, 66, 67, 68, 69 };  // ASCII: A, B, C, D, E
            // Line 2: Create byte array to write
            
            fs.Write(data, 0, data.Length);
            // Line 3: Write bytes to file
            //         Parameter 1: byte array source
            //         Parameter 2: offset in array (start from index 0)
            //         Parameter 3: number of bytes to write
            
            fs.WriteByte(70);  // ASCII: F
            // Line 4: Write single byte
        }

        // ─────────────────────────────────────────────────────────────────────
        // READ BYTES
        // ─────────────────────────────────────────────────────────────────────
        
        using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read))
        {
            // Read single byte
            int firstByte = fs.ReadByte();
            // Line 5: ReadByte returns int (not byte)
            //         Returns -1 at end of file
            Console.WriteLine($"First byte: {firstByte} = '{(char)firstByte}'");
            // Output: First byte: 65 = 'A'

            // Read into buffer
            byte[] buffer = new byte[10];
            // Line 6: Create buffer to hold read data
            
            int bytesRead = fs.Read(buffer, 0, buffer.Length);
            // Line 7: Read bytes into buffer
            //         Parameter 1: destination buffer
            //         Parameter 2: offset in buffer to start writing
            //         Parameter 3: max bytes to read
            //         Returns: actual bytes read (may be less)
            
            Console.WriteLine($"Bytes read: {bytesRead}");
            // Output: Bytes read: 5 (B, C, D, E, F)
            
            Console.WriteLine($"Content: {System.Text.Encoding.ASCII.GetString(buffer, 0, bytesRead)}");
            // Output: Content: BCDEF
        }

        // ─────────────────────────────────────────────────────────────────────
        // SEEK - MOVE POSITION
        // ─────────────────────────────────────────────────────────────────────
        
        using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read))
        {
            Console.WriteLine($"Initial position: {fs.Position}");
            // Position: Current byte position in stream (0-indexed)
            // Output: Initial position: 0
            
            Console.WriteLine($"File length: {fs.Length}");
            // Length: Total size of file in bytes
            // Output: File length: 6
            
            // Seek to position
            fs.Seek(3, SeekOrigin.Begin);
            // Line 8: Move to position 3 from beginning
            //         SeekOrigin.Begin - from start of file
            //         SeekOrigin.Current - from current position
            //         SeekOrigin.End - from end of file
            
            Console.WriteLine($"Position after seek: {fs.Position}");
            // Output: Position after seek: 3
            
            int byteAtPos3 = fs.ReadByte();
            Console.WriteLine($"Byte at position 3: {(char)byteAtPos3}");
            // Output: Byte at position 3: D

            // Seek from end
            fs.Seek(-2, SeekOrigin.End);
            // Line 9: Move to 2 bytes before end
            //         Negative offset means go backwards
            
            int secondLast = fs.ReadByte();
            Console.WriteLine($"Second last byte: {(char)secondLast}");
            // Output: Second last byte: E
        }
    }
}
```

---

## 🛠️ Path Class Utilities

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PATH CLASS - FILE PATH MANIPULATION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;

class PathDemo
{
    static void Main()
    {
        string filePath = @"C:\Users\Demo\Documents\report.txt";

        // ─────────────────────────────────────────────────────────────────────
        // EXTRACT PATH COMPONENTS
        // ─────────────────────────────────────────────────────────────────────
        
        Console.WriteLine($"Full path: {filePath}");
        // Output: C:\Users\Demo\Documents\report.txt

        Console.WriteLine($"Directory: {Path.GetDirectoryName(filePath)}");
        // Output: C:\Users\Demo\Documents
        // Returns parent directory path

        Console.WriteLine($"Filename: {Path.GetFileName(filePath)}");
        // Output: report.txt
        // Returns filename with extension

        Console.WriteLine($"Filename (no ext): {Path.GetFileNameWithoutExtension(filePath)}");
        // Output: report
        // Returns filename without extension

        Console.WriteLine($"Extension: {Path.GetExtension(filePath)}");
        // Output: .txt
        // Returns extension including the dot

        Console.WriteLine($"Root: {Path.GetPathRoot(filePath)}");
        // Output: C:\
        // Returns drive letter and root separator

        // ─────────────────────────────────────────────────────────────────────
        // COMBINE PATHS
        // ─────────────────────────────────────────────────────────────────────
        
        string dir = @"C:\Demo";
        string filename = "test.txt";
        
        // ❌ BAD: Manual concatenation
        string badPath = dir + "\\" + filename;
        // May cause issues with trailing slashes
        
        // ✅ GOOD: Path.Combine
        string goodPath = Path.Combine(dir, filename);
        // Output: C:\Demo\test.txt
        // Handles trailing slashes automatically
        // Works cross-platform (uses correct separator)

        // Combine multiple segments
        string multiPath = Path.Combine("C:", "Users", "Demo", "file.txt");
        // Output: C:\Users\Demo\file.txt

        // ─────────────────────────────────────────────────────────────────────
        // SPECIAL DIRECTORIES
        // ─────────────────────────────────────────────────────────────────────
        
        Console.WriteLine($"Temp folder: {Path.GetTempPath()}");
        // Output: C:\Users\Demo\AppData\Local\Temp\
        
        Console.WriteLine($"Temp filename: {Path.GetTempFileName()}");
        // Creates and returns path to new temporary file
        // Output: C:\Users\Demo\AppData\Local\Temp\tmpXXXX.tmp

        Console.WriteLine($"Random filename: {Path.GetRandomFileName()}");
        // Returns random filename (does NOT create file)
        // Output: ab12cd34.xyz

        // ─────────────────────────────────────────────────────────────────────
        // PATH VALIDATION
        // ─────────────────────────────────────────────────────────────────────
        
        Console.WriteLine($"Is path rooted: {Path.IsPathRooted(filePath)}");
        // Output: True (starts with C:\)
        // Relative paths return False

        Console.WriteLine($"Has extension: {Path.HasExtension(filePath)}");
        // Output: True

        // Change extension
        string newExtPath = Path.ChangeExtension(filePath, ".pdf");
        Console.WriteLine($"Changed extension: {newExtPath}");
        // Output: C:\Users\Demo\Documents\report.pdf
        // Does NOT rename actual file, just returns new path string

        // ─────────────────────────────────────────────────────────────────────
        // INVALID CHARACTERS
        // ─────────────────────────────────────────────────────────────────────
        
        char[] invalidPathChars = Path.GetInvalidPathChars();
        // Characters not allowed in paths
        
        char[] invalidFileChars = Path.GetInvalidFileNameChars();
        // Characters not allowed in filenames
        // Includes: \ / : * ? " < > |
    }
}
```

---

## ✅ Best Practices

### 1. Always Use `using` Statement

```csharp
// ✅ CORRECT: Resources automatically disposed
using (StreamWriter writer = new StreamWriter(path))
{
    writer.WriteLine("Data");
}  // Dispose called here, even if exception occurs

// ❌ WRONG: Resources may leak
StreamWriter writer = new StreamWriter(path);
writer.WriteLine("Data");
writer.Close();  // May not execute if exception before this line
```

### 2. Handle Exceptions Properly

```csharp
try
{
    using (StreamReader reader = new StreamReader(filePath))
    {
        string content = reader.ReadToEnd();
    }
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"File not found: {ex.FileName}");
}
catch (UnauthorizedAccessException ex)
{
    Console.WriteLine($"Access denied: {ex.Message}");
}
catch (IOException ex)
{
    Console.WriteLine($"I/O error: {ex.Message}");
}
```

### 3. Check Before Operations

```csharp
// Check file exists before reading
if (File.Exists(filePath))
{
    string content = File.ReadAllText(filePath);
}

// Check directory exists before writing
string directory = Path.GetDirectoryName(filePath);
if (!Directory.Exists(directory))
{
    Directory.CreateDirectory(directory);
}
```

### 4. Use Path.Combine for Paths

```csharp
// ❌ WRONG: Platform-specific separator
string path = baseDir + "\\" + filename;

// ✅ CORRECT: Cross-platform safe
string path = Path.Combine(baseDir, filename);
```

---

## ⚠️ Common Pitfalls

### 1. Forgetting to Dispose

```csharp
// ❌ File handle leak - other programs can't access file
StreamWriter writer = new StreamWriter(path);
writer.WriteLine("Data");
// Forgot to close! File locked until program exits
```

### 2. Reading Large Files into Memory

```csharp
// ❌ May crash on large files
string allData = File.ReadAllText(hugeFilePath);  // Out of memory!

// ✅ Process line by line
foreach (string line in File.ReadLines(hugeFilePath))
{
    ProcessLine(line);  // Only one line in memory
}
```

### 3. Not Handling File Locking

```csharp
// ❌ Another process may have file open
File.Delete(filePath);  // IOException!

// ✅ Handle the exception
try
{
    File.Delete(filePath);
}
catch (IOException)
{
    Console.WriteLine("File is in use by another process");
}
```

---

## 🎤 Interview Questions

### Basic Level

**Q1: What is the difference between `File` and `FileInfo` classes?**
> `File` is a static class with static methods for quick one-off operations. `FileInfo` is an instance class that creates an object representing a file, better for multiple operations on the same file as it caches security checks.

**Q2: Why should we use `using` statement with file operations?**
> The `using` statement ensures that `Dispose()` is called even if an exception occurs, properly releasing file handles and flushing buffers. Without it, files may remain locked.

**Q3: What is the difference between `Write()` and `WriteLine()`?**
> `Write()` outputs text without adding a newline at the end. `WriteLine()` adds a newline character after the text.

### Intermediate Level

**Q4: What is the difference between `ReadAllText()` and `ReadAllLines()`?**
> `ReadAllText()` returns the entire file as a single string including newlines. `ReadAllLines()` returns a string array where each element is one line without the newline characters.

**Q5: When would you use `BinaryWriter` instead of `StreamWriter`?**
> Use `BinaryWriter` for storing primitive data types (int, double, etc.) in compact binary format. Use `StreamWriter` for human-readable text files. Binary is smaller and faster but not human-readable.

**Q6: What are the different `FileMode` options?**
> `Create` (create/overwrite), `CreateNew` (create only, error if exists), `Open` (open existing, error if not), `OpenOrCreate` (open or create), `Append` (open and seek to end), `Truncate` (open and clear content).

### Advanced Level

**Q7: Explain buffering in file streams.**
> Streams use internal buffers to reduce disk I/O. Data written is first stored in memory buffer, then flushed to disk when buffer is full or stream is closed. `AutoFlush = true` bypasses buffering for immediate writes.

**Q8: How do you handle reading files with different encodings?**
> Specify the encoding in the StreamReader constructor: `new StreamReader(path, Encoding.UTF8)`. If encoding is unknown, use `StreamReader` with `detectEncodingFromByteOrderMarks: true`.

---

## 📝 Summary

| Concept | Key Points |
|---------|------------|
| **File Class** | Static methods for quick single operations |
| **FileInfo Class** | Instance-based for multiple operations |
| **StreamWriter** | Text output with WriteLine/Write |
| **StreamReader** | Text input with ReadLine/ReadToEnd |
| **BinaryWriter** | Binary primitives output |
| **BinaryReader** | Binary primitives input |
| **FileStream** | Low-level byte operations with Seek |
| **Path Class** | Path manipulation utilities |
| **using Statement** | Essential for proper resource cleanup |
| **FileMode** | Controls how file is opened |
| **FileAccess** | Controls read/write permissions |

### Quick Reference

```csharp
// Quick file operations
File.WriteAllText(path, text);      // Write all text
File.ReadAllText(path);             // Read all text
File.AppendAllText(path, text);     // Append text
File.ReadAllLines(path);            // Read lines array

// Stream operations
using (var writer = new StreamWriter(path)) { writer.WriteLine("line"); }
using (var reader = new StreamReader(path)) { string content = reader.ReadToEnd(); }

// Path utilities
Path.Combine(dir, file);            // Combine paths
Path.GetFileName(path);             // Get filename
Path.GetDirectoryName(path);        // Get directory
```
