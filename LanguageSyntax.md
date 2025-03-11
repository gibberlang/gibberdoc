# Gibber Language Syntax Specification

## Overview

Gibber is designed to transpile to TypeScript while introducing syntax features that make it more suitable for AI-assisted development. This specification outlines the core syntax elements of Gibber.

## Key aspects of the syntax specification include:

- Rich Metadata - Extensive use of annotations like @purpose, @requires, and @ensures to make intent explicit
- Type System - Strong typing with constrained types like Range<0..100>
- Pattern Templates - First-class support for common design patterns
- Effect System - Tracking and controlling side effects
- Verification Blocks - Built-in testing and verification
- Explicit Scoping - Named blocks and clear scope boundaries
- TypeScript Interoperability - Direct interop with TypeScript for practical adoption

## File Structure

Gibber files use the `.gib` extension. Each file typically contains:

```
// Optional file-level metadata
@module("Authentication")
@description("Handles user authentication and session management")
@author("AI-Human Collaboration Team")

// Imports from other modules
import { User, Session } from "./user.gib"
import { Logger } from "@gibber/logging"

// Module code
// ...
```

## Comments

Gibber supports three types of comments:

```gibber
// Single-line comment

/* 
   Multi-line comment
*/

/**
 * Documentation comment
 * These are structured and can contain metadata
 * @param {string} name - Description of parameter
 * @returns {boolean} Description of return value
 */
```

## Variables and Constants

Variable declarations use explicit typing:

```gibber
// Variable declaration
var name: String = "Gibber"

// Constant declaration
const PI: Float = 3.14159

// Type inference is available but explicit types are encouraged
var inferred = "This is a string" // Type String is inferred

// Constrained type (valid value range)
var percentage: Range<0..100> = 75
```

## Basic Types

Gibber provides the following primitive types:

```gibber
// Numeric types
var integer: Int = 42
var decimal: Float = 3.14
var scientific: Float = 6.02e23
var money: Decimal = 19.99d // precise decimal

// Text types
var character: Char = 'a'
var text: String = "Hello, Gibber!"

// Boolean
var flag: Bool = true

// Void type (for functions that don't return values)
var nothing: Void = void

// Optional types (can be null/undefined)
var optional: Optional<String> = null
var shorthand: String? = null // Equivalent to Optional<String>
```

## Control Structures

Gibber provides familiar control structures with explicit block scoping:

```gibber
// If-else statement
if (condition) {
    // code
} else if (anotherCondition) {
    // code
} else {
    // code
}

// Switch statement with exhaustiveness checking
switch (value) {
    case "first":
        // code
        break
    case "second":
        // code
        break
    default:
        // code
}

// For loop with range
for (var i in range(0, 10)) {
    // code
}

// For-each loop
for (var item in collection) {
    // code
}

// While loop
while (condition) {
    // code
}

// Do-while loop
do {
    // code
} while (condition)

// Named blocks for clearer scoping
block initializationPhase {
    // code with clear scope
}
```

## Functions

Functions include rich metadata and explicit contracts:

```gibber
@purpose("Authenticate a user with given credentials")
@requires(username.length > 0, "Username must not be empty")
@requires(password.length >= 8, "Password must be at least 8 characters")
@ensures(result != null, "Result is never null")
@complexity(time: O(1), space: O(1))
@throws("AuthError", "When credentials are invalid")
func authenticate(username: String, password: String) -> AuthResult {
    // Implementation
    return new AuthResult(true, "session123")
}

// Function with optional parameters
func greet(name: String, greeting: String = "Hello") -> String {
    return `${greeting}, ${name}!`
}

// Function with named parameters
func configure(server: String, @named port: Int = 8080, @named secure: Bool = false) {
    // Implementation
}
// Called as: configure("example.com", port: 443, secure: true)

// Anonymous functions
var callback = func(x: Int) -> Int {
    return x * 2
}

// Arrow functions for simple expressions
var double = (x: Int) => x * 2
```

## Classes and Objects

Classes in Gibber are similar to TypeScript but with added metadata:

```gibber
@purpose("Represents a user in the system")
@pattern("Entity")
class User {
    // Properties with access modifiers
    public username: String
    private password: String
    protected loginAttempts: Int = 0
    
    // Constructor
    @purpose("Create a new user with given credentials")
    constructor(username: String, password: String) {
        this.username = username
        this.password = password
    }
    
    // Methods
    @purpose("Check if provided password matches user's password")
    @pure // No side effects
    public validatePassword(input: String) -> Bool {
        return this.password == input
    }
    
    // Static methods
    @purpose("Create an anonymous user")
    static createAnonymous() -> User {
        return new User("anonymous", "")
    }
}
```

## Interfaces

Interfaces define contracts for classes:

