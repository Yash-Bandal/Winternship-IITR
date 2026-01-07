# Request And Response - FreshMart Case Study


<br>

## 1. Problem Overview (FreshMart Case)

FreshMart’s loyalty system failed due to:
* Missing request data
* Invalid inputs (string instead of number)
* No validation before business logic
* Partial operations (points deducted but order failed)
* Confusing and inconsistent error responses

**Goal:** Build a robust API that validates early, fails safely, and responds consistently.

<br>

## 2. Mental Model (Restaurant Analogy)

* **Request** → Waiter taking the order
* **Validation Middleware** → Quality check before cooking
* **Route Logic** → Chef preparing the dish
* **Error Handler** → Manager explaining what went wrong
   - Example - You have 300 rs, dish is 500rs -> Manager says `Insufficient balance`
* **Zod** - Security Service -> Like GUard

Golden rule:

> Routes should assume data is valid. Middleware decides whether the request is allowed to reach the route.


<br>

---

<br>

## 3. Project Setup (From Scratch..)

### 3.1 Initialize Project

```bash
mkdir freshmart-api
cd freshmart-api
npm init -y
```

### 3.2 Install Dependencies

```bash
npm install express zod
npm install -D typescript ts-node-dev @types/node @types/express
```
Setup `tsconfig.json`
```bash
npx tsc --init
```
### 3.3 TypeScript Config (`tsconfig.json`)

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
  }
}
```
### 3.4 Update `package.json` -> script entry
```
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
      "dev": "ts-node-dev --respawn --transpile-only src/app.ts"
  },
```

<br>

## 4. Folder Structure

```
src/
├── app.ts
├── schemas.ts
├── routes/
│   ├── redeem.route.ts
│   └── transfer.route.ts
├── middlewares/
│   └── validate.ts
├── errors/
│   └── ApiError.ts
├── db/
│   └── fakeDb.ts
```


<br>

## 5. Core Building Blocks

### 5.1 Custom Error Class (`errors/ApiError.ts`)

```ts
export class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}
```

Purpose:
* Carry HTTP status + message together
* Differentiate expected vs unexpected errors


<br>

### 5.2 Zod Schemas (`schemas.ts`)

```ts
import { z } from "zod";

export const RedeemSchema = z.object({
  customerId: z.string().uuid(),
  points: z.number().int().positive(),
});

export const TransferSchema = z.object({
  fromCustomerId: z.string().uuid(),
  toCustomerId: z.string().uuid(),
  points: z.number().int().positive(),
});

export type RedeemRequest = z.infer<typeof RedeemSchema>;
export type TransferRequest = z.infer<typeof TransferSchema>;
```


<br>

### 5.3 Validation Middleware (`middlewares/validate.ts`)

```ts
import { RequestHandler } from "express";
import { z } from "zod";

export function validate(schema: z.ZodTypeAny): RequestHandler {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        status: "error",
        error: result.error.issues[0].message,
      });
    }

    req.body = result.data; // safe + typed
    next();
  };
}
```

Why middleware:

* Avoids repeating validation logic
* Blocks invalid requests early


## 5.4 `/src/db/fakeDB.ts`
```js
/**
 * Fake in-memory database
 * (acts like a balances table)
 */
export const balances: Record<string, number> = {
    // Redeem customer
    "550e8400-e29b-41d4-a716-446655440000": 300,

    // Transfer customers
    "3fa85f64-5717-4562-b3fc-2c963f66afa6": 500,
    "9c1b8f20-8f25-4d1e-9c77-9d7bdf9a6a33": 200,
};

```
<br>

## 6. Application Entry (`app.ts`)

```ts
import express, { Request, Response, NextFunction } from "express";
import redeemRouter from "./routes/redeem.route";
import transferRouter from "./routes/transfer.route";
import { ApiError } from "./errors/ApiError";

const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.json({ status: "ok" });
});

app.use("/api", redeemRouter);
app.use("/api", transferRouter);

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof ApiError) {
    return res.status(err.statusCode).json({
      status: "error",
      error: err.message,
    });
  }

  console.error(err);
  res.status(500).json({
    status: "error",
    error: "Internal server error",
  });
});

app.listen(1200, () => {
  console.log("Server running on http://localhost:1200");
});
```


<br>

## 7. Redeem Endpoint (`routes/redeem.route.ts`)

```ts
import { Router, Request, Response } from "express";
import { validate } from "../middleware/validate";
import { RedeemSchema, RedeemRequest } from "../schema"
import { ApiError } from "../errors/ApiError";
import { balances } from "../db/fakeDb";

const router = Router();

/**
 * POST /api/redeem
 */
router.post(
    "/redeem",
    validate(RedeemSchema),
    (req: Request<{}, {}, RedeemRequest>, res: Response) => {
        const { customerId, points } = req.body;

        const currentBalance = balances[customerId];

        if (currentBalance === undefined) {
            throw new ApiError(404, "Customer not found");
        }

        if (points > currentBalance) {
            throw new ApiError(400, "Insufficient points");
        }

        // Deduct points
        balances[customerId] -= points;

        res.json({
            status: "success",
            data: {
                customerId,
                redeemed: points,
                remainingPoints: balances[customerId],
            },
        });
    }
);

export default router;


```


<br>

## 8. Transfer Challenge Implementation (`routes/transfer.route.ts`)

```ts
import { Router, Request, Response } from "express";
import { validate } from "../middleware/validate";
import { TransferSchema, TransferRequest } from "../schema";
import { ApiError } from "../errors/ApiError";
import { balances } from "../db/fakeDb";

const router = Router();

/**
 * POST /api/transfer
 */
router.post(
    "/transfer",
    validate(TransferSchema),
    (req: Request<{}, {}, TransferRequest>, res: Response) => {
        const { fromCustomerId, toCustomerId, points } = req.body;

        if (balances[fromCustomerId] === undefined) {
            throw new ApiError(404, "Sender not found");
        }

        if (balances[toCustomerId] === undefined) {
            throw new ApiError(404, "Recipient not found");
        }

        if (balances[fromCustomerId] < points) {
            throw new ApiError(400, "Insufficient points for transfer");
        }

        // Perform transfer (atomic in-memory)
        balances[fromCustomerId] -= points;
        balances[toCustomerId] += points;

        res.json({
            status: "success",
            data: {
                fromCustomerId,
                toCustomerId,
                transferred: points,
                senderRemainingPoints: balances[fromCustomerId],
            },
        });
    }
);

export default router;

```


<br>

## 9. Testing with Postman

### 9.1 Redeem

**POST** `/api/redeem`

```json
{
"customerId": "550e8400-e29b-41d4-a716-446655440000",
"points": 200
}
```

### 9.2 Transfer

**POST** `/api/transfer`

```json
{
  "fromCustomerId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "toCustomerId": "9c1b8f20-8f25-4d1e-9c77-9d7bdf9a6a33",
  "points": 200
}
```

<br>

## If POSTMAN Not installed
### Redeem 
```
Invoke-RestMethod -Method POST -Uri http://localhost:11000/api/redeem -ContentType application/json -Body '{"customerId": "550e8400-e29b-41d4-a716-446655440000","points": 10}'
```

### Transfer
```
 Invoke-RestMethod -Method POST -Uri http://localhost:11000/api/transfer -ContentType application/json -Body '{"fromCustomerId":"3fa85f64-5717-4562-b3fc-2c963f66afa6","toCustod0b-4903-b2f6-0281a340899f-9c77-9d7bdf9a6a33","points":50}'
```

# Done.......
