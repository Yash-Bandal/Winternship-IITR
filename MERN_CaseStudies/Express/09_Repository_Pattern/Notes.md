# 9. Repository Pattern 
## Course Registration System

<br>

## 1. Problem Statement

Riverdale University faces issues in course registration due to multiple storage systems (spreadsheets, databases, paper records):

* Lost enrollment requests
* Double-booked seats
* Difficult upgrades to new storage systems
* Hard-to-test business logic due to tight coupling

**Goal:**
Build a course registration system where business logic is independent of how/where data is stored.

<br>

## 2. What is the Repository Pattern?

The Repository Pattern separates:

* **Business Logic** (rules, validations)
* **Data Access Logic** (how data is stored/retrieved)

It provides a **common interface** for data access so storage can be swapped without changing logic.

<br>

## 3. Project Structure

```
repository-pattern-course/
│
├── src/
│   ├── app.ts
│   ├── server.ts
│
│   ├── models/
│   │   └── Course.ts
│
│   ├── repositories/
│   │   ├── interfaces/
│   │   │   └── ICourseRepository.ts
│   │   └── InMemoryCourseRepository.ts
│
│   ├── services/
│   │   └── CourseService.ts
│
│   └── routes/
│       └── course.routes.ts
│
├── tsconfig.json
└── package.json
```

**Layer Responsibilities**

* `routes` → HTTP handling only
* `services` → business rules
* `repositories` → data access only
* `models` → domain entities

<br>

## 4. Setup & Installation

### 4.1 Initialize Project

```bash
npm init -y
```

### 4.2 Install Dependencies

```bash
npm install express
npm install -D typescript ts-node-dev @types/node @types/express
```

<br>

## 5. TypeScript Configuration

### 5.1 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

<br>

## 6. Domain Model

### 6.1 Course Model

```ts
// src/models/Course.ts
export interface Course {
  id: string;
  name: string;
  capacity: number;
  students: string[];
}
```

<br>

## 7. Repository Interface

### 7.1 ICourseRepository

```ts
// src/repositories/interfaces/ICourseRepository.ts
import { Course } from "../../models/Course";

export interface ICourseRepository {
  findAll(): Promise<Course[]>;
  findById(id: string): Promise<Course | null>;
  save(course: Course): Promise<void>;
  delete(courseId: string): Promise<void>;
  enrollStudent(courseId: string, studentId: string): Promise<void>;
  findByStudentId(studentId: string): Promise<Course[]>;
}
```

This interface is the contract used by business logic.

<br>

## 8. In-Memory Repository Implementation

```ts
// src/repositories/InMemoryCourseRepository.ts
import { ICourseRepository } from "./interfaces/ICourseRepository";
import { Course } from "../models/Course";

export class InMemoryCourseRepository implements ICourseRepository {
  private courses: Course[] = [];

  async findAll(): Promise<Course[]> {
    return this.courses;
  }

  async findById(id: string): Promise<Course | null> {
    return this.courses.find(c => c.id === id) || null;
  }

  async save(course: Course): Promise<void> {
    const index = this.courses.findIndex(c => c.id === course.id);
    if (index >= 0) this.courses[index] = course;
    else this.courses.push(course);
  }

  async delete(courseId: string): Promise<void> {
    this.courses = this.courses.filter(c => c.id !== courseId);
  }

  async enrollStudent(courseId: string, studentId: string): Promise<void> {
    const course = await this.findById(courseId);
    if (course && !course.students.includes(studentId)) {
      course.students.push(studentId);
    }
  }

  async findByStudentId(studentId: string): Promise<Course[]> {
    return this.courses.filter(c => c.students.includes(studentId));
  }
}
```

<br>

## 9. Service Layer (Business Logic)

```ts
// src/services/CourseService.ts
import { ICourseRepository } from "../repositories/interfaces/ICourseRepository";
import { Course } from "../models/Course";

export class CourseService {
  constructor(private courseRepo: ICourseRepository) {}

  async createCourse(course: Course) {
    await this.courseRepo.save(course);
    return course;
  }

  async enroll(courseId: string, studentId: string) {
    const course = await this.courseRepo.findById(courseId);
    if (!course) throw new Error("Course not found");
    if (course.students.length >= course.capacity)
      throw new Error("Course full");

    await this.courseRepo.enrollStudent(courseId, studentId);
    return { message: "Enrolled successfully" };
  }

  async getStudentCourses(studentId: string) {
    return this.courseRepo.findByStudentId(studentId);
  }

  async deleteCourse(courseId: string) {
    await this.courseRepo.delete(courseId);
    return { message: "Course deleted" };
  }
}
```

<br>

## 10. Routes Layer

```ts
// src/routes/course.routes.ts
import { Router } from "express";
import { CourseService } from "../services/CourseService";

export const courseRoutes = (courseService: CourseService) => {
  const router = Router();

  router.post("/", async (req, res) => {
    const course = await courseService.createCourse(req.body);
    res.status(201).json(course);
  });

  router.post("/:id/enroll", async (req, res) => {
    try {
      const result = await courseService.enroll(
        req.params.id,
        req.body.studentId
      );
      res.json(result);
    } catch (err: any) {
      res.status(400).json({ error: err.message });
    }
  });

  router.get("/students/:id", async (req, res) => {
    const courses = await courseService.getStudentCourses(req.params.id);
    res.json(courses);
  });

  router.delete("/:id", async (req, res) => {
    const result = await courseService.deleteCourse(req.params.id);
    res.json(result);
  });

  return router;
};
```


  
 <br>

  

## 11. App & Server Bootstrap

### 11.1 app.ts

```ts
import express from "express";
import { InMemoryCourseRepository } from "./repositories/InMemoryCourseRepository";
import { CourseService } from "./services/CourseService";
import { courseRoutes } from "./routes/course.routes";

const app = express();
app.use(express.json());

const courseRepo = new InMemoryCourseRepository();
const courseService = new CourseService(courseRepo);

app.use("/courses", courseRoutes(courseService));

export default app;
```

### 11.2 server.ts

```ts
import app from "./app";

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```


  
 <br>

  

## 12. Run the Application

Add script in `package.json`:

```json
"scripts": {
  "dev": "ts-node-dev src/server.ts"
}
```

Run:

```bash
npm run dev
```


  
 <br>

  

## 13. Postman Testing

### 13.1 Create Course

  
**POST** `/courses`
  
`http://localhost:3000/courses`

```json
{
  "id": "CS101",
  "name": "Computer Science Basics",
  "capacity": 2,
  "students": []
}
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7684d1d7-f668-46af-bada-6e550d008571" />

  

  
 <br>

  

### 13.2 Enroll Student

**POST** `/courses/CS101/enroll`

  `http://localhost:3000/courses/CS101/enroll`
  
```json
{
  "studentId": "STU1"
}
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fe5de4ad-9ad3-4d9c-8b10-7abc3d4385fa" />

  
 <br>

  

### 13.3 Get Student Courses

**GET** `/courses/students/STU1`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5090fd52-b163-4365-87c2-83935ea29502" />

  
  
 <br>

  
### 13.4 Delete Course

**DELETE** `/courses/CS101`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/eef12ff1-c3d2-4e08-8a2c-9fe8f7cbcf38" />


  
 <br>

  
## 14. Key Takeaways

* Business logic depends on interfaces, not storage
* Storage can be swapped without code changes
* Easy to test using mock repositories
* Clean separation of concerns

This is a correct, production-grade Repository Pattern implementation.