```gibber
@purpose("Defines authentication provider capabilities")
interface AuthProvider {
    @purpose("Authenticate user with given credentials")
    authenticate(username: String, password: String) -> AuthResult
    
    @purpose("Log out a user session")
    logout(sessionId: String) -> Void
    
    // Read-only property
    @purpose("Check if provider is currently available")
    readonly isAvailable: Bool
}
```

## Enums

Enums provide type-safe sets of related constants:

```gibber
@purpose("Possible authentication result statuses")
enum AuthStatus {
    SUCCESS,
    INVALID_CREDENTIALS,
    ACCOUNT_LOCKED,
    SYSTEM_ERROR
}

// Enum with associated values
@purpose("Authentication result with associated data")
enum AuthResult {
    SUCCESS(sessionId: String),
    FAILURE(reason: String),
    MFA_REQUIRED(methods: Array<String>)
}
```

## Pattern Templates

Pattern templates are a unique feature of Gibber, enabling reuse of common patterns:

```gibber
// Define a repository pattern template
@pattern("Repository<T>")
template Repository<T> {
    @requires(T must have property "id")
    interface {
        findById(id: String) -> Optional<T>
        findAll() -> Array<T>
        save(item: T) -> T
        delete(id: String) -> Bool
    }
    
    // Optional default implementation
    implementation {
        private items: Map<String, T> = new Map()
        
        findById(id: String) -> Optional<T> {
            return this.items.get(id)
        }
        
        // Other method implementations...
    }
}

// Use the template
@instantiate(Repository<User>)
class UserRepository {
    // Inherits interface and default implementations
    // Can override specific methods as needed
    
    @override
    findById(id: String) -> Optional<User> {
        // Custom implementation
        // ...
    }
}
```

## Error Handling

Gibber provides structured error handling:

```gibber
// Try-catch-finally
try {
    // Code that might throw
} catch (e: NetworkError) {
    // Handle NetworkError
} catch (e: ValidationError) {
    // Handle ValidationError
} catch (e) {
    // Handle any other error
} finally {
    // Always executed
}

// Result type for functions that might fail
func divide(a: Int, b: Int) -> Result<Int, DivisionError> {
    if (b == 0) {
        return failure(new DivisionError("Division by zero"))
    }
    return success(a / b)
}

// Using the result
var result = divide(10, 2)
if (result.isSuccess) {
    var value = result.value
} else {
    var error = result.error
}

// Pattern matching on result
match (divide(10, 0)) {
    success(value) => {
        // Use value
    },
    failure(DivisionError error) => {
        // Handle specific error
    },
    failure(error) => {
        // Handle other errors
    }
}
```

## Modules and Namespaces

Code organization in Gibber uses modules and namespaces:

```gibber
// Declaring a module
@module("Authentication")
// Module code...

// Namespace for grouping related elements
namespace Validation {
    @purpose("Validates user input")
    func validateEmail(email: String) -> Bool {
        // Implementation
    }
    
    @purpose("Validates password strength")
    func validatePassword(password: String) -> ValidationResult {
        // Implementation
    }
}

// Using namespace elements
var isValid = Validation.validateEmail("user@example.com")
```

## Decorators and Annotations

Gibber uses annotations extensively for metadata:

```gibber
// Core annotations for intent and behavior
@purpose("...")      // Describes the purpose
@requires(...)       // Precondition
@ensures(...)        // Postcondition
@complexity(...)     // Time/space complexity
@pure                // No side effects
@throws(...)         // Possible exceptions
@deprecated(...)     // Marked for removal

// Custom annotations
@author("...")
@version("...")
@todo("...")
@example("...")
```

## Verification Blocks

Verification blocks provide built-in testing:

```gibber
@verify {
    // Assertions that must hold true
    assert authenticate("admin", "correct") != null
    assert authenticate("admin", "wrong") == null
    
    // Property-based tests
    forAll (password: String where password.length < 8) {
        assert !authenticate("user", password)
    }
}
```

## Effect System

The effect system tracks and controls side effects:

```gibber
// Pure function (no side effects)
@pure
func add(a: Int, b: Int) -> Int {
    return a + b
}

// Function with explicit side effects
@effects("filesystem", "network")
func saveUserToRemote(user: User) -> Void {
    // Implementation with side effects
}

// Function with limited effects
@effects("database")
func saveUser(user: User) -> Void {
    // Only database effects allowed
}
```

## TypeScript Interoperability

Gibber code can interoperate with TypeScript:

```gibber
// Import from TypeScript
import { Component } from "react"

// Export for TypeScript
@export
class GibberComponent {
    // Implementation
}

// Inline TypeScript blocks
@typescript {
    // Direct TypeScript code
    interface ReactProps {
        title: string;
        count: number;
    }
}

// TypeScript type references
var props: @tstype("ReactProps") = { title: "Hello", count: 42 }
```
