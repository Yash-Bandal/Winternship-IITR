# 10. Dependency Injection & Inversion of Control (IoC)

## Healthcare Appointment System – From Scratch

<br>

## 1. What We Are Building

A simple **Healthcare Appointment Booking API** where:

* Appointment logic does **not create** SMS / Email / Billing services
* All dependencies are **injected** using a DI container (TypeDI)
* Services can be swapped (SMS → Email, Real → Mock) **without changing business logic**

<br>

## 2. Core Concepts (In Short)

### Dependency Injection (DI)

* A class receives its dependencies from outside
* No `new SMSService()` inside business logic

### Inversion of Control (IoC)

* Object creation is handled by a **container** (TypeDI)
* Business classes focus only on logic, not wiring

<br>

## 3. Project Structure

```
di-healthcare-system/
│
├── src/
│   ├── main.ts
│   ├── app.ts
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
│   └── routes/
│       └── appointment.routes.ts
│
├── tsconfig.json
└── package.json
```

<br>

## 4. Setup & Installation

```bash
npm init -y
npm install express typedi reflect-metadata
npm install -D typescript ts-node-dev @types/node @types/express
```

<br>

## 5. TypeScript Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

These two lines are MANDATORY for TypeDI:
```
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
```

<br>

## 6. Enable Reflect Metadata (MANDATORY)

```ts
// src/main.ts
import "reflect-metadata";
import app from "./app";

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

<br>

## 7. Define Interfaces (Contracts)

### NotificationService

```ts
// src/notifications/NotificationService.ts
export interface NotificationService {
  send(to: string, message: string): Promise<void>;
}
```

### BillingService

```ts
// src/billing/BillingService.ts
export interface BillingService {
  charge(patient: string, amount: number): Promise<void>;
}
```

<br>

## 8. Implement Services

### SMS Notification Service

```ts
// src/notifications/SMSService.ts
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

### Email Notification Service

```ts
// src/notifications/EmailService.ts
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

### Billing Service (Stripe Example)

```ts
// src/billing/StripeBillingService.ts
import { Service } from "typedi";
import { BillingService } from "./BillingService";

@Service()
export class StripeBillingService implements BillingService {
  async charge(patient: string, amount: number) {
    console.log(`Charged ₹${amount} to ${patient} via Stripe`);
  }
}
```

<br>

## 9. Appointment Service (Business Logic)

```ts
// src/appointments/AppointmentService.ts
import { Service, Inject } from "typedi";
import { NotificationService } from "../notifications/NotificationService";
import { EmailService } from "../notifications/EmailService";
import { BillingService } from "../billing/BillingService";
import { SMSService } from "../notifications/SMSService";
import { StripeBillingService } from "../billing/StripeBillingService";

@Service()
export class AppointmentService {
    constructor(
        
        @Inject(() => StripeBillingService)
        private billing: BillingService,
        
        //Switch between any one - as needed easily
        //SMS service
        // @Inject(() => SMSService)
        // private notifier: NotificationService,

        //Email Service
        @Inject(() => EmailService)
        private notifier: NotificationService,

    ) { }

    async bookAppointment(
        patient: string,
        time: string,
        amount: number
    ) {
        await this.billing.charge(patient, amount);
        await this.notifier.send(
            patient,
            `Your appointment is booked for ${time}`
        );

        return { status: "confirmed" };
    }
}

```

<br>

## 10. Express App Setup

```ts
// src/app.ts
import express from "express";
import { Container } from "typedi";
import { AppointmentService } from "./appointments/AppointmentService";
import { appointmentRoutes } from "./routes/appointment.routes";

const app = express();
app.use(express.json());

const appointmentService = Container.get(AppointmentService);

app.use("/appointments", appointmentRoutes(appointmentService));

export default app;
```

<br>

## 11. Routes Layer

```ts
// src/routes/appointment.routes.ts
import { Router } from "express";
import { AppointmentService } from "../appointments/AppointmentService";

export const appointmentRoutes = (appointmentService: AppointmentService) => {
  const router = Router();

  router.post("/book", async (req, res) => {
    const { patient, time, amount } = req.body;

    const result = await appointmentService.bookAppointment(
      patient,
      time,
      amount
    );

    res.json(result);
  });

  return router;
};
```

<br>

## 12. Run the Application

```json
"scripts": {
  "dev": "ts-node-dev src/main.ts"
}
```

```bash
npm run dev
```

<br>

## 13. Postman Testing

### Book Appointment

**POST** `/appointments/book`

```json
{
  "patient": "yash@example.com",
  "time": "Monday 10am",
  "amount": 500
}
```

### Response

```json
{ "status": "confirmed" }
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fea9d3a0-ab26-4ed1-a9c2-858625d31220" />

### Console Output

```
Charged ₹500 to yash@example.com via Stripe
SMS sent to yash@example.com: Your appointment is booked for Monday 10am
```

OR
```
Charged ₹500 to yash@example.com via Stripe
Email sent to yash@example.com: Your appointment is booked for Monday 10am
```

<br>

## 14.  IMPORTANT: Changing Injection (SMS → Email)

### ONLY CHANGE THIS LINE

```ts
// Before
@Inject(() => SMSService)
private notifier: NotificationService;
```

### Replace With

```ts
@Inject(() => EmailService)
private notifier: NotificationService;
```

- No change in AppointmentService logic
- No change in routes
- No change in app.ts

This proves **Dependency Injection + IoC** is working correctly.

<br>

## 15. Final Core Idea (Interview Ready)

> Dependency Injection provides dependencies from outside, while Inversion of Control hands over object creation to a container, making systems modular, testable, and easy to extend.

<br>

## 16. What We Achieved

* No hardcoded dependencies
* Swappable services
* Clean separation of concerns
* Safe testing possible
* Production-grade DI setup

<br>


