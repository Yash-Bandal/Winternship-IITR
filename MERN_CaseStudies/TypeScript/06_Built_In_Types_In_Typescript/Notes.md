# [6. Built-in Types in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/06-built-in-types-in-typescript/)

## 6.1 Lesson Focus (Core Idea)

This lesson focuses on **using TypeScript’s built-in types to model real-world data safely**. We discuss how primitive and special types (`number`, `string`, `boolean`, `void`, `null`, `undefined`, `never`, `symbol`, `object`) help prevent type confusion, enforce correct function behavior, and handle missing or exceptional values.

What we do in this lesson:

* Use built-in types to represent financial-style data
* Handle optional / missing values safely
* Use `void` for non-returning functions and `never` for unreachable paths
* Combine types using unions (e.g., `string | undefined`)

<br>

## 6.2 Interactive Challenge / Mini-Project

**Your Turn!**

* Create a function `processTransaction` that takes:
  * `amount` (number)
  * `description` (string or undefined)
  * `isCredit` (boolean)
* If the amount is negative, throw an error (`never` case).
* If the description is missing, handle it safely.
* Print a transaction summary.

<br>

## 6.3 Structure of Code

1. Function with explicitly typed parameters
2. Union type to allow missing values
3. Error handling using `never`
4. Conditional logic for safe defaults
5. Console output summary

<br>

## 6.4 Answer (Solution Code)

```ts
// | is important here: description can be string OR undefined
function processTransaction(
  amount: number,
  description: string | undefined,
  isCredit: boolean
): void {
  
  // Handle invalid (negative) amount
  // This path never returns normally
  if (amount < 0) {
    throw new Error("Amount cannot be negative");
  }

    // Handle missing description safely
    //check if desc is(==) undef / ? has value
    //=== strict check, if === undef, return 'no desc'
  const finalDescription = description === undefined
    ? "No desc provided"
    : description;

  // Print transaction summary
  console.log("Transaction Summary:");
  console.log("Amount:", amount);
  console.log("Type:", isCredit ? "Credit" : "Debit");
  console.log("Description:", finalDescription);
}

// Valid calls
processTransaction(500, "Paid successfully", true);
processTransaction(200, undefined, false);
```

<br>

## 6.5 Output

```text
Transaction Summary:
Amount: 500
Type: Credit
Description: Paid successfully

Transaction Summary:
Amount: 200
Type: Debit
Description: No desc provided
```

<br>

## 6.6 Small Explanation

* Built-in types ensure **correct data handling** at compile time.
* Union types (`string | undefined`) force explicit checks for missing values.
* Throwing an error represents a `never` execution path.
* `void` confirms the function does not return a value.

This lesson shows how built-in types make programs **safe, predictable, and robust**.
