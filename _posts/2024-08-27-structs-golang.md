---
layout: distill
title: "Understanding Structs in Go: A Beginner's Guide"
date: 2024-08-27 12:00:00
description: Learn the basics of structs in Go and how to use them effectively in your programs.
tags: golang, structs
categories: golang
featured: false
toc:
  - name: What is a Struct?
  - name: Defining a Struct
  - name: Initializing Structs
  - name: Accessing and Modifying Struct Fields
  - name: Embedding and Inheritance with Structs
  - name: 'Methods on Structs: Pass by Value vs. Pass by Reference'
---

## What is a Struct?

In Go, a struct is a composite data type that groups together variables under a single name. These variables, known as **fields**, can have different types and are used to represent data more complex than a single primitive type. Structs are the closest thing Go has to classes in object-oriented languages.

Structs provide a way to create more complex data structures that mirror real-world entities. For example, if you were writing an application to manage a library, you could define a struct to represent a book, with fields for the title, author, and publication year.

## Defining a Struct

To define a struct in Go, use the `type` keyword followed by the name of the struct and the `struct` keyword. Inside the curly braces `{}`, list the fields along with their types. Here's an example:

```go
package main

import "fmt"

// Defining a struct named Book
type Book struct {
    Title  string
    Author string
    Pages  int
}

func main() {
    // Using the struct in the main function
    fmt.Println("Structs in Go")
}
```

In this example, `Book` is a struct with three fields: `Title` (a string), `Author` (a string), and `Pages` (an integer).

## Initializing Structs

Once you have defined a struct, you can create instances of it in several ways.

### Using a Literal

You can create and initialize a struct in a single step using a struct literal:

```go
package main

import "fmt"

type Book struct {
    Title  string
    Author string
    Pages  int
}

func main() {
    // Initializing a struct using a literal
    myBook := Book{
        Title:  "The Go Programming Language",
        Author: "Alan Donovan",
        Pages:  380,
    }

    fmt.Println(myBook)
}
```

This creates a `Book` instance called `myBook` with the specified values.

### Using the `new` Keyword

Alternatively, you can create a pointer to a struct using the `new` keyword:

```go
package main

import "fmt"

type Book struct {
    Title  string
    Author string
    Pages  int
}

func main() {
    // Creating a pointer to a struct
    myBook := new(Book)
    myBook.Title = "The Go Programming Language"
    myBook.Author = "Alan Donovan"
    myBook.Pages = 380

    fmt.Println(*myBook)
}
```

This approach is useful when you want to work with pointers.

## Accessing and Modifying Struct Fields

Once you have a struct instance, you can access and modify its fields using the dot `.` notation:

```go
package main

import "fmt"

type Book struct {
    Title  string
    Author string
    Pages  int
}

func main() {
    myBook := Book{
        Title:  "The Go Programming Language",
        Author: "Alan Donovan",
        Pages:  380,
    }

    // Accessing fields
    fmt.Println("Title:", myBook.Title)
    fmt.Println("Author:", myBook.Author)
    fmt.Println("Pages:", myBook.Pages)

    // Modifying fields
    myBook.Pages = 400
    fmt.Println("Updated Pages:", myBook.Pages)
}
```

Here, the `Title`, `Author`, and `Pages` fields of the `myBook` instance are accessed and modified.

## Embedding and Inheritance with Structs

Go doesn't have traditional inheritance like some other languages, but it does allow for composition through struct embedding. You can embed one struct within another, allowing the embedded struct's fields and methods to be accessed directly from the outer struct.

```go
package main

import "fmt"

// Define a basic struct
type Person struct {
    Name string
    Age  int
}

// Embedding Person struct in Author struct
type Author struct {
    Person
    BooksPublished int
}

func main() {
    author := Author{
        Person: Person{
            Name: "Alan Donovan",
            Age:  45,
        },
        BooksPublished: 5,
    }

    // Accessing embedded struct's fields
    fmt.Println("Author Name:", author.Name)
    fmt.Println("Author Age:", author.Age)
    fmt.Println("Books Published:", author.BooksPublished)
}
```

In this example, the `Author` struct embeds the `Person` struct, and you can access the `Name` and `Age` fields directly on an `Author` instance.

Here’s the final section of the article focused on methods, including passing by value and by reference:

## Methods on Structs: Pass by Value vs. Pass by Reference

In Go, you can define methods on structs to associate behaviors with the data they encapsulate. These methods can either receive a copy of the struct (pass by value) or a reference to the struct (pass by reference). Understanding the difference between these two approaches is essential for writing efficient and bug-free Go code.

### Pass by Value

When a method is defined with a value receiver, the method operates on a copy of the struct. Any modifications made within the method do not affect the original struct.

```go
package main

import "fmt"

type Rectangle struct {
    Width  int
    Height int
}

// Method with a value receiver
func (r Rectangle) Area() int {
    return r.Width * r.Height
}

func (r Rectangle) Scale(factor int) {
    r.Width *= factor
    r.Height *= factor
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    fmt.Println("Area:", rect.Area()) // Outputs: Area: 50

    rect.Scale(2)
    fmt.Println("Scaled Width:", rect.Width)  // Outputs: Scaled Width: 10
    fmt.Println("Scaled Height:", rect.Height) // Outputs: Scaled Height: 5
}
```

In this example, calling `rect.Scale(2)` does not change the `Width` or `Height` of `rect` because the method operates on a copy of the struct. The original struct remains unchanged.

### Pass by Reference

To modify the original struct within a method, you can use a pointer receiver. This approach allows the method to operate directly on the struct's memory address, enabling it to make changes that persist outside the method scope.

```go
package main

import "fmt"

type Rectangle struct {
    Width  int
    Height int
}

// Method with a pointer receiver
func (r *Rectangle) Scale(factor int) {
    r.Width *= factor
    r.Height *= factor
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    rect.Scale(2)
    fmt.Println("Scaled Width:", rect.Width)  // Outputs: Scaled Width: 20
    fmt.Println("Scaled Height:", rect.Height) // Outputs: Scaled Height: 10
}
```

Here, `Scale` is defined with a pointer receiver `*Rectangle`, allowing it to modify the original `rect` struct. After scaling, the `Width` and `Height` values are updated as expected.

### Choosing Between Pass by Value and Pass by Reference

- **Pass by Value**: Use this when the method does not need to modify the original struct. It can be more efficient when working with small structs, as copying a small amount of data is generally inexpensive.
  
- **Pass by Reference**: Use this when the method needs to alter the struct’s data. It is more efficient for larger structs because it avoids copying the entire struct, which can be costly in terms of performance.

Understanding when to use each approach allows you to write clearer, more efficient Go code that behaves as intended.
```


## Conclusion

Structs are a fundamental part of Go's type system and are essential for creating more complex and meaningful data structures. By understanding how to define, initialize, and manipulate structs, as well as how to use methods and embedding, you can write more organized and maintainable Go code.

Start experimenting with structs in your projects, and you'll see how they can help you manage and structure your data more effectively.
```
