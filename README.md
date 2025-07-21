

# STX-ClearSight: Supply Chain Transparency Smart Contract



## 🧭 Overview

**ClearSight** is a Clarity smart contract designed to bring **transparency**, **traceability**, and **trust** to supply chains by managing product lifecycle events with:

* Role-Based Access Control (RBAC)
* Status transitions and validation
* Historical status and ownership logs
* Audit trails
* Data integrity through input validation

It enables authorized manufacturers, distributors, and retailers to track products from registration to delivery and verification.

---

## ⚙️ Key Features

* ✅ **Role Management (RBAC):** Admins can assign specific roles (Manufacturer, Distributor, Retailer).
* 🛠️ **Product Registration & Updates:** Only manufacturers can register and update product statuses.
* 🔍 **Input Validation:** Product ID, location, and reason fields are validated for correctness and length.
* 🧾 **Status History:** Every status change is logged with timestamp, actor, reason, and location.
* 📘 **Audit Trail:** Immutable logs for all major actions (register, update).
* 🔄 **Status Transition Control:** Only valid transitions are allowed (e.g., `registered → in-transit`).

---

## 📐 Data Structures

### Roles (`user-roles`)

```clarity
{ user: principal } → { role: uint }
```

* ROLE-ADMIN: `u1`
* ROLE-MANUFACTURER: `u2`
* ROLE-DISTRIBUTOR: `u3`
* ROLE-RETAILER: `u4`

---

### Products (`products`)

```clarity
{ product-id: string } → {
  manufacturer: principal,
  timestamp: uint,
  current-owner: principal,
  current-status: string,
  verified: bool,
  status-update-count: uint
}
```

---

### Status History (`status-history`)

```clarity
{ product-id: string, update-number: uint } → {
  status: string,
  timestamp: uint,
  changed-by: principal,
  reason: string,
  location: string
}
```

---

### Product History (`product-history`)

(Currently unused but defined for future extensions)

```clarity
{ product-id: string, timestamp: uint } → {
  owner: principal,
  action: string,
  location: string,
  previous-owner: (optional principal)
}
```

---

### Audit Log (`audit-log`)

```clarity
{ transaction-id: uint, timestamp: uint } → {
  actor: principal,
  action: string,
  product-id: string,
  details: string
}
```

---

## 🚨 Error Codes

| Error Constant           | Code  | Description                         |
| ------------------------ | ----- | ----------------------------------- |
| `ERR-NOT-AUTHORIZED`     | `100` | Caller is not authorized for action |
| `ERR-PRODUCT-EXISTS`     | `101` | Product already registered          |
| `ERR-PRODUCT-NOT-FOUND`  | `102` | Product ID not found                |
| `ERR-INVALID-STATUS`     | `103` | Status provided is not valid        |
| `ERR-INVALID-TRANSITION` | `109` | Illegal status transition           |
| `ERR-INVALID-ROLE`       | `108` | Provided role is not allowed        |
| `ERR-INVALID-REASON`     | `110` | Reason string is invalid            |

---

## 📊 Valid Status Transitions

| From          | To                          |
| ------------- | --------------------------- |
| `registered`  | `in-transit`, `transferred` |
| `in-transit`  | `delivered`, `returned`     |
| `delivered`   | `verified`, `rejected`      |
| `transferred` | `verified`                  |

---

## 🛠️ How to Use

### 1. 👑 Assign Roles (Admin only)

```clarity
(assign-role user-role)
```

```clarity
(assign-role 'SP123... u2) ;; Assigns manufacturer role
```

---

### 2. 🏷️ Register a Product (Manufacturer only)

```clarity
(register-product product-id location)
```

```clarity
(register-product "product-001" "Lagos")
```

---

### 3. 🚚 Update Product Status (Manufacturer only)

```clarity
(update-product-status product-id new-status reason location)
```

```clarity
(update-product-status "product-001" "in-transit" "Shipped to warehouse" "Ibadan")
```

---

### 4. 📖 Get Product Details

```clarity
(get-product-details product-id)
```

```clarity
(get-product-details "product-001")
```

---

### 5. 📜 Get Status History

```clarity
(get-status-history product-id update-number)
```

```clarity
(get-status-history "product-001" u0)
```

---

## 🔐 Security Considerations

* All mutations are protected with RBAC.
* Input validations prevent blank or malformed strings.
* Unauthorized transitions are explicitly rejected.

---
