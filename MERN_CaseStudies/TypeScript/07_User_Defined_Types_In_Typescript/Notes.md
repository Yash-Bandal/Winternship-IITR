# [7. User-Defined Types in TypeScript](https://sudarshansudarshan.github.io/winternship/case-studies/07-user-defined-types-in-typescript/)

## 7.1 Lesson Focus (Core Idea)

This lesson focuses on **modeling real-world data using custom, user-defined types**. We learn how TypeScript lets us go beyond primitives by defining **structured data contracts** using arrays, tuples, enums, interfaces, and classes.

What is discussed and practiced in this lesson:

* Grouping related data using arrays and tuples
* Representing fixed sets of values using enums
* Defining object shapes using interfaces
* Organizing data + behavior using classes
* Using these types together to build scalable, maintainable systems

The goal is to make complex data **type-safe, readable, and easy to extend**.

<br>

## 7.2 Interactive Challenge / Mini-Project

**Your Turn!**

* Define an enum `Role` for staff roles (Doctor, Nurse, Admin).
* Create an interface `Staff` with fields `id`, `name`, and `role`.
* Create an array of staff members using the interface and enum.
* Write a function that prints a summary of all staff members.

<br>

## 7.3 Structure of Code

1. Enum to represent fixed role values
2. Interface to define staff object structure
3. Array of typed objects
4. Function operating on typed data
5. Additional examples using typed arrays

<br>

## 7.4 Answer (Solution Code)

```ts
// 1. Enum for staff roles (fixed set of values)
enum Role {
  Doctor = "Doctor",
  Nurse = "Nurse",
  Admin = "Admin"
}

// 2. Interface defining the structure of a Staff object
interface Staff {
  id: number;
  name: string;
  role: Role;
}

// 3. Array of staff members using interface + enum
let staffMembers: Staff[] = [
  { id: 1, name: "Dr. Pandey", role: Role.Doctor },
  { id: 2, name: "Namita", role: Role.Nurse },
  { id: 3, name: "Admindada", role: Role.Admin }
];

// 4. Function to print staff summary
function printStaffSummary(staffList: Staff[]): void {
  console.log("Hospital Staff Summary:");
  for (let staff of staffList) {
    console.log(`${staff.name} — ${staff.role}`);
  }
}

printStaffSummary(staffMembers);

// 5. Typed arrays examples
let pID: number[] = [1, 2, 3, 4];       // number array
let pName: Array<string> = [];         // generic array syntax
```

<br>

## 7.5 Output

```text
Hospital Staff Summary:
Dr. Pandey — Doctor
Namita — Nurse
Admindada — Admin
```

<br>

## 7.6 Small Explanation

* **Enums** restrict values to a known set, preventing invalid states.
* **Interfaces** define object structure and enforce consistency.
* **Arrays** store collections of typed data.
* **Functions** operating on interfaces ensure safe data usage.
* Combining user-defined types allows TypeScript to model **complex systems cleanly**.

This lesson shows how TypeScript scales from simple variables to **real-world application models**.
