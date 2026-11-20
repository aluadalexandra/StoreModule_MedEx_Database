# STORE MODULE DATABASE
## PostgreSQL Database Structure and Connectivity

*Database Module:* store_module  
*Database Platform:* PostgreSQL (Supabase)

---

# 1. TABLE: merchant_profiles

## Table Structure

| Column Name | Data Type | Constraint |
|---|---|---|
| merchant_id | UUID | PRIMARY KEY |
| user_id | UUID | UNIQUE, FOREIGN KEY |
| shop_name | VARCHAR(150) | NOT NULL |
| shop_address_id | UUID | FOREIGN KEY |
| is_pharmacist | BOOLEAN | DEFAULT FALSE |
| pharmacist_name | VARCHAR(100) | NULL |
| pharmacist_licence_no | VARCHAR(50) | NULL |
| contact_number | VARCHAR(20) | NULL |
| is_24_hours | BOOLEAN | DEFAULT FALSE |
| is_temporarily_closed | BOOLEAN | DEFAULT FALSE |
| approval_status | VARCHAR(20) | NOT NULL |
| approved_by | UUID | FOREIGN KEY |
| approved_at | TIMESTAMPTZ | NULL |
| rejection_reason | TEXT | NULL |
| created_at | TIMESTAMPTZ | DEFAULT NOW() |
| updated_at | TIMESTAMPTZ | DEFAULT NOW() |

## Connectivity 1: user_id

- Foreign Key: user_id
- Reference Database: user_address
- Reference Table: users
- Reference Column: user_id (PK)
- Cardinality: One-to-One (1:1)
- This Table: One
- Reference Table: One
- Status: Connected in SQL

## Connectivity 2: shop_address_id

- Foreign Key: shop_address_id
- Reference Database: user_address
- Reference Table: addresses
- Reference Column: address_id (PK)
- Cardinality: Many-to-One (M:1)
- This Table: Many
- Reference Table: One
- Status: Connected in SQL

## Connectivity 3: approved_by

- Foreign Key: approved_by
- Reference Database: user_address
- Reference Table: users
- Reference Column: user_id (PK)
- Cardinality: Many-to-One (M:1)
- This Table: Many
- Reference Table: One
- Status: Connected in SQL

## SQL Code

