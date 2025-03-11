# Gibber Pattern Templates and Standard Library

## Pattern Templates Overview

Pattern templates are one of Gibber's most distinctive features, designed specifically to help both humans and LLMs implement common programming patterns consistently and correctly. Unlike traditional libraries or frameworks, pattern templates in Gibber are first-class language constructs that encode both structure and behavior.

## Why Pattern Templates?

LLMs excel at pattern recognition but can struggle with consistent implementation across larger codebases. Gibber's pattern templates address this by:

1. Making patterns explicit and named
2. Ensuring consistent implementation
3. Providing built-in verification for pattern correctness
4. Reducing the cognitive load for both humans and AI

## Core Pattern Template Concepts

### Template Definition

A pattern template defines a reusable programming pattern with structural and behavioral requirements:

```gibber
@pattern("Repository")
template Repository<T> {
    @requires(T has property "id": String)
    
    // Interface that implementations must fulfill
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
        
        findAll() -> Array<T> {
            return Array.from(this.items.values())
        }
        
        save(item: T) -> T {
            this.items.set(item.id, item)
            return item
        }
        
        delete(id: String) -> Bool {
            return this.items.delete(id)
        }
    }
    
    // Pattern-specific verification
    @verify {
        // Verify core Repository behavior
        test "save and find" {
            const repo = new RepositoryImpl<TestItem>()
            const item = { id: "1", value: "test" }
            repo.save(item)
            assert repo.findById("1") == item
        }
    }
}
```

### Template Instantiation

Patterns can be instantiated for specific types:

```gibber
// Define an entity type
class User {
    id: String
    name: String
    email: String
    
    constructor(id: String, name: String, email: String) {
        this.id = id
        this.name = name
        this.email = email
    }
}

// Instantiate the Repository pattern for User
@instantiate(Repository<User>)
class UserRepository {
    // Inherits interface and default implementations from Repository<User>
    
    // Add custom methods
    findByEmail(email: String) -> Optional<User> {
        return this.findAll().find(user => user.email == email)
    }
    
    // Override default implementation if needed
    @override
    findAll() -> Array<User> {
        // Custom implementation that might use a database
        // ...
    }
}
```

### Pattern Composition

Patterns can be composed together:

```gibber
// Instantiate multiple patterns
@instantiate(Repository<User>)
@instantiate(Validator<User>)
@instantiate(Factory<User>)
class UserService {
    // Inherits from all three patterns
    // ...
}
```

## Standard Pattern Templates

Gibber's standard library provides a rich set of pre-defined pattern templates:

### Data Access Patterns

```gibber
// Repository Pattern (data access abstraction)
@pattern("Repository<T>")

// Unit of Work (transaction management)
@pattern("UnitOfWork")

// Data Mapper (object-relational mapping)
@pattern("DataMapper<T, R>")

// Active Record (self-persisting objects)
@pattern("ActiveRecord<T>")

// Query Object (encapsulated queries)
@pattern("Query<T>")
```

### Behavioral Patterns

```gibber
// Observer (publish-subscribe)
@pattern("Observable<T>")
@pattern("Observer<T>")

// Strategy (interchangeable algorithms)
@pattern("Strategy<Input, Output>")

// Command (encapsulated operations)
@pattern("Command<T>")
@pattern("CommandHandler<T>")

// State Machine
@pattern("StateMachine<S, E>")

// Visitor (separates algorithms from object structure)
@pattern("Visitor<T>")
```

### Creational Patterns

```gibber
// Factory (object creation)
@pattern("Factory<T>")

// Builder (step-by-step object construction)
@pattern("Builder<T>")

// Singleton (single instance)
@pattern("Singleton<T>")

// Prototype (cloning objects)
@pattern("Prototype<T>")

// Dependency Injection
@pattern("Injectable")
@pattern("Injector")
```

### Structural Patterns

