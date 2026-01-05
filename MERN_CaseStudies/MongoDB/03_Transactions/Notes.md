# Transactions 

# Setup Transactios and Session

---

> [!Note]
> ##  Why This Setup Was Needed

> MongoDB **multi-document transactions require a replica set**.\
> A standalone MongoDB server (default Windows service mode) does **not** support transactions.\
> Therefore, we temporarily:
>     * Stopped MongoDB Windows Service
>     * Started MongoDB manually with replica set enabled
>     * Used a custom writable data directory

---

<br>

## 1  Stopping MongoDB Windows Service (Initial Step)

**Why:**
* Avoid port conflicts
* Disable standalone mode
* Gain full control over startup options

**Steps:**
1. Windows `win + R` -> Open `services.msc` 
<img width="998" height="732" alt="image" src="https://github.com/user-attachments/assets/a1f64086-b223-434a-b166-a9ffba947006" />

2. Locate **MongoDB Server**
3. Click **Stop**

At this point, no MongoDB server is running.

<br>

## Now
1. Open Terminal / CMD
2. Try
   ```cmd
   C:\Users\admin>mongod --replSet rs0 --dbpath "C:\Program Files\MongoDB\Server\7.0\data"
   ```
3. If you see
   ```
   Waiting for connections
   and many warnings
   ```
  Start session , else if error / see direct
  ```
  C> :...
  ```
  Try technique below 
  
## Why `C:\Program Files` Failed (Important Error Handling)

### Error Encountered

```
WiredTiger.lock: Permission denied
```

### Root Cause
  * `C:\Program Files` is a **protected Windows directory**
  * Manual `mongod` does **not** run with elevated write permissions
  * WiredTiger requires full read/write access

### Correct Resolution
Use a **user-writable directory**:

```
C:\data\db
```
<img width="624" height="52" alt="image" src="https://github.com/user-attachments/assets/bf557ee7-60bf-4823-9286-c45fc6559c00" />

This is the **recommended MongoDB practice on Windows**.

You may see a lot of warnings, ignore them 

---

## 1. Creating the Data Directory inside C drive

Manually created:

```
C:\data\db
```

This directory is:
    * Writable
    * Safe for WiredTiger
    * Common in MongoDB documentation

<br>

## 2. Starting MongoDB with Replica Set Enabled

MongoDB was started manually using:

```bash
mongod --replSet rs0 --dbpath C:\data\db
```

### Key Points
* `--replSet rs0` enables replica set mode
* `rs0` is the replica set name
* Terminal **must remain open**

### Successful Startup Indicator

```
Waiting for connections
mongod startup complete
```

Warnings about `ReadConcernMajorityNotAvailableYet` are **expected** at this stage.

<br>

## 3. Connecting via mongosh

In a **separate terminal**: / Not the mongod above one..seperate

```bash
mongosh
```

This connects to the running `mongod` instance.


<img width="1756" height="577" alt="image" src="https://github.com/user-attachments/assets/02fc2929-a315-4750-b21c-423cdb9404fc" />

<br>

## 7. Initializing the Replica Set

Replica sets require **explicit initialization**.

```js
rs.initiate()
```

### Verification

```js
rs.status()
```

Confirmed:

* `set: "rs0"`
* `stateStr: "PRIMARY"`

At this point:

>  MongoDB supports transactions

<br>

## 4. Database & Collections Setup

```js
use fintrust

db.createCollection("users")
db.createCollection("transactions")
```

Inserted users:

```js
db.users.insertMany([
  { name: "Alice", balance: 500 },
  { name: "Bob", balance: 300 }
])
```

<br>

## 5. Starting a Transaction (Correct Way in mongosh)

### Important mongosh Rule

In transactions, **do not use the global `db` object**.

Always bind the session to a database:

```js
const session = db.getMongo().startSession();
session.startTransaction();
const txDb = session.getDatabase("fintrust");
```

<br>

## 6. Transaction Operations (ACID Demonstration)

### a Deduct from Alice

```js
txDb.users.updateOne(
  { name: "Alice" },
  { $inc: { balance: -100 } }
)
```

### b Credit Bob

```js
txDb.users.updateOne(
  { name: "Bob" },
  { $inc: { balance: 100 } }
)
```

### c Insert Transaction Log

```js
txDb.transactions.insertOne({
  from: "Alice",
  to: "Bob",
  amount: 100,
  type: "transfer",
  status: "completed",
  date: new Date()
})
```

<br>

## 7 Error Encountered: Transaction Aborted

### Error Seen

```
NoSuchTransaction: Transaction has been aborted
```

### Why This Happened
* Transactions are **time-bound**
* Shell inactivity or errors can auto-abort
* MongoDB aborts to **protect atomicity**

