
---

# 🏦 Scenario: Loan Disbursement System

A bank disburses a **loan of ₹5,00,000** to a customer.

Steps involved:

1. Validate customer
2. Debit bank’s pool account
3. Credit customer account
4. Save transaction history
5. Generate loan agreement PDF
6. Call CIBIL API
7. Risk check sub-step
8. System monitoring

All will use different propagation types.

---

# 🎯 MAIN METHOD

```java
@Service
public class LoanService {

    @Transactional // REQUIRED (default)
    public void disburseLoan(Long customerId, double amount) {
        validationService.validateCustomer(customerId);      // SUPPORTS
        accountService.debitBankPool(amount);                 // REQUIRED
        accountService.creditCustomer(customerId, amount);    // REQUIRED
        transactionService.saveTxn(customerId, amount);       // REQUIRES_NEW
        documentService.generateLoanDoc(customerId);          // NESTED
        cibilService.checkScore(customerId);                  // NOT_SUPPORTED
        monitoringService.systemHealth();                     // NEVER
    }
}
```

---

# 1️⃣ REQUIRED → Core Money Movement 💰

```java
@Transactional
public void debitBankPool(double amount) { }

@Transactional
public void creditCustomer(Long id, double amount) { }
```

Join the main transaction (TX-1).

If anything fails → rollback everything.

---

# 2️⃣ SUPPORTS → Validation

```java
@Transactional(propagation = Propagation.SUPPORTS)
public void validateCustomer(Long id) { }
```

If called inside TX → join
If standalone → no TX

Used for reads.

---

# 3️⃣ REQUIRES_NEW → Transaction History 📜

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveTxn(Long id, double amount) { }
```

Even if main TX fails, audit log persists.

---

# 4️⃣ NESTED → Loan Document Generation 📄

```java
@Transactional(propagation = Propagation.NESTED)
public void generateLoanDoc(Long id) {
    riskService.riskSubCheck(id); // nested inside nested
}
```

Creates **savepoint** inside TX-1.

If doc generation fails → only doc part rolls back, loan still processed.

---

# 5️⃣ MANDATORY → Risk Sub-Check ⚠️

```java
@Transactional(propagation = Propagation.MANDATORY)
public void riskSubCheck(Long id) { }
```

Must run inside existing transaction (loan doc TX).

If called alone → exception.

---

# 6️⃣ NOT_SUPPORTED → CIBIL API 🌍

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void checkScore(Long id) {
    // External API call
}
```

Suspends TX to avoid DB lock during network call.

---

# 7️⃣ NEVER → System Monitoring 🖥️

```java
@Transactional(propagation = Propagation.NEVER)
public void systemHealth() { }
```

Fails if called within transaction.

Used for pure system-level checks.

---

# 🔥 FULL FLOW DIAGRAM

```
TX-1 START (REQUIRED)

  validateCustomer() → SUPPORTS → joins TX-1

  debitBankPool() → REQUIRED → TX-1
  creditCustomer() → REQUIRED → TX-1

  TX-1 PAUSE
    TX-2 START → saveTxn() → COMMIT
  TX-1 RESUME

  NESTED TX SAVEPOINT → generateLoanDoc()
        riskSubCheck() → MANDATORY (must be inside TX)
  If fails → rollback to savepoint only

  TX-1 PAUSE
    checkScore() → NOT_SUPPORTED (no TX)
  TX-1 RESUME

  systemHealth() → NEVER → throws error if TX exists ❌

TX-1 COMMIT
```

---

# 🎯 What Rolls Back When?

| Failure                     | Result                   |
| --------------------------- | ------------------------ |
| Debit fails                 | Everything rolls back    |
| Doc generation fails        | Only doc step rolls back |
| History save fails          | Loan still processed     |
| CIBIL API fails             | Loan still processed     |
| Monitoring called inside TX | Exception                |

---

# 💣 Interview Killer Closing Line

> “Propagation lets us separate **critical financial operations** from **side effects, auditing, and external calls**, ensuring consistency, performance, and fault isolation in distributed banking systems.”

---
