Here’s a fully polished **README.md** version of your Medicine Inventory API guide. I’ve expanded the formatting, added structure, and enriched details so it feels like a professional developer-facing document.  

---

```markdown
# Medicine Inventory API v0

A complete developer-focused guide to the **Medicine Inventory API**.  
This document explains every endpoint, schema, and relationship in depth—so by reading it, you can understand how the system behaves, how data flows, and how to use it correctly in real-world scenarios.

---

## 📌 Overview

- **Base URL:** `http://localhost:8080`
- **OpenAPI docs:** `/v3/api-docs`
- **Version:** v0

### Controllers
- **medicine-controller:** CRUD for medicines, stock aggregation, and batch linkage
- **batch-controller:** CRUD for batches and batch-level inventory
- **order-controller:** Payment processing for orders

---

## 🧩 Domain Model & Relationships

The API models a pharmacy inventory system with three core entities: **Medicine**, **Batch**, and **OrderItem**.

### Entities

#### Medicine
- `id` (int64): Unique identifier  
- `name` (string): Medicine name  
- `category` (string): Category (e.g., Analgesic, Antibiotic)  
- `price` (double): Unit price  
- `sku` (string): Stock keeping unit  
- `requiresRx` (boolean): Whether prescription is required  
- `batches` (array of Batch): Associated batches  
- `inStock` (boolean): Availability flag  
- `stockStatus` (string): Human-readable stock status  
- `totalQuantity` (int32): Sum of quantities across all batches  

#### Batch
- `id` (int64): Unique identifier  
- `batchNo` (string): Batch number  
- `expiryDate` (date `YYYY-MM-DD`): Expiry date  
- `qtyAvailable` (int32): Quantity available in this batch  
- `orderItems` (array of OrderItem): Fulfilled order items  

#### OrderItem
- `id` (int64): Unique identifier  
- `quantity` (int32): Quantity purchased  
- `priceAtPurchase` (double): Price at time of purchase  

---

### Relationships

- **Medicine → Batch (1:N):**  
  One medicine can have many batches. Stock is distributed across batches.  

- **Batch → OrderItem (1:N):**  
  One batch can fulfill many order items.  

- **Derived fields on Medicine:**  
  - `totalQuantity = sum(qtyAvailable)`  
  - `inStock = totalQuantity > 0`  
  - `stockStatus = "Available" | "Low stock" | "Out of stock"`  

---

### Conceptual Diagram

```
Medicine (id, name, category, price, sku, requiresRx, inStock, stockStatus, totalQuantity)
  └── batches: [Batch]

Batch (id, batchNo, expiryDate, qtyAvailable)
  └── orderItems: [OrderItem]

OrderItem (id, quantity, priceAtPurchase)
```

---

## 🚀 API Endpoints

### Medicine Controller

#### `GET /medicines`
Retrieve all medicines with batches and stock summary.  
**Response:** Array of Medicine objects.

#### `GET /medicines/{id}`
Retrieve a single medicine by ID.  
**Response:** Medicine object with nested batches.

#### `POST /medicines`
Create a new medicine.  
**Request body:** Medicine object (batches optional).  
**Response:** Created Medicine object.

#### `PUT /medicines/{id}`
Update an existing medicine by ID.  
**Request body:** Full Medicine object.  
**Response:** Updated Medicine object.

#### `DELETE /medicines/{id}`
Delete a medicine by ID.  
**Response:** `200 OK`  

---

### Batch Controller

#### `GET /batches`
Retrieve all batches.  
**Response:** Array of Batch objects.

#### `GET /batches/{id}`
Retrieve a single batch by ID.  
**Response:** Batch object.

#### `POST /batches`
Create a new batch.  
**Request body:** Batch object.  
**Response:** Created Batch object.

#### `PUT /batches/{id}`
Update an existing batch by ID.  
**Response:** Updated Batch object.

#### `DELETE /batches/{id}`
Delete a batch by ID.  
**Response:** `200 OK`  

---

### Order Controller

#### `POST /api/orders/pay`
Process payment for an order.  
**Request body:** Array of order items.  
**Response:** `200 OK`  