```gibber
// Adapter (interface compatibility)
@pattern("Adapter<From, To>")

// Decorator (dynamic behavior extension)
@pattern("Decorator<T>")

// Proxy (controlled access)
@pattern("Proxy<T>")

// Composite (tree structures)
@pattern("Composite<T>")

// Facade (simplified interface)
@pattern("Facade<T>")
```

### Validation Patterns

```gibber
// Validator (input validation)
@pattern("Validator<T>")

// Specification (business rules)
@pattern("Specification<T>")

// Chain of Responsibility (processing pipeline)
@pattern("Chain<T>")
```

### Asynchronous Patterns

```gibber
// Promise (async operations)
@pattern("Promise<T>")

// Future (async computation)
@pattern("Future<T>")

// Reactive (event streams)
@pattern("Observable<T>")
@pattern("Subscriber<T>")
```

## Standard Library Overview

Gibber's standard library complements pattern templates with core utilities and data structures designed for reliability and explicit intent.

### Core Data Structures

#### Collections

```gibber
// Arrays with enhanced operations
Array<T> {
    @pure filter(predicate: (item: T) -> Bool) -> Array<T>
    @pure map<U>(transform: (item: T) -> U) -> Array<U>
    @pure reduce<U>(initial: U, reducer: (acc: U, item: T) -> U) -> U
    @pure sort(comparator?: (a: T, b: T) -> Int) -> Array<T>
    @pure groupBy<K>(keyFn: (item: T) -> K) -> Map<K, Array<T>>
    // ...more operations
}

// Maps
Map<K, V> {
    @pure entries() -> Array<Tuple<K, V>>
    @pure keys() -> Array<K>
    @pure values() -> Array<V>
    @pure filter(predicate: (key: K, value: V) -> Bool) -> Map<K, V>
    @pure map<W>(transform: (key: K, value: V) -> W) -> Map<K, W>
    // ...more operations
}

// Sets
Set<T> {
    @pure union(other: Set<T>) -> Set<T>
    @pure intersection(other: Set<T>) -> Set<T>
    @pure difference(other: Set<T>) -> Set<T>
    @pure isSubsetOf(other: Set<T>) -> Bool
    // ...more operations
}

// Tuples
Tuple<T1, T2, ...> {
    @pure first() -> T1
    @pure second() -> T2
    @pure nth<N extends number>(n: N) -> TupleElement<N>
    // ...more operations
}
```

#### Data Handling

```gibber
// Result type for operations that might fail
Result<T, E> {
    @pure isSuccess: Bool
    @pure isFailure: Bool
    @pure value: T // Throws if failure
    @pure error: E // Throws if success
    @pure map<U>(fn: (value: T) -> U) -> Result<U, E>
    @pure mapError<F>(fn: (error: E) -> F) -> Result<T, F>
    @pure flatMap<U>(fn: (value: T) -> Result<U, E>) -> Result<U, E>
    @pure recover(fn: (error: E) -> T) -> T
    // ...more operations
}

// Optional values
Optional<T> {
    @pure hasValue: Bool
    @pure value: T // Throws if no value
    @pure orElse(defaultValue: T) -> T
    @pure map<U>(fn: (value: T) -> U) -> Optional<U>
    @pure flatMap<U>(fn: (value: T) -> Optional<U>) -> Optional<U>
    // ...more operations
}
```

### Functional Utilities

```gibber
// Function utilities
Function {
    @pure compose<A, B, C>(f: (b: B) -> C, g: (a: A) -> B) -> (a: A) -> C
    @pure pipe<A, B, C>(g: (a: A) -> B, f: (b: B) -> C) -> (a: A) -> C
    @pure curry<A, B, C>(f: (a: A, b: B) -> C) -> (a: A) -> (b: B) -> C
    @pure memoize<A, B>(f: (a: A) -> B) -> (a: A) -> B
    // ...more operations
}

// Lazy evaluation
Lazy<T> {
    @pure value: T // Computes only on first access
    @pure map<U>(fn: (value: T) -> U) -> Lazy<U>
    // ...more operations
}
```

### Concurrency and Asynchronous Operations