### Correct Handling (Industry Practice)
1. Abort and discard old transaction
2. Start a **fresh transaction**
3. Re-run all steps

This behavior is **expected and correct**.

<br>

## 8. Committing the Transaction

```js
session.commitTransaction();
session.endSession();
```

After commit:
* Changes become visible to all clients
* Data is durable

<br>

## 13. Final Verification (Outside Transaction)

```js
db.users.find().pretty()
```

Confirmed:

* Alice → 400
* Bob → 400

```js
db.transactions.find().pretty()
```

Confirmed:
* One transaction log exists

This proves:
* Atomicity
* Consistency
* Isolation
* Durability

<br>


<br>

---

<br>
<br>

# Transactions – Refund Flow (Challenge)

### Your Turn!
You’re building a new feature for **FinTrust**:
If a user disputes a payment, you must execute the following workflow:
1. **Credit Sender:** Add the refund amount back to the **sender’s balance**.
2. **Debit Recipient:** Subtract the amount from the **recipient’s balance**.
3. **Update Status:** Update the original transaction’s status to `"refunded"`.
4. **Audit:** Log a new transaction record as a **refund**.

###  Ensure:
* **Atomicity:** If *any* step fails (e.g., the recipient doesn’t have enough balance to refund), **no changes are made**.
* **Integrity:** All operations are **ACID-compliant**.





## Precondition (Existing Data)

A completed transfer already exists in the database:

```json
{
  _id: ObjectId("695be65a1b67fb18ee46b79d"),
  from: "Alice",
  to: "Bob",
  amount: 100,
  type: "transfer",
  status: "completed"
}
```

Current balances before refund:

* Alice → 400
* Bob → 400

<br>

##  Step 1 – Identify Original Transaction

```js
db.transactions.find(
  { type: "transfer", status: "completed" },
  { _id: 1 }
)
```

**Output:**

```js
{ _id: ObjectId("695be65a1b67fb1adasd8ee46b79d") }
```

<br>

##  Step 2 – Start Session & Transaction

```js
const session = db.getMongo().startSession();
session.startTransaction();
const txDb = session.getDatabase("fintrust");
```
No output

<br>

##  Step 3 – Fetch Original Transaction (Inside TX)

```js
const originalTxn = txDb.transactions.findOne({
  _id: ObjectId("695be65a1b67fb18ee46b79d")
});
originalTxn
```

**Output (simplified):**

```json
{
  from: "Alice",
  to: "Bob",
  amount: 100,
  status: "completed"
}
```

<br>

##  Step 4 – Validate Recipient Balance

```js
const recipient = txDb.users.findOne({ name: originalTxn.to });
recipient
```

**Output:**

```json
{ name: "Bob", balance: 400 }
```

Validation:

```
400 >= 100  // OK
```

<br>

## Step 5 – Reverse Balances (Inside TX)

### Debit Bob

```js
txDb.users.updateOne(
  { name: originalTxn.to },
  { $inc: { balance: -originalTxn.amount } }
);
```

### Credit Alice

```js
txDb.users.updateOne(
  { name: originalTxn.from },
  { $inc: { balance: originalTxn.amount } }
);
```

**Inside-transaction state:**

* Alice → 500
* Bob → 300

<br>

## Step 6 – Update Original Transaction

```js
txDb.transactions.updateOne(
  { _id: originalTxn._id },
  { $set: { status: "refunded", refundedAt: new Date() } }
);
```

**Result:**

* Prevents double refunds

<br>

##  Step 7 – Insert Refund Transaction Record

```js
txDb.transactions.insertOne({
  type: "refund",
  originalTxnId: originalTxn._id,
  from: originalTxn.to,
  to: originalTxn.from,
  amount: originalTxn.amount,
  status: "completed",
  date: new Date()
});
```

<br>

##  Step 8 – Commit Transaction

```js
session.commitTransaction();
session.endSession();
```

**Guarantee:**
* If any step failed → automatic rollback

<br>

## Final Verification (Outside Transaction)

### Verify Balances

```js
db.users.find().pretty();
```

**Output:**

```json
{ name: "Alice", balance: 500 }
{ name: "Bob", balance: 300 }
```

### Verify Transactions

```js
db.transactions.find().pretty();
```

**Output:**
* Original transfer → `status: "refunded"`
* New refund transaction → exists

<br>

---

<br>




##  Restoring MongoDB Windows Service

### Steps

1. Press `CTRL + C` in the manual `mongod` terminal
2. Wait for clean shutdown
3. Open `services.msc`
4. Start **MongoDB Server** service

### Result

* MongoDB runs in standalone mode
* CRUD works normally
* Transactions are disabled (expected)
