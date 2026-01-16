# 8. MVC Pattern 

## Scalable Library System – Maplewood Public Library

<br>

<br>

---
### We built a Library Management API using:

Model
Book interface defining data shape

**Repository**
- Handles data storage (InMemoryBookRepository)
  → No business logic here

**Service**
- Contains all rules (borrow, return, validations)
  → “Book already borrowed”, “Book not found”, etc.

**Controlle**r
- Handles HTTP routes and delegates work to service

**Dependency Injection**
- Services and controllers receive dependencies instead of creating them

---


<br>



## 1. Problem Statement

The library system had major issues:

* Business rules mixed with HTTP request handling
* Data access logic tangled with policies
* Difficult to add features like reservations or fines
* Hard to test or modify without breaking code

**Goal:** Build a clean, scalable system using **MVC + Service + Repository + Dependency Injection**.

<br>

## 2. Architecture Overview

```
Client Request
   ↓
Controller  (HTTP layer)
   ↓
Service     (business rules)
   ↓
Repository  (data access)
   ↓
Model       (data structure)
```

Each layer has a single responsibility.

<br>

## 3. Project Structure

```
case_8/
│
├── src/
│   ├── app.ts
│   ├── server.ts
│   │
│   ├── models/
│   │   └── Book.ts
│   │
│   ├── repositories/
│   │   ├── interfaces/
│   │   │   └── IBookRepository.ts
│   │   └── InMemoryBookRepository.ts
│   │
│   ├── services/
│   │   └── BookService.ts
│   │
│   ├── controllers/
│   │   └── BookController.ts
│   │
│   └── routes/
│       └── book.routes.ts
│
├── package.json
├── tsconfig.json
└── nodemon.json
```

<br>

## 4. Initialization & Scripts

### Initialize Project

```bash
npm init -y
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



## 5. TypeScript Configuration

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

### nodemon.json

```json
{
  "watch": ["src"],
  "ext": "ts",
  "exec": "ts-node-dev src/server.ts"
}
```

<br>

## 6. Model Layer

### src/models/Book.ts

```ts
export interface Book {
  id: string;
  title: string;
  author: string;
  isBorrowed: boolean;
}
```

<br>

## 7. Repository Layer

### Repository Interface

#### src/repositories/interfaces/IBookRepository.ts

```ts
import type  {Book} from "../../models/Book"

export interface IBookRepository {
    findAll(): Promise<Book[]>;
    findById(id: string): Promise<Book | null>;
    save(book: Book): Promise<void>;
    update(book: Book): Promise<void>;
}

```

### In-Memory Implementation

#### src/repositories/InMemoryBookRepository.ts

```ts
import  type { IBookRepository } from "./interfaces/IBookRepository";
import type { Book } from "../models/Book";

export class InMemoryBookRepository implements IBookRepository {
    private books: Book[] = [];

    async findAll(): Promise<Book[]> {
        return this.books;
    }

    async findById(id: string): Promise<Book | null> {
        return this.books.find(book => book.id === id) || null;
    }

    async save(book: Book): Promise<void> {
        this.books.push(book);
    }

    async update(updatedBook: Book): Promise<void> {
        const index = this.books.findIndex(b => b.id === updatedBook.id);
        if (index !== -1) {
            this.books[index] = updatedBook;
        }
    }
}

```


<br>



## 8. Service Layer (Business Logic)

### src/services/BookService.ts

```ts
import { IBookRepository } from "../repositories/interfaces/IBookRepository";
import { Book } from "../models/Book";

export class BookService {
    constructor(private bookRepository: IBookRepository) { }

    async addBook(book: Book): Promise<void> {
        await this.bookRepository.save(book);
    }

    async borrowBook(bookId: string): Promise<Book> {
        const book = await this.bookRepository.findById(bookId);

        if (!book) {
            throw new Error("Book not found");
        }

        if (book.isBorrowed) {
            throw new Error("Book already borrowed");
        }

        const updatedBook: Book = {
            ...book,
            isBorrowed: true,
        };

        await this.bookRepository.update(updatedBook);
        return updatedBook;
    }

