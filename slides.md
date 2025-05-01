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

# 使用 Inversify.js 进行依赖注入
基于 TypeScript 的 DI 容器

---

# 什么是依赖注入？

- 实现控制反转 (IoC) 的设计模式
- 将对象创建与对象使用解耦
- 使测试和维护更加容易
- 促进组件之间的松耦合

---

# DI 和依赖倒置原则

- 依赖倒置原则 (DIP) 是 SOLID 中的 'D'
- 它指出：
  1. 高层模块不应该依赖于低层模块
  2. 两者都应该依赖于抽象
- 依赖注入是实现 DIP 的一种技术
- 示例：
  ```ts
  // 不使用 DIP
  class UserService {
    private database = new MySQLDatabase(); // 紧耦合
  }

  // 使用 DIP 和 DI
  class UserService {
    constructor(private database: IDatabase) {} // 松耦合
  }
  ```

---

# 为什么选择 Inversify.js？

- 轻量级 TypeScript IoC 容器
- 使用装饰器实现简单配置
- 类型安全的依赖注入
- 与 TypeScript 的元数据反射配合良好

---

# 基本概念

定义接口

```ts
interface IWeapon {
  hit(): string;
}

interface IWarrior {
  fight(): string;
}
```

---

# 设置 Inversify

`injectable` 和 `inject`

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

# 容器配置

绑定和使用

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

# 绑定类型 - `.to()`

1. 绑定到类 (`.to`)

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

# 绑定类型 - `.toConstantValue()`

2. 绑定到常量值 (`.toConstantValue`)

<div class="max-h-[400px] overflow-y-auto">

```ts
container.bind<string>('API_URL').toConstantValue('https://api.example.com');
```

</div>

---

# 绑定类型 - `.toDynamicValue()`

3. 绑定到动态值 (`.toDynamicValue`)

<div class="max-h-[400px] overflow-y-auto">

```ts
container.bind<Date>('CurrentDate').toDynamicValue(() => {
  return new Date();
});
```

</div>

---

# 绑定类型 - `.toFactory()`

4. 绑定到工厂 (`.toFactory`)

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

# 作用域

InversifyJS 提供三种作用域类型：

- 单例作用域 - 所有请求共享同一个实例
- 瞬态作用域 - 每个请求创建一个新实例
- 请求作用域 - 在请求上下文中共享同一个实例

---

# 作用域 - 单例作用域

1. 单例作用域

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

# 作用域 - 瞬态作用域

2. 瞬态作用域

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

# 作用域 - 请求作用域

3. 请求作用域

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

container.bind<RequestLogger>('Logger').to(RequestLogger).inRequestScope();

// Usage in request middleware
app.use((req, res, next) => {
  container.bind<IRequestContext>('Context').toConstantValue({ id: req.id, timestamp: new Date() });
  next();
});
```

</div>

---

# 高级特性

1. 标签绑定 - 使用标签区分相似依赖
2. 命名绑定 - 为多个实现使用命名绑定
3. 上下文绑定 - 基于注入上下文进行绑定
4. 循环依赖 - 处理循环依赖
5. 中间件 - 添加中间件处理横切关注点

---

# 高级特性 - 标签绑定

<div class="max-h-[400px] overflow-y-auto">

```ts
interface IWeapon { hit(): string; }

@injectable()
class Katana implements IWeapon { hit() { return 'cut!'; } }

@injectable()
class Shuriken implements IWeapon { hit() { return 'throw!'; } }

// Bind with tags
container.bind<IWeapon>('IWeapon').to(Katana).whenTargetTagged('type', 'melee');
container.bind<IWeapon>('IWeapon').to(Shuriken).whenTargetTagged('type', 'ranged');

// Usage with tags
@injectable()
class Ninja {
  constructor(
    @inject('IWeapon') @tagged('type', 'melee') private meleeWeapon: IWeapon,
    @inject('IWeapon') @tagged('type', 'ranged') private rangedWeapon: IWeapon
  ) {}
}
```

</div>

---

# 高级特性 - 命名绑定

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

# 高级特性 - 上下文绑定

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

# 高级特性 - 循环依赖

```ts
interface IA { b: IB; doA(): string; }
interface IB { a: IA; doB(): string; }

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

# 高级特性 - 中间件

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

# 最佳实践

