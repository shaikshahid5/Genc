# 🏥 Medicine Inventory & Order Management System  
### Complete Project Understanding – Internal Flow & Design

This document explains **what your project actually is**, how each part is connected, and what exactly is happening inside the system — from database to business logic.  
This is NOT API usage guide. This is the **brain of the project.**

After reading this you will clearly know:

✔ What each entity represents  
✔ How tables are connected  
✔ How stock is calculated  
✔ How order deduction works  
✔ Why certain fields exist  
✔ What data links to what  

---

# 1️⃣ PROJECT PURPOSE – REAL WORLD MEANING

This project models a **real pharmacy inventory system**.

### ❌ Wrong Thinking

Paracetamol = 100 units Amoxicillin = 50 units

### ✔ Real Pharmacy Thinking
Stock is divided by BATCH & EXPIRY

| Medicine | Batch | Expiry | Qty |
|---------|-------|--------|-----|
| PCM     | B1    | 2025   | 30  |
| PCM     | B2    | 2026   | 70  |

Because:
- Medicines expire  
- Different suppliers  
- Legal tracking  
- Recall management  

👉 That is the CORE reason this project exists.

---

# 2️⃣ WHAT EXACTLY YOUR PROJECT MANAGES

Your system manages 3 main things:

1. Medicine Definition  
2. Physical Stock (Batches)  
3. Sales History (OrderItems)

## Flow in one line:

Medicine → has many Batches → batches produce OrderItems when sold

---

# 3️⃣ ENTITIES – REAL MEANING (NOT THEORY)

## 🟢 1. Medicine – “Product Definition”

Medicine table DOES NOT store real stock.  
It stores:

- Name  
- Price  
- Category  
- SKU  
- Prescription requirement  

### Medicine = Identity of Drug  
Not physical packets.

### Derived Fields on Medicine

These are NOT manually entered:

- totalQuantity  
- inStock  
- stockStatus  

They are CALCULATED from batches.

---

## 🟡 2. Batch – “Real Physical Stock”

Batch represents:

✔ Actual strip/box group  
✔ With expiry date  
✔ With quantity available  

### Why batch is needed

Because:

- 2024 batch cannot be mixed with 2026  
- expired stock must be blocked  
- deduction must follow FEFO  

Batch is the REAL STOCK HOLDER.

---

## 🔵 3. OrderItem – “Sale Record”

OrderItem represents:

- How much sold  
- From which batch  
- At what price  

It freezes:

✔ priceAtPurchase  
✔ quantity  
✔ linkage to batch  

---

# 4️⃣ DATABASE RELATIONSHIPS – EXACT TRUTH

## Medicine → Batch (1 : N)

medicine.id ←── batch.medicine_id

Meaning:

- One medicine can have many batches  
- Batch BELONGS to exactly one medicine  
- FK lives in BATCH table  

### Real Life Meaning
Paracetamol can have:

- Batch B1  
- Batch B2  
- Batch B3  

---

## Batch → OrderItem (1 : N)

batch.id ←── order_items.batch_id

Meaning:

- One batch can serve many orders  
- Each order line comes from one batch  

---

# 5️⃣ DERIVED STOCK LOGIC – HEART OF SYSTEM

## Total Quantity Logic

Medicine.totalQuantity = SUM(batch.qtyAvailable)

Medicine never stores stock manually.

---

## InStock Logic

inStock = totalQuantity > 0

---

## StockStatus Logic

0        → "Out of stock" 1 – 49   → "Low stock" 50+      → "Available"

These fields reflect BATCH data.

---

# 6️⃣ FEFO – CORE BUSINESS ALGORITHM

### FEFO = First Expiry First Out

When order comes:

System MUST:

1. Ignore expired batches  
2. Sort batches by expiry ASC  
3. Deduct from earliest first  

---

## Real Deduction Example

### Before Order

| Batch | Expiry | Qty |
|------|--------|----|
| B1   | 2025   | 30 |
| B2   | 2026   | 70 |

### Customer buys 40

Process:

1. Take 30 from B1  
2. Take 10 from B2  

### After

| Batch | Qty |
|------|----|
| B1 | 0 |
| B2 | 60 |

---

# 7️⃣ ORDER FLOW INSIDE SYSTEM

When payment request comes:

## Step 1 – Validation

- Medicine exists?  
- Non expired batches exist?  
- Total qty enough?  

---

## Step 2 – FEFO Deduction

Loop through batches:

need = order quantity

