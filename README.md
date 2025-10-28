# 🐍 Oracle Database with Python — `python-oracledb` Complete Guide

---

## 🌟 Overview

![python-oracledb-arch](https://github.com/user-attachments/assets/ba461622-2514-4f1d-bcea-6ab6d063c498)

`python-oracledb` is Oracle’s **official open-source driver for Python**, enabling Python applications to connect seamlessly to **Oracle Database** for SQL, PL/SQL, and advanced data operations.

* ✅ Successor to `cx_Oracle` (fully compatible)
* ✅ Supports both **Thin** (no client needed) and **Thick** (client library) modes
* ✅ Implements the **Python DB API 2.0** standard
* ✅ Built for performance, scalability, and modern async use

---

## ⚙️ Architecture & Modes

`python-oracledb` operates in **two modes** — each suited for different environments.

### **Thin Mode (Default)**

* Pure Python (no Oracle Client installation)
* Simple deployment and lightweight
* Ideal for containers, cloud apps, and quick scripts
* Works with Oracle DB **12.1 and later**
* Limited access to advanced Oracle features

### **Thick Mode (Optional)**

* Uses Oracle Client libraries (like **Instant Client**)
* Enables full Oracle Database feature set
* Supports older Oracle versions (as far back as 9.2)
* Reads Oracle network files (`tnsnames.ora`, `sqlnet.ora`, `oraaccess.xml`)

```python
import oracledb
oracledb.init_oracle_client(lib_dir="/opt/oracle/instantclient_21_12")
```

---

## 🧩 Installation

```bash
pip install oracledb
```

* Works on Windows, macOS, and Linux
* Python 3.7+ required
* For Thick mode: install **Oracle Instant Client** separately

---

## 🔌 Supported Versions

| Component     | Supported Versions    |
| ------------- | --------------------- |
| **Python**    | 3.7 – 3.14            |
| **Oracle DB** | 11.2 – 23c            |
| **Modes**     | Thin / Thick          |
| **OS**        | Windows, Linux, macOS |

---

## 🔑 Connecting to Oracle Database

### Basic Connection

```python
import oracledb

conn = oracledb.connect(
    user="hr",
    password="hr",
    dsn="localhost/orclpdb1"
)
```

### Using Context Managers (Recommended)

```python
with oracledb.connect(user="hr", password="hr", dsn="localhost/orclpdb1") as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT first_name, last_name FROM employees")
        for row in cur:
            print(row)
```

🪄 Context managers automatically clean up connections and cursors.

---

## 🧵 Connection Pooling

Connection pools boost performance by reusing connections instead of recreating them.

```python
pool = oracledb.create_pool(
    user="hr",
    password="hr",
    dsn="localhost/orclpdb1",
    min=2,
    max=10,
    increment=1
)

conn = pool.acquire()
cur = conn.cursor()
cur.execute("SELECT COUNT(*) FROM employees")
print(cur.fetchone())
pool.release(conn)
```

💡 **Tip:** Always reuse connections in web or multi-threaded apps.

---

## 🧮 Executing SQL and PL/SQL

### Querying Data

```python
cur.execute(
    "SELECT employee_id, first_name FROM employees WHERE department_id = :dept",
    dept=50
)
for emp_id, fname in cur:
    print(emp_id, fname)
```

### Inserting Data

```python
cur.execute(
    "INSERT INTO employees (employee_id, first_name, last_name) VALUES (:1, :2, :3)",
    (300, "John", "Doe")
)
conn.commit()
```

### Bulk Inserts

```python
rows = [
    (301, "Alice", "Brown"),
    (302, "Bob", "Green"),
]
cur.executemany(
    "INSERT INTO employees (employee_id, first_name, last_name) VALUES (:1, :2, :3)",
    rows
)
conn.commit()
```

### Calling PL/SQL Procedures

```python
cur.callproc("update_salary", [101, 5000])
```

---

## 📦 Working with JSON & SODA

Oracle’s **Simple Oracle Document Access (SODA)** API enables document-style access to JSON data.

### JSON Query

```python
cur.execute("SELECT data FROM json_table WHERE data.name = :name", name="Alice")
for row in cur:
    print(row[0])  # Returns a Python dict
```

### SODA Example

```python
db = conn.db
collection = db.create_collection("mydocs")
collection.insert_one({"id": "1", "name": "test", "value": 100})

result = collection.find().filter({"value": {"$gt": 50}}).get_one()
print(result)
```

---

## ⚡ Advanced Features

| Feature                                 | Description                                |
| --------------------------------------- | ------------------------------------------ |
| **Array Binding**                       | Insert/update multiple rows efficiently    |
| **LOB Support**                         | Handle CLOB, BLOB, and NCLOB data          |
| **Scrollable Cursors**                  | Move backward and forward through results  |
| **Implicit Results**                    | Retrieve multiple result sets from PL/SQL  |
| **Continuous Query Notification (CQN)** | Receive notifications when data changes    |
| **Advanced Queuing (AQ)**               | Use Oracle message queues                  |
| **Session Tagging**                     | Manage pooled session state                |
| **Client Result Caching**               | Cache query results on client side         |
| **SODA**                                | Work with JSON documents directly          |
| **High Availability**                   | Support for Application Continuity and FAN |
| **Async I/O**                           | Run async queries for high concurrency     |

---

## ⚙️ Asynchronous Example

```python
import asyncio
import oracledb

async def main():
    async with await oracledb.connect_async(
        user="hr", password="hr", dsn="localhost/orclpdb1"
    ) as conn:
        async with conn.cursor() as cur:
            await cur.execute("SELECT first_name FROM employees")
            async for row in cur:
                print(row)

asyncio.run(main())
```

---

## 🔄 Migrating from `cx_Oracle`

| Change                      | Description                                                    |
| --------------------------- | -------------------------------------------------------------- |
| **Keyword-only parameters** | `connect()` and `create_pool()` no longer take positional args |
| **Removed parameters**      | `encoding`, `nencoding`, `threaded` removed                    |
| **Default pool behavior**   | Now waits for free connections instead of raising errors       |
| **JSON columns**            | Fetched as native Python dicts/lists                           |
| **Error codes**             | New standardized `DPY-` prefix                                 |
| **Renamed classes**         | `SessionPool` → `ConnectionPool`                               |

🧠 Migration is straightforward — most `cx_Oracle` scripts need only small adjustments.

---

## 🚨 Error Handling

```python
try:
    cur.execute("SELECT * FROM invalid_table")
except oracledb.DatabaseError as e:
    print("Database error:", e)
```

### Common Exceptions

* `DatabaseError`
* `InterfaceError`
* `OperationalError`
* `ProgrammingError`
* `IntegrityError`

---

## 🧠 Performance Optimization Tips

1. ✅ Use **connection pooling**
2. ✅ Batch inserts with **executemany()**
3. ✅ Commit in **controlled batches**
4. ✅ Use **fetchmany()** for large datasets
5. ✅ Tune **statement caching** and **fetch size**
6. ✅ Use **context managers** for auto-cleanup
7. ✅ Handle **timeouts** and **dead connections** gracefully
8. ✅ Use **Thin mode** unless advanced features are required
9. ✅ Avoid unnecessary conversions
10. ✅ Keep **Instant Client** up to date if using Thick mode

---

## ⚠️ Limitations

* Thin mode does not support every Oracle feature (e.g., DRCP, advanced encryption)
* Must call `init_oracle_client()` before any connection in Thick mode
* Some features (AQ, SODA) require specific database versions
* Large LOB operations may require streaming techniques

---

## 💡 Quick Reference Snippets

### ✅ Basic Query

```python
with oracledb.connect(user="hr", password="hr", dsn="localhost/orclpdb1") as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT * FROM departments")
        for row in cur:
            print(row)
```

### ✅ Connection Pool

```python
pool = oracledb.create_pool(user="hr", password="hr", dsn="localhost/orclpdb1")
conn = pool.acquire()
with conn.cursor() as cur:
    cur.execute("SELECT COUNT(*) FROM employees")
    print(cur.fetchone())
pool.release(conn)
```

### ✅ Async Query

```python
async with await oracledb.connect_async(user="hr", password="hr", dsn="localhost/orclpdb1") as conn:
    async with conn.cursor() as cur:
        await cur.execute("SELECT first_name FROM employees")
        async for row in cur:
            print(row)
```

---

## ✅ Best Practices

* 🌐 Use **Thin mode** for simplicity, **Thick mode** for advanced use
* 🧩 Always initialize the Oracle client early in Thick mode
* 🔐 Use **parameterized queries** to prevent SQL injection
* 💾 Manage transactions explicitly — avoid autocommit
* 📦 Reuse connections (especially in APIs and web apps)
* 🚦 Implement retry logic for transient network/database errors
* 🧭 Use SODA and JSON features for flexible data storage
* 🧰 Tune and test your connection pool under real workloads

---
