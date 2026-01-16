# 10. Dependency Injection & Inversion of Control 
## Healthcare Appointment System

<br>

## 1. Problem Statement

Sunrise Family Clinic started small but grew rapidly:

* Scheduling, notifications, and billing logic became tightly coupled
* Adding new features (Email/SMS reminders) caused breaking changes
* Switching providers required rewriting business logic
* Testing triggered real SMS and billing calls

**Goal:**
Design the system so scheduling, notifications, and billing can be developed, tested, and swapped independently.

<br>

## 2. Core Concepts

### 2.1 Dependency Injection (DI)

Dependency Injection means:

* A class does **not create** its own dependencies
* Dependencies are **provided from outside**

```ts
// BAD
this.notifier = new SMSService();

// GOOD
constructor(notifier: NotificationService) {}
```

<br>

### 2.2 Inversion of Control (IoC)

* Control of creating dependencies is moved **outside the class**
* An **IoC Container** (TypeDI) handles creation and wiring

Classes focus only on **what they do**, not **how dependencies are created**.

<br>

## 3. Why DI Is Needed

Without DI:

* Hardcoded services
* Difficult testing
* Provider switching breaks code

With DI:

* Loose coupling
* Easy mocking
* Safe provider swapping
* Clean, scalable architecture

<br>

## 4. Project Structure

```
healthcare-di-system/
│
├── src/
│   ├── main.ts
│
│   ├── appointments/
│   │   └── AppointmentService.ts
│
│   ├── notifications/
│   │   ├── NotificationService.ts
│   │   ├── SMSService.ts
│   │   └── EmailService.ts
│
│   ├── billing/
│   │   ├── BillingService.ts
│   │   └── StripeBillingService.ts
│
│   └── tests/
│       └── AppointmentService.test.ts
│
├── tsconfig.json
└── package.json
```

<br>

## 5. Setup & Configuration

### Initialize Project

```bash
npm init -y
```

###  Install Dependencies

```bash
npm install typedi reflect-metadata
```



### Install Dependencies

```bash
npm install express
npm install -D typescript ts-node-dev @types/express
```

### package.json (scripts)

```json
"scripts": {
  "dev": "ts-node-dev --respawn --transpile-only src/server.ts"
}
```


<br>



## TypeScript Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

<br>

### 5.2 Enable Decorators (tsconfig.json)

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

<br>

### 5.3 Enable Reflect Metadata

At the **top-level entry file**:

```ts
import "reflect-metadata";
```

<br>

## 6. Defining Interfaces (Contracts)

### 6.1 NotificationService

```ts
// notifications/NotificationService.ts
export interface NotificationService {
  send(to: string, message: string): Promise<void>;
}
```

### 6.2 BillingService

```ts
// billing/BillingService.ts
export interface BillingService {
  charge(patient: string, amount: number): Promise<void>;
}
```

Interfaces ensure business logic depends on **abstractions, not implementations**.

<br>

## 7. Implementing Injectable Services

### 7.1 SMS Notification

```ts
// notifications/SMSService.ts
import { Service } from "typedi";
import { NotificationService } from "./NotificationService";

@Service()
export class SMSService implements NotificationService {
  async send(to: string, message: string) {
    console.log(`SMS sent to ${to}: ${message}`);
  }
}
```

<br>

### 7.2 Email Notification

```ts
// notifications/EmailService.ts
import { Service } from "typedi";
import { NotificationService } from "./NotificationService";

@Service()
export class EmailService implements NotificationService {
  async send(to: string, message: string) {
    console.log(`Email sent to ${to}: ${message}`);
  }
}
```

<br>

### 7.3 Stripe Billing

```ts
// billing/StripeBillingService.ts
import { Service } from "typedi";
import { BillingService } from "./BillingService";

@Service()
export class StripeBillingService implements BillingService {
  async charge(patient: string, amount: number) {
    console.log(`Charged $${amount} to ${patient} via Stripe`);
  }
}
```

<br>

## 8. Injecting Dependencies (Constructor Injection)

```ts
// appointments/AppointmentService.ts
import { Service, Inject } from "typedi";
import { NotificationService } from "../notifications/NotificationService";
import { BillingService } from "../billing/BillingService";
import { SMSService } from "../notifications/SMSService";
import { StripeBillingService } from "../billing/StripeBillingService";

@Service()
export class AppointmentService {
  constructor(
    @Inject(() => SMSService) private notifier: NotificationService,
    @Inject(() => StripeBillingService) private billing: BillingService
  ) {}

  async bookAppointment(patient: string, time: string, amount: number) {
    await this.billing.charge(patient, amount);
    await this.notifier.send(patient, `Your appointment is booked for ${time}`);
    return { status: "confirmed" };
  }
}
```

Appointment logic does not know **how** notification or billing works.

<br>

## 9. Swapping Implementations (IoC Container)

```ts
// main.ts
import "reflect-metadata";
import { Container } from "typedi";
import { AppointmentService } from "./appointments/AppointmentService";
import { EmailService } from "./notifications/EmailService";
import { NotificationService } from "./notifications/NotificationService";

Container.set(NotificationService, new EmailService());

const service = Container.get(AppointmentService);
service.bookAppointment("alice@example.com", "Monday 10am", 50);
```

Only the container changes — business logic stays untouched.

<br>

## 10. Testing with Mock Dependencies

```ts
class MockNotifier implements NotificationService {
  messages: string[] = [];
  async send(to: string, message: string) {
    this.messages.push(`${to}: ${message}`);
  }
}

class MockBilling implements BillingService {
  charges: string[] = [];
  async charge(patient: string, amount: number) {
    this.charges.push(`${patient}: $${amount}`);
  }
}
```

```ts
Container.set(NotificationService, new MockNotifier());
Container.set(BillingService, new MockBilling());

const service = Container.get(AppointmentService);
await service.bookAppointment("bob@example.com", "Tuesday 2pm", 75);

Container.reset();
```

No real SMS or billing occurs during tests.

<br>

## 11. Output of This Design

* Modular appointment system
* Notification providers are swappable
* Billing providers are swappable
* Safe, isolated unit testing
* Clean separation of responsibilities

<br>

## 12. Core Idea (Interview Ready)

> Dependency Injection removes hardcoded dependencies by supplying them externally, while Inversion of Control hands over object creation to a container, making systems modular, testable, and easy to extend.

<br>

## 13. Best Practices

| Pitfall                       | Best Practice              |
| <br><br><br><br><br><br><br><br><br>-- | <br><br><br><br><br><br><br><br>-- |
| Hardcoding services           | Always inject dependencies |
| Depending on concrete classes | Depend on interfaces       |
| Forgetting container cleanup  | Reset container in tests   |
| Too many dependencies         | Keep services focused      |

<br>

## 14. Final Summary

DI + IoC allows:

* Writing business logic once
* Swapping implementations freely
* Testing safely without side effects

This is essential for scalable, production-grade systems.