    async returnBook(bookId: string): Promise<Book> {
        const book = await this.bookRepository.findById(bookId);

        if (!book) {
            throw new Error("Book not found");
        }

        if (!book.isBorrowed) {
            throw new Error("Book is not borrowed");
        }

        const updatedBook: Book = {
            ...book,
            isBorrowed: false,
        };

        await this.bookRepository.update(updatedBook);
        return updatedBook;
    }

    async getAllBooks(): Promise<Book[]> {
        return this.bookRepository.findAll();
    }
}

```


<br>



## 9. Controller Layer

### src/controllers/BookController.ts

```ts
import { Request, Response } from "express";
import { BookService } from "../services/BookService";

export class BookController {
  constructor(private bookService: BookService) {}

  async addBook(req: Request, res: Response): Promise<void> {
    try {
      await this.bookService.addBook(req.body);
      res.status(201).json({ message: "Book added" });
    } catch (error: any) {
      res.status(400).json({ error: error.message });
    }
  }

  async borrowBook(req: Request, res: Response): Promise<void> {
    try {
      const book = await this.bookService.borrowBook(req.params.id);
      res.json(book);
    } catch (error: any) {
      res.status(400).json({ error: error.message });
    }
  }

  async returnBook(req: Request, res: Response): Promise<void> {
    try {
      const book = await this.bookService.returnBook(req.params.id);
      res.json(book);
    } catch (error: any) {
      res.status(400).json({ error: error.message });
    }
  }

  async getAllBooks(req: Request, res: Response): Promise<void> {
    const books = await this.bookService.getAllBooks();
    res.json(books);
  }
}
```


<br>



## 10. Routes Layer

### src/routes/book.routes.ts

```ts
import { Router } from "express";
import { BookController } from "../controllers/BookController";

export const createBookRoutes = (bookController: BookController) => {
  const router = Router();

  router.post("/books", (req, res) => bookController.addBook(req, res));
  router.get("/books", (req, res) => bookController.getAllBooks(req, res));
  router.post("/books/:id/borrow", (req, res) => bookController.borrowBook(req, res));
  router.post("/books/:id/return", (req, res) => bookController.returnBook(req, res));

  return router;
};
```



<br>


## 11. App & Server Setup (Dependency Injection)

### src/app.ts

```ts
import express from "express";
import { InMemoryBookRepository } from "./repositories/InMemoryBookRepository";
import { BookService } from "./services/BookService";
import { BookController } from "./controllers/BookController";
import { createBookRoutes } from "./routes/book.routes";

const app = express();
app.use(express.json());

const bookRepository = new InMemoryBookRepository();
const bookService = new BookService(bookRepository);
const bookController = new BookController(bookService);

app.use("/api", createBookRoutes(bookController));

export default app;
```

### src/server.ts

```ts
import app from "./app";

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Library system running on port ${PORT}`);
});
```


<br>



## 12. Running the Server

```bash
npm run dev
```

Expected output:

```
Library system running on port 3000
```

<br>


## 13. Postman Testing

### Add Book

**POST** `/api/books`

```json
{
  "id": "1",
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isBorrowed": false
}
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ffab259f-ad95-432a-b594-8eff78c950bf" />


### Borrow Book

**POST** `/api/books/1/borrow`
Borrow once
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/664aa053-7e9c-4d4a-aa54-53d92ebe9f20" />

Try Borrowing again (sending request again)
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fb6930d5-ffc6-4fe4-9d4b-e4cbab3d75c5" />


### Return Book

**POST** `/api/books/1/return`
Return Once
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4aedfd57-f37d-45a7-9648-fa1b2f17a406" />

Try returning again
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bbc20b2a-acf1-4d72-95bb-759a50e2da37" />


### Get All Books

**GET** `/api/books`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/11207f3c-6422-4a36-946e-2793f008b8b3" />


<br>

## 14. Outcome

* Clear separation of concerns (MVC)
* Business rules isolated in services
* Storage logic isolated in repositories
* Easy to extend and test
* Clean, scalable Express architecture
