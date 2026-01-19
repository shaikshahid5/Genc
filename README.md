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

## This is the REAL SOUL of your project.
