# [5. `any` Type in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/05-any-type-in-typescript/)

## 5.1 Lesson Focus (Core Idea)

This lesson focuses on **flexibility vs safety**. The `any` type allows TypeScript to accept **unknown or changing data**, but it disables compile-time checks. It is useful in early stages or dynamic scenarios, with the intent to **gradually replace `any` with safer types** later.

<br>

## 5.2 Interactive Challenge / Mini-Project

**Your Turn!**

* Create a function `recordAnswer` that takes a question ID and an answer of type `any`.
* Store answers in an object.
* Add at least three answers: a string, a number, and an array.
* Print all recorded answers.

<br>

## 5.3 Structure of Code

1. Object to store dynamic answers
2. Function accepting `any` type
3. Storing mixed-type values
4. Iterating and printing stored data

<br>

## 5.4 Answer (Solution Code)

```ts
// Object with any type values
let recordedAnswers: any = {};

function recordAnswer(questionId: string, answer: any): void {
  recordedAnswers[questionId] = answer;
}

recordAnswer("Q1", "Yes");
recordAnswer("Q2", 1);
recordAnswer("Q3", ["OptionA", "OptionB"]);

// Print all answers
for (let key in recordedAnswers) {
  console.log("Received answer: " + key + ": " + recordedAnswers[key]);
}
```

<br>

## 5.5 Output

```text
Received answer: Q1: Yes
Received answer: Q2: 1
Received answer: Q3: OptionA,OptionB
```

<br>

## 5.6 Small Explanation

* `any` disables type checking and accepts **any value**.
* Mixed data (string, number, array) can be stored without compiler errors.
* TypeScript will **not warn** if unsafe operations are performed.
* `any` should be used **temporarily** when data shape is unknown.

This lesson highlights why `any` offers flexibility but should be handled with caution.
