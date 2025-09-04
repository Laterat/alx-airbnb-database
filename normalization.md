# Database Normalization - Airbnb Database

## **Objective**

The goal of normalization is to reduce redundancy and ensure data integrity.  
We will apply the three normal forms (1NF, 2NF, and 3NF) to the given Airbnb database schema.  
The final design will be in **Third Normal Form (3NF)**.

---

## **1. First Normal Form (1NF)**

**Rule:**

- Each table should have a **primary key**.
- Data must be **atomic** (no repeating groups or arrays).
- No multi-valued attributes.

**Issues Identified:**

- The `User` table is fine as all attributes are atomic.
- The `Property` table has an attribute `location` which is a single string (not atomic enough, could include city, state, country, etc. in one field).
- The `Message` table is fine because `sender_id` and `recipient_id` are separate attributes.
- No multi-valued attributes in other tables.

**Solution:**

- Break `location` into separate attributes: `street`, `city`, `state`, `country`.

**Revised Property Table:**
| Column | Type |
|---------------|------------|
| property_id | UUID (PK) |
| host_id | UUID (FK → User.user_id) |
| name | VARCHAR |
| description | TEXT |
| street | VARCHAR |
| city | VARCHAR |
| state | VARCHAR |
| country | VARCHAR |
| price_per_night | DECIMAL |
| created_at | TIMESTAMP |
| updated_at | TIMESTAMP |

---

## **2. Second Normal Form (2NF)**

**Rule:**

- Table must be in **1NF**.
- **No partial dependency**: Non-key attributes must depend on the whole primary key.

**Analysis:**

- All tables have **single-column primary keys**, so no partial dependency exists.
- The schema remains in **2NF** as is.

---

## **3. Third Normal Form (3NF)**

**Rule:**

- Table must be in **2NF**.
- **No transitive dependency**: Non-key attributes must depend **only on the primary key**, not on other non-key attributes.

**Potential Issues Identified:**

1. In `Booking` table, `total_price` can be derived from `price_per_night * number_of_nights`.
   - Keeping it may introduce redundancy.
2. In `Payment` table, `payment_method` is currently an ENUM. If new methods are added, the database schema must be altered.
   - Solution: Create a separate `PaymentMethod` table and reference it.

---

### **Changes Made**

#### **Booking Table**

- Remove `total_price` to avoid redundancy.

| Column      | Type                                |
| ----------- | ----------------------------------- |
| booking_id  | UUID (PK)                           |
| property_id | UUID (FK → Property.property_id)    |
| user_id     | UUID (FK → User.user_id)            |
| start_date  | DATE                                |
| end_date    | DATE                                |
| status      | ENUM (pending, confirmed, canceled) |
| created_at  | TIMESTAMP                           |

---

#### **Payment Table**

- Replace `payment_method` ENUM with a **PaymentMethod** lookup table.

| Column       | Type                               |
| ------------ | ---------------------------------- |
| payment_id   | UUID (PK)                          |
| booking_id   | UUID (FK → Booking.booking_id)     |
| amount       | DECIMAL                            |
| payment_date | TIMESTAMP                          |
| method_id    | INT (FK → PaymentMethod.method_id) |

**PaymentMethod Table:**
| Column | Type |
|--------------|------------|
| method_id | INT (PK) |
| method_name | VARCHAR (credit_card, PayPal, Stripe, etc.) |

---

## **Final ERD Relationships**

- **User → Property**: 1-to-Many (`host_id`)
- **Property → Booking**: 1-to-Many
- **User → Booking**: 1-to-Many
- **Booking → Payment**: 1-to-Many
- **PaymentMethod → Payment**: 1-to-Many
- **Property → Review**: 1-to-Many
- **User → Review**: 1-to-Many
- **User → Message (Sender & Receiver)**: Many-to-Many
- **Property → Location**: Implicit through normalized address fields.

---

## **Summary of Normalization**

| Step | Change Made                                                | Reason                                                 |
| ---- | ---------------------------------------------------------- | ------------------------------------------------------ |
| 1NF  | Split `location` into `street`, `city`, `state`, `country` | Ensure atomic data                                     |
| 2NF  | No changes needed                                          | No composite keys present                              |
| 3NF  | Removed `total_price` from `Booking`                       | Avoid redundancy, can be calculated dynamically        |
| 3NF  | Added `PaymentMethod` table                                | Prevent schema changes when adding new payment methods |

---

## **Final Tables**

1. **User**
2. **Property** (with normalized location fields)
3. **Booking** (no total_price)
4. **Payment**
5. **PaymentMethod**
6. **Review**
7. **Message**

The database now fully satisfies **Third Normal Form (3NF)**.

---
