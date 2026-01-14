#  [3. Variables in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/03-variables-in-typescript/)

## 3.1 Lesson Focus (Core Idea)

Variables are the **foundation of TypeScript**. This lesson focuses on **declaring variables correctly, using types safely, and understanding scope**, so data in your program is clear, predictable, and protected from misuse.

<br>

## 3.2 Interactive Challenge / Mini-Project
**Your Turn!**
* Declare a variable called `city` and assign it your favorite city as a string.
* Declare a variable called `temperature` with type `number` and assign it a value.
* Create a variable called `isRaining` and let TypeScript infer its type.
* Write a function `weatherReport` that takes `city`, `temperature`, and `isRaining` and prints:
  `In <city>, it is <temperature>°C. Is it raining? <true/false>`
* Call the function using your variables.


<br>

## 3.3 Structure of Code

1. Variable declaration with explicit type
2. Variable declaration using type inference
3. Function with typed parameters
4. Function call using declared variables


<br>

## 3.4 Answer (Solution Code)

```ts
// Type inference example
var num = 2; // inferred as number
console.log("value of num " + num);

// Explicitly typed variables
let city: string = "Tokyo";
let temperature: number = 22;

// Type inferred as boolean
let isRaining = false;

// Function using typed parameters
function weatherReport(city: string, temperature: number, isRaining: boolean): void {
  console.log(`In ${city}, temperature is ${temperature} degree, and is it raining, ${isRaining}`);
}

weatherReport(city, temperature, isRaining);
```


<br>

## 3.5 Output

```text
value of num 2
In Tokyo, temperature is 22 degree, and is it raining, false
```

<br>

## 3.6 Small Explanation

* `city` and `temperature` use **explicit typing** for clarity and safety.
* `isRaining` uses **type inference**, where TypeScript decides the type automatically.
* The function enforces correct data types through its parameters.
* Any wrong type passed (e.g., string instead of number) causes a **compile-time error**.

This lesson highlights how TypeScript variables make programs **robust, readable, and error-free**.
