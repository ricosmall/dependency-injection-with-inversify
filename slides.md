---
theme: seriph
background: https://picsum.photos/seed/technology/1920/1080
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  ## Clean Architecture
  Presentation on Clean Architecture principles and implementation.
drawings:
  persist: false
css: unocss
---

# Dependency Injection with Inversify.js
A TypeScript-powered DI Container

---

# What is Dependency Injection?

- Design pattern that implements Inversion of Control (IoC)
- Decouples object creation from object usage
- Makes testing and maintenance easier
- Promotes loose coupling between components

---

# DI and Dependency Inversion Principle

- Dependency Inversion Principle (DIP) is the 'D' in SOLID
- States that:
  1. High-level modules should not depend on low-level modules
  2. Both should depend on abstractions
- Dependency Injection is a technique to achieve DIP
- Example:
  ```ts
  // Without DIP
  class UserService {
    private database = new MySQLDatabase(); // Tightly coupled
  }

  // With DIP and DI
  class UserService {
    constructor(private database: IDatabase) {} // Loosely coupled
  }
  ```

---

# Why Inversify.js?

- Lightweight IoC container for TypeScript
- Decorators for easy configuration
- Type-safe dependency injection
- Works great with TypeScript's metadata reflection

---

# Basic Concepts

Define interfaces

```ts
interface IWeapon {
  hit(): string;
}

interface IWarrior {
  fight(): string;
}
```

---

# Setting Up Inversify

`injectable` & `inject`

```ts
// Step 1: Import reflect-metadata
import 'reflect-metadata';
import { injectable, inject, Container } from 'inversify';

// Step 2: Use decorators
@injectable()
class Katana implements IWeapon {
  hit() {
    return 'cut!';
  }
}

@injectable()
class Samurai implements IWarrior {
  constructor(
    @inject('IWeapon') private weapon: IWeapon
  ) {}

  fight() {
    return this.weapon.hit();
  }
}
```

---

# Container Configuration

Bind and use

```ts
// Create and configure container
const container = new Container();

// Bind interfaces to implementations
container.bind<IWeapon>('IWeapon').to(Katana);
container.bind<IWarrior>('IWarrior').to(Samurai);

// Resolve dependencies
const warrior = container.get<IWarrior>('IWarrior');
console.log(warrior.fight()); // "cut!"
```

---

# Binding Types - `.to()`

1. Bind to Class (`.to`)

<div class="max-h-[400px] overflow-y-auto">

```ts
interface ILogger {
  log(message: string): void;
}

@injectable()
class ConsoleLogger implements ILogger {
  log(message: string) {
    console.log(message);
  }
}

container.bind<ILogger>('ILogger').to(ConsoleLogger);
```

</div>

---

# Binding Types - `.toConstantValue()`

2. Bind to Constant Value (`.toConstantValue`)

<div class="max-h-[400px] overflow-y-auto">

```ts
container.bind<string>('API_URL').toConstantValue('https://api.example.com');
```

</div>

---

# Binding Types - `.toDynamicValue()`

3. Bind to Dynamic Value (`.toDynamicValue`)

<div class="max-h-[400px] overflow-y-auto">

```ts
container.bind<Date>('CurrentDate').toDynamicValue(() => {
  return new Date();
});
```

</div>

---

# Binding Types - `.toFactory()`

4. Bind to Factory (`.toFactory`)

<div class="max-h-[400px] overflow-y-auto">

```ts
interface IWeaponFactory {
  createWeapon(type: string): IWeapon;
}

container.bind<IWeaponFactory>('IWeaponFactory').toFactory((context) => {
  return {
    createWeapon: (type: string) => {
      if (type === 'katana') {
        return context.container.get<IWeapon>('Katana');
      } else {
        return context.container.get<IWeapon>('Bow');
      }
    }
  };
});
```

</div>

---

# Scopes

InversifyJS provides three scope types:

- Singleton Scope - The same instance is shared across all requests
- Transient Scope - A new instance is created for each request
- Request Scope - The same instance is shared within a request context

---

# Scopes - Singleton Scope

1. Singleton Scope

```ts
// Same instance for all requests
@injectable()
class DatabaseConnection {
  private id = Math.random();
  getId() { return this.id; }
}

container.bind<DatabaseConnection>('DB')
  .to(DatabaseConnection)
  .inSingletonScope();

// Both will have the same id
const db1 = container.get<DatabaseConnection>('DB');
const db2 = container.get<DatabaseConnection>('DB');
console.log(db1.getId() === db2.getId()); // true
```

---

# Scopes - Transient Scope

2. Transient Scope

```ts
// New instance per request
@injectable()
class RequestHandler {
  private requestId = Math.random();
  getId() { return this.requestId; }
}

container.bind<RequestHandler>('Handler')
  .to(RequestHandler)
  .inTransientScope();

// Each will have different ids
const handler1 = container.get<RequestHandler>('Handler');
const handler2 = container.get<RequestHandler>('Handler');
console.log(handler1.getId() === handler2.getId()); // false
```

---

# Scopes - Request Scope

3. Request Scope

