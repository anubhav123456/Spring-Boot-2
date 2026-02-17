## Mapping DTO to Table with JPA Annotations

JPA (Java Persistence API) annotations are used to map Java objects to database tables. Let's explore each annotation and their attributes in detail.

## @Table Annotation

The `@Table` annotation is used at the class level to specify the details of the database table that the entity will be mapped to.

### Attributes of @Table:

#### 1. **name**
Specifies the name of the database table.
```java
@Entity
@Table(name = "employee_master")
public class Employee {
    // fields and methods
}
```
- If not specified, the table name defaults to the entity class name
- Use this when your table name differs from the class name

#### 2. **schema**
Specifies the database schema where the table belongs.
```java
@Entity
@Table(name = "employee", schema = "hr_schema")
public class Employee {
    // fields and methods
}
```
- Useful in multi-schema databases
- Helps organize tables into logical groups
- If not specified, uses the default schema

#### 3. **uniqueConstraints**
Defines unique constraints at the table level (across multiple columns).
```java
@Entity
@Table(
    name = "employee",
    uniqueConstraints = {
        @UniqueConstraint(
            name = "uk_employee_email", 
            columnNames = {"email"}
        ),
        @UniqueConstraint(
            name = "uk_employee_dept_role", 
            columnNames = {"department_id", "role_id"}
        )
    }
)
public class Employee {
    // fields and methods
}
```
- Creates database-level unique constraints
- Can span multiple columns (composite unique keys)
- Helps maintain data integrity
- Name attribute gives meaningful names to constraints

#### 4. **indexes**
Defines database indexes to improve query performance.
```java
@Entity
@Table(
    name = "employee",
    indexes = {
        @Index(
            name = "idx_employee_lastname", 
            columnList = "last_name"
        ),
        @Index(
            name = "idx_employee_dept_salary", 
            columnList = "department_id, salary"
        ),
        @Index(
            name = "idx_employee_email_unique", 
            columnList = "email", 
            unique = true
        )
    }
)
public class Employee {
    // fields and methods
}
```
- Improves query performance for frequently searched columns
- `columnList` can include multiple columns (composite indexes)
- `unique` attribute can create unique indexes
- Index names should be meaningful for maintenance

## @Column Annotation

The `@Column` annotation is used at the field or property level to specify mapping details for a database column.

### Attributes of @Column:

#### 1. **name**
Specifies the name of the database column.
```java
@Column(name = "emp_first_name")
private String firstName;
```
- Maps Java field to specific column name
- Essential when column name differs from field name

#### 2. **unique**
Specifies whether the column should have a unique constraint.
```java
@Column(name = "email", unique = true)
private String email;

@Column(name = "username", unique = true)
private String username;
```
- Creates a unique constraint on this column
- Equivalent to `@UniqueConstraint` at column level
- Ensures no duplicate values in this column

#### 3. **nullable**
Specifies whether the column can store NULL values.
```java
@Column(name = "first_name", nullable = false)
private String firstName;

@Column(name = "middle_name", nullable = true)
private String middleName;

@Column(name = "last_name", nullable = false)
private String lastName;
```
- `false` means column cannot be null (NOT NULL constraint)
- `true` (default) allows null values
- Essential for data validation at database level

#### 4. **length**
Specifies the column length (primarily for String/VARCHAR columns).
```java
@Column(name = "username", length = 50)
private String username;

@Column(name = "description", length = 500)
private String description;

@Column(name = "status", length = 20)
private String status;
```
- Used for string-based column types
- Default length is 255 for VARCHAR
- Database will create column with specified size

#### 5. **updatable**
Specifies whether the column should be included in UPDATE statements.
```java
@Column(name = "created_date", updatable = false)
private LocalDateTime createdDate;

@Column(name = "created_by", updatable = false)
private String createdBy;

@Column(name = "last_modified_date")
private LocalDateTime lastModifiedDate;
```
- `false`: column won't be updated after initial insert
- Useful for audit fields (creation timestamp, created by)
- `true` (default): column can be updated

#### 6. **insertable**
Specifies whether the column should be included in INSERT statements.
```java
@Column(name = "id", insertable = false)
private Long id;  // Auto-generated, don't insert

@Column(name = "calculated_field", insertable = false, updatable = false)
private BigDecimal calculatedField;  // Database-generated
```
- `false`: column won't be included in INSERT operations
- Useful for auto-generated or computed columns
- `true` (default): column is included in inserts

#### 7. **precision**
Specifies the total number of digits for decimal/numeric columns.
```java
@Column(name = "salary", precision = 10, scale = 2)
private BigDecimal salary;  // Can store up to 99999999.99

@Column(name = "tax_rate", precision = 5, scale = 4)
private BigDecimal taxRate;  // Can store up to 9.9999

@Column(name = "quantity", precision = 8)
private BigDecimal quantity;  // 8 digits total, 0 decimal places
```
- Total number of digits (including both sides of decimal point)
- Used with `scale` for precise decimal storage
- Essential for financial applications

#### 8. **scale**
Specifies the number of digits after the decimal point.
```java
@Column(name = "price", precision = 8, scale = 2)
private BigDecimal price;  // 123456.78 format

@Column(name = "interest_rate", precision = 5, scale = 3)
private BigDecimal interestRate;  // 12.345 format

@Column(name = "latitude", precision = 8, scale = 6)
private BigDecimal latitude;  // 12.345678 format
```
- Number of digits to the right of decimal point
- Must be less than or equal to `precision`
- Defines decimal places for precise calculations

## Complete Example

Here's a comprehensive example combining all concepts:

```java
@Entity
@Table(
    name = "employees",
    schema = "hr_system",
    uniqueConstraints = {
        @UniqueConstraint(name = "uk_emp_email", columnNames = "email"),
        @UniqueConstraint(name = "uk_emp_empno", columnNames = "employee_number")
    },
    indexes = {
        @Index(name = "idx_emp_lastname", columnList = "last_name"),
        @Index(name = "idx_emp_dept_salary", columnList = "department_id, salary"),
        @Index(name = "idx_emp_hire_date", columnList = "hire_date")
    }
)
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "employee_id", updatable = false, insertable = false)
    private Long id;

    @Column(name = "employee_number", unique = true, nullable = false, length = 20)
    private String employeeNumber;

    @Column(name = "first_name", nullable = false, length = 50)
    private String firstName;

    @Column(name = "last_name", nullable = false, length = 50)
    private String lastName;

    @Column(name = "email", nullable = false, length = 100)
    private String email;

    @Column(name = "salary", precision = 10, scale = 2)
    private BigDecimal salary;

    @Column(name = "bonus_percentage", precision = 5, scale = 2)
    private BigDecimal bonusPercentage;

    @Column(name = "hire_date", updatable = false)
    private LocalDate hireDate;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Column(name = "is_active", nullable = false)
    private Boolean active = true;

    // Constructors, getters, and setters
}
```

