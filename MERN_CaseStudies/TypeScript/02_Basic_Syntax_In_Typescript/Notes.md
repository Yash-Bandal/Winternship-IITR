# [2. Basic Syntax in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/02-basic-syntax-in-typescript/)

## 2.1 Interactive Challenge / Mini-Project

**Your Turn!**

* Create a variable for your favorite fruit and print it.
* Write a function that takes a number and prints double its value.
* Add a single-line and a multi-line comment.
* Define a class called `Person` with a method `sayHello` that prints a greeting.

<br>

## 2.2 Structure of Code

1. Variable declaration with type
2. Function definition with typed parameter
3. Single-line and multi-line comments
4. Class definition with a method
5. Object creation and method call

<br>

## 2.3 Answer (Solution Code)

```ts
// Favorite fruit variable
let favoriteFruit: string = "Mango";
console.log("Favorite fruit: " + favoriteFruit);

/* Function to print double the value */
function printDouble(num: number): void {
  console.log("Double value: " + num * 2);
}

printDouble(10);

// Person class definition
class Person {
  sayHello(): void {
    console.log("Hello! Nice to meet you.");
  }
}

let person = new Person();
person.sayHello();
```

<br>

## 2.4 Output

```text
Favorite fruit: Mango
Double value: 20
Hello! Nice to meet you.
```

### Second
```ts
// Task1 - Printing fruit
let fruit: string = "Orange";
console.log(fruit);

/*
Task 2 - Double Function
*/

function doubleN(num: number): void{
    console.log(num*num);
}

doubleN(4);
doubleN(10);

//Task 3: Class
class Person{
    sayHello(pers: string): void{
        console.log(`Hello  ${pers}`);
    }
}

let pers = new Person();
pers.sayHello("Yash");
```

<br>

## 2.5 Small Explanation

* Variables store typed data (`string`, `number`).
* Functions define reusable logic with typed inputs.
* Comments improve readability but don’t affect execution.
* Classes group related behavior using methods.
* TypeScript enforces structure and clarity at compile time.