**Server responsibilities:**
- Validate stock availability  
- Deduct quantities from batches (FEFO: first-expiry-first-out)  
- Create `OrderItem` entries under batches  
- Recalculate `Medicine.totalQuantity`, `inStock`, and `stockStatus`  

---

## 📊 Stock & Status Behavior

- **Aggregation:**  
  - `totalQuantity = sum(qtyAvailable)`  
  - `inStock = totalQuantity > 0`  
  - `stockStatus` thresholds:  
    - `>= 50`: Available  
    - `1–49`: Low stock  
    - `0`: Out of stock  

- **Expiry Handling:**  
  - Do not fulfill orders from expired batches  
  - Deduct from earliest expiry first (FEFO)  

- **Atomicity:**  
  - All-or-nothing order processing  

---

## ⚠️ Validation & Error Handling

- **400 Bad Request:** Invalid payload, malformed date  
- **404 Not Found:** Medicine or batch not found  
- **409 Conflict:** Duplicate SKU or conflicting resource  
- **422 Unprocessable Entity:** Insufficient stock, expired batch used  
- **500 Internal Server Error:** Unexpected server failure  

---

## 🛠️ Practical Workflow Example

### Add a medicine, add a batch, then place an order

1. **Create medicine**
```bash
curl -X POST "http://localhost:8080/medicines" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Paracetamol",
    "category": "Analgesic",
    "price": 50.0,
    "sku": "MED-001",
    "requiresRx": false,
    "batches": [],
    "inStock": false,
    "stockStatus": "Not available",
    "totalQuantity": 0
  }'
```

2. **Create batch**
```bash
curl -X POST "http://localhost:8080/batches" \
  -H "Content-Type: application/json" \
  -d '{
    "batchNo": "BATCH-2026",
    "expiryDate": "2026-01-19",
    "qtyAvailable": 100,
    "orderItems": []
  }'
```

3. **Associate batch with medicine**
```bash
curl -X PUT "http://localhost:8080/medicines/1" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "name": "Paracetamol",
    "category": "Analgesic",
    "price": 50.0,
    "sku": "MED-001",
    "requiresRx": false,
    "batches": [
      {
        "id": 10,
        "batchNo": "BATCH-2026",
        "expiryDate": "2026-01-19",
        "qtyAvailable": 100,
        "orderItems": []
      }
    ],
    "inStock": true,
    "stockStatus": "Available",
    "totalQuantity": 100
  }'
```

4. **Place order**
```bash
curl -X POST "http://localhost:8080/api/orders/pay" \
  -H "Content-Type: application/json" \
  -d '[{
    "medicineId": 1,
    "quantity": 2,
    "priceAtPurchase": 50.0
  }]'
```

---

## 📖 Schema Reference

### Medicine
- `id`: int64  
- `name`: string  
- `category`: string  
- `price`: double  
- `sku`: string  
- `requiresRx`: boolean  
- `batches`: array of Batch  
- `inStock`: boolean  
- `stockStatus`: string  
- `totalQuantity`: int32  

### Batch
- `id`: int64  
- `batchNo`: string  
- `expiryDate`: date  
- `qtyAvailable`: int32  
- `orderItems`: array of OrderItem  

### OrderItem
- `id`: int64  
- `quantity`: int32  
- `priceAtPurchase`: double  

---

## 📝 Common Implementation Decisions

- **Batch association:** Decide whether batches are created with a `medicineId` or attached via `PUT /medicines/{id}`.  
- **Stock thresholds:** Define clear thresholds for `stockStatus`.  
- **Order payload:** Replace placeholder schema with `{ "medicineId": 1, "quantity": 2, "priceAtPurchase": 50.0 }`.  
- **Deletion behavior:** Decide whether deleting a medicine cascades to batches or is blocked.  

---

## ❌ Error Examples

- **Insufficient stock:**  
  `422 Unprocessable Entity` → `"Insufficient stock for medicine MED-001"`

- **Expired batch attempt:**  
  `422 Unprocessable Entity` → `"No non-expired batches available"`

- **Invalid date:**  
  `400 Bad Request` → `"Invalid date format for expiryDate"`

---

## 📜 Change Log

- **v0:** Initial endpoints for medicines, batches
