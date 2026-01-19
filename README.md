# 🏥 Medicine Inventory & Order Management System  
### Complete Project Understanding – Internal Flow, Design & API

This README is the **complete soul of the project**.  
It explains:

✔ Business purpose  
✔ Domain model  
✔ Database relationships  
✔ Stock calculation  
✔ FEFO algorithm  
✔ Transaction behavior  
✔ Exact API contracts  
✔ Internal execution steps  
✔ Error scenarios  

Anyone reading this can fully understand the project **without seeing code**.

---

# 1️⃣ PROJECT PURPOSE – REAL WORLD MEANING

This system models how a **real pharmacy manages medicines**.

## ❌ Wrong Model

Paracetamol = 100 units Amoxicillin = 50 units

## ✔ Real Model – BATCH BASED

| Medicine | Batch | Expiry | Qty |
|---------|-------|--------|-----|
| PCM     | B1    | 2025   | 30  |
| PCM     | B2    | 2026   | 70  |

Reason:

- Medicines expire  
- Different suppliers  
- Legal tracking  
- Recall management  

👉 That is the FOUNDATION of this project.

---

# 2️⃣ WHAT THIS PROJECT MANAGES

### Three Core Concepts

1. **Medicine** – product definition  
2. **Batch** – physical stock unit  
3. **OrderItem** – sales history  

### One Line Flow

Medicine → has many Batches → batches create OrderItems when sold

---

# 3️⃣ ENTITIES – EXACT MEANING

## 🟢 Medicine – Product Definition

Medicine DOES NOT hold real stock.  
It only holds identity:

- name  
- price  
- category  
- sku  
- requiresRx  

### Derived Fields (Auto Calculated)

- totalQuantity  
- inStock  
- stockStatus  

👉 These come from BATCHES, not manual input.

---

## 🟡 Batch – Real Stock

Represents:

- physical quantity  
- expiry date  
- batch number  

### Batch = Actual Sellable Units

Why needed:

- expiry control  
- FEFO deduction  
- legal audit  

---

## 🔵 OrderItem – Sale Record

Stores:

- quantity sold  
- price at that time  
- from which batch  

---

# 4️⃣ DATABASE RELATIONSHIPS

## Medicine → Batch (1:N)

medicine.id ←── batch.medicine_id

Meaning:

- One medicine → many batches  
- Batch belongs to one medicine  
- FK stored in BATCH table  

---

## Batch → OrderItem (1:N)

batch.id ←── order_items.batch_id

Meaning:

- One batch can serve many orders  
- Each order line comes from one batch  

---

# 5️⃣ STOCK DERIVATION LOGIC

## Total Quantity

Medicine.totalQuantity = SUM(batch.qtyAvailable)

## In Stock

inStock = totalQuantity > 0

## Status

0        → "Out of stock" 1–49     → "Low stock" 50+      → "Available"

---

# 6️⃣ FEFO – CORE ALGORITHM

### First Expiry First Out

When selling:

1. Ignore expired batches  
2. Sort by expiry ASC  
3. Deduct sequentially  

## Example

### Before

| Batch | Expiry | Qty |
|------|--------|----|
| B1   | 2025   | 30 |
| B2   | 2026   | 70 |

### Buy 40

- 30 from B1  
- 10 from B2  

### After

| Batch | Qty |
|------|----|
| B1 | 0 |
| B2 | 60 |

---

# 7️⃣ ORDER INTERNAL FLOW

### Step 1 – Validation
- medicine exists  
- non expired batches  
- enough quantity  

### Step 2 – FEFO Deduction

need = order qty

for batch in sorted batches: if expired → skip

if batch.qty >= need: deduct need create OrderItem break else: deduct batch.qty create OrderItem need -= batch.qty

If need > 0 → FAIL

### Step 3 – Update Medicine

Recalculate:

- totalQuantity  
- inStock  
- stockStatus  

### Step 4 – Save OrderItem

