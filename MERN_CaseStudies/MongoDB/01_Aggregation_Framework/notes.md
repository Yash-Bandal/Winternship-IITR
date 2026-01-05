# Aggregation Framework



## Task

1. Find the **average rating for each genre in 2024**, but **only include genres with more than 10,000 total views**.\
**Output fields:**
   * genre
   * totalViews
   * avgRating (rounded to 1 decimal)

<br>

 ---
> [!Note]
> 1. First Install mongoDB
> 2. Set up MongoDB Compass / VSCode
> 3. Connect to mongoDB server `localhost:27017`
> 4. Start by creating a database -> collections
 ---

## 1. Database & Collection Setup

```js
use case1

db.createCollection("movieflix")
```

<br>

## 2. Insert Data of Movies

```js
db.movieflix.insertMany([
  {
    movie: "Edge of Tomorrow",
    genre: "Sci-Fi",
    country: "USA",
    views: 15000,
    rating: 8.2,
    year: 2024
  },
  {
    movie: "Interstellar",
    genre: "Sci-Fi",
    country: "USA",
    views: 18000,
    rating: 8.6,
    year: 2024
  },
  {
    movie: "Parasite",
    genre: "Drama",
    country: "South Korea",
    views: 9000,
    rating: 8.5,
    year: 2024
  },
  {
    movie: "Dune",
    genre: "Sci-Fi",
    country: "USA",
    views: 12000,
    rating: 8.3,
    year: 2024
  },
  {
    movie: "3 Idiots",
    genre: "Drama",
    country: "India",
    views: 14000,
    rating: 8.4,
    year: 2024
  },
  {
    movie: "Jawan",
    genre: "Action",
    country: "India",
    views: 22000,
    rating: 7.9,
    year: 2024
  }
])
```

<br>

### 3. Verify
```js
> db.movieflix.find().pretty()
```
<img width="484" height="851" alt="image" src="https://github.com/user-attachments/assets/d6847391-55ca-4b43-b057-7d7cb1a74f47" />

## 4. Use Aggregation Pipeline

```js
db.movieflix.aggregate([
  {
    $match: { year: 2024 }
  },
  {
    $group: {
      _id: "$genre",
      totalViews: { $sum: "$views" },
      avgRating: { $avg: "$rating" }
    }
  },
  {
    $match: { totalViews: { $gt: 10000 } }
  },
  {
    $project: {
      _id: 0,
      genre: "$_id",
      totalViews: 1,
      avgRating: { $round: ["$avgRating", 1] }
    }
  }
])
```
<img width="438" height="797" alt="image" src="https://github.com/user-attachments/assets/cc44983b-938e-4660-9dd2-1c6e2e7c4cc8" />


## 5. Understanding

### 1`$match`
Filters documents to only include records from the year 2024.

### 2 `$group`

Groups documents by `genre` and computes:
* `totalViews` using `$sum`
* `avgRating` using `$avg`

### 3 Second `$match`
Filters **aggregated results**, keeping only genres where total views exceed 10,000.

### 4 `$project`
Reshapes the final output:
* Removes `_id`
* Renames genre
* Rounds average rating to 1 decimal place

<br>

## Output

```json
[
{
  totalViews: 22000,
  genre: 'Action',
  avgRating: 7.9
}
{
  totalViews: 23000,
  genre: 'Drama',
  avgRating: 8.4
}
  {
  totalViews: 45000,
  genre: 'Sci-Fi',
  avgRating: 8.4
}
]
```