CREATE TABLE store_module.merchant_profiles (
    merchant_id UUID PRIMARY KEY,

    user_id UUID UNIQUE
        REFERENCES user_address.users(user_id),

    shop_name VARCHAR(150) NOT NULL,

    shop_address_id UUID
        REFERENCES user_address.addresses(address_id),

    is_pharmacist BOOLEAN DEFAULT FALSE,
    pharmacist_name VARCHAR(100) NULL,
    pharmacist_licence_no VARCHAR(50) NULL,
    contact_number VARCHAR(20) NULL,
    is_24_hours BOOLEAN DEFAULT FALSE,
    is_temporarily_closed BOOLEAN DEFAULT FALSE,
    approval_status VARCHAR(20) NOT NULL,

    approved_by UUID
        REFERENCES user_address.users(user_id),

    approved_at TIMESTAMPTZ NULL,
    rejection_reason TEXT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

---

# 2. TABLE: merchant_hours

## Table Structure

| Column Name | Data Type | Constraint |
|---|---|---|
| merchant_id | UUID | PRIMARY KEY, FOREIGN KEY |
| day_of_week | SMALLINT | CHECK (0-6) |
| opens_at | TIME | NOT NULL |
| closes_at | TIME | NOT NULL |
| is_closed | BOOLEAN | DEFAULT FALSE |
| updated_at | TIMESTAMPTZ | DEFAULT NOW() |

## Connectivity

- Foreign Key: merchant_id
- Reference Database: store_module
- Reference Table: merchant_profiles
- Reference Column: merchant_id (PK)
- Cardinality: One-to-One (1:1)
- This Table: One
- Reference Table: One
- Status: Connected in SQL

## SQL Code

CREATE TABLE store_module.merchant_hours (
    merchant_id UUID PRIMARY KEY
        REFERENCES store_module.merchant_profiles(merchant_id),

    day_of_week SMALLINT
        CHECK (day_of_week BETWEEN 0 AND 6),

    opens_at TIME NOT NULL,
    closes_at TIME NOT NULL,
    is_closed BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

---

# 3. TABLE: product_categories

## Table Structure

| Column Name | Data Type | Constraint |
|---|---|---|
| category_id | INTEGER | GENERATED ALWAYS AS IDENTITY, PRIMARY KEY |
| name | VARCHAR(100) | UNIQUE, NOT NULL |
| is_active | BOOLEAN | DEFAULT TRUE |
| created_at | TIMESTAMPTZ | DEFAULT NOW() |

## Connectivity

- Referenced by: products.category_id
- Reference Database: store_module
- Reference Table: products
- Reference Column: category_id
- Cardinality: One-to-Many (1:M)
- This Table: One
- Reference Table: Many
- Status: Relationship defined in SQL

## SQL Code

CREATE TABLE store_module.product_categories (
    category_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    name VARCHAR(100) UNIQUE NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

---

# 4. TABLE: products

## Table Structure

| Column Name | Data Type | Constraint |
|---|---|---|
| product_id | UUID | PRIMARY KEY |
| merchant_id | UUID | FOREIGN KEY |
| category_id | INTEGER | FOREIGN KEY |
| name | VARCHAR(150) | NOT NULL |
| generic_name | VARCHAR(150) | NOT NULL |
| brand_name | VARCHAR(150) | NOT NULL |
| description | TEXT | NULL |
| unit | VARCHAR(50) | NULL |
| strength | VARCHAR(50) | NULL |
| dosage_form | VARCHAR(50) | NULL |
| image_path | TEXT | NULL |
| price | NUMERIC(10,2) | NOT NULL |
| requires_prescription | BOOLEAN | DEFAULT FALSE |
| is_active | BOOLEAN | DEFAULT FALSE |
| created_at | TIMESTAMPTZ | DEFAULT NOW() |
| updated_at | TIMESTAMPTZ | DEFAULT NOW() |

## Connectivity 1: merchant_id

- Foreign Key: merchant_id
- Reference Database: store_module
- Reference Table: merchant_profiles
- Reference Column: merchant_id (PK)
- Cardinality: Many-to-One (M:1)
- This Table: Many
- Reference Table: One
- Status: Connected in SQL

## Connectivity 2: category_id

- Foreign Key: category_id
- Reference Database: store_module
- Reference Table: product_categories
- Reference Column: category_id (PK)
- Cardinality: Many-to-One (M:1)
- This Table: Many
- Reference Table: One
- Status: Connected in SQL

## SQL Code

CREATE TABLE store_module.products (
    product_id UUID PRIMARY KEY,

    merchant_id UUID
        REFERENCES store_module.merchant_profiles(merchant_id),

    category_id INTEGER
        REFERENCES store_module.product_categories(category_id),

    name VARCHAR(150) NOT NULL,
    generic_name VARCHAR(150) NOT NULL,
    brand_name VARCHAR(150) NOT NULL,
    description TEXT NULL,
    unit VARCHAR(50) NULL,
    strength VARCHAR(50) NULL,
    dosage_form VARCHAR(50) NULL,
    image_path TEXT NULL,
    price NUMERIC(10,2) NOT NULL,
    requires_prescription BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

---

# 5. TABLE: inventory

## Table Structure

| Column Name | Data Type | Constraint |
|---|---|---|
| product_id | UUID | PRIMARY KEY, FOREIGN KEY |
| stock_quantity | INTEGER | DEFAULT 0, CHECK (>= 0) |
| reserved_quantity | INTEGER | NOT NULL, DEFAULT 0 |
| reorder_level | INTEGER | DEFAULT 10 |
| last_restocked_at | TIMESTAMPTZ | NULL |
| updated_at | TIMESTAMPTZ | DEFAULT NOW() |

## Connectivity

- Foreign Key: product_id
- Reference Database: store_module
- Reference Table: products
- Reference Column: product_id (PK)
- Cardinality: One-to-One (1:1)
- This Table: One
- Reference Table: One
- Status: Connected in SQL

## SQL Code

CREATE TABLE store_module.inventory (
    product_id UUID PRIMARY KEY
        REFERENCES store_module.products(product_id),

    stock_quantity INTEGER DEFAULT 0
        CHECK (stock_quantity >= 0),

    reserved_quantity INTEGER NOT NULL DEFAULT 0,
    reorder_level INTEGER DEFAULT 10,
    last_restocked_at TIMESTAMPTZ NULL,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

---

# 6. RELATIONSHIP SUMMARY

| Child Table | Foreign Key | Parent Table | Referenced Column | Cardinality |
|---|---|---|---|---|
| merchant_profiles | user_id | users | user_id | One-to-One |
| merchant_profiles | shop_address_id | addresses | address_id | Many-to-One |
| merchant_profiles | approved_by | users | user_id | Many-to-One |
| merchant_hours | merchant_id | merchant_profiles | merchant_id | One-to-One |
| products | merchant_id | merchant_profiles | merchant_id | Many-to-One |
| products | category_id | product_categories | category_id | Many-to-One |
| inventory | product_id | products | product_id | One-to-One |

---

# 7. DATABASE NOTES

1. The database contains five tables:
   - merchant_profiles
   - merchant_hours
   - product_categories
   - products
   - inventory

2. The user_id relationship connects merchant_profiles to user_address.users with One-to-One (1:1) cardinality.

3. The shop_address_id relationship connects merchant_profiles to user_address.addresses with Many-to-One (M:1) cardinality.

4. The approved_by relationship connects merchant_profiles to user_address.users with Many-to-One (M:1) cardinality.

5. The user_id column has both UNIQUE and FOREIGN KEY constraints.

6. The merchant_hours table has a One-to-One (1:1) relationship with merchant_profiles because merchant_id is both its Primary Key and Foreign Key.

7. The day_of_week column accepts values from 0 to 6 through its CHECK constraint.

8. The product_categories table uses GENERATED ALWAYS AS IDENTITY for automatic category_id generation.

9. The product_categories table has a One-to-Many (1:M) relationship with products.

10. The products table has Many-to-One (M:1) relationships with merchant_profiles and product_categories.

11. The inventory table has a One-to-One (1:1) relationship with products because product_id is both its Primary Key and Foreign Key.

12. The stock_quantity column cannot accept negative values because of its CHECK constraint.

13. The reserved_quantity column has a default value of 0.

14. The reorder_level column has a default value of 10.

15. The created_at and updated_at columns in all five tables use TIMESTAMPTZ.

16. The Foreign Key references to user_address require the referenced schemas, tables, and columns to exist in the same PostgreSQL database as store_module.

17. Cardinality describes the relationship between tables, while Foreign Key constraints enforce the actual database connection.

18. The merchant_hours and inventory tables currently allow only one record per merchant and product, respectively, because their Foreign Key columns are also their Primary Keys.

---

*END OF STORE MODULE DATABASE DOCUMENTATION*