# Identity Generation Type

The `@GeneratedValue(strategy = GenerationType.IDENTITY)` annotation relies on the database's auto-increment feature to generate unique primary keys.

---

## How it Works:

```java
@Entity
@Data
@NoArgsConstructor
public class Person 
{
    private String name;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;  // Note: Should be Integer (object), not INTEGER (primitive)
}
```

---

## Your Example Explained Step by Step:

| Operation | What Happens | Generated ID | Explanation |
|-----------|--------------|--------------|-------------|
| **ABC1** | `INSERT INTO person (name) VALUES ('ABC1')` | 1 | First record, starts at 1 |
| **ABC2** | `INSERT INTO person (name) VALUES ('ABC2')` | 2 | Auto-increments to 2 |
| **1200, ABC3** | `INSERT INTO person (id, name) VALUES (1200, 'ABC3')` | 1200 | **Manual override** - you explicitly set ID 1200 |
| **3, ABC4** | `INSERT INTO person (id, name) VALUES (3, 'ABC4')` | 3 | Another manual override with ID 3 |
| **ABC5** | `INSERT INTO person (name) VALUES ('ABC5')` | 1201 | Database continues from **max ID + 1** (1200+1) |


---

## Key Concept: Auto-increment Tracking

The database maintains a counter for the next value to use. In your example:
- After ABC1: Next value = 2
- After ABC2: Next value = 3  
- After manual insert (1200): Next value = 1201
- After manual insert (3): Next value = 1201 (unchanged, as 3 < 1200)
- Final insert: Uses 1201

---

## Important Characteristics:

1. **Database-managed**: The database handles ID generation, not Hibernate
2. **Post-insert**: You only get the ID after inserting (unlike UUID which can be generated before)
3. **Always increasing**: IDs typically increase but may have gaps

---

## Common Database Implementations:
```sql
-- MySQL / MariaDB
AUTO_INCREMENT

-- PostgreSQL / H2
SERIAL / IDENTITY

-- SQL Server
IDENTITY(1,1)

-- Oracle (with sequences)
SEQUENCE + TRIGGER
```

---

## Advantages vs Disadvantages:

| Pros | Cons |
|------|------|
| ✅ Simple and efficient | ❌ Database-dependent |
| ✅ Good for indexing | ❌ Not suitable for distributed systems |
| ✅ Human-readable | ❌ Predictable (security concern) |
| ✅ Compact storage (4 bytes) | ❌ Conflicts when merging databases |

---