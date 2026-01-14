# [9. Type Aliases in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/09-type-aliases/)

## 9.1 Lesson Focus (Core Idea)

This lesson focuses on **creating reusable, meaningful names for complex types** using **type aliases**. Type aliases help avoid repetition, improve readability, and make large systems easier to maintain.

What is discussed and practiced in this lesson:
* Creating aliases for primitives, unions, objects, and functions
* Using aliases to simplify function signatures
* Modeling real-world entities (customers, orders) with clear type names
* Improving code self-documentation and consistency
The goal is to make TypeScript code **cleaner, scalable, and easier to reason about**.

<br>

## 9.2 Interactive Challenge / Mini-Project

**Your Turn!**

* Define a `CustomerID` alias for `string`.
* Create a `Customer` object alias with `id`, `name`, and optional `email`.
* Define an `OrderStatus` union type.
* Create a `ProcessOrder` function type alias that accepts an order ID and a callback.
* Implement and use the function with sample customers.

<br>


## 9.3 Structure of Code

1. Primitive type alias for identifiers
2. Object type alias with optional properties
3. Union type alias for limited states
4. Function type alias for callbacks
5. Concrete implementation using aliases

<br>

## 9.4 Answer (Solution Code)

```ts
// 1. CustomerID alias (primitive alias)
type CustomerID = string;

// 2. Customer object alias
type Customer = {
  id: CustomerID;
  name: string;
  email?: string; // optional field
};

// 3. Order status union alias
type OrderStatus = "pending" | "shipped" | "cancelled";

// 4. processOrder function type alias
type ProcessOrder = (
  orderId: number,
  callback: (status: OrderStatus) => void
) => void;

// 5. Implementation of the function alias
const processOrder: ProcessOrder = (orderId, callback) => {
  console.log("Processing order:", orderId);
  callback("shipped");
};

// Sample customers
const customer1: Customer = {
  id: "C101",
  name: "Alice",
  email: "alice@mail.com"
};

const customer2: Customer = {
  id: "C102",
  name: "Bob"
};

console.log("Customers:", customer1, customer2);

// Execute order processing
processOrder(5001, (status) => {
  console.log("Order status:", status);
});
```

<br>

## 9.5 Output

```text
Customers: 
{
  id: 'C101', 
  name: 'Alice',
  email: 'alice@mail.com'
}
{
  id: 'C102',
  name: 'Bob'
}
Processing order: 5001
Order status: shipped
```

<br>

## 9.6 Small Explanation

* **Type aliases** give meaningful names to complex types.
* Primitive aliases (`CustomerID`) clarify intent.
* Object aliases (`Customer`) enforce structure and reuse.
* Function aliases (`ProcessOrder`) simplify signatures and callbacks.
* Union aliases restrict values to valid states only.

This lesson shows how type aliases make large codebases **cleaner, safer, and easier to scale**.
