# 7. Request Validation 

## Case Study: BrightFuture University – Admissions API


<br>


## 1. Problem Statement

BrightFuture University receives thousands of online applications. Many are incomplete or invalid:

* Missing grades or essays
* Invalid birthdates like "202"
* Wrong file uploads instead of documents
* Incomplete contact information

**Goal:** Ensure every application is complete, accurate, and validated *before* processing.


<br>



## 2. Why Request Validation Is Important

* **Garbage In, Garbage Out** – bad data breaks systems
* **Better UX** – users get clear feedback on what to fix
* **Security** – blocks malformed or malicious input
* **Efficiency** – errors are caught early, not deep in logic


<br>



## 3. Project Structure

```
brightfuture-admissions/
│
├── src/
│   ├── app.ts
│   ├── server.ts
│   │
│   ├── routes/
│   │   └── application.routes.ts
│   │
│   ├── validators/
│   │   └── application.validator.ts
│   │
│   └── types/
│       └── application.type.ts
│
├── package.json
├── tsconfig.json
└── nodemon.json
```


<br>



## 4. Initialization & Setup

### 4.1 Initialize Project

```bash
npm init -y
```

### 4.2 Install Dependencies

```bash
npm install express express-validator
npm install -D typescript ts-node-dev @types/express
```


<br>



## 5. TypeScript Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### nodemon.json

```json
{
  "watch": ["src"],
  "ext": "ts",
  "exec": "ts-node-dev src/server.ts"
}
```

### package.json (scripts)

```json
"scripts": {
  "dev": "ts-node-dev --respawn --transpile-only src/server.ts"
}
```


<br>



## 6. Application Data Model

### src/types/application.type.ts

```ts
export interface Application {
    name: string;
    email: string;
    birthdate: string;
    grades: number[];
    essay: string;
    recommendationLetter: string;
    portfolioLink?: string;
}

```


<br>



## 7. Validation Rules (express-validator)

### src/validators/application.validator.ts

```ts
import { body } from "express-validator";

export const applicationValidation = [
    body("name")
        .isString()
        .notEmpty()
        .withMessage("Name is required"),

    body("email")
        .isEmail()
        .withMessage("Valid email is required"),

    body("birthdate")
        .isISO8601()
        .withMessage("Birthdate must be a valid date (YYYY-MM-DD)"),

    body("grades")
        .isArray({ min: 1 })
        .withMessage("At least one grade is required"),

    body("grades.*")
        .isNumeric()
        .withMessage("All grades must be numbers"),

    body("essay")
        .isLength({ min: 100 })
        .withMessage("Essay must be at least 100 characters"),

    body("recommendationLetter")
        .isURL()
        .withMessage("A valid recommendation letter link is required"),

    // Mini-challenge requirement
    body("portfolioLink")
        .optional()
        .isURL()
        .withMessage("A valid portfolio link is required for art applicants"),
];

```


<br>



## 8. Route Implementation

### src/routes/application.routes.ts

```ts
import { Router, Request, Response } from "express";
import { validationResult } from "express-validator";
import { applicationValidation } from "../validators/application.validator"

const router = Router();

router.post("/apply", applicationValidation, (req: Request, res: Response) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
        return res.status(400).json({
            errors: errors.array(),
        });
    }

    // Valid application
    res.status(200).json({
        status: "Application received!",
    });
});

export default router;

```


<br>



## 9. Express App Setup

### src/app.ts

```ts
import express from "express";
import applicationRoutes from "../routes/application.routes"

const app = express();

app.use(express.json());

app.use("/api", applicationRoutes);

export default app;

```

### src/server.ts

```ts
import app from "./app";

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```


<br>



## 10. Running the Server

```bash
npm run dev
```

Server runs on:

```
http://localhost:3000
```

<br>


## 11. Postman Testing

### 11.1 Valid Request (Success)

**POST** `/api/apply`

```json
{
  "name": "Rahul Sharma",
  "email": "rahul@gmail.com",
  "birthdate": "2002-06-15",
  "grades": [85, 90, 78],
  "essay": "This essay is intentionally written longer than one hundred characters to pass validation and demonstrate a valid application submission.",
  "recommendationLetter": "https://university.com/rec-letter.pdf",
  "portfolioLink": "https://portfolio.com/rahul"
}
```

**Response**

```json
{
  "status": "Application received!"
}
```



<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0452961d-0648-4a80-a4a4-8f61c9638acc" />

<br>

### 11.2 Invalid Request (Validation Errors)

```json
{
  "name": "",
  "email": "wrong-email",
  "birthdate": "202",
  "grades": [],
  "essay": "Too short",
  "recommendationLetter": "not-a-url"
}
```

**Response (400)**

```json
{
  "errors": [
    { "msg": "Name is required", "param": "name" },
    { "msg": "Valid email is required", "param": "email" },
    { "msg": "Birthdate must be a valid date (YYYY-MM-DD)", "param": "birthdate" },
    { "msg": "At least one grade is required", "param": "grades" },
    { "msg": "Essay must be at least 100 characters", "param": "essay" },
    { "msg": "A valid recommendation letter link is required", "param": "recommendationLetter" }
  ]
}
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9c0393c4-8e14-4912-af61-601617ac883b" />

<br>

## 12. Validation Flow

```
Client Request
   ↓
Validation Middleware
   ↓
Errors? → YES → 400 + error list
   ↓ NO
Controller Logic
   ↓
Success Response
```

<br>

## 13. Best Practices

* Always validate **before** business logic
* Validate **every required field**
* Return **clear, actionable errors**
* Handle arrays using `field.*`
* Keep validation logic separate from routes

<br>

## 14. Outcome

* No incomplete applications reach the system
* Clear feedback for users
* Clean, scalable Express architecture
* Industry-standard validation pattern
