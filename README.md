# Mini Banking System – Transaction Flow and Distribution Planning

Diagrams: open [Mini_Banking_System.drawio](Mini_Banking_System.drawio) with [app.diagrams.net](https://app.diagrams.net) or the Draw.io extension in VS Code.

## Part 1 – Transaction Management

**Scenario:** Account A = 1000 DT. T1 transfers 200 DT from A and T2 transfers 300 DT from A, at the same time.

### 1. Concurrency issue
**Lost update.** Both transactions read A = 1000 before either writes. T2 overwrites T1's write, so A = 700 instead of 500.

### 2. Locking mechanism
- **Exclusive lock (X)** on an account before a transfer changes it.
- **Shared lock (S)** to view a balance.
- Locks are kept until COMMIT.

### 3. Pessimistic or optimistic?
**Pessimistic.** Transfers on the same account often conflict, and money must never be wrong. It is better to lock first than to detect the conflict later and retry.

### 4. Schedules

Unsafe (no locks):

| Time | T1 | T2 | A |
|---|---|---|---|
| t1 | Read(A) = 1000 | | 1000 |
| t2 | | Read(A) = 1000 | 1000 |
| t3 | Write(A) = 800 | | 800 |
| t4 | | Write(A) = 700 | 700 |
| t5 | Commit | Commit | **700 ❌ Not safe** |

Safe (exclusive locks):

| Time | T1 | T2 | A |
|---|---|---|---|
| t1 | X-Lock(A) | | 1000 |
| t2 | Read(A) = 1000 | X-Lock(A) → wait | 1000 |
| t3 | Write(A) = 800 | waiting | 800 |
| t4 | Commit + Unlock(A) | | 800 |
| t5 | | X-Lock(A) granted | 800 |
| t6 | | Read(A) = 800 | 800 |
| t7 | | Write(A) = 500 | 500 |
| t8 | | Commit + Unlock(A) | **500 ✅ Safe** |

## Part 2 – Distributed Database Planning

### 1. Horizontal fragmentation
Split the `Customers` table by branch:
- `Customers_Tunis = σ branch='Tunis' (Customers)`
- `Customers_Sousse = σ branch='Sousse' (Customers)`
- `Customers_Sfax = σ branch='Sfax' (Customers)`

### 2. Vertical fragmentation
Move login info to its own table:
- `Customers(customer_id, name, phone, address, branch)`
- `Customer_Login(customer_id, email, password_hash)`

Login data is sensitive and only used for authentication.

### 3. Replication
| Data | Replicate to all branches? | Why |
|---|---|---|
| Customer info | ✅ Yes | Small and rarely changes, so any branch can serve any customer |
| Account balances | ❌ No (one copy at the home branch) | Changes on every transfer, so keeping 3 copies in sync is costly |
| Transaction history | ❌ No (home branch + backup) | Very large and mostly used by the home branch |

### 4. Allocation for transaction history
**Static.** Each transaction is stored at the branch that owns the account. Customers mostly use their home branch, so the access pattern is predictable. Static allocation is simple and never needs to move data.
