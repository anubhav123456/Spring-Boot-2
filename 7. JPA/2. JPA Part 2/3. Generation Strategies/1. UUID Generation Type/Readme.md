# UUID Generation Type

The `@GeneratedValue(strategy = GenerationType.UUID)` annotation in your code automatically generates a unique UUID (Universally Unique Identifier) as the primary key for your `Person` entity.

---

## How it Works in Your Code:

```java
@Entity
@Data
@NoArgsConstructor
public class Person 
{
    private String name;
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;
}
```

When you save a new `Person` object:
```java
Person person = new Person();
person.setName("John Doe");
personRepository.save(person); // id is automatically generated
```

---

## Key Characteristics:

1. **Auto-generation**: Hibernate automatically generates a UUID before inserting the record
2. **String Storage**: The UUID is stored as a String in the database
3. **Unique across tables/systems**: UUIDs are globally unique, not just within a single table

## Example UUID Generated:
```
"123e4567-e89b-12d3-a456-426614174000"
```

---

## Advantages:
- **Global uniqueness**: Perfect for distributed systems
- **Security**: Harder to guess than sequential IDs
- **No coordination needed**: Can generate IDs offline
- **Merge-friendly**: No conflicts when merging databases

---

## Disadvantages:
- **Storage size**: 36 characters vs 4-8 bytes for integers
- **Performance**: Slightly slower for indexing and joins
- **Readability**: Not human-friendly for debugging

---