<div class="max-h-[400px] overflow-y-auto">

```ts
// Same instance within a request context
interface IRequestContext {
  id: string;
  timestamp: Date;
}

@injectable()
class RequestLogger {
  constructor(@inject('Context') private context: IRequestContext) {}
  log(message: string) {
    console.log(`[${this.context.id}] ${message}`);
  }
}

container.bind<RequestLogger>('Logger')
  .to(RequestLogger)
  .inRequestScope();

// Usage in request middleware
app.use((req, res, next) => {
  container.bind<IRequestContext>('Context')
    .toConstantValue({ id: req.id, timestamp: new Date() });
  next();
});
```

</div>

---

# Advanced Features

1. Tagged Bindings - Use tags to differentiate between similar dependencies
2. Named Bindings - Use named bindings for multiple implementations
3. Contextual Bindings - Bind based on the injection context
4. Circular Dependencies - Handle circular dependencies
5. Middleware - Add middleware for cross-cutting concerns

---

# Advanced Features - Tagged Bindings

```ts
interface IWeapon {
  hit(): string;
}

@injectable()
class Katana implements IWeapon {
  hit() { return 'cut!'; }
}

@injectable()
class Shuriken implements IWeapon {
  hit() { return 'throw!'; }
}

// Bind with tags
container.bind<IWeapon>('IWeapon')
  .to(Katana)
  .whenTargetTagged('type', 'melee');

container.bind<IWeapon>('IWeapon')
  .to(Shuriken)
  .whenTargetTagged('type', 'ranged');

// Usage with tags
@injectable()
class Ninja {
  constructor(
    @inject('IWeapon') @tagged('type', 'melee') private meleeWeapon: IWeapon,
    @inject('IWeapon') @tagged('type', 'ranged') private rangedWeapon: IWeapon
  ) {}
}
```

---

# Advanced Features - Named Bindings

```ts
// Bind with names
container.bind<IWeapon>('IWeapon')
  .to(Katana)
  .whenTargetNamed('primary');

container.bind<IWeapon>('IWeapon')
  .to(Shuriken)
  .whenTargetNamed('secondary');

// Usage with named bindings
@injectable()
class Ninja {
  constructor(
    @inject('IWeapon') @named('primary') private primary: IWeapon,
    @inject('IWeapon') @named('secondary') private secondary: IWeapon
  ) {}
}
```

---

# Advanced Features - Contextual Bindings

```ts
interface IWeapon {
  hit(): string;
}

@injectable()
class Katana implements IWeapon {
  hit() { return 'cut!'; }
}

@injectable()
class TrainingKatana implements IWeapon {
  hit() { return 'practice cut!'; }
}

// Bind based on context
container.bind<IWeapon>('IWeapon').to(Katana).when((request) => {
  return request.parentRequest?.serviceIdentifier === 'Warrior';
});

container.bind<IWeapon>('IWeapon').to(TrainingKatana).when((request) => {
  return request.parentRequest?.serviceIdentifier === 'Student';
});
```

---

# Advanced Features - Circular Dependencies

```ts
interface IA {
  b: IB;
  doA(): string;
}

interface IB {
  a: IA;
  doB(): string;
}

@injectable()
class A implements IA {
  constructor(@inject('IB') public b: IB) {}
  doA() { return 'A' + this.b.doB(); }
}

@injectable()
class B implements IB {
  constructor(@inject('IA') public a: IA) {}
  doB() { return 'B' + this.a.doA(); }
}

// Bind with lazy evaluation to handle circular deps
container.bind<IA>('IA').to(A).inSingletonScope();
container.bind<IB>('IB').to(B).inSingletonScope();
```

---

# Advanced Features - Middleware

```ts
// Create middleware
const measurePerformance = (planAndResolve: (next: () => any) => any) => {
  return (next: () => any) => {
    const start = Date.now();
    const result = planAndResolve(next);
    const end = Date.now();
    console.log(`Resolution took ${end - start}ms`);
    return result;
  };
};

// Apply middleware
container.applyMiddleware(measurePerformance);

// Every resolution will now be measured
const warrior = container.get<IWarrior>('IWarrior');
```

---

# Best Practices

1. Use meaningful identifiers
2. Keep container configuration centralized
3. Avoid service locator pattern
4. Use interfaces for better abstraction
5. Leverage TypeScript decorators

---

# Best Practices

1. Use Meaningful Identifiers

```ts
// Bad ❌
container.bind<ILogger>('x').to(ConsoleLogger);

// Good ✅
container.bind<ILogger>('ILogger').to(ConsoleLogger);
```

---

# Best Practices

2. Centralize Container Configuration

```ts
// container.ts
export const container = new Container();

// Organize bindings by module
export class DatabaseModule {
  static configure(container: Container): void {
    container.bind<IDatabase>('IDatabase').to(PostgresDatabase);
    container.bind<IUserRepository>('IUserRepository').to(UserRepository);
  }
}

// Configure all modules in one place
DatabaseModule.configure(container);
AuthModule.configure(container);
```

---

# Best Practices


