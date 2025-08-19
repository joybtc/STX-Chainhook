

# STX-Chainhook - Supply Chain Tracking Smart Contract

## Overview

This Clarity smart contract implements a **supply chain management system** on the Stacks blockchain. It ensures transparency, traceability, and accountability by recording the movement of products from **manufacturers → transporters → retailers** while maintaining an auditable history of changes.

The contract supports:

* Role-based access control (manufacturer, transporter, retailer).
* Secure product registration by manufacturers.
* Product lifecycle updates (location, status).
* Immutable history tracking for auditing.
* Role assignment and revocation by contract owner.

---

## ✨ Features

* **Role Management**

  * Assign and revoke roles (Manufacturer, Transporter, Retailer).
  * Enforce permissions for different supply chain operations.

* **Product Management**

  * Register products with metadata (name, origin, manufacturer, timestamp).
  * Update product location as it moves through the supply chain.
  * Track product status.

* **History Tracking**

  * Every product update is recorded in a history map.
  * Provides an auditable trail of product movement.

---

## ⚖️ Error Codes

| Code   | Meaning        |
| ------ | -------------- |
| `u100` | Not authorized |
| `u101` | Invalid role   |
| `u102` | Not found      |
| `u103` | Invalid input  |
| `u104` | Already exists |

---

## 🔑 Roles

The contract uses **role-based access control**:

* **Manufacturer (`"manufacturer"`)** – Can create products and update locations.
* **Transporter (`"transporter"`)** – Can update product location.
* **Retailer (`"retailer"`)** – Read-only role (can query data).

Only the **contract owner** can assign or revoke roles.

---

## 🗂️ Data Structures

### Roles Map

```clarity
roles: principal => { role: string, is-active: bool }
```

* Assigns a role to an address.

### Products Map

```clarity
products: uint => {
  id: uint,
  name: string,
  manufacturer: principal,
  origin: string,
  timestamp: uint,
  current-location: string,
  status: string
}
```

* Stores product details by ID.

### Product History Map

```clarity
product-history: {product-id: uint, change-id: uint} => {
  timestamp: uint,
  location: string,
  status: string
}
```

* Stores an immutable trail of changes for each product.

---

## ⚙️ Functions

### 🔒 Private Helpers

* `is-valid-role(role)` → Checks if role is manufacturer, transporter, or retailer.
* `is-valid-string-length(str, max-len)` → Ensures strings are within valid bounds.
* `validate-strings(name, origin, location)` → Ensures product metadata is valid.
* `is-contract-owner()` → Verifies if caller is contract owner.
* `check-role(address, role)` → Checks if an address holds a given role.
* `safe-get-role(address)` → Safely fetches a role (returns default if none).
* `safe-get-product(product-id)` → Safely fetches a product.

---

### 👥 Role Management

#### `assign-role(address, role)`

* Assigns a role (`manufacturer`, `transporter`, `retailer`).
* Only contract owner can call.
* Fails if role already active.

#### `revoke-role(address)`

* Revokes role from an address.
* Only contract owner can call.

---

### 📦 Product Management

#### `add-product(name, origin, location)`

* Creates a new product entry.
* Only manufacturers can call.
* Requires valid strings for name, origin, and location.
* Increments `product-counter`.

#### `update-location(product-id, new-location)`

* Updates product location.
* Allowed for **manufacturer** and **transporter**.
* Records change in `product-history`.
* Increments `change-counter`.

---

### 🔍 Read-Only Functions

#### `get-product(product-id)`

* Returns product details by ID.

#### `get-role(address)`

* Returns role and status of an address.

#### `get-product-history(product-id)`

* Returns the **first history entry** of a product.
* *(Can be extended to fetch full history)*.

---

## 🚀 Usage Flow

1. **Contract Owner** assigns roles:

   * Assign `manufacturer` to company A.
   * Assign `transporter` to company B.
   * Assign `retailer` to company C.

2. **Manufacturer** creates a product:

   ```clarity
   (contract-call? .supply-chain add-product "Laptop" "Nigeria" "Lagos")
   ```

3. **Transporter** updates product location:

   ```clarity
   (contract-call? .supply-chain update-location u0 "Port Harcourt")
   ```

4. **Retailer** queries product details:

   ```clarity
   (contract-call? .supply-chain get-product u0)
   ```

5. **Auditor** checks product history:

   ```clarity
   (contract-call? .supply-chain get-product-history u0)
   ```

---
