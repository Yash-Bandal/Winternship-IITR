# Getting Started with Express



## Installation of Prerequisites

### Step 1: Install Node.js

1. Go to `nodejs.org`
2. Download the **LTS version**
3. Run the installer
4. Verify installation:

```bash
node -v
npm -v
```

<br>

## Folder Structure
```
greenfield-center/
│
├── app.js
└── package.json
└── package-lock.json
```

### Step 2: Initialize the Project

```bash
mkdir greenfield-center
cd greenfield-center
npm init -y
```

This creates `package.json`, which:

* Tracks project metadata
* Manages dependencies

<br>

### Step 3: Install Express

```bash
npm install express
```

What happens internally:

* Express is downloaded into `node_modules`
* `package.json` is updated with the dependency

<br>

## 1. Creating Your First Express Server

### Step 4: Create `app.js`

```js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});

app.listen(port, () => {
  console.log(`Community Center server running at http://localhost:${port}`);
});
```

<br>

## 2. Step-by-Step Code Walkthrough 

### A. Creating the Express App

* `require('express')` loads Express
* `express()` creates the app instance
* `port = 3000` defines where the server listens

<br>

### B. Defining Routes

```js
app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});
```

Explanation:

* `app.get()` defines a GET route
* `req` represents the incoming request
* `res` is used to send a response

This ensures every visitor receives a clear welcome message.

<br>

### C. Adding an Events Route

```js
app.get('/events', (req, res) => {
  const events = [
    'Yoga Class - Monday 7pm',
    'Gardening Workshop - Wednesday 5pm',
    'Book Club - Friday 6pm'
  ];
  res.json(events);
});
```

* `/events` returns structured data
* `res.json()` formats and sends JSON automatically

This allows neighbors to check events anytime.

<br>

### D. Starting the Server

```js
app.listen(port, () => {
  console.log(`Community Center server running at http://localhost:${port}`);
});
```

* Node.js starts the HTTP server
* Express handles incoming requests
* The server is now live and listening

<br>

### E. Running and Testing

Start the server:

```bash
node app.js
```

Test URLs:

* `http://localhost:3000/`
* `http://localhost:3000/events`

Troubleshooting:

* Port in use → change port number
* "Cannot GET" error → route not defined

<br>

##  How Express Handles Requests (Flow)

```
Browser
  ↓
HTTP Request
  ↓
Node.js Server
  ↓
Express App
  ↓
Route Match
  ↓
Handler Function
  ↓
Response
```


<br>

---

<br>

#  Challenge

### Task

Add a new route `/contact` that returns:

```json
{
  "email": "contact@greenfieldcenter.org",
  "phone": "+1-555-123-4567"
}
```

Test at:

```
http://localhost:3000/contact
```

```js
const express = require('express');
const app = express();
const port = 1234;


app.get('/temp' , (req,res) =>{
    const numb = [
        '1 - one',
        '2 - two'
    ];
    console.log("temp")
    res.json(numb);
});

app.get('/events', (req, res) => {
    const events = [
        'Yoga Class - Monday 7pm',
        'Gardening Workshop - Wednesday 5pm',
        'Book Club - Friday 6pm'
    ];
    res.json(events);
});

app.get('/contact', (req, res) => {
    res.json({
        email: "contact@greenfieldcenter.org",
        phone: "+1-555-123-4567"
    });
});


app.get('/', (req, res) => {
  res.send('Welcome to Greenfield Community Center!');
});

app.listen(port, () => {
    console.log(`Community Center server running at http://localhost:${port}`);
});
```

<img width="511" height="275" alt="image" src="https://github.com/user-attachments/assets/2afc7e9f-aea3-4bd6-b2a1-03ad3037997e" />