for batch in sorted batches: if expired → skip

if batch.qty >= need: deduct need create OrderItem need = 0 break else: deduct batch.qty create OrderItem need -= batch.qty

If need > 0 → FAIL

---

## Step 3 – Update Medicine

Recalculate:

- totalQuantity  
- inStock  
- stockStatus  

---

## Step 4 – Save OrderItem

Each deduction creates:

OrderItem ├─ quantity ├─ priceAtPurchase └─ linked to Batch

---

# 8️⃣ TRANSACTION BEHAVIOR

Order process is:

✔ SINGLE TRANSACTION  
✔ ATOMIC  

If anything fails:

- no deduction  
- no order items  
- no partial update  

---

# 9️⃣ WHY EACH FIELD EXISTS

## priceAtPurchase

Because:
- price may change tomorrow  
- bill must keep old price  

---

## requiresRx

To block UI sale without prescription.

---

## sku

Business unique code for:

- barcode  
- search  
- duplication prevention  

---

# 🔟 EDGE CASES HANDLED

✔ Expired batch skip  
✔ Multiple batch deduction  
✔ Insufficient stock block  
✔ Negative qty protection  
✔ Concurrent safety  

---

# 1️⃣1️⃣ COMPLETE DATA FLOW VIEW

USER ORDER ↓ Service Layer ↓ Find Medicine ↓ Load Batches ↓ FEFO Algorithm ↓ Create OrderItems ↓ Update Batch Qty ↓ Recalculate Medicine ↓ Commit Transaction

---

# 1️⃣2️⃣ WHAT THIS PROJECT PROVES TECHNICALLY

✔ Real domain modeling  
✔ 1:N relationships  
✔ Derived fields  
✔ Transaction management  
✔ Business algorithm  
✔ Not simple CRUD  

---

# 1️⃣3️⃣ SUMMARY IN ONE STORY

- Medicine is just a product  
- Batch is real stock  
- OrderItem is sale history  
- Stock comes from batches  
- Deduction follows FEFO  
- Everything is transactional  

---
# 🧪 ULTRA DETAILED API DOCUMENTATION  
### Medicine Inventory & Order Management System

This section explains the APIs **line-by-line**:

- What EXACT data you must send  
- What EXACT data you will receive  
- Which table is affected  
- Which relation is touched  
- What happens internally  
- Common mistakes  

---

# 🌐 BASE INFORMATION

- **Base URL:** `http://localhost:8080`  
- **Swagger:** `/v3/api-docs`  
- **Content-Type:** `application/json`

Controllers:

1. medicine-controller  
2. batch-controller  
3. order-controller  

---

# 1️⃣ MEDICINE CONTROLLER – /medicines

## 1.1 GET /medicines

### Purpose
Fetch ALL medicines with:

- their batches  
- stock summary  
- derived fields  

### What DB Happens

SELECT * FROM medicines LEFT JOIN batches ON medicines.id = batches.medicine_id

### Response Structure