### Step 5 – Commit Transaction

---

# 8️⃣ TRANSACTION GUARANTEE

✔ Atomic  
✔ No partial deduction  
✔ Rollback on failure  

---

# 9️⃣ WHY EACH FIELD EXISTS

### priceAtPurchase
- price can change later  
- invoice must preserve old price  

### sku
- unique business identity  

### requiresRx
- prescription validation  

---

# 🔟 EDGE CASES

✔ expired batch skip  
✔ multi batch deduction  
✔ insufficient block  
✔ negative guard  
✔ concurrent safety  

---

# 1️⃣1️⃣ API DOCUMENTATION – EXACT CONTRACTS

## BASE

- http://localhost:8080  
- /v3/api-docs  
- Content-Type: application/json  

Controllers:

1. medicine-controller  
2. batch-controller  
3. order-controller  

---

## 1. MEDICINE CONTROLLER

### GET /medicines

#### Purpose
Get medicines with batches + derived stock.

#### DB

SELECT * FROM medicines LEFT JOIN batches ON medicines.id = batches.medicine_id

#### Response

```json
[
 {
  "id": 1,
  "name": "Paracetamol",
  "category": "Analgesic",
  "price": 50,
  "sku": "MED-001",
  "requiresRx": false,

  "batches":[
   {
    "id":10,
    "batchNo":"B1",
    "expiryDate":"2026-01-19",
    "qtyAvailable":100,
    "orderItems":[]
   }
  ],

  "inStock":true,
  "stockStatus":"Available",
  "totalQuantity":100
 }
]


---

GET /medicines/{id}

Flow:

1. find medicine


2. load batches


3. load orderItems



Errors:

404 not found

500 db error



---

POST /medicines

YOU MUST SEND

{
 "name":"Dolo",
 "category":"Fever",
 "price":20,
 "sku":"MED-1",
 "requiresRx":false,
 "batches":[]
}

DO NOT SEND

❌ id
❌ totalQuantity
❌ inStock
❌ stockStatus

System generates them.


---

PUT /medicines/{id}

Used to:

update info

attach batches

recalc stock



---

DELETE /medicines/{id}

Risk:

if batches exist without cascade → 409



---

2. BATCH CONTROLLER

POST /batches

{
 "batchNo":"B2026",
 "expiryDate":"2026-10-01",
 "qtyAvailable":100,
 "orderItems":[]
}

⚠ Not linked to medicine until mapping done.


---

PUT /batches/{id}

Change qty / expiry → triggers recalc.


---

DELETE /batches/{id}

Blocked if orderItems exist.


---

3. ORDER CONTROLLER

POST /api/orders/pay

[
 {
  "medicineId":1,
  "quantity":2,
  "priceAtPurchase":50
 }
]

Internal Execution

1. validate medicine


2. load non expired batches


3. FEFO deduction


4. create OrderItems


5. update batches


6. recalc medicine


7. commit



Errors

422 insufficient

422 expired only

404 medicine



---

1️⃣2️⃣ WHAT EACH API TOUCHES

API	Tables

POST /medicines	medicines
POST /batches	batches
PUT /medicines	medicines + batches
POST /pay	batches + order_items + medicines



---

1️⃣3️⃣ COMMON MISTAKES

❌ sending derived fields
❌ using expired batch
❌ not linking batch
❌ expecting partial order


---

1️⃣4️⃣ DEBUG STEPS

1. check expiry


2. check totals


3. check batch.medicine_id


4. check transaction




---

1️⃣5️⃣ INTERVIEW SUMMARY

This project proves:

✔ real domain modeling
✔ JPA 1:N
✔ FEFO algorithm
✔ transactions
✔ derived fields
✔ not CRUD


---

FINAL STORY

Medicine = product

Batch = real stock

OrderItem = sale

Stock derived from batches

FEFO protects safety

Everything transactional



---

This document is the COMPLETE understanding of your project.