```gibber
// Promise for asynchronous operations
Promise<T> {
    @effects("async")
    then<U>(onFulfilled: (value: T) -> U | Promise<U>) -> Promise<U>
    
    @effects("async")
    catch<U>(onRejected: (error: Error) -> U | Promise<U>) -> Promise<U | T>
    
    @effects("async")
    finally(onFinally: () -> Void) -> Promise<T>
    
    // Static methods
    @effects("async")
    static all<T>(promises: Array<Promise<T>>) -> Promise<Array<T>>
    
    @effects("async")
    static race<T>(promises: Array<Promise<T>>) -> Promise<T>
    
    // ...more operations
}

// Async/await syntax
@effects("async")
func fetchUserData(userId: String) -> Promise<User> {
    const response = await api.fetch(`/users/${userId}`)
    return await response.json()
}
```

### IO and Effects

```gibber
// Controlled file IO
@effects("filesystem")
FileSystem {
    readFile(path: String) -> Result<String, FileError>
    writeFile(path: String, content: String) -> Result<Void, FileError>
    exists(path: String) -> Bool
    // ...more operations
}

// Network operations
@effects("network")
Http {
    get(url: String) -> Promise<Response>
    post(url: String, data: Any) -> Promise<Response>
    put(url: String, data: Any) -> Promise<Response>
    delete(url: String) -> Promise<Response>
    // ...more operations
}

// Logging with explicit levels
@effects("logging")
Logger {
    debug(message: String, ...args: Array<Any>) -> Void
    info(message: String, ...args: Array<Any>) -> Void
    warn(message: String, ...args: Array<Any>) -> Void
    error(message: String, ...args: Array<Any>) -> Void
    // ...more operations
}
```

### Date and Time

```gibber
// Immutable date/time handling
DateTime {
    @pure now() -> DateTime
    @pure parse(str: String) -> Result<DateTime, DateTimeError>
    @pure format(pattern: String) -> String
    @pure plus(duration: Duration) -> DateTime
    @pure minus(duration: Duration) -> DateTime
    @pure diff(other: DateTime) -> Duration
    // ...more operations
}

Duration {
    @pure seconds() -> Int
    @pure minutes() -> Int
    @pure hours() -> Int
    @pure days() -> Int
    @pure plus(other: Duration) -> Duration
    @pure minus(other: Duration) -> Duration
    // ...more operations
}
```

### Text Processing

```gibber
// String utilities with explicit intent
String {
    @pure isEmpty() -> Bool
    @pure contains(substring: String) -> Bool
    @pure startsWith(prefix: String) -> Bool
    @pure endsWith(suffix: String) -> Bool
    @pure split(separator: String) -> Array<String>
    @pure replace(search: String, replacement: String) -> String
    @pure match(pattern: Regex) -> Optional<Array<String>>
    @pure trim() -> String
    @pure toLowerCase() -> String
    @pure toUpperCase() -> String
    // ...more operations
}

// Regular expressions
Regex {
    @pure test(str: String) -> Bool
    @pure exec(str: String) -> Optional<Array<String>>
    @pure replace(str: String, replacement: String) -> String
    // ...more operations
}
```

### Math and Numerics

```gibber
// Mathematical operations
Math {
    @pure abs(x: Number) -> Number
    @pure min(x: Number, y: Number) -> Number
    @pure max(x: Number, y: Number) -> Number
    @pure floor(x: Number) -> Int
    @pure ceil(x: Number) -> Int
    @pure round(x: Number) -> Int
    @pure pow(base: Number, exponent: Number) -> Number
    @pure sqrt(x: Number) -> Number
    @pure random() -> Float // Returns 0.0 <= x < 1.0
    // ...more operations
}

// Numeric utilities with range checking
Number {
    @pure clamp(min: Number, max: Number) -> Number
    @pure isInRange(min: Number, max: Number) -> Bool
    @pure toFixed(digits: Int) -> String
    // ...more operations
}
```

### Serialization and Parsing

```gibber
// JSON handling with explicit error cases
JSON {
    @pure parse(str: String) -> Result<Any, JSONError>
    @pure stringify(value: Any) -> Result<String, JSONError>
    // ...more operations
}

// Structured data parsing
Parser<T> {
    @pure parse(input: String) -> Result<T, ParseError>
    @pure map<U>(transform: (value: T) -> U) -> Parser<U>
    @pure andThen<U>(next: Parser<U>) -> Parser<Tuple<T, U>>
    @pure orElse(alternative: Parser<T>) -> Parser<T>
    // ...more operations
}
```

## Pattern Template Examples

### Repository Pattern Example

```gibber
// Entity class
class User {
    id: String
    name: String
    email: String
    
    constructor(id: String, name: String, email: String) {
        this.id = id
        this.name = name
        this.email = email
    }
}

// Implementing repository pattern
@instantiate(Repository<User>)
class UserRepository {
    private db: Database
    
    constructor(db: Database) {
        this.db = db
    }
    
    @override
    @effects("database")
    async findById(id: String) -> Promise<Optional<User>> {
        const result = await this.db.query(
            "SELECT * FROM users WHERE id = ?",
            [id]
        )
        
        if (result.rows.length === 0) {
            return undefined
        }
        
        const row = result.rows[0]
        return new User(row.id, row.name, row.email)
    }
    
    @override
    @effects("database")
    async findAll() -> Promise<Array<User>> {
        const result = await this.db.query("SELECT * FROM users")
        
        return result.rows.map(row => 
            new User(row.id, row.name, row.email)
        )
    }
    
    @override
    @effects("database")
    async save(user: User) -> Promise<User> {
        await this.db.query(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?) " +
            "ON DUPLICATE KEY UPDATE name = ?, email = ?",
            [user.id, user.name, user.email, user.name, user.email]
        )
        
        return user
    }
    
    @override
    @effects("database")
    async delete(id: String) -> Promise<Bool> {
        const result = await this.db.query(
            "DELETE FROM users WHERE id = ?",
            [id]
        )
        
        return result.affectedRows > 0
    }
    
    // Custom repository method
    @effects("database")
    async findByEmail(email: String) -> Promise<Optional<User>> {
        const result = await this.db.query(
            "SELECT * FROM users WHERE email = ?",
            [email]
        )
        
        if (result.rows.length === 0) {
            return undefined
        }
        
        const row = result.rows[0]
        return new User(row.id, row.name, row.email)
    }
}
```

### Observer Pattern Example

```gibber
// Observer pattern implementation
@pattern("Observable<T>")
template Observable<T> {
    interface {
        subscribe(observer: Observer<T>) -> Subscription
        unsubscribe(subscription: Subscription) -> Bool
        notify(value: T) -> Void
    }
    
    implementation {
        private observers: Map<String, Observer<T>> = new Map()
        
        subscribe(observer: Observer<T>) -> Subscription {
            const id = generateUUID()
            this.observers.set(id, observer)
            return new Subscription(id)
        }
        
        unsubscribe(subscription: Subscription) -> Bool {
            return this.observers.delete(subscription.id)
        }
        
        notify(value: T) -> Void {
            for (const observer of this.observers.values()) {
                observer.update(value)
            }
        }
    }
}

// Observer interface
interface Observer<T> {
    update(value: T) -> Void
}

// Subscription class
class Subscription {
    id: String
    
    constructor(id: String) {
        this.id = id
    }
}

// Usage example
class UserService implements Observer<User> {
    @effects("logging")
    update(user: User) -> Void {
        Logger.info(`User updated: ${user.name}`)
    }
}

@instantiate(Observable<User>)
class UserManager {
    // Inherits Observable implementation
    
    @effects("database", "notification")
    updateUser(user: User) -> Void {
        // Update user in database
        database.saveUser(user)
        
        // Notify observers
        this.notify(user)
    }
}

// Client code
const userManager = new UserManager()
const userService = new UserService()

// Subscribe to updates
const subscription = userManager.subscribe(userService)

// Update a user
userManager.updateUser(new User("1", "Alice", "alice@example.com"))

// Unsubscribe when done
userManager.unsubscribe(subscription)
```

