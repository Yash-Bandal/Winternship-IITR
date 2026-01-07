# HTTP Methods & Status Codes

## Status Codes
- [Reference](https://github.com/Yash-Bandal/Postman-API-Essentials/blob/main/Notes.md#5-run-and-check-response)

| Situation                                           | Status                        |
| --------------------------------------------------- | ----------------------------- |
| Request OK, data returned                           | **200 OK**                    |
| New resource created                                | **201 Created**               |
| No content in response                              | **204 No Content**            |
| Client mistake / bad input                          | **400 Bad Request**           |
| Resource not found                                  | **404 Not Found**             |
| Conflict with existing resource (duplicate id etc.) | **409 Conflict**              |
| Server error                                        | **500 Internal Server Error** |


<br>



## 1. Project Goal

Build a RESTful API for **products** that supports full CRUD operations and can be tested using **Postman**.


<br>

## 2. Folder Structure

```
food-store-api/
│
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts
│   └── products.router.ts
└── node_modules/
```


<br>

## 3. Project Initialization

### 3.1 Initialize Node project

```bash
npm init -y
```

### 3.2 Install dependencies

```bash
npm install express
npm install -D typescript @types/express ts-node-dev
```


<br>

## 4. Configuration Files

### 4.1 `package.json` scripts

```json
{
  "scripts": {
    "dev": "ts-node-dev src/index.ts"
  }
}
```

### 4.2 `tsconfig.json`

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


<br>

## 5. Server Setup (`src/index.ts`)

```ts
import express from "express";
import productsRouter from "./products.router";

const app = express();

app.use(express.json());

app.use("/products", productsRouter);

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Purpose:

* Initializes Express server
* Enables JSON body parsing
* Mounts products router


<br>

## 6. Products Router (`src/products.router.ts`)

### 6.1 Data Model

```ts
interface Product {
  id: string;
  name: string;
  price: number;
  inStock: boolean;
}

let products: Product[] = [
  { id: "1", name: "Bananas", price: 1.5, inStock: true },
  { id: "2", name: "Apples", price: 2.0, inStock: false }
];
```


<br>

## 7. CRUD Routes Implementation

### 7.1 GET – List all products

* **Route:** `GET /products`
* **Status:** 200

```ts
router.get("/", (req, res) => {
  res.status(200).json(products);
});
```


<br>

### 7.2 GET – Get product by ID

* **Route:** `GET /products/:id`
* **Status:** 200 / 404

```ts
router.get("/:id", (req, res) => {
  const product = products.find(p => p.id === req.params.id);

  if (!product) {
    return res.status(404).json({ error: "Product not found" });
  }

  res.status(200).json(product);
});
```


<br>

### 7.3 POST – Add new product

* **Route:** `POST /products`
* **Status:** 201 / 400

```ts
router.post("/", (req, res) => {
  const { name, price, inStock } = req.body;

  if (!name || typeof price !== "number" || typeof inStock !== "boolean") {
    return res.status(400).json({ error: "Invalid input" });
  }

  const newProduct = {
    id: String(products.length + 1),
    name,
    price,
    inStock
  };

  products.push(newProduct);
  res.status(201).json(newProduct);
});
```


<br>

### 7.4 PUT – Replace entire product

* **Route:** `PUT /products/:id`
* **Status:** 200 / 400 / 404

```ts
router.put("/:id", (req, res) => {
  const index = products.findIndex(p => p.id === req.params.id);

  if (index === -1) {
    return res.status(404).json({ error: "Product not found" });
  }

  const { name, price, inStock } = req.body;

  if (!name || typeof price !== "number" || typeof inStock !== "boolean") {
    return res.status(400).json({ error: "Invalid input" });
  }

  products[index] = { id: req.params.id, name, price, inStock };
  res.status(200).json(products[index]);
});
```


<br>

### 7.5 PATCH – Update price only

* **Route:** `PATCH /products/:id/price`
* **Status:** 200 / 400 / 404

```ts
router.patch("/:id/price", (req, res) => {
  const product = products.find(p => p.id === req.params.id);

  if (!product) {
    return res.status(404).json({ error: "Product not found" });
  }

  const { price } = req.body;

  if (typeof price !== "number" || price < 0) {
    return res.status(400).json({ error: "Invalid price" });
  }

  product.price = price;
  res.status(200).json(product);
});
```


<br>

### 7.6 PATCH – Update inStock only

* **Route:** `PATCH /products/:id/inStock`
* **Status:** 200 / 400 / 404

```ts
router.patch("/:id/inStock", (req, res) => {
  const product = products.find(p => p.id === req.params.id);

  if (!product) {
    return res.status(404).json({ error: "Product not found" });
  }

  const { inStock } = req.body;

  if (typeof inStock !== "boolean") {
    return res.status(400).json({ error: "inStock must be boolean" });
  }

  product.inStock = inStock;
  res.status(200).json(product);
});
```


<br>

### 7.7 DELETE – Remove product

* **Route:** `DELETE /products/:id`
* **Status:** 204 / 404

```ts
router.delete("/:id", (req, res) => {
  const index = products.findIndex(p => p.id === req.params.id);

  if (index === -1) {
    return res.status(404).json({ error: "Product not found" });
  }

  products.splice(index, 1);
  res.sendStatus(204);
});
```


---

### `array.splice()`
**Syntax:**
```
array.splice(startIndex, deleteCount, item1, item2, ...);
```
<img src = "https://github.com/Yash-Bandal/Winternship-IITR/blob/0ee25073c28a34bea5fa5f7fa96af42533a4f168/MERN_CaseStudies/Express/Vault/1.png" height = 400>
<img src = "https://github.com/Yash-Bandal/Winternship-IITR/blob/0ee25073c28a34bea5fa5f7fa96af42533a4f168/MERN_CaseStudies/Express/Vault/2.png" height = 400>

---

<br>

### Final `products.router.ts`
```js
import {Router, Request , Response} from "express";

const router = Router();

interface Product{
    id: string;
    name: string;
    price:number;
    inStock: boolean
}

//define products object wrt interace
let products: Product[] = [
    { id: "1", name: "Bananas", price: 1.5, inStock: true },
    { id: "2", name: "Apples", price: 2.0, inStock: false }
];

//GET
// GET /products - list all products
router.get("/", (req: Request, res: Response) => {
    res.status(200).json(products);
});

// GET /products/:id - get one product
router.get("/:id", (req: Request, res: Response) => {
    //find product p such that p.id is equalt to req.params.id
    const product = products.find(p =>p.id === req.params.id);

    //only if no error found
    if(!product){
        return res.status(404).json({error: "Product not found"});
    }
    //success if no error
    res.status(200).json(product);  
})


// POST /products - add new product
router.post("/", (req: Request, res:Response) => {
    //recieve from frontend / postman
    const {name, price, inStock} = req.body;

    //validation check
    if(!name || typeof price !== "number" || typeof inStock !== "boolean" ){
        return res.status(400).json({error:  "Invalid Input"});
    }

    //new product
    const newProduct:Product = {
        id: String(products.length + 1),
        name,
        price,
        inStock
    };

    products.push(newProduct);

    res.status(201).json(newProduct);
});

// PUT /products/:id - replace product
router.put("/:id", (req: Request, res:Response) => {
    //find index
    const index = products.findIndex(pi =>pi.id === req.params.id);

    //validation check 
    if(index === -1){
        return res.status(401).json({error: "Product Not Found"});
    }

    //work on replacement
    //recieve from frontend / postman
    const { name, price, inStock } = req.body;

    //validation check
    if (!name || typeof price !== "number" || typeof inStock !== "boolean") {
        return res.status(400).json({ error: "Invalid Input" });
    }

    products[index] = {id: req.params.id, name, price, inStock};

    res.status(200).json(products[index]);
});

//patch - PATCH update price only
router.patch("/:id/price" , (req: Request, res:Response) => {
    //product
    const product = products.find(p => p.id === req.params.id);
    
    //validate
    if(!product){
        return res.status(404).json({error: "Product not found"});
    }

    //actual logic
    //1 fetch
    // const price = req.body.price; //dot notation
    const {price} = req.body;


    //2 validate
    if(typeof price !== "number" || price < 0){
        return res.status(400).json({error: "Invalid Price"});
    }

    product.price = price;
    res.status(200).json(product);
});

//===============================================
//PATCH update inStock only (challenge)
router.patch("/:id/inStock", (req: Request, res: Response) => {
    const product = products.find(p => p.id === req.params.id);

    if (!product) {
        return res.status(404).json({ error: "Product not found" });
    }

    const { inStock } = req.body;

    if (typeof inStock !== "boolean") {
        return res.status(400).json({ error: "inStock must be boolean" });
    }

    product.inStock = inStock;
    res.status(200).json(product);
});

//============================================

//array.splice(startIndex, deleteCount, item1, item2, ...);

//Delete
router.delete("/:id", (req: Request, res:Response)  => {
    const index = products.findIndex(p => p.id === req.params.id);

    if(index === -1){
        return res.status(404).json({error: "Product Not found"});
    }

    //splice can replace as well as delete
    products.splice(index, 1);  //index, deteleCount - from that index , replaceitem(extra)
    res.sendStatus(204); //item not found
});

export default router;


```

| Action       | Method | Route                 | Status          |
| ------------ | ------ | --------------------- | --------------- |
| List         | GET    | /products             | 200             |
| One          | GET    | /products/:id         | 200 / 404       |
| Add          | POST   | /products             | 201 / 400       |
| Replace      | PUT    | /products/:id         | 200 / 400 / 404 |
| Update price | PATCH  | /products/:id/price   | 200 / 400 / 404 |
| Update stock | PATCH  | /products/:id/inStock | 200 / 400 / 404 |
| Delete       | DELETE | /products/:id         | 204 / 404       |




<br>

## 8. Running the Server

```bash
npm run dev
```

Server URL:

```
http://localhost:3000
```

<br>

## 9. Postman Testing (CRUD)

### 9.1 GET all products

* Method: GET
* URL: `/products`
* Expected: `200 OK`

### 9.2 GET product by ID

* Method: GET
* URL: `/products/1`
* Expected: `200 OK` or `404 Not Found`

### 9.3 POST add product

* Method: POST
* URL: `/products`
* Body:

```json
{
  "name": "Milk",
  "price": 3.5,
  "inStock": true
}
```

* Expected: `201 Created`

### 9.4 PUT replace product

* Method: PUT
* URL: `/products/1`
* Body:

```json
{
  "name": "Green Bananas",
  "price": 1.8,
  "inStock": false
}
```

* Expected: `200 OK`

### 9.5 PATCH update price

* Method: PATCH
* URL: `/products/1/price`
* Body:

```json
{
  "price": 2.2
}
```

* Expected: `200 OK`

### 9.6 PATCH update inStock

* Method: PATCH
* URL: `/products/1/inStock`
* Body:

```json
{
  "inStock": false
}
```

* Expected: `200 OK`

### 9.7 DELETE product

* Method: DELETE
* URL: `/products/1`
* Expected: `204 No Content`

---
