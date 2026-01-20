
# 🛢️ PetroManage — Asset Registration & Lifecycle Management Module  
### Oil & Gas Asset & Operations Management System

---

## 📘 1. Overview

The **Asset Registration & Lifecycle Management Module** is the core foundation of the **PetroManage** platform.  
It enables oil & gas organizations to efficiently **register**, **track**, and **manage** operational assets such as rigs, pipelines, and storage facilities.

The module provides:
- Standardized **asset registration**
- Complete **lifecycle tracking**
- Advanced **filtering & search**
- **DTO-driven REST APIs**
- **MySQL-backed persistence** using JPA/Hibernate

---

## 📋 2. TODO

- [x] Implement CRUD operations for Asset entity  
- [x] Connect to MySQL database  
- [x] DTO request/response structure  
- [ ] Add Swagger/OpenAPI documentation  
- [ ] Add global exception handling  
- [ ] Add unit tests (Controller/Service)  
- [ ] Add lifecycle rule validations  

---

## ⭐ 3. Features

### 🔹 Asset Registration
- Register assets with:
  - **name**
  - **type** (`RIG`, `PIPELINE`, `STORAGE`)
  - **location**
  - **status**

### 🔹 Lifecycle Status Tracking
Supported lifecycle statuses:
- `REGISTERED`
- `OPERATIONAL`
- `MAINTENANCE`
- `UNDER_INSPECTION`
- `DECOMMISSIONED`

### 🔹 Search & Filtering
Retrieve assets by:
- Type  
- Status  
- Location  

### 🔹 Additional Features
- MySQL persistent storage  
- CRUD + partial update  
- Layered architecture  

---

## 🏗️ 4. Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | Java 17+, Spring Boot |
| Database | MySQL |
| ORM | Hibernate / JPA |
| Build Tool | Maven |
| Documentation | Swagger (optional) |
| Testing | JUnit 5 (optional) |

---

## ⚙️ 5. Prerequisites

- Java **17+**  
- MySQL installed (local or cloud)  
- Maven **3.8+**  
- IDE (IntelliJ / Eclipse / VS Code)  

---

## 🚀 6. Getting Started

### 6.1 Clone the Repository
```bash
git clone <your-repository-url>
cd petromanage-asset-module
```
## 6.2 Configure MySQL
Edit application.properties:

```bash
  spring.datasource.url=jdbc:mysql://localhost:3306/petromanage
  spring.datasource.username=root
  spring.datasource.password=admin

  spring.jpa.hibernate.ddl-auto=update
  spring.jpa.show-sql=true
```
## 6.3 Run the Application
```bash
    mvn spring-boot:run
```
Server starts at:
  ```bash
  http://localhost:8080
  ```
---

## 🌐 7. API Endpoints
  ### ➤ Base URL
  ```
    /api/assets
  ```
### 🔹 7.1 Asset CRUD & Status APIs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/assets` | Get all assets (supports filters) |
| GET | `/api/assets/{id}` | Get asset by ID |
| POST | `/api/assets` | Create a new asset |
| PUT | `/api/assets/{id}` | Update full asset details |
| PATCH | `/api/assets/{id}/status` | Update only asset status |
| DELETE | `/api/assets/{id}` | Delete asset |

---

## 📦 8. Data Model: Asset Entity

| Field | Type | Description |
|--------|------|-------------|
| assetId | Long | Primary key |
| name | String | Asset name |
| type | Enum | RIG, PIPELINE, STORAGE |
| location | String | Asset location |
| status | Enum | Lifecycle status |
| createdAt | Timestamp | Auto-created |
| updatedAt | Timestamp | Auto-updated |

### Notes:
- Use `@Enumerated(EnumType.STRING)`
- Use `@CreationTimestamp` / `@UpdateTimestamp` for auditing
- Suggested unique constraint: `(name, location)`

---

## 📮 9. Sample API Requests

### 🟦 Create Asset (POST)
```json
{
  "name": "Rig-Alpha",
  "type": "RIG",
  "location": "Abu Dhabi",
  "status": "REGISTERED"
}
```

### UPDATE Asset (PUT)
```json
{
  "name": "Rig-Alpha Updated",
  "type": "RIG",
  "location": "Abu Dhabi - Zone A",
  "status": "OPERATIONAL"
}
```

---

## 🧱 10. Architecture Overview

### 🔹 Controller Layer
Handles all HTTP requests.

**Class:**  
`com.petromanage.asset.controller.AssetController`

---

### 🔹 Service Layer
Contains business logic & lifecycle rules.

**Classes:**  
- `AssetService`  
- `AssetServiceImpl`

---

### 🔹 Repository Layer
Interacts with MySQL database using JPA.

**Class:**  
`AssetRepository`

---

### 🔹 DTO Layer
Defines API request & response models.

**Files:**  
- `AssetRequestDTO`  
- `AssetResponseDTO`

---

### 🔹 Config Layer
Handles CORS configuration.

**Class:**  
`com.petromanage.config.CorsConfig`


---

## 📘 11. Swagger / API Documentation

To enable Swagger UI, add:

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.5.0</version>
</dependency>
```
### Swagger UI URL:

```bash
http://localhost:8080/swagger-ui

```

---


##  📌 12. Summary

The Asset Registration & Lifecycle Management Module is the backbone of PetroManage, enabling:
✔ Accurate asset tracking
✔ Reliable lifecycle management
✔ Clean and scalable architecture
✔ Easy integration via REST APIs

This module forms the base for future PetroManage features such as:

Maintenance scheduling
Compliance tracking
Real-time analytics
