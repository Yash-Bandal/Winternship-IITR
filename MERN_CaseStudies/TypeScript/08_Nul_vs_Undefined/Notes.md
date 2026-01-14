# [8. null vs undefined in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/08-null-vs-undefined/)

## 8.1 Lesson Focus (Core Idea)

This lesson focuses on **clearly representing missing data**. We distinguish between values that are **intentionally empty (`null`)** and values that are **not yet set or optional (`undefined`)**.

What is discussed and practiced in this lesson:

* Semantic difference between `null` and `undefined`
* When to use nullable types vs optional properties
* Safe checks and defaults to avoid runtime errors
* Modeling real user data where fields may be missing or intentionally empty

The goal is to make data intent **explicit, readable, and safe** for other developers and future code.

<br>

## 8.2 Interactive Challenge / Mini-Project

**Your Turn!**

* Define a type `Profile` with:
  * `username` (string)
  * `bio` (string or null)
  * optional `avatarUrl` (string)
* Create two profiles:
  * one with `bio` set to `null` and no avatar
  * one with both fields set
* Write a function `showProfile` that:
  * prints a default message if `bio` is `null`
  * prints a default avatar message if `avatarUrl` is `undefined`

<br>

## 8.3 Structure of Code

1. Custom type with nullable and optional properties
2. Objects representing different data states
3. Explicit checks for `null`
4. Explicit checks for `undefined`
5. Safe output with meaningful defaults

<br>

## 8.4 Answer (Solution Code)

```ts
// 1. Define the Profile type
// bio can be explicitly empty (null)
// avatarUrl may be missing (undefined)
type Profile = {
  username: string;
  bio: string | null;
  avatarUrl?: string;
};

// 2. Create profiles with different states
let profile1: Profile = {
  username: "Yash Bandal",
  bio: null // intentionally empty bio
};

let profile2: Profile = {
  username: "Raj Goel",
  bio: "Traveler",
  avatarUrl: "https://example.com/RG.png"
};

// 3. Function to safely display profile info
function showProfile(profile: Profile): void {
  // Handle null bio explicitly
  const bioText = profile.bio === null
    ? "No bio yet"
    : profile.bio;

  // Handle undefined avatar explicitly
  const avatarText = profile.avatarUrl === undefined
    ? "Default avatar image used"
    : `Avatar: ${profile.avatarUrl}`;

  console.log("Username:", profile.username);
  console.log("Bio:", bioText);
  console.log(avatarText);
  console.log("---------------------------");
}

showProfile(profile1);
showProfile(profile2);

// let a= null;
// console.log(a)
```

<br>

## 8.5 Output

```text
Username: Yash Bandal
Bio: No bio yet
Default avatar image used
---------------------------
Username: Raj Goel
Bio: Traveler
Avatar: https://example.com/RG.png
---------------------------
```

<br>

## 8.6 Small Explanation

* `null` is used when a value is **intentionally empty**.
* `undefined` means a value is **not provided or not yet set**.
* Union types (`string | null`) and optional properties (`?`) make intent explicit.
* Strict checks avoid runtime errors and improve code clarity.

This lesson reinforces **semantic correctness** when handling missing data.
