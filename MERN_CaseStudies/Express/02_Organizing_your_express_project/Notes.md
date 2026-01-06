# Organizing Your Express Project

## [Reference](https://github.com/Yash-Bandal/Web-Development-Essentials_CWH/blob/main/NodeJS/Express/Express.md#6-express-router)


##  Common Express Project Structure

A scalable Express project typically looks like:

```
greenfield-center/
├── app.js                # Main application entry point
├── package.json          # Project metadata and dependencies
├── routes/               # Route definitions
│   ├── events.js
│   ├── classes.js
│   └── contact.js
├── controllers/          # Business logic (optional)
│   ├── eventsController.js
│   └── classesController.js
├── models/               # Database models
│   └── event.js
├── public/               # Static assets
│   ├── css/
│   ├── js/
│   └── images/
├── views/                # Template files
│   └── index.ejs
├── middleware/           # Custom middleware
│   └── auth.js
├── .env                  # Environment variables
└── README.md             # Documentation
```

<br>

## Folder Responsibilities

### routes/

* Defines application endpoints
* Maps URLs to handlers

### controllers/

* Contains request-handling logic
* Keeps route files clean

### models/

* Defines data structures
* Used when working with databases

### public/

* Stores static files like images and stylesheets

### views/

* Contains HTML templates for server-rendered pages

### middleware/

* Handles cross-cutting concerns (auth, logging, validation)

<br>

##  Modularizing the Application

### A. Moving Routes to Separate Files

**routes/events.js**

```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json([
    'Yoga Class - Monday 7pm',
    'Gardening Workshop - Wednesday 5pm',
    'Book Club - Friday 6pm'
  ]);
});

module.exports = router;
```

<br>

### B. Using Routes in `app.js`

```js
const express = require('express');
const app = express();

const eventsRouter = require('./routes/events');

app.use(express.json());

app.use('/events', eventsRouter);

app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});

const port = 3000;
app.listen(port, () => {
  console.log(`Community Center server running at http://localhost:${port}`);
});
```

<br>

### C. Adding More Features Easily

To add a new `/classes` route:

**routes/classes.js**

```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json([
    'Art Class - Tuesday 4pm',
    'Music Class - Thursday 3pm'
  ]);
});

module.exports = router;
```

Mount it in `app.js`:

```js
const classesRouter = require('./routes/classes');
app.use('/classes', classesRouter);
```

<br>

## Serving Static Files

To serve static assets:

```js
app.use(express.static('public'));
```

Files in `public/` become accessible at:

```
http://localhost:3000/filename
```

<br>

##  Step-by-Step Walkthrough

### A. Creating the Structure

```bash
mkdir routes public
```

* Move route logic into separate files
* Keep `app.js` focused on setup only



### B. Adding and Testing Features

* Add `/classes` route
* Add a static file like `logo.png`
* Access via `http://localhost:3000/logo.png`

<br>

---

<br>

##  Challenge

### Task
Your Turn!
- Add a new route /contact in routes/contact.js that returns the center’s contact info as JSON.
- Mount it in app.js at `/contact`.
- Test by visiting http://localhost:3000/contact.
# Organizing Your Express Project

Learn about organizing an Express project for scalability and teamwork

---

## 1. Problem Statement

### Greenfield Community Center’s Growing Pains

The Greenfield Community Center website is live.

Initially, everything worked from a single file:

* Welcome message
* Events list

As the site grew, new requirements appeared:

* Class registrations
* Contact messages
* News updates

All new code was added to the same file, leading to:

* Poor readability
* Difficult debugging
* Accidental breakages when fixing bugs

### The Challenge

How do you organize an Express project so that:

* New features are easy to add
* Bugs can be fixed safely
* Multiple developers can work together
* The application remains manageable as it grows

---

## 2. Learning Objectives

By the end of this tutorial, you will be able to:

* Understand why project structure matters in Express
* Set up a modular and maintainable folder structure
* Separate routes, logic, and data
* Initialize a scalable Express application

---

## 3. Concept Introduction with Analogy

### Analogy: From a Single Room to a Modern Community Center

Originally, everything happened in one room:

* Meetings
* Classes
* Storage
* Administration

As more people joined, chaos followed.

The solution was to create **dedicated rooms**:

* One for classes
* One for events
* One for staff

A well-structured Express project works the same way:

* Each folder has a clear purpose
* New features do not clutter existing code
* Developers can work independently without conflict

