# CRUD Operations

## Task:
Implement complete **CRUD (Create, Read, Update, Delete)** operations for a food delivery backend using MongoDB.\
You’re managing FastBite’s menu database. Complete these tasks using MongoDB CRUD operations:
1. Add a new vegan dish called “Tofu Buddha Bowl” (cuisine: “Asian”, price: $9.50, tags: [“vegan”, “gluten-free”], available: true).
2. Find all available vegan dishes under $12, showing only their name and price.
3. Update the price of “Tofu Buddha Bowl” to $10.00 and add a “popular” tag.
4. Delete the dish “Old Special Soup” from the menu.

<br>

## Database & Collection Setup

```js
use fastbite

db.createCollection("menu")
```

<br>

## Our Menu Structure


```js
{
  name: String,
  cuisine: String,
  price: Number,
  tags: [String],
  available: Boolean
}
```



## Setup our Inventory Menu (Veg Only )

```js
db.menu.insertMany([
  {
    name: "Margherita Pizza",
    cuisine: "Italian",
    price: 11.5,
    tags: ["vegetarian"],
    available: true
  },
  {
    name: "Vegan Salad Bowl",
    cuisine: "Continental",
    price: 9.0,
    tags: ["vegan", "gluten-free"],
    available: true
  },
  {
    name: "Paneer Butter Masala",
    cuisine: "Indian",
    price: 12.5,
    tags: ["vegetarian"],
    available: true
  },
  {
    name: "Old Special Soup",
    cuisine: "Asian",
    price: 7.5,
    tags: ["vegan"],
    available: false
  },
  {
    name: "Veg Hakka Noodles",
    cuisine: "Asian",
    price: 10.0,
    tags: ["vegetarian"],
    available: true
  }
])
```
### Verify..
```js
db.menu.find().pretty()
```
<img width="550" height="914" alt="image" src="https://github.com/user-attachments/assets/9f35328d-8405-403a-9c35-d72f0a03ee67" />

<br>

<br>

---

<br>


## Task 1 – CREATE (insertOne)

### Requirement

Add a new vegan dish:

* Name: Tofu Buddha Bowl
* Cuisine: Asian
* Price: 9.50
* Tags: vegan, gluten-free
* Available: true

### Implementation

```js
db.menu.insertOne({
  name: "Tofu Buddha Bowl",
  cuisine: "Asian",
  price: 9.5,
  tags: ["vegan", "gluten-free"],
  available: true
})
```

<img width="548" height="641" alt="image" src="https://github.com/user-attachments/assets/f2f60077-b564-4667-a082-d680e3732bee" />


<br>

---

<br>


##  Task 2 – READ (find)

### Requirement

Find all **available vegan dishes under $12**, returning only name and price.

### Implementation

```js
db.menu.find(
  {
    available: true,
    tags: "vegan",
    price: { $lt: 12 }
  },
  {
    _id: 0,
    name: 1,
    price: 1
  }
)
```
<img width="389" height="607" alt="image" src="https://github.com/user-attachments/assets/8a2282a8-ca38-4eea-b116-8df910b019e4" />

<br>

<br>

---

<br>


## Task 3 – UPDATE (updateOne)

### Requirement

Update **Tofu Buddha Bowl**:

* Change price to $10.00
* Add tag `popular`

### Implementation

```js
db.menu.updateOne(
  { name: "Tofu Buddha Bowl" },
  {
    $set: { price: 10.0 },
    $push: { tags: "popular" }
  }
)
```

<img width="340" height="427" alt="image" src="https://github.com/user-attachments/assets/e5bd4c29-b0d0-475c-8570-ea52ea29a6f0" />

<br>

<br>

---

<br>


## Task 4 – DELETE (deleteOne)

### Requirement

Delete the dish **Old Special Soup**.

### Implementation
### First Check if present in database
```js
db.menu.find({ name: "Old Special Soup" }).pretty()
```
### Delete If present
```js
db.menu.deleteOne({ name: "Old Special Soup" })
```
<img width="538" height="203" alt="image" src="https://github.com/user-attachments/assets/7d69fc04-15fc-4d69-b3f5-16a26aef3b65" />

<br>

---

<br>

