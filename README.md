# 🏥 Medicine Inventory & Order Management System  
### Enterprise-Grade REST API Documentation

This document is a **complete knowledge base** of the system.  
After reading this you will understand:

✔ Business logic behind pharmacy systems  
✔ How batches & expiry work  
✔ How orders consume stock  
✔ Exact database design  
✔ API contracts  
✔ Internal algorithms  
✔ Failure scenarios  

---

# 1. BUSINESS THEORY — BEFORE CODING

## 1.1 How Real Pharmacies Work

Medicine stock is NOT a single number.

❌ Wrong model  
Paracetamol → 100 units

✔ Real model  
Paracetamol arrives in BATCHES:

| Batch | Expiry | Qty |
|------|--------|-----|
| B1 | 2025-06 | 30 |
| B2 | 2026-01 | 70 |

### When customer buys 40:

System MUST:

1. Use earliest expiry first  
2. Deduct 30 from B1  
3. Deduct 10 from B2  

👉 This is called **FEFO – First Expiry First Out**

---

## 1.2 Core Business Requirements

### Stock Rules

1. Expired batches → CANNOT be sold  
2. Total stock = SUM of all batch quantities  
3. Medicine status is DERIVED:

```
if total = 0      → Out of stock
if total 1–49     → Low stock
if total ≥ 50     → Available
```

### Order Rules

- Order must be FULLY available  
- Partial deduction not allowed  
- Price at purchase must be stored  
- Stock update must be ATOMIC  

---

# 2. SYSTEM ARCHITECTURE

```
Client (React/Postman)
        ↓
Controller Layer
        ↓
Service Layer (Business + FEFO)
        ↓
Repository Layer
        ↓
MySQL Database
```

---

# 3. DOMAIN MODEL — HEART OF SYSTEM

## 3.1 Entities in Simple English

### 🟢 Medicine = PRODUCT

- Basic info  
- List of batches  
- Calculated stock  

### 🟡 Batch = PHYSICAL STOCK UNIT

- Expiry  
- Quantity  
- Linked to one medicine  

### 🔵 OrderItem = CONSUMPTION RECORD

- How much sold  
- At what price  
- From which batch  

---

## 3.2 Java Model (Conceptual)

### Medicine

```java
class Medicine {
 Long id;
 String name;
 String category;
 double price;
 String sku;
 boolean requiresRx;

 List<Batch> batches;

 // Derived
 boolean inStock;
 String stockStatus;
 int totalQuantity;
}
```

### Batch

```java
class Batch {
 Long id;
 String batchNo;
 LocalDate expiryDate;
 int qtyAvailable;

 List<OrderItem> orderItems;
}
```

### OrderItem

```java
class OrderItem {
 Long id;
 int quantity;
 double priceAtPurchase;
}
```

---

# 4. DATABASE DESIGN

## 4.1 Tables

### medicines

| id | name | sku | price | requires_rx |

### batches

| id | batch_no | expiry_date | qty | medicine_id |

### order_items

| id | qty | price | batch_id |

---

## 4.2 Relationships

### 1 Medicine → Many Batches

```
Medicine (1)
   |
   |---- Batch A
   |---- Batch B
```

### 1 Batch → Many OrderItems

```
Batch B1
   |
   |--- OrderItem 1
   |--- OrderItem 2
```

---

# 5. DERIVED FIELD LOGIC

## 5.1 Stock Calculation

```java
totalQuantity =
  batches.stream()
         .mapToInt(Batch::getQtyAvailable)
         .sum();
```

## 5.2 Status Logic

```java
if(total == 0)
   status = "Out of stock";
else if(total < 50)
   status = "Low stock";
else
   status = "Available";
```

---

# 6. FEFO ALGORITHM (MOST IMPORTANT)

## 6.1 Flow

1. Get all batches of medicine  
2. Remove expired  
3. Sort by expiry ASC  
4. Deduct sequentially  

## 6.2 Pseudocode

```text
need = order qty

for batch in batches sorted by expiry:

  if batch.expired → skip

  if batch.qty >= need:
       deduct need
       need = 0
       break
  else:
       deduct batch.qty
       need -= batch.qty
```

If need > 0 → FAIL ORDER

---

# 7. API SPECIFICATION

## BASE
- http://localhost:8080  
- /v3/api-docs

---

## 7.1 MEDICINE CONTROLLER

### GET /medicines

**Response**

```json
{
 "id":1,
 "name":"Paracetamol",
 "totalQuantity":100,
 "stockStatus":"Available",
 "batches":[]
}
```

---

### POST /medicines

**Request**

```json
{
 "name":"Dolo",
 "category":"Fever",
 "price":20,
 "sku":"MED-1",
 "requiresRx":false
}
```

---

## 7.2 BATCH CONTROLLER

### POST /batches

```json
{
 "batchNo":"B2026",
 "expiryDate":"2026-10-01",
 "qtyAvailable":100
}
```

---

## 7.3 ORDER CONTROLLER

### POST /api/orders/pay

```json
[
 {
  "medicineId":1,
  "quantity":2,
  "priceAtPurchase":50
 }
]
```

### Internal Steps

1. Validate medicine  
2. Check non-expired batches  
3. Run FEFO deduction  
4. Create OrderItem  
5. Update totals  

---

# 8. ERROR SCENARIOS

### 422 – Insufficient stock
```
"Only 10 units available"
```

### 422 – All batches expired
```
"No valid batches"
```

### 409 – Duplicate SKU

### 400 – Bad date format

---

# 9. TRANSACTION BEHAVIOR

Order process is:

✔ SINGLE TRANSACTION

If any step fails:

❌ No deduction  
❌ No order items  
❌ Data unchanged  

---

# 10. DESIGN DECISIONS EXPLAINED

### Why priceAtPurchase?

Price may change tomorrow  
Invoice must keep old price

### Why not simple stock column?

Because:
- expiry  
- legal tracking  
- recalls  

### Why FEFO not FIFO?

Medical safety requirement

---

# 11. EDGE CASES HANDLED

- Multiple batches needed  
- Expired + valid mix  
- Concurrent orders  
- Zero stock race  
- Negative qty protection  

---

# 12. EXTENSIONS POSSIBLE

- Prescription module  
- GST billing  
- Supplier tracking  
- Returns  
- Audit logs  

---

# 13. LEARNING OUTCOME

This project demonstrates:

- JPA relationships  
- Derived fields  
- Transactions  
- Business validation  
- Real domain modeling  
- Not toy CRUD

---