3. Avoid Service Locator Pattern
```ts
// Bad ❌
class UserService {
  doSomething() {
    const db = container.get<IDatabase>('IDatabase'); // Direct container usage
  }
}

// Good ✅
@injectable()
class UserService {
  constructor(
    @inject('IDatabase') private db: IDatabase
  ) {}
}
```

---

# Best Practices


4. Use Interfaces for Better Abstraction
```ts
// Define clear interfaces
interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// Multiple implementations
@injectable()
class SmtpEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // SMTP implementation
  }
}

@injectable()
class MockEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Test implementation
  }
}
```

--- 

# Best Practices


5. Leverage TypeScript Decorators
```ts
// Use custom decorators for common patterns
function LogMethod() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    descriptor.value = async function (...args: any[]) {
      console.log(`Calling ${propertyKey} with:`, args);
      const result = await original.apply(this, args);
      console.log(`${propertyKey} returned:`, result);
      return result;
    };
  };
}

@injectable()
class UserService {
  @LogMethod()
  async createUser(userData: UserData): Promise<User> {
    // Implementation
  }
}
```

---

# Testing with Inversify

```ts
// Easy mocking
const mockWeapon: IWeapon = {
  hit: () => 'mock hit!'
};

container.rebind<IWeapon>('IWeapon')
  .toConstantValue(mockWeapon);

// Test your components
const warrior = container.get<IWarrior>('IWarrior');
expect(warrior.fight()).toBe('mock hit!');
```

---

# Real-world Example

```ts
// User service example
interface IUserRepository {
  findById(id: string): Promise<User>;
}

interface IAuthService {
  validateUser(user: User): Promise<boolean>;
}

@injectable()
class UserService {
  constructor(
    @inject('IUserRepository') private repo: IUserRepository,
    @inject('IAuthService') private auth: IAuthService
  ) {}

  async login(id: string): Promise<boolean> {
    const user = await this.repo.findById(id);
    return this.auth.validateUser(user);
  }
}
```


---

# Real-world Example: E-commerce System

<div class="max-h-[400px] overflow-y-auto">

```ts
// Domain Interfaces
interface IProduct {
  id: string;
  name: string;
  price: number;
}

interface IOrder {
  id: string;
  userId: string;
  products: IProduct[];
  total: number;
}

// Repository Interfaces
interface IProductRepository {
  findById(id: string): Promise<IProduct>;
  findByCategory(category: string): Promise<IProduct[]>;
  save(product: IProduct): Promise<void>;
}

interface IOrderRepository {
  create(order: Omit<IOrder, 'id'>): Promise<IOrder>;
  findByUserId(userId: string): Promise<IOrder[]>;
}

// Service Interfaces
interface IPaymentService {
  processPayment(amount: number, userId: string): Promise<boolean>;
}

interface INotificationService {
  notifyUser(userId: string, message: string): Promise<void>;
}

// Implementation Example
@injectable()
class OrderService {
  constructor(
    @inject('IOrderRepository') private orderRepo: IOrderRepository,
    @inject('IProductRepository') private productRepo: IProductRepository,
    @inject('IPaymentService') private paymentService: IPaymentService,
    @inject('INotificationService') private notificationService: INotificationService
  ) {}

  @LogMethod()
  async createOrder(userId: string, productIds: string[]): Promise<IOrder> {
    // Fetch products
    const products = await Promise.all(
      productIds.map(id => this.productRepo.findById(id))
    );

    // Calculate total
    const total = products.reduce((sum, product) => sum + product.price, 0);

    // Process payment
    const paymentSuccess = await this.paymentService.processPayment(total, userId);
    if (!paymentSuccess) {
      throw new Error('Payment failed');
    }

    // Create order
    const order = await this.orderRepo.create({
      userId,
      products,
      total
    });

    // Notify user
    await this.notificationService.notifyUser(
      userId,
      `Order ${order.id} created successfully!`
    );

    return order;
  }
}

// Container Configuration
const container = new Container();

// Repositories
container.bind<IProductRepository>('IProductRepository')
  .to(PostgresProductRepository)
  .inSingletonScope();

container.bind<IOrderRepository>('IOrderRepository')
  .to(PostgresOrderRepository)
  .inSingletonScope();

// Services
container.bind<IPaymentService>('IPaymentService')
  .to(StripePaymentService)
  .inSingletonScope();

container.bind<INotificationService>('INotificationService')
  .to(EmailNotificationService)
  .inSingletonScope();

container.bind<OrderService>('OrderService')
  .to(OrderService)
  .inSingletonScope();

// Usage
const orderService = container.get<OrderService>('OrderService');
const order = await orderService.createOrder('user123', ['prod1', 'prod2']);
```

This real-world example demonstrates:
- Clear interface definitions
- Proper dependency injection
- Service composition
- Repository pattern
- Error handling
- Async operations
- Logging decorator usage
- Container configuration

</div>

---

# Thank You!

Questions?

Resources:
- [Inversify Documentation](https://inversify.io/)
- [GitHub Repository](https://github.com/inversify/InversifyJS)
- [TypeScript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)