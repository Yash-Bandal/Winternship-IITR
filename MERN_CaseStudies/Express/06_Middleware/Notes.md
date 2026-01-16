# 6. Middleware

## 1. Objective

Design and implement a **multi-step hospital discharge workflow** using **Express middleware** to understand:
* What middleware is
* How middleware chaining works
* Why order matters
* How errors stop the pipeline
* How to test middleware-driven flows using Postman

<br>

## 2. Problem Mapping (Real World → Backend)

Hospital Discharge Steps → Express Middleware

| Hospital Step        | Backend Concept    |
| -------------------- | ------------------ |
| Doctor sign-off      | Middleware check   |
| Pharmacy review      | Middleware check   |
| Insurance approval   | Middleware check   |
| Follow-up scheduling | Middleware check   |
| Tracking paperwork   | Logging middleware |

Each middleware acts as a **checkpoint**. If any checkpoint fails, discharge stops.

<br>

## 3. Project Structure

```
citycare-hospital/
│
├── src/
│   ├── app.ts
│   │
│   ├── routes/
│   │   └── discharge.ts
│   │
│   ├── middleware/
│   │   ├── logRequest.ts
│   │   ├── doctorCheck.ts
│   │   ├── pharmacyCheck.ts
│   │   ├── insuranceCheck.ts
│   │   ├── followupCheck.ts
│   │   └── errorHandler.ts
│   │
│   └── types/
│       └── express.d.ts
│
├── tsconfig.json
├── package.json
└── README.md
```

<br>

## 4. Initialization & Setup

### 4.1 Install Dependencies

```bash
npm install express cors
npm install -D typescript ts-node-dev @types/node @types/express
```

<br>

### 4.2 package.json (scripts)

```json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js"
  }
}
```

<br>

### 4.3 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

<br>

## 5. Type Extension (Important for TypeScript)

### src/types/express.d.ts

```ts
import "express-serve-static-core";

declare module "express-serve-static-core" {
    interface Request {
        dischargeLog?: {
            step: string;
            time: string;
        }[];
    }
}

```

Used to safely attach `req.dischargeLog` inside middleware.

<br>

## 6. Middleware Implementation

### 6.1 Logging Middleware `logRequest.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function logDischargeRequest(
    req: Request,
    res: Response,
    next: NextFunction
) {
    req.dischargeLog = req.dischargeLog || [];
    req.dischargeLog.push({
        step: "requestReceived",
        time: new Date().toISOString()
    });
    next();
}

```

Purpose:

* Tracks every discharge request
* Makes bottlenecks visible

<br>

### 6.2 Doctor Sign-off Check `doctorCheck.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function doctorSignoffCheck(
    req: Request,
    res: Response,
    next: NextFunction
) {
    if (!req.body.doctorSigned) {
        return res.status(400).json({
            error: "Doctor sign-off required before discharge"
        });
    }

    req.dischargeLog!.push({
        step: "doctorSignoff",
        time: new Date().toISOString()
    });

    next();
}
```

<br>

### 6.3 Pharmacy Review `pharmacyCheck.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function pharmacyReview(
    req: Request,
    res: Response,
    next: NextFunction
) {
    if (!req.body.pharmacyChecked) {
        return res.status(400).json({
            error: "Pharmacy review required before discharge"
        });
    }

    req.dischargeLog!.push({
        step: "pharmacyReview",
        time: new Date().toISOString()
    });

    next();
}

```

<br>

### 6.4 Insurance Approval `insuranceCheck.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function insuranceCheck(
    req: Request,
    res: Response,
    next: NextFunction
) {
    if (!req.body.insuranceApproved) {
        return res.status(403).json({
            error: "Insurance approval required before discharge"
        });
    }

    req.dischargeLog!.push({
        step: "insuranceApproved",
        time: new Date().toISOString()
    });

    next();
}

```

<br>

### 6.5 Follow-up Scheduling `followupCheck.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function followupCheck(
    req: Request,
    res: Response,
    next: NextFunction
) {
    if (!req.body.followupScheduled) {
        return res.status(400).json({
            error: "Follow-up appointment must be scheduled"
        });
    }

    req.dischargeLog!.push({
        step: "followupCheck",
        time: new Date().toISOString()
    });

    next();
}

