# Sequence Generation Type

Sequence generation uses a database sequence object to generate unique numeric IDs. Unlike IDENTITY, sequences are independent of tables and can be shared across multiple entities.

---

## How Sequences Work:

```java
@Entity
@Data
@NoArgsConstructor
public class Person 
{
    private String name;
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Integer id;  // Using default sequence
}
```

---

## Sequence Patterns You Mentioned:

| Pattern | Example | Description |
|---------|---------|-------------|
| **1, 2, 3, 4, 5...** | `1, 2, 3, 4, 5` | Default sequence with increment by 1 |
| **2, 4, 6, 8, 10...** | `2, 4, 6, 8, 10` | Sequence with increment by 2 |
| **1, 3, 5, 7, 9...** | `1, 3, 5, 7, 9` | Odd numbers sequence |

---

## Custom Sequence Example Explained:

```java
@Entity
@Data
@NoArgsConstructor
public class Person {
    private String name;
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequences")
    @SequenceGenerator(
        name = "sequences",      // Name used in @GeneratedValue
        initialValue = 4,         // Starting point
        allocationSize = 1        // How many values to increment by
    )
    private Integer id;
}
```

**What happens when you save:**
- **abc1 → 4**: First person gets ID 4 (initialValue)
- **abc2 → 5**: Second person gets ID 5 (incremented by allocationSize=1)

---

## Sharing Sequence Across Entities:

```java
// Same sequence generator for both entities
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequences")
    @SequenceGenerator(name = "sequences", initialValue = 4, allocationSize = 1)
    private Integer id;
}

@Entity  
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequences")
    @SequenceGenerator(name = "sequences", initialValue = 4, allocationSize = 1)
    private Integer id;
}
```


**How IDs are allocated:**

| Operation | Table | ID Generated | Sequence Current Value |
|-----------|-------|--------------|------------------------|
| Save Person1 | Person | 4 | 5 |
| Save Person2 | Person | 5 | 6 |
| Save Author1 | Author | 6 | 7 |
| Save Author2 | Author | 7 | 8 |
| Save Person3 | Person | 8 | 9 |

---

## Database-Level Sequence (Example for PostgreSQL):
```sql
-- This is what Hibernate creates
CREATE SEQUENCE sequences START WITH 4 INCREMENT BY 1;

-- Both tables use the same sequence
CREATE TABLE person (id INTEGER DEFAULT nextval('sequences'), name VARCHAR);
CREATE TABLE author (id INTEGER DEFAULT nextval('sequences'), name VARCHAR);
```
---

## allocationSize Explained:

```java
@SequenceGenerator(name = "seq", initialValue = 1, allocationSize = 50)
```

- **allocationSize = 1**: Fetches one value at a time (simpler but more database calls)
- **allocationSize = 50**: Fetches 50 values at once (better performance, possible gaps)

**With allocationSize = 50:**
```
Hibernate: select nextval('seq') -- Gets 1-50
Person1 gets 1
Person2 gets 2
... (until 50)
Hibernate: select nextval('seq') -- Gets 51-100
Person51 gets 51
```

---

## Advantages vs IDENTITY:

| Aspect | SEQUENCE | IDENTITY |
|--------|----------|----------|
| **Performance** | ✅ Can batch insert (with allocationSize > 1) | ❌ No batch inserts possible |
| **Sharing** | ✅ Can be shared across tables | ❌ Per table only |
| **Control** | ✅ Full control over generation | ❌ Database controls it |
| **Pre-allocation** | ✅ Yes (reduces DB calls) | ❌ No |
| **Gaps** | Possible (due to caching) | Possible (due to rollbacks) |

---

## Key Points:
1. **Sequence is a database object**, independent of tables
2. **Shared sequences** work across multiple entities
3. **initialValue** sets where the sequence starts
4. **allocationSize** determines the increment step
5. **Gaps can occur** if the server restarts or transactions rollback

---