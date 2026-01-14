# [1. Introduction to TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/01-introduction-to-typescript/)


## Core Idea
TypeScript is about safety, clarity, and scalability.\
The core idea of this lesson is that TypeScript adds static typing to JavaScript, allowing developers to:
- Catch errors before runtime
- Write self-documenting code
- Build large, maintainable applications with confidence
- TypeScript acts as a safety layer over JavaScript, helping teams prevent bugs early and making code easier to understand as projects grow.

<br>

## 1.1 Interactive Challenge / Mini-Project

**Your Turn!**
* Change the `message` variable to your own name and print a personalized greeting.
* Declare a variable for your age and print it with a message.
* Try assigning a number to a variable declared as a `string` and observe what happens.

<br>

## 1.2 Structure of Code

1. Declare variables with explicit TypeScript types
2. Assign valid values to those variables
3. Print values using `console.log`
4. Intentionally try an invalid assignment to see TypeScript's error handling

<br>

## 1.3 Answer (Solution Code)

```ts
let message: string = "Yash";
let age: number = 21;

console.log("Hello, my name is " + message);
console.log("I am " + age + " years old");

// Invalid assignment (will cause error)
// message = 25;
```

<br>

## 1.4 Output

```text
Hello, my name is Yash
I am 21 years old
```

<br>

## 1.5 Small Explanation

* `message: string` allows only text values.
* `age: number` allows only numeric values.
* When you try `message = 25`, TypeScript throws a **compile-time error**.
* This shows how TypeScript catches mistakes **before the program runs**.

This mini-challenge demonstrates TypeScript’s core strength: **type safety from the start**.
