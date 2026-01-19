# 🏥 Medicine Inventory & Order Management System  
### Enterprise-Grade REST API – Complete Technical Documentation

This README is a **full knowledge document** of the project — not just endpoint listing.  
After reading this, any developer can:

✔ Understand pharmacy domain logic  
✔ Know how batches & expiry work  
✔ Understand FEFO deduction  
✔ Use all APIs correctly  
✔ Understand DB relationships  
✔ Debug errors  
✔ Explain project in interview  

---

# 1. DOMAIN THEORY — WHY THIS SYSTEM EXISTS

## 1.1 Real Pharmacy Stock Concept

Medicine stock is NOT stored as:

❌ Paracetamol = 100 units

Because medicines expire.

✔ Correct model:

| Batch | Expiry | Qty |
|------|--------|-----|
| B1 | 2025-06 | 30 |
| B2 | 2026-01 | 70 |

### If customer buys 40:

System must:

1. Use earliest expiry first  
2. Take 30 from B1  
3. Take 10 from B2  

👉 **FEFO – First Expiry First Out**

---

## 1.2 Core Business Rules

### Stock Rules
1. Expired batches → cannot be sold  
2. Medicine stock = SUM of batch.qtyAvailable  
3. Medicine fields are DERIVED:

```
totalQuantity = sum(all batches)

inStock = totalQuantity > 0

stockStatus =
  0        → "Out of stock"
  1–49     → "Low stock"
  ≥50      → "Available"
```

### Order Rules
- Order must be fully available  
- Partial deduction NOT allowed  
- Deduction must be atomic  
- priceAtPurchase must be stored  

---

# 2. SYSTEM ARCHITECTURE

```
Client (React / Postman)
        ↓
Controller Layer  – REST endpoints
        ↓
Service Layer – Business + FEFO
        ↓
Repository Layer – JPA
        ↓
MySQL Database
```

---

# 3. DOMAIN MODEL (From Swagger)

## 3.1 Entities Exactly As Per API

### 🟢 Medicine Schema

```json
{
 "id": 0,
 "name": "string",
 "category": "string",
 "price": 0,
 "sku": "string",
 "requiresRx": true,
 "batches": [Batch],
 "inStock": true,
 "stockStatus": "string",
 "totalQuantity": 0
}
```

### 🟡 Batch Schema

```json
{
 "id": 0,
 "batchNo": "string",
 "expiryDate": "2026-01-19",
 "qtyAvailable": 0,
 "orderItems": [OrderItem]
}
```

### 🔵 OrderItem Schema

```json
{
 "id": 0,
 "quantity": 0,
 "priceAtPurchase": 0
}
```

---

## 3.2 Relationships (Real Meaning)

### Medicine → Batch  (1 : N)

One medicine can have many batches because:

- different suppliers  
- different expiry dates  
- different arrivals  

### Batch → OrderItem (1 : N)

One batch can serve many orders.

---

# 4. DATABASE DESIGN

## 4.1 Tables

### medicines
| id | name | category | price | sku | requires_rx |

### batches
| id | batch_no | expiry_date | qty_available | medicine_id |

### order_items
| id | quantity | price_at_purchase | batch_id |

---

# 5. DERIVED FIELD LOGIC

### Stock Calculation

```java
totalQuantity =
  batches.stream()
         .mapToInt(Batch::getQtyAvailable)
         .sum();
```

### Status

```java
if(total == 0)
   "Out of stock"
else if(total < 50)
   "Low stock"
else
   "Available"
```

---

# 6. FEFO ALGORITHM — CORE LOGIC

### Steps

1. Get batches of medicine  
2. Remove expired  
3. Sort by expiry ASC  
4. Deduct sequentially  

### Pseudocode

```
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

if need > 0 → fail order
```

---

# 7. API DOCUMENTATION (From Swagger)

## BASE
- http://localhost:8080  
- /v3/api-docs  

---

## 7.1 MEDICINE CONTROLLER

### GET /medicines

**Response**

```json
[
 {
  "id": 1,
  "name": "string",
  "category": "string",
  "price": 0,
  "sku": "string",
  "requiresRx": true,
  "batches": [],
  "inStock": true,
  "stockStatus": "string",
  "totalQuantity": 0
 }
]
```

---

### GET /medicines/{id}

Returns single medicine with batches and orderItems.

---

### POST /medicines

**Request Body**

Same as Medicine schema.

Used to create new medicine.

---

### PUT /medicines/{id}

Used to:

- update details  
- attach batches  
- modify stock info  

---

### DELETE /medicines/{id}

Removes medicine.

---

## 7.2 BATCH CONTROLLER

### GET /batches
Returns all batches.

### GET /batches/{id}
Single batch with orderItems.

### POST /batches
Create batch.

### PUT /batches/{id}
Update batch qty / expiry.

### DELETE /batches/{id}

---

## 7.3 ORDER CONTROLLER

### POST /api/orders/pay

**Request (Swagger shows dynamic map)**

```json
[
 { "medicineId":1, "quantity":2, "priceAtPurchase":50 }
]
```

### Internal Flow

1. Validate medicine  
2. Check non-expired batches  
3. Run FEFO  
4. Create OrderItem  
5. Update totals  

---

# 8. ERROR HANDLING

### 400 – Bad Request
- invalid date  
- wrong JSON

### 409 – Conflict
- duplicate SKU

### 422 – Business Errors
- insufficient stock  
- only expired batches

---

# 9. TRANSACTION MANAGEMENT

Order payment is:

✔ SINGLE TRANSACTION

If any step fails:

❌ no stock deduction  
❌ no order items  
❌ rollback  

---

# 10. DESIGN DECISIONS

### Why priceAtPurchase?
Future price may change.

### Why batches?
Legal + expiry tracking.

### Why FEFO?
Medical safety.

---

# 11. EDGE CASES

- Multiple batches needed  
- Expired + valid mix  
- Concurrent orders  
- Negative qty guard  

---

# 12. INTERVIEW EXPLANATION

This project proves:

- JPA 1:N mapping  
- Derived fields  
- Transaction  
- Real domain modeling  
- Not CRUD toy  

---

# 13. HOW TO TEST

1. Create Medicine  
2. Create Batch  
3. Attach  
4. Call /api/orders/pay  

---

# END — COMPLETE PROJECT GUIDE
