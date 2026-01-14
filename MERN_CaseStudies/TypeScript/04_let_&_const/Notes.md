# [4. let & const in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/04-let-const/)

## 4.1 Lesson Focus (Core Idea)

This lesson focuses on **controlling mutability and scope** in TypeScript. You learn how `let` allows controlled changes within a block, while `const` locks values to prevent accidental reassignment—making your code safer and more predictable.


<br>


## 4.2 Interactive Challenge / Mini-Project

**Your Turn!**
* Declare a variable `score` using `let` and assign it a number.
* Inside a block (`if` statement), modify `score` and print it.
* Declare a constant `COUNTRY` and assign it your favorite country.
* Try to change the value of `COUNTRY` and observe what happens.
* Try to redeclare `score` in the same block and see the result.


<br>

## 4.3 Structure of Code

1. Variable declaration using `let`
2. Block scope modification
3. Constant declaration using `const`
4. Attempted reassignment and redeclaration (commented)


<br>


## 4.4 Answer (Solution Code)

```ts
let score: number = 101;
console.log(score);

if (score > 100) {
  score = 200; // allowed
  // let score = 200; // Error: Cannot redeclare block-scoped variable
  console.log(score);
}

const COUNTRY: string = "INDIA";
// COUNTRY = "BHUTAN"; // Error: Cannot assign to 'COUNTRY' because it is a constant
console.log(COUNTRY);
```


<br>

## 4.5 Output

```text
101
200
INDIA
```


<br>

## 4.6 Small Explanation
* `let` allows reassignment but **prevents redeclaration in the same scope**.
* `let` variables are **block scoped**, so behavior is predictable inside `{}`.
* `const` must be initialized and **cannot be reassigned**.
* TypeScript throws **compile-time errors** for invalid reassignment or redeclaration.\
This lesson reinforces safe variable handling using **block scope and immutability**.
