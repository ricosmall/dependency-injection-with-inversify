---
theme: seriph
background: https://picsum.photos/seed/sum/1920/1080
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

```ts
import { injectable, inject, Container } from 'inversify';

// Define interfaces
interface IWeapon {
  hit(): string;
}

interface IWarrior {
  fight(): string;
}
```

---

# Setting Up Inversify

```ts
// Step 1: Import reflect-metadata
import 'reflect-metadata';

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

- Singleton Scope
- Transient Scope
- Request Scope

---

# Scopes - Singleton Scope

1. Singleton Scope (Default)

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

- Tagged bindings
- Named bindings
- Contextual bindings
- Circular dependencies
- Middleware

```ts
// Tagged binding example
container.bind<IWeapon>('IWeapon')
  .to(Katana)
  .whenTargetTagged('faction', 'samurai');
```

---

# Best Practices

1. Use meaningful identifiers
2. Keep container configuration centralized
3. Avoid service locator pattern
4. Use interfaces for better abstraction
5. Leverage TypeScript decorators

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

# Thank You!

Questions?

Resources:
- [Inversify Documentation](https://inversify.io/)
- [GitHub Repository](https://github.com/inversify/InversifyJS)
- [TypeScript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)