```

<br>

### 6.6 Error Handling Middleware (LAST) `errorHandler.ts`

```ts
import { Request, Response, NextFunction } from "express";

export default function errorHandler(
    err: Error,
    req: Request,
    res: Response,
    next: NextFunction
) {
    console.error(" Discharge failed");
    console.error("Discharge log:", req.dischargeLog);

    res.status(500).json({
        error: err.message || "Internal server error",
        log: req.dischargeLog
    });
}

```

<br>

## 7. Route Definition (Middleware Chain)

### src/routes/discharge.ts

```ts
import express from "express";

import logDischargeRequest from "../middleware/logRequest";
import doctorSignoffCheck from "../middleware/doctorCheck";
import pharmacyReview from "../middleware/pharmacyCheck";
import insuranceCheck from "../middleware/insuranceCheck";
import followupCheck from "../middleware/followupCheck";

const router = express.Router();

router.post(
    "/",
    logDischargeRequest,
    doctorSignoffCheck,
    pharmacyReview,
    insuranceCheck,
    followupCheck,
    (req, res) => {
        req.dischargeLog!.push({
            step: "dischargeComplete",
            time: new Date().toISOString()
        });

        res.json({
            status: "Discharge complete",
            patient: req.body.patientName,
            log: req.dischargeLog
        });
    }
);

export default router;

```

Order = business logic

<br>

## 8. App Initialization

### src/app.ts

```ts
import express from "express";
import cors from "cors";

import dischargeRouter from "./routes/discharge";
import errorHandler from "./middleware/errorHandler";

const app = express();

// Built-in middleware
app.use(express.json());

// Third-party middleware
app.use(cors());

// Routes
app.use("/discharge", dischargeRouter);

// Error middleware (ALWAYS LAST)
app.use(errorHandler);

app.listen(3000, () => {
    console.log("🏥 CityCare Hospital system running on port 3000");
});

```

<br>

## 9. Execution Flow

```
Request
 ↓
Logging middleware
 ↓
Doctor sign-off check
 ↓
Pharmacy review
 ↓
Insurance approval
 ↓
Follow-up check
 ↓
Final handler
```

If any step fails → response is sent immediately.

<br>

## 10. Postman Testing

### 10.1 Success Case

POST `/discharge`

```json
{
  "patientName": "John Doe",
  "doctorSigned": true,
  "pharmacyChecked": true,
  "insuranceApproved": true,
  "followupScheduled": true
}
```

Expected:

* Status: Discharge complete
* Full discharge log returned


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/45ec2044-e9fb-4ce1-a348-0023e495aac3" />


<br>

### 10.2 Missing Doctor Sign-off

```json
{
  "patientName": "Jane",
  "pharmacyChecked": true,
  "insuranceApproved": true,
  "followupScheduled": true
}
```

Response:

```json
{ "error": "Doctor sign-off required before discharge" }
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3363d013-18d4-4395-805b-53cda5c6c601" />

<br>

### 10.3 Missing Insurance Approval

```json
{
  "patientName": "Alex",
  "doctorSigned": true,
  "pharmacyChecked": true,
  "followupScheduled": true
}
```

Response:

```json
{ "error": "Insurance approval required before discharge" }
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a120d525-dbde-4a18-b280-7c003b28d1c2" />

<br>

## 11. Key Learnings

* Middleware is a pipeline
* Each middleware has a single responsibility
* Order of middleware = workflow logic
* Errors stop execution immediately
* Error middleware must be last
* Middleware is ideal for approvals, checks, logging, auth

<br>

## 12. Real-world Usage

This same pattern is used for:

* Authentication & authorization
* Payment workflows
* Approval systems
* File processing pipelines
* Request auditing & tracing

<br>

## 13. Final Takeaway

Middleware turns Express into a **step-by-step workflow engine**, making complex processes visible, debuggable, and reliable.