1. 使用有意义的标识符
2. 保持容器配置集中
3. 避免服务定位器模式
4. 使用接口实现更好的抽象
5. 利用 TypeScript 装饰器

---

# 最佳实践

1. 使用有意义的标识符

```ts
// Bad ❌
container.bind<ILogger>('x').to(ConsoleLogger);

// Good ✅
container.bind<ILogger>('ILogger').to(ConsoleLogger);
```

---

# 最佳实践

2. 中心化容器配置

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

# 最佳实践


3. 避免服务定位器模式

```ts
// Bad ❌
class UserService {
  doSomething() {
    const db = container.get<IDatabase>('IDatabase'); // 直接使用容器
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

# 最佳实践


4. 使用接口实现更好的抽象

```ts
// 定义清晰的接口
interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// 多种实现
@injectable()
class SmtpEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // SMTP 实现
  }
}

@injectable()
class MockEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // 测试实现
  }
}
```

--- 

# 最佳实践

5. 利用 TypeScript 装饰器

<div class="max-h-[400px] overflow-y-auto">

```ts
// 为常见模式使用自定义装饰器
function LogMethod() {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
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
    // 实现
  }
}
```

</div>

---

# 使用 Inversify 进行测试

```ts
// 简单模拟
const mockWeapon: IWeapon = {
  hit: () => 'mock hit!'
};

container.rebind<IWeapon>('IWeapon')
  .toConstantValue(mockWeapon);

// 测试你的组件
const warrior = container.get<IWarrior>('IWarrior');
expect(warrior.fight()).toBe('mock hit!');
```

---

# 真实世界示例

<div class="max-h-[400px] overflow-y-auto">

```ts
// 用户服务示例
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

</div>


---

# 真实世界示例：电子商务系统

<div class="max-h-[400px] overflow-y-auto">

```ts
// 领域接口
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

// 数据仓库接口
interface IProductRepository {
  findById(id: string): Promise<IProduct>;
  findByCategory(category: string): Promise<IProduct[]>;
  save(product: IProduct): Promise<void>;
}

interface IOrderRepository {
  create(order: Omit<IOrder, 'id'>): Promise<IOrder>;
  findByUserId(userId: string): Promise<IOrder[]>;
}

// 服务接口
interface IPaymentService {
  processPayment(amount: number, userId: string): Promise<boolean>;
}

interface INotificationService {
  notifyUser(userId: string, message: string): Promise<void>;
}

// 实现示例
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
    // 获取产品
    const products = await Promise.all(
      productIds.map(id => this.productRepo.findById(id))
    );

    // 计算总额
    const total = products.reduce((sum, product) => sum + product.price, 0);

    // 处理支付
    const paymentSuccess = await this.paymentService.processPayment(total, userId);
    if (!paymentSuccess) {
      throw new Error('Payment failed');
    }

    // 创建订单
    const order = await this.orderRepo.create({
      userId,
      products,
      total
    });

    // 通知用户
    await this.notificationService.notifyUser(
      userId,
      `Order ${order.id} created successfully!`
    );

    return order;
  }
}

// 容器配置
const container = new Container();

// 数据仓库
container.bind<IProductRepository>('IProductRepository')
  .to(PostgresProductRepository)
  .inSingletonScope();

container.bind<IOrderRepository>('IOrderRepository')
  .to(PostgresOrderRepository)
  .inSingletonScope();

// 服务
container.bind<IPaymentService>('IPaymentService')
  .to(StripePaymentService)
  .inSingletonScope();

container.bind<INotificationService>('INotificationService')
  .to(EmailNotificationService)
  .inSingletonScope();

container.bind<OrderService>('OrderService')
  .to(OrderService)
  .inSingletonScope();

// 使用
const orderService = container.get<OrderService>('OrderService');
const order = await orderService.createOrder('user123', ['prod1', 'prod2']);
```

这个真实世界示例展示了：
- 清晰的接口定义
- 适当的依赖注入
- 服务组合
- 仓储模式
- 错误处理
- 异步操作
- 日志装饰器使用
- 容器配置

</div>

---

# 谢谢！

还有问题吗？

资源：
- [Inversify 文档](https://inversify.io/)
- [GitHub 仓库](https://github.com/inversify/InversifyJS)
- [TypeScript 装饰器](https://www.typescriptlang.org/docs/handbook/decorators.html)