### State Machine Pattern Example

```gibber
// Define possible states and events
enum OrderState {
    CREATED,
    PAID,
    SHIPPED,
    DELIVERED,
    CANCELLED
}

enum OrderEvent {
    PAY,
    SHIP,
    DELIVER,
    CANCEL,
    REFUND
}

// State machine pattern implementation
@instantiate(StateMachine<OrderState, OrderEvent>)
class OrderStateMachine {
    constructor() {
        // Define state transitions
        this.addTransition(OrderState.CREATED, OrderEvent.PAY, OrderState.PAID)
        this.addTransition(OrderState.PAID, OrderEvent.SHIP, OrderState.SHIPPED)
        this.addTransition(OrderState.SHIPPED, OrderEvent.DELIVER, OrderState.DELIVERED)
        this.addTransition(OrderState.CREATED, OrderEvent.CANCEL, OrderState.CANCELLED)
        this.addTransition(OrderState.PAID, OrderEvent.CANCEL, OrderState.CANCELLED)
        this.addTransition(OrderState.PAID, OrderEvent.REFUND, OrderState.CANCELLED)
        
        // Define callbacks
        this.onTransition(OrderState.CREATED, OrderState.PAID, (order) => {
            this.sendConfirmationEmail(order)
        })
        
        this.onTransition(OrderState.PAID, OrderState.SHIPPED, (order) => {
            this.sendShippingNotification(order)
        })
    }
    
    @effects("email")
    private sendConfirmationEmail(order: Order) -> Void {
        // Send email logic
    }
    
    @effects("notification")
    private sendShippingNotification(order: Order) -> Void {
        // Send notification logic
    }
}

// Order class using the state machine
class Order {
    id: String
    customer: Customer
    items: Array<OrderItem>
    state: OrderState
    stateMachine: OrderStateMachine
    
    constructor(id: String, customer: Customer, items: Array<OrderItem>) {
        this.id = id
        this.customer = customer
        this.items = items
        this.state = OrderState.CREATED
        this.stateMachine = new OrderStateMachine()
    }
    
    @effects("state")
    processEvent(event: OrderEvent) -> Result<OrderState, StateError> {
        return this.stateMachine.transition(this.state, event).map(newState => {
            this.state = newState
            return newState
        })
    }
}

// Client code
const order = new Order("ORD-123", customer, items)

// Process events
order.processEvent(OrderEvent.PAY)   // State changes to PAID
order.processEvent(OrderEvent.SHIP)  // State changes to SHIPPED
order.processEvent(OrderEvent.REFUND) // Error: Invalid transition
```

### Validation Pattern Example

```gibber
// Define validation rules for User
@instantiate(Validator<User>)
class UserValidator {
    @pure
    validateName(user: User) -> ValidationResult {
        if (!user.name || user.name.trim().length === 0) {
            return error("Name is required")
        }
        
        if (user.name.length > 100) {
            return error("Name must be at most 100 characters")
        }
        
        return success()
    }
    
    @pure
    validateEmail(user: User) -> ValidationResult {
        if (!user.email || user.email.trim().length === 0) {
            return error("Email is required")
        }
        
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailRegex.test(user.email)) {
            return error("Email is invalid")
        }
        
        return success()
    }
    
    @pure
    @override
    validate(user: User) -> ValidationResult {
        // The validate method is required by the Validator pattern
        return this.validateName(user)
            .and(this.validateEmail(user))
    }
}

// Using the validator
class UserService {
    private validator: UserValidator
    private repository: UserRepository
    
    constructor() {
        this.validator = new UserValidator()
        this.repository = new UserRepository()
    }
    
    @effects("database")
    createUser(user: User) -> Result<User, ValidationError | DatabaseError> {
        // Validate user
        const validationResult = this.validator.validate(user)
        
        if (validationResult.hasErrors()) {
            return failure(new ValidationError(validationResult.errors))
        }
        
        // Save user
        return this.repository.save(user)
    }
}
```
