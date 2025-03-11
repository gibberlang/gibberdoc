# Gibber Core Type System

## Overview

Gibber's type system is designed to enhance code reliability and AI comprehension through explicit constraints, rich metadata, and predictable patterns. The type system balances static safety with flexibility, ensuring that both humans and AI can reason effectively about code behavior.

## Design Principles

The Gibber type system is built on these foundational principles:

1. **Intent Transparency** - Types express not just data structure but intended use
2. **Constraint Explicitness** - Value constraints are part of the type itself
3. **Effect Awareness** - Types track potential side effects of operations
4. **Pattern Recognition** - Common type patterns are standardized for consistency
5. **Verification Integration** - Types participate in automatic verification

## Core Types

### Primitive Types

Gibber provides the following primitive types:

| Type | Description | Examples |
|------|-------------|----------|
| `Int` | Integer values | `42`, `-10` |
| `Float` | Floating-point numbers | `3.14`, `-0.01`, `6.02e23` |
| `Decimal` | Precise decimal values | `19.99d`, `1000.00d` |
| `Bool` | Boolean values | `true`, `false` |
| `Char` | Single character | `'a'`, `'\n'` |
| `String` | Text sequence | `"Hello"` |
| `Void` | No value | `void` |

### Constrained Primitive Types

Primitives can be constrained to specific ranges or patterns:

```gibber
// Numeric ranges
type Percentage = Range<0..100>
type Probability = Range<0.0..1.0>
type PositiveInt = Range<1..∞>

// String patterns
type Email = String @matches("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")
type Username = String @matches("^[a-z0-9_]{3,16}$")

// Enumerations of allowed values
type Direction = OneOf<"north", "south", "east", "west">
type HttpStatus = OneOf<200, 404, 500> // Limited to specific values
```

### Compound Types

Gibber provides various ways to compose types:

```gibber
// Arrays
type Names = Array<String>
type Matrix = Array<Array<Float>>

// Maps
type UserMap = Map<String, User>
type ConfigMap = Map<String, OneOf<String, Int, Bool>>

// Tuples (fixed-length arrays with heterogeneous types)
type Point = Tuple<Float, Float> // (x, y)
type RGB = Tuple<Int, Int, Int>  // (r, g, b)

// Records (named fields)
type Person = Record<{
    name: String,
    age: Int,
    email: Optional<String>
}>

// Union types
type NumberOrString = Union<Int, String>
type Result = Union<Success, Failure>

// Intersection types
type AdminUser = Intersection<User, AdminPermissions>
```

### Optional and Nullable Types

Gibber distinguishes between optional and nullable values:

```gibber
// Optional types may or may not have a value
type OptionalAge = Optional<Int>  // Can be undefined
var age: OptionalAge = undefined  // Valid
var age: OptionalAge = 30         // Valid

// Shorthand notation
type OptionalName = String?       // Equivalent to Optional<String>

// Nullable types can have a value or be null
type NullableName = Nullable<String>  // Can be null
var name: NullableName = null     // Valid
var name: NullableName = "Alice"  // Valid

// Types that are both optional and nullable
type MaybeString = Optional<Nullable<String>>  // Can be undefined or null
```

## Advanced Type Features

### Result Types

For functions that might fail, Gibber provides a built-in Result type:

```gibber
// Result type definition
type Result<T, E> = Union<Success<T>, Failure<E>>

// Function returning a result
func divide(a: Int, b: Int) -> Result<Float, DivisionError> {
    if (b == 0) {
        return failure(new DivisionError("Division by zero"))
    }
    return success(a / b)
}

// Using results with pattern matching
match (divide(10, 2)) {
    success(value) => {
        // Use the value
    },
    failure(error) => {
        // Handle the error
    }
}
```

### Effect Types

Gibber tracks side effects through the type system:

```gibber
// Pure function type (no side effects)
type PureFunction<Args, Return> = (args: Args) -> Return

// Function with specific effects
type IOFunction<Args, Return> = (args: Args) -> Return with effects "io"
type NetworkFunction<Args, Return> = (args: Args) -> Return with effects "network"

// Combined effects
type DatabaseLogger<Args, Return> = (args: Args) -> Return with effects "database", "logging"

// Usage
@effects("database")
func saveUser(user: User) -> Void {
    // Implementation...
}

// Type checking ensures effect compatibility
var dbFunc: DatabaseLogger<User, Void> = saveUser  // Valid
var pureFunc: PureFunction<User, Void> = saveUser  // Compile error - effects mismatch
```

### Refinement Types

Refinement types add predicates to base types:

```gibber
// Type refined with predicate
type PositiveNumber = Int where value > 0
type ValidEmail = String where isValidEmail(value)

// Using refinement types
func sendEmail(email: ValidEmail, subject: String) -> Void {
    // No need to validate email - type system guarantees it's valid
}

// Asserting a refinement
var userInput = getUserInput()
var email = assert<ValidEmail>(userInput)  // Runtime check
sendEmail(email, "Welcome!")
```

### Type States

Type states represent objects that change their type based on state:

```gibber
// Connection that changes type based on state
@typestate
interface Connection {
    state Disconnected {
        connect() -> Connected
    }
    
    state Connected {
        send(message: String) -> Connected
        disconnect() -> Disconnected
    }
}

// Usage enforces valid state transitions
var conn: Connection.Disconnected = new Connection()
var connected = conn.connect()      // Type is now Connection.Connected
connected.send("Hello")             // Valid
conn.send("Test")                   // Compile error - wrong state
```

### Dependent Types

Gibber supports limited dependent types for advanced constraints:

```gibber
// Array with length in type
type Vector<T, N: Int> = Array<T> where length == N

// Matrix with dimension types
type Matrix<T, Rows: Int, Cols: Int> = Array<Array<T>> 
    where length == Rows && all(row => row.length == Cols)

// Usage
func dotProduct(a: Vector<Float, 3>, b: Vector<Float, 3>) -> Float {
    // Dimensions guaranteed by type system
    return a[0] * b[0] + a[1] * b[1] + a[2] * b[2]
}
```

## Type Inference

Gibber provides type inference but encourages explicit typing:

```gibber
// Type inference
var name = "Alice"              // Inferred as String
var numbers = [1, 2, 3]         // Inferred as Array<Int>
var mixed = [1, "two", true]    // Inferred as Array<Union<Int, String, Bool>>

// Type annotation recommended for clarity
var name: String = "Alice"      // Explicit is preferred
```

## Generics

Gibber supports powerful generics with constraints:

```gibber
// Generic function
func first<T>(items: Array<T>) -> Optional<T> {
    if (items.length > 0) {
        return items[0]
    }
    return undefined
}

// Generic with constraints
func sortBy<T, K>(items: Array<T>, keyFn: (item: T) -> K) -> Array<T>
    where K implements Comparable {
    // Implementation...
}

// Generic class
class Box<T> {
    private value: T
    
    constructor(value: T) {
        this.value = value
    }
    
    get() -> T {
        return this.value
    }
    
    set(newValue: T) -> Void {
        this.value = newValue
    }
}

// Using generics
var numberBox = new Box<Int>(42)
var stringBox = new Box("Hello")  // Type inferred as Box<String>
```

## Type Compatibility

Gibber uses structural typing similar to TypeScript:

```gibber
// Structural typing example
interface Named {
    name: String
}

class Person {
    name: String
    age: Int
    
    constructor(name: String, age: Int) {
        this.name = name
        this.age = age
    }
}

// Valid assignment because Person has a name property
var named: Named = new Person("Alice", 30)
```

## Type Aliases and Type Operators

Gibber allows for type manipulation:

```gibber
// Type alias
type UserID = String

// Type operators
type Nullable<T> = Union<T, Null>
type ReadOnly<T> = T with { readonly: true }
type DeepReadOnly<T> = /* recursive application of readonly */

// Pick and Omit
type UserView = Pick<User, "name", "email">
type PublicUser = Omit<User, "password", "privateData">

// Mapped types
type Optional<T> = { [K in keyof T]?: T[K] }
type Mutable<T> = { -readonly [K in keyof T]: T[K] }
```

## Type Annotations and Metadata

Gibber types can include rich metadata:

```gibber
// Type with metadata
@purpose("Represents a user in the system")
@since("1.0.0")
@example("new User('alice@example.com', 'password123')")
type User = {
    @purpose("Unique identifier for the user")
    @format("email")
    email: String,
    
    @purpose("User's encrypted password")
    @sensitive
    password: String,
    
    @purpose("User's profile information")
    @optional
    profile: UserProfile?
}
```

## Type Verification

Types in Gibber participate in verification:

```gibber
// Verify type properties
@verify type PositiveNumber {
    assert new PositiveNumber(5) is valid
    assert new PositiveNumber(0) is invalid
    assert new PositiveNumber(-1) is invalid
}

// Verify type behavior
@verify type Result<String, Error> {
    assert success("value").isSuccess == true
    assert failure(new Error()).isFailure == true
    
    forAll (s: String) {
        assert success(s).value == s
    }
}
```

## Integration with TypeScript

Gibber types transpile to TypeScript:

```gibber
// Gibber type
type User = {
    id: String,
    name: String,
    age: Range<0..120>
}

// Transpiles to TypeScript as:
/*
interface User {
    id: string;
    name: string;
    age: number; // Runtime checks added for range
}
*/

// TypeScript interop
@typescript
type ReactProps = {
    title: string;
    onClick: () => void;
}

// Using TypeScript types
func renderButton(props: @tstype("ReactProps")) -> String {
    // Implementation...
}
```