```json
[
 {
  "id": 1,
  "name": "Paracetamol",
  "category": "Analgesic",
  "price": 50.0,
  "sku": "MED-001",
  "requiresRx": false,

  "batches": [
     {
       "id": 10,
       "batchNo": "B1",
       "expiryDate": "2026-01-19",
       "qtyAvailable": 100,
       "orderItems": []
     }
  ],

  "inStock": true,
  "stockStatus": "Available",
  "totalQuantity": 100
 }
]

Field Meaning – Exact

Field	Comes From

id	medicines.id
batches	SELECT from batches table
totalQuantity	SUM(batches.qtyAvailable)
inStock	totalQuantity > 0
stockStatus	threshold logic



---

1.2 GET /medicines/{id}

You Send

/medicines/1

System Does

1. Find medicine by ID


2. Load batches where medicine_id = 1


3. For each batch load orderItems



Fails When

ID not exist → 404

DB error → 500



---

1.3 POST /medicines

What You MUST Send

{
 "name": "Paracetamol",
 "category": "Analgesic",
 "price": 50.0,
 "sku": "MED-001",
 "requiresRx": false,
 "batches": []
}

What You SHOULD NOT Send

❌ id
❌ totalQuantity
❌ inStock
❌ stockStatus

Because:

these are GENERATED by system


What Happens Internally

1. Insert into medicines table


2. No batch created


3. Derived fields default:



totalQuantity = 0
inStock = false
stockStatus = "Out of stock"

Response

Created medicine with ID.


---

1.4 PUT /medicines/{id}

Used For

Update name / price

Attach batches

Recalculate stock


You Send FULL OBJECT

{
 "id": 1,
 "name": "Paracetamol",
 "category": "Analgesic",
 "price": 55.0,
 "sku": "MED-001",
 "requiresRx": false,

 "batches": [
   {
     "id": 10,
     "batchNo": "B1",
     "expiryDate": "2026-01-19",
     "qtyAvailable": 100
   }
 ]
}

Internal Steps

1. Update medicines row


2. Link batch.medicine_id = 1


3. Recalculate totals




---

1.5 DELETE /medicines/{id}

Deletes

medicine row

MAY cascade batches (based on config)


Risk

If batches exist and no cascade → 409 Conflict


---

2️⃣ BATCH CONTROLLER – /batches

2.1 GET /batches

Returns raw batch table.

No medicine aggregation.


---

2.2 POST /batches

What You Send

{
 "batchNo": "BATCH-2026",
 "expiryDate": "2026-01-19",
 "qtyAvailable": 100,
 "orderItems": []
}

IMPORTANT

This batch is NOT linked to medicine yet
until:

PUT /medicines/{id}
OR

created with medicine relation in code


DB Effect

INSERT INTO batches


---

2.3 PUT /batches/{id}

Use For

change qty

correct expiry


After Update

Medicine totals must be recalculated.


---

2.4 DELETE /batches/{id}

Blocked if:

orderItems exist → to protect history



---

3️⃣ ORDER CONTROLLER – /api/orders/pay

3.1 POST /api/orders/pay

This is MOST COMPLEX API

You Send

[
 {
   "medicineId": 1,
   "quantity": 2,
   "priceAtPurchase": 50.0
 }
]

What Each Field Means

Field	Meaning

medicineId	which medicine buying
quantity	how many units
priceAtPurchase	current price



---

INTERNAL FLOW – LINE BY LINE

Step 1 – Validate Medicine

find medicine by id
if not exist → 404


---

Step 2 – Load Batches

SELECT * FROM batches
WHERE medicine_id = ?

Filter:

expiryDate >= today

qtyAvailable > 0


If none → 422 expired stock


---

Step 3 – FEFO Deduction

Algorithm:

1. Sort batches by expiry ASC


2. need = quantity



For each batch:

if expired → skip

if batch.qty >= need

deduct need

create OrderItem

break


else

deduct batch.qty

need -= batch.qty



If need > 0 →
❌ 422 Insufficient stock


---

Step 4 – Create OrderItems

For every deduction piece:

{
 "quantity": usedQty,
 "priceAtPurchase": 50,
 "batch_id": 10
}

INSERT INTO order_items


---

Step 5 – Update Batch

UPDATE batches
SET qty_available = newQty


---

Step 6 – Recalculate Medicine

totalQuantity = SUM(batches)
inStock = total > 0
stockStatus = rule


---

Step 7 – Commit

All in ONE transaction.


---

Response

Usually 200 OK with order summary.


---

4️⃣ VALIDATIONS EXACT

Medicine Create

❌ duplicate SKU → 409
❌ negative price → 400

Batch Create

❌ past expiry → 400
❌ negative qty → 400

Order

❌ expired only → 422
❌ not enough → 422


---

5️⃣ REAL EXAMPLES

Scenario A – Simple Order

Stock

Batch	Expiry	Qty

B1	2025	30
B2	2026	70


Request qty = 40

Result

B1 → 0

B2 → 60

2 OrderItems created



---

Scenario B – Expired Mix

| B1 | 2023 | 50 (expired) | | B2 | 2026 | 20 |

Buy 30 → FAIL 422


---

6️⃣ WHAT EACH API TOUCHES

POST /medicines

→ medicines table only

POST /batches

→ batches table only

PUT /medicines

→ medicines + batches

POST /pay

→ batches
→ order_items
→ medicines (derived)


---

7️⃣ COMMON MISTAKES

❌ sending totalQuantity in POST
❌ using expired batch
❌ not attaching batch to medicine
❌ partial order assumption


---

8️⃣ DEBUG CHECKLIST

If order fails:

1. check expiry


2. check totalQuantity


3. check batch.medicine_id


4. check transaction logs




---

9️⃣ INTERVIEW EXPLANATION OF API

This API shows:

FEFO deduction

transactional stock

1:N mapping

derived fields

real pharmacy logic



---

This is COMPLETE API UNDERSTANDING of your project.
## This is the REAL SOUL of your project.
