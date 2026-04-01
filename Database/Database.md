

1. **Core design principles**
2. **Domain grouping (bounded contexts)**
3. **Full entity breakdown**
4. **Relationships (ERD logic)**
5. **Scaling + real-world considerations**

---

# 🧠 1. Database Design Principles (Critical)

Before tables:

### Must support:

* Multi-tenancy (`tenant_id` everywhere)
* Auditability (who did what)
* Extensibility (labs vary a lot)
* High write throughput (samples, results)

### Strategy:

* **PostgreSQL (primary relational DB)**
* Normalize core entities
* Denormalize for performance where needed

---

# 🧩 2. Domain-Based Schema Segmentation

We divide schema into logical modules:

1. Identity & Tenant
2. Patient & Orders
3. Sample Management
4. Test & Results
5. Reporting
6. Billing
7. Inventory
8. Audit & Logs

---

# 🧱 3. FULL ENTITY DESIGN

---

## 🔐 3.1 Identity & Tenant

### tenants

```sql
tenants (
  id UUID PK,
  name TEXT,
  plan_type TEXT,
  status TEXT,
  created_at TIMESTAMP
)
```

### users

```sql
users (
  id UUID PK,
  tenant_id UUID FK,
  name TEXT,
  email TEXT UNIQUE,
  phone TEXT,
  password_hash TEXT,
  status TEXT,
  created_at TIMESTAMP
)
```

### roles

```sql
roles (
  id UUID PK,
  tenant_id UUID,
  name TEXT
)
```

### user_roles

```sql
user_roles (
  user_id UUID FK,
  role_id UUID FK
)
```

### permissions

```sql
permissions (
  id UUID PK,
  name TEXT
)
```

### role_permissions

```sql
role_permissions (
  role_id UUID,
  permission_id UUID
)
```

---

## 🧾 3.2 Patient & Orders

### patients

```sql
patients (
  id UUID PK,
  tenant_id UUID,
  first_name TEXT,
  last_name TEXT,
  dob DATE,
  gender TEXT,
  phone TEXT,
  email TEXT,
  address TEXT,
  created_at TIMESTAMP
)
```

### doctors

```sql
doctors (
  id UUID PK,
  tenant_id UUID,
  name TEXT,
  specialization TEXT,
  contact TEXT
)
```

### orders

```sql
orders (
  id UUID PK,
  tenant_id UUID,
  patient_id UUID FK,
  doctor_id UUID FK,
  status TEXT,
  total_amount NUMERIC,
  created_at TIMESTAMP
)
```

### order_tests

```sql
order_tests (
  id UUID PK,
  order_id UUID FK,
  test_id UUID FK,
  status TEXT,
  price NUMERIC
)
```

---

## 🧪 3.3 Sample Management

### samples

```sql
samples (
  id UUID PK,
  tenant_id UUID,
  order_id UUID FK,
  barcode TEXT UNIQUE,
  sample_type TEXT,
  status TEXT,
  collected_at TIMESTAMP,
  received_at TIMESTAMP
)
```

### sample_tracking

```sql
sample_tracking (
  id UUID PK,
  sample_id UUID FK,
  status TEXT,
  location TEXT,
  updated_at TIMESTAMP,
  updated_by UUID
)
```

👉 This gives **chain-of-custody**

---

## 🔬 3.4 Test Definitions & Processing

### tests

```sql
tests (
  id UUID PK,
  tenant_id UUID,
  name TEXT,
  category TEXT,
  description TEXT
)
```

### test_parameters

```sql
test_parameters (
  id UUID PK,
  test_id UUID FK,
  name TEXT,
  unit TEXT,
  normal_range_min NUMERIC,
  normal_range_max NUMERIC
)
```

---

## 🧾 3.5 Results

### results

```sql
results (
  id UUID PK,
  order_test_id UUID FK,
  status TEXT,
  validated_by UUID,
  validated_at TIMESTAMP
)
```

### result_values

```sql
result_values (
  id UUID PK,
  result_id UUID FK,
  parameter_id UUID FK,
  value TEXT,
  flag TEXT, -- abnormal / critical
  machine_source TEXT
)
```

---

## 📊 3.6 Reporting

### reports

```sql
reports (
  id UUID PK,
  order_id UUID FK,
  report_url TEXT,
  status TEXT,
  generated_at TIMESTAMP,
  approved_by UUID
)
```

---

## 💰 3.7 Billing

### invoices

```sql
invoices (
  id UUID PK,
  order_id UUID FK,
  total_amount NUMERIC,
  discount NUMERIC,
  tax NUMERIC,
  final_amount NUMERIC,
  status TEXT
)
```

### payments

```sql
payments (
  id UUID PK,
  invoice_id UUID FK,
  method TEXT,
  amount NUMERIC,
  status TEXT,
  paid_at TIMESTAMP
)
```

---

## 📦 3.8 Inventory

### inventory_items

```sql
inventory_items (
  id UUID PK,
  tenant_id UUID,
  name TEXT,
  unit TEXT
)
```

### inventory_stock

```sql
inventory_stock (
  id UUID PK,
  item_id UUID FK,
  quantity NUMERIC,
  lot_number TEXT,
  expiry_date DATE
)
```

### inventory_transactions

```sql
inventory_transactions (
  id UUID PK,
  item_id UUID,
  change NUMERIC,
  type TEXT,
  created_at TIMESTAMP
)
```

---

## 📜 3.9 Audit Logs

### audit_logs

```sql
audit_logs (
  id UUID PK,
  tenant_id UUID,
  user_id UUID,
  action TEXT,
  entity_type TEXT,
  entity_id UUID,
  timestamp TIMESTAMP
)
```

---

# 🔗 4. RELATIONSHIPS (ERD LOGIC)

Core relationships:

* **Tenant → Users → Roles**
* **Patient → Orders → Order Tests**
* **Order → Samples → Tracking**
* **Order Test → Results → Result Values**
* **Order → Report**
* **Order → Invoice → Payment**

---

## 🔄 Data Flow Mapping

```
Patient → Order → Test → Sample → Result → Report → Invoice → Payment
```

---

# ⚡ 5. ADVANCED DESIGN (WHAT MAKES THIS ELITE)

---

## ✅ 5.1 Multi-Tenancy

Every major table:

```sql
tenant_id UUID
```

Add:

```sql
INDEX (tenant_id)
```

---

## ✅ 5.2 Soft Deletes

```sql
deleted_at TIMESTAMP
```

Never hard delete medical data.

---

## ✅ 5.3 Versioning (Important for Reports)

```sql
report_versions (
  id,
  report_id,
  version_number,
  data_snapshot JSONB
)
```

---

## ✅ 5.4 JSONB for Flexibility

Use in:

* result_values (raw machine data)
* workflows
* dynamic test configs

---

## ✅ 5.5 Indexing Strategy

* `patient_id`
* `order_id`
* `barcode`
* `tenant_id`
* `status`

---

## ✅ 5.6 Partitioning (for scale)

Partition:

* `orders`
* `results`
* `audit_logs`

By:

* tenant_id OR date

---

# 🧠 Final Insight

This schema gives you:

* Full traceability (legal requirement)
* Real-time operations
* Flexibility for different lab types
* Scalability to millions of records

---

# 👉 Next Step

We should now design one of these (your choice):

1. **API design (endpoints + contracts)**
2. **Workflow engine schema (VERY important differentiator)**
3. **Instrument integration data model**
4. **Event schema (Kafka topics design)**

Pick one, and we’ll go deep again.
