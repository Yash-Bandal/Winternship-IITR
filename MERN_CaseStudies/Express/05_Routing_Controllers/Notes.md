# Routing Controllers – Bakery Case Study

## 1. Project Goal

Implement a clean Express backend using **routing-controllers** with:

* Modular routing (Controllers)
* Request validation (DTOs)
* Middleware (logging, non-blocking checks)
* Clear Postman testing flow


**Your Turn!**
- Create a BakingController for /baking routes.
- Add a POST /baking/start endpoint to start baking an order.
- Add a GET /baking/status/:id endpoint to check the baking status of an order.

<br>

## 2. Project Structure

```
src/
│
├── app.ts                 # App bootstrap (routing-controllers config)
├── server.ts              # Server start
│
├── controllers/
│   ├── OrderController.ts
│   └── BakingController.ts
│
├── models/
│   └── Order.ts           # DTO + validation rules
│
├── data/
│   └── orders.ts          # In-memory data store
│
├── middleware/
│   ├── LoggingMiddleware.ts
│   └── AllergyMiddleware.ts (non-blocking / optional)
│
└── validators/
    └── NoPeanut.ts        # Custom validator (recommended approach)
```

<br>

## 3. App Bootstrap

### `app.ts`
* Uses `useExpressServer`
* Enables validation and body parsing internally
* No manual `express.json()`

```ts
import "reflect-metadata";
import express from "express";
import { useExpressServer } from "routing-controllers";
import { OrderController } from "./controllers/OrderController";
import { BakingController } from "./controllers/BakingController";

export const app = express();

useExpressServer(app, {
    controllers: [OrderController, BakingController],
    validation: true,
    defaultErrorHandler: true,
    
});

```

### `server.ts`

```ts
import { app } from "./app";

app.listen(3000, () => {
    console.log("Server running on http://localhost:3000");
});

```

<br>

## 4. Data Layer (Fake DB)

### `orders.ts`

```ts
export interface StoredOrder {
    id: string;
    status: "pending" | "baking" | "ready";
    customerName: string;
    flavor: string;
    quantity: number;
    pickupDate: string;
}

export const orders: StoredOrder[] = [
    //sample
    // {
    //     "customerName": "Maria",
    //     "flavor": "vanila",
    //     "quantity": 1,
    //     "pickupDate": "2026-02-01"
    // }

];


```

* In-memory storage
* Data resets on server restart

<br>

## 5. DTO & Validation

### `Order.ts`

```ts
import { IsInt, IsString, IsDateString, Min, Max } from "class-validator";

export class Order {
    @IsString()
    customerName!: string;

    @IsString()
    flavor!: string;

    @IsInt()
    @Min(1)
    @Max(100)
    quantity!: number;

    @IsDateString()
    pickupDate!: string;
}

```

* Validation happens **before controller logic**
* Custom validator is the correct place for allergy rules

<br>

## 6. Controllers

### 6.1 OrderController

```ts
import {
    JsonController,
    Get,
    Post,
    Param,
    Body,
    UseBefore,
    UseAfter
} from "routing-controllers";
import { Order } from "../models/Order";
import { orders } from "../data/orders";
import { AllergyMiddleware } from "../middleware/AllergyMiddleware";
import { LoggingMiddleware } from "../middleware/LoggingMiddleware";
import { StoredOrder } from "../data/orders";

@JsonController("/orders")
@UseBefore(AllergyMiddleware)

@UseBefore(LoggingMiddleware)
export class OrderController {

    @Get("/")
    getAll() {
        return orders;
    }

    @Get("/:id")
    getOne(@Param("id") id: string) {
        const order = orders.find(o => o.id === id);
        if (!order) throw new Error("Order not found");
        return order;
    }

    @Post("/")
    
    create(@Body() order: Order) {
        
        //Allergy check 
        // if (order.flavor.toLowerCase().includes("peanut")) {
        //     throw new Error("Peanut allergy detected");
        // }

        const newOrder:StoredOrder = {
            id: String(orders.length + 1),
            status: "pending",
            ...order
        };

        orders.push(newOrder);
        return newOrder;
    }
}

```

Responsibilities:

* Accept new orders
* Validate input
* Assign ID and initial status

<br>

### 6.2 BakingController

```ts
import { JsonController, Post, Get, Param } from "routing-controllers";
import { orders } from "../data/orders";

@JsonController("/baking")
export class BakingController {

    @Post("/start/:id")
    startBaking(@Param("id") id: string) {
        const order = orders.find(o => o.id === id);
        if (!order) throw new Error("Order not found");

        order.status = "baking";
        return { message: "Baking started", order };
    }

    @Get("/status/:id")
    getStatus(@Param("id") id: string) {
        const order = orders.find(o => o.id === id);
        if (!order) throw new Error("Order not found");

        return { status: order.status };
    }
}

```

Responsibilities:

* Change order status
* Read-only status checks

<br>

## 7. Middleware (Correct Usage)
### AllergyMiddleware.ts

```ts
import { ExpressMiddlewareInterface } from "routing-controllers";
import { Request, Response, NextFunction } from "express";
import { error } from "node:console";

export class AllergyMiddleware implements ExpressMiddlewareInterface {
    use(req: Request, res: Response, next: NextFunction) {
        console.log("ALLERGY MIDDLEWARE BODY:", req.body);

        const flavor = req.body?.flavor;
        if (typeof flavor === "string" && flavor.toLowerCase().includes("peanut")) {
            throw new Error("Peanut allergy detected");
            // console.warn(" Peanut order detected");
           
        }

        next();
    }

}

```
* Should **NOT block requests**
* Only for logging / monitoring
* Body-based blocking belongs in validation or controller

  
### LoggingMiddleware.ts

```ts
import { ExpressMiddlewareInterface } from "routing-controllers";
import { Request, Response, NextFunction } from "express";

export class LoggingMiddleware implements ExpressMiddlewareInterface {
    use(req: Request, res: Response, next: NextFunction) {
        console.log(`${req.method} ${req.url}`);
        next();
    }
}

```





Used for:

* Request logging
* Cross-cutting concerns




## Run
```
 npx ts-node-dev src/server.ts
```


<br>

## 8. Execution Flow

```
Request
 ↓
Body Parsing (routing-controllers)
 ↓
DTO Validation (class-validator)
 ↓
Controller Logic
 ↓
Response
```

<br>

## 9. Postman Testing Flow

### 9.1 Create Order

POST `/orders`

```json
{
  "customerName": "Maria",
  "flavor": "vanilla",
  "quantity": 2,
  "pickupDate": "2026-02-01"
}
```

Expected:

* status: pending
* id generated


POST `/orders`

```json
{
  "customerName": "Kid",
  "flavor": "Peanut Butter",
  "quantity": 2,
  "pickupDate": "2026-02-01"
}
```

Expected:

* status: error

**Try**
`/GET /orders`

```
http://localhost:3000/orders
```


<br>

### 9.2 Start Baking

POST `/baking/start/1`
```
http://localhost:3000/baking/start/1
```

Expected:

* status changes to baking
```
{
    "message": "Baking started",
    "order": {
        "id": "1",
        "status": "baking",
        "customerName": "Maria",
        "flavor": "Vanila",
        "quantity": 2,
        "pickupDate": "2026-02-01"
    }
}
```

<br>

### 9.3 Check Baking Status
  
GET `/baking/status/1`

```json
{ "status": "baking" }
```

<br>

### 9.4 Error Cases

* Invalid quantity → validation error
* Peanut flavor → validation error
* Invalid ID → "Order not found"

<br>