---

## 4. Why Project Structure Matters

### Key Benefits

* **Readability**: Easy to locate and understand code
* **Maintainability**: Faster bug fixes and updates
* **Teamwork**: Multiple developers work in parallel
* **Scalability**: Project grows without becoming messy

---

## 5. Common Express Project Structure

A scalable Express project typically looks like:

```
greenfield-center/
├── app.js                # Main application entry point
├── package.json          # Project metadata and dependencies
├── routes/               # Route definitions
│   ├── events.js
│   ├── classes.js
│   └── contact.js
├── controllers/          # Business logic (optional)
│   ├── eventsController.js
│   └── classesController.js
├── models/               # Database models
│   └── event.js
├── public/               # Static assets
│   ├── css/
│   ├── js/
│   └── images/
├── views/                # Template files
│   └── index.ejs
├── middleware/           # Custom middleware
│   └── auth.js
├── .env                  # Environment variables
└── README.md             # Documentation
```

---

## 6. Folder Responsibilities

### routes/

* Defines application endpoints
* Maps URLs to handlers

### controllers/

* Contains request-handling logic
* Keeps route files clean

### models/

* Defines data structures
* Used when working with databases

### public/

* Stores static files like images and stylesheets

### views/

* Contains HTML templates for server-rendered pages

### middleware/

* Handles cross-cutting concerns (auth, logging, validation)

---

## 7. Modularizing the Application

### A. Moving Routes to Separate Files

**routes/events.js**

```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json([
    'Yoga Class - Monday 7pm',
    'Gardening Workshop - Wednesday 5pm',
    'Book Club - Friday 6pm'
  ]);
});

module.exports = router;
```

---

### B. Using Routes in `app.js`

```js
const express = require('express');
const app = express();

const eventsRouter = require('./routes/events');

app.use(express.json());

app.use('/events', eventsRouter);

app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});

const port = 3000;
app.listen(port, () => {
  console.log(`Community Center server running at http://localhost:${port}`);
});
```

---

### C. Adding More Features Easily

To add a new `/classes` route:

**routes/classes.js**

```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json([
    'Art Class - Tuesday 4pm',
    'Music Class - Thursday 3pm'
  ]);
});

module.exports = router;
```

Mount it in `app.js`:

```js
const classesRouter = require('./routes/classes');
app.use('/classes', classesRouter);
```

---

## 8. Serving Static Files

To serve static assets:

```js
app.use(express.static('public'));
```

Files in `public/` become accessible at:

```
http://localhost:3000/filename
```



## 10. Step-by-Step Walkthrough

### A. Creating the Structure

```bash
mkdir routes public
```

* Move route logic into separate files
* Keep `app.js` focused on setup only

<br>

### B. Adding and Testing Features

* Add `/classes` route
* Add a static file like `logo.png`
* Access via `http://localhost:3000/logo.png`



<br>

## Code

### Step 1: Create the Route File

Inside the `routes/` folder, create a new file:

```
routes/contact.js
```

<br>

### Step 2: Write the Contact Route

```js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({
    email: 'info@greenfieldcenter.org',
    phone: '555-123-4567'
  });
});

module.exports = router;
```

Explanation:

* `express.Router()` creates a modular router
* `router.get('/')` handles GET requests
* `res.json()` sends structured response data

<br>

### Step 3: Import the Route in `app.js`

At the top of `app.js`, import the contact router:

```js
const contactRouter = require('./routes/contact');
```

<br>

### Step 4: Mount the Route

Add this line in `app.js`:

```js
app.use('/contact', contactRouter);
```

This means:

* `/contact` → handled by `contact.js`
* `/contact/` → matches `router.get('/')`

<br>

### Step 5: Final `app.js` (Relevant Part)

```js
const express = require('express');
const app = express();

const eventsRouter = require('./routes/events');
const classesRouter = require('./routes/classes');
const contactRouter = require('./routes/contact');

app.use(express.json());

app.use('/events', eventsRouter);
app.use('/classes', classesRouter);
app.use('/contact', contactRouter);

app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});

const port = 3000;
app.listen(port, () => {
  console.log(`Community Center server running at http://localhost:${port}`);
});
```

<br>

### Step 6: Run and Test

Start the server:

```bash
node app.js
```

Test in browser:

```
http://localhost:3000/contact
```

You should receive the contact information in JSON format.

<br>

