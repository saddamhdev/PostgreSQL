
<details>
  <summary><strong>First example</strong></summary>

  To further illustrate the use of `@OneToMany` and `@ManyToOne` with `mappedBy` in a Spring Boot application using PostgreSQL, I’ll provide a different example with new entities: `Department` and `Employee`. This example will demonstrate a bidirectional relationship, a join query, and how `mappedBy` works with a different field name. I’ll keep it concise while covering the key aspects, including why `mappedBy` refers to the field name in the "many" side entity.

---

### **Example Scenario: Department and Employee**

- **Relationship**: A `Department` can have multiple `Employee`s, but each `Employee` belongs to one `Department` (one-to-many/many-to-one relationship).
- **Goal**: Show how to set up the entities, use `mappedBy`, and perform a join query to fetch departments with their employees.

---

### **1. Entity Classes**

#### **Department Entity (One Side)**
```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String deptName;
    
    @OneToMany(mappedBy = "dept", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
    
    // Helper method to maintain bidirectional relationship
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDept(this);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getDeptName() { return deptName; }
    public void setDeptName(String deptName) { this.deptName = deptName; }
    public List<Employee> getEmployees() { return employees; }
    public void setEmployees(List<Employee> employees) { this.employees = employees; }
}
```

#### **Employee Entity (Many Side)**
```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String empName;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department dept; // Field name is "dept"
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getEmpName() { return empName; }
    public void setEmpName(String empName) { this.empName = empName; }
    public Department getDept() { return dept; }
    public void setDept(Department dept) { this.dept = dept; }
}
```

- **Key Points**:
  - The `Department` entity has a `@OneToMany` relationship with `Employee`, and `mappedBy = "dept"` points to the `dept` field in the `Employee` entity.
  - The `Employee` entity has a `@ManyToOne` relationship with `Department`, and the foreign key column in the database is `department_id` (defined by `@JoinColumn(name = "department_id")`).
  - The `mappedBy = "dept"` indicates that the `dept` field in `Employee` owns the relationship, and the foreign key (`department_id`) is stored in the `employees` table.

---

### **2. PostgreSQL Schema**

The corresponding database schema in PostgreSQL would look like this:

```sql
CREATE TABLE departments (
    id BIGINT PRIMARY KEY,
    dept_name VARCHAR(255)
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    emp_name VARCHAR(50),
    department_id BIGINT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

- The `department_id` column in the `employees` table is the foreign key linking to the `id` column of the `departments` table.
- The name `department_id` comes from the `@JoinColumn(name = "department_id")` in the `Employee` entity, not from the `dept` field name or `mappedBy`.

---

### **3. Repository with Join Query**

To query departments along with their employees, you can use a Spring Data JPA repository with a JPQL query.

```java
@Repository
public interface DepartmentRepository extends JpaRepository<Department, Long> {
    // Fetch departments with their employees using a JOIN FETCH
    @Query("SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees e WHERE d.deptName LIKE %:name%")
    List<Department> findByDeptNameContainingWithEmployees(@Param("name") String name);
    
    // Native SQL query example
    @Query(value = "SELECT d.* FROM departments d LEFT JOIN employees e ON d.id = e.department_id WHERE d.dept_name LIKE %:name%", nativeQuery = true)
    List<Department> findByDeptNameContainingWithEmployeesNative(@Param("name") String name);
}
```

- **Explanation**:
  - The JPQL query uses `d.employees` to reference the `employees` collection in `Department`, and `JOIN FETCH` ensures that the `employees` are loaded in a single query to avoid the N+1 problem.
  - The native SQL query uses the actual database column `department_id` in the `ON` clause, matching the `@JoinColumn` name.
  - The `mappedBy = "dept"` does not appear in the queries; it’s only used by JPA to map the relationship.

---

### **4. Service Layer**

A service to create and query departments with employees:

```java
@Service
public class DepartmentService {
    @Autowired
    private DepartmentRepository departmentRepository;
    
    public Department createDepartmentWithEmployee(String deptName, String empName) {
        Department department = new Department();
        department.setDeptName(deptName);
        
        Employee employee = new Employee();
        employee.setEmpName(empName);
        
        department.addEmployee(employee); // Synchronizes both sides
        return departmentRepository.save(department);
    }
    
    public List<Department> findDepartmentsByName(String name) {
        return departmentRepository.findByDeptNameContainingWithEmployees(name);
    }
}
```

- **Explanation**:
  - The `addEmployee` method ensures the bidirectional relationship is synchronized by setting the `dept` field in `Employee` and adding the `Employee` to the `employees` list in `Department`.
  - The `save` operation persists both the `Department` and its `Employee`s due to `cascade = CascadeType.ALL`.

---

### **5. Why `mappedBy = "dept"`?**

- The `mappedBy = "dept"` in the `Department` entity’s `@OneToMany` annotation points to the `dept` field in the `Employee` entity, which is annotated with `@ManyToOne`.
- This tells JPA that the `Employee` entity owns the relationship, and the foreign key (`department_id`) is stored in the `employees` table.
- The name `"dept"` is the Java field name in `Employee`, not a database column or a predefined value. If the field in `Employee` were named `department` instead of `dept`, you would use `mappedBy = "department"`.

For example, if you change the `Employee` entity to:

```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String empName;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department; // Field name changed to "department"
    
    // Getters and setters
}
```

Then the `Department` entity would need:

```java
@OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
private List<Employee> employees = new ArrayList<>();
```

- The `mappedBy` value must always match the field name in the `Employee` entity.

---

### **6. Configuration (`application.properties`)**

Ensure your Spring Boot application is configured for PostgreSQL:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/yourdb
spring.datasource.username=youruser
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=true
```

- **`spring.jpa.hibernate.ddl-auto=update`**: Automatically creates/updates the database schema based on the entities.
- **`spring.jpa.show-sql=true`**: Logs the SQL queries to verify the foreign key (`department_id`) and join operations.

---

### **7. Example Usage**

#### **Creating Data**
```java
Department department = departmentService.createDepartmentWithEmployee("Engineering", "John Doe");
```

- This creates a `Department` with `deptName = "Engineering"` and an `Employee` with `empName = "John Doe"`.
- The `addEmployee` method ensures the `dept` field in `Employee` is set, and the `department_id` foreign key is populated in the `employees` table.

#### **Querying Data**
```java
List<Department> departments = departmentService.findDepartmentsByName("Eng");
```

- This fetches all departments with a name containing "Eng" (e.g., "Engineering") along with their employees in a single query.

#### **Sample Database Content**
After saving, the database might look like:

**departments table**:
```
id  | dept_name
----+------------
1   | Engineering
```

**employees table**:
```
id  | emp_name  | department_id
----+-----------+--------------
1   | John Doe  | 1
```

---

### **8. Join Query Example**

When you run the JPQL query:
```java
@Query("SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees e WHERE d.deptName LIKE %:name%")
```

- JPA generates SQL similar to:
  ```sql
  SELECT DISTINCT d.*, e.*
  FROM departments d
  LEFT JOIN employees e ON d.id = e.department_id
  WHERE d.dept_name LIKE '%Eng%';
  ```
- The `LEFT JOIN FETCH` ensures that the `employees` collection is populated in one query.
- The `department_id` foreign key (from `@JoinColumn`) is used in the `ON` clause, while `mappedBy = "dept"` defines the relationship mapping in JPA.

---

### **9. Why `mappedBy = "dept"` Instead of a Column Name?**

- The `mappedBy = "dept"` refers to the Java field name in the `Employee` entity (`private Department dept`), not the database column (`department_id`) or the `deptName` field in `Department`.
- It tells JPA that the `Employee` entity’s `dept` field owns the relationship, and the foreign key is managed by the `@JoinColumn(name = "department_id")` in `Employee`.
- This avoids a redundant join table and ensures the relationship is mapped to the `department_id` column in the `employees` table.

---

### **10. Key Differences from Previous Example**

Compared to the `Customer`/`Order` example:
- **Field Name**: The `Employee` entity uses `dept` instead of `customer` for the `@ManyToOne` field, so `mappedBy = "dept"` reflects this change.
- **Foreign Key**: The foreign key column is `department_id` instead of `customer_id`, showing that the column name is defined by `@JoinColumn` and can be customized.
- **Entities**: The example uses `Department` and `Employee` instead of `Customer` and `Order`, demonstrating the same concept in a different business context.
- **Query**: The join query searches by `deptName` instead of `name`, but the structure (using `JOIN FETCH`) is similar.

---

If you need another example with a different relationship (e.g., many-to-many) or have specific questions about this setup (e.g., handling queries, performance, or a different field name), let me know, and I’ll provide a tailored response!
</details>

<details>
  <summary> <strong> Second Example</strong> </summary>
  Let’s look at another example of a `@OneToMany` and `@ManyToOne` relationship in Spring Boot with PostgreSQL, using different entities to illustrate the concept and the use of `mappedBy`. This time, we’ll use a `Department` and `Employee` relationship, where one `Department` can have many `Employees`, and each `Employee` belongs to one `Department`. I’ll also show how to perform a join query in a repository and explain the role of `mappedBy` and the foreign key column.

---

### **Scenario**

- **Entities**: `Department` (one) and `Employee` (many).
- **Relationship**: A `Department` can have multiple `Employees`, but each `Employee` is associated with only one `Department`.
- **Foreign Key**: The `employees` table will have a `department_id` column referencing the `departments` table.
- **Join Query**: We’ll create a repository method to fetch departments with their employees using a JPQL join.

---

### **Example Code**

#### **1. Entities**

**Department Entity** (One side):
```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String deptName;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
    
    // Helper method to maintain bidirectional relationship
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }
    
    // Getters and setters
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getDeptName() {
        return deptName;
    }
    
    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }
    
    public List<Employee> getEmployees() {
        return employees;
    }
    
    public void setEmployees(List<Employee> employees) {
        this.employees = employees;
    }
}
```

**Employee Entity** (Many side):
```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String empName;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    // Getters and setters
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getEmpName() {
        return empName;
    }
    
    public void setEmpName(String empName) {
        this.empName = empName;
    }
    
    public Department getDepartment() {
        return department;
    }
    
    public void setDepartment(Department department) {
        this.department = department;
    }
}
```

- **Explanation**:
  - **Department**: The `@OneToMany` annotation on `employees` uses `mappedBy = "department"`, which points to the `department` field in the `Employee` entity. This indicates that the `Employee` entity owns the relationship, and the foreign key is stored in the `employees` table.
  - **Employee**: The `@ManyToOne` annotation on `department` specifies that each `Employee` is linked to one `Department`. The `@JoinColumn(name = "department_id")` defines the foreign key column in the `employees` table.
  - **Helper Method**: The `addEmployee` method in `Department` ensures the bidirectional relationship is synchronized by setting both sides of the relationship.

---

#### **2. PostgreSQL Schema**

The corresponding database schema in PostgreSQL would look like this:

```sql
CREATE TABLE departments (
    id BIGINT PRIMARY KEY,
    dept_name VARCHAR(255)
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    emp_name VARCHAR(255),
    department_id BIGINT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

- **department_id**: This is the foreign key column in the `employees` table, as specified by `@JoinColumn(name = "department_id")`. It links to the `id` column of the `departments` table.
- The `department_id` name is user-defined, not predefined, and matches the `@JoinColumn` annotation.

---

#### **3. Repository with Join Query**

Here’s a Spring Data JPA repository that includes a join query to fetch departments with their employees:

```java
@Repository
public interface DepartmentRepository extends JpaRepository<Department, Long> {
    // JPQL query to fetch departments with their employees
    @Query("SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees e WHERE d.deptName LIKE %:name%")
    List<Department> findByDeptNameContainingWithEmployees(@Param("name") String name);
    
    // Native SQL query
    @Query(value = "SELECT d.* FROM departments d LEFT JOIN employees e ON d.id = e.department_id WHERE d.dept_name LIKE %:name%", nativeQuery = true)
    List<Department> findByDeptNameContainingWithEmployeesNative(@Param("name") String name);
}
```

- **Explanation**:
  - **JPQL Query**: Uses `LEFT JOIN FETCH d.employees e` to eagerly load the `employees` collection for each `Department`. The `mappedBy = "department"` ensures JPA knows the relationship is managed by the `department` field in `Employee`.
  - **Native SQL Query**: Uses the actual table and column names (`departments`, `employees`, `department_id`). The `department_id` column matches the `@JoinColumn` annotation in the `Employee` entity.
  - The `DISTINCT` keyword avoids duplicate `Department` entries in the result set when multiple `Employees` are associated with a single `Department`.

---

#### **4. Service Layer**

A service to demonstrate creating and querying departments with employees:

```java
@Service
public class DepartmentService {
    @Autowired
    private DepartmentRepository departmentRepository;
    
    public Department createDepartmentWithEmployee(String deptName, String empName) {
        Department department = new Department();
        department.setDeptName(deptName);
        
        Employee employee = new Employee();
        employee.setEmpName(empName);
        
        department.addEmployee(employee);
        return departmentRepository.save(department);
    }
    
    public List<Department> findDepartmentsByName(String name) {
        return departmentRepository.findByDeptNameContainingWithEmployees(name);
    }
}
```

- **Explanation**:
  - The `createDepartmentWithEmployee` method creates a `Department` and an `Employee`, links them using the `addEmployee` helper method, and saves them.
  - The `findDepartmentsByName` method uses the JPQL query to fetch departments and their employees.

---

#### **5. Configuration (`application.properties`)**

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/yourdb
spring.datasource.username=youruser
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

- This configures the PostgreSQL database and enables SQL logging to verify the generated queries.

---

### **Why `mappedBy = "department"`?**

- The `mappedBy = "department"` in the `Department` entity’s `@OneToMany` annotation points to the `department` field in the `Employee` entity:
  ```java
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "department_id")
  private Department department; // Field name is "department"
  ```
- The value `"department"` is the Java field name in the `Employee` entity, not a database column name or the `deptName` field (e.g., "HR" or "IT").
- It tells JPA that the `Employee` entity’s `department` field manages the relationship, and the foreign key (`department_id`) is stored in the `employees` table.

If you named the field in `Employee` something else, like `dept`, you’d use `mappedBy = "dept"`:

```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String empName;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department dept; // Field name changed to "dept"
    
    // Getters and setters
}

@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String deptName;
    
    @OneToMany(mappedBy = "dept", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
    
    // Getters, setters, and addEmployee method
}
```

- The database schema remains unchanged (still uses `department_id` as the foreign key column), but `mappedBy` must match the field name (`dept`).

---

### **Join Query in Action**

Suppose you have the following data in PostgreSQL:

- **departments**:
  ```
  id | dept_name
  ---+----------
  1  | HR
  2  | IT
  ```

- **employees**:
  ```
  id | emp_name | department_id
  ---+----------+--------------
  1  | John     | 1
  2  | Jane     | 1
  3  | Mike     | 2
  ```

Running the query:
```java
List<Department> departments = departmentRepository.findByDeptNameContainingWithEmployees("HR");
```

- **JPQL Query Generated**:
  ```sql
  SELECT DISTINCT d.* FROM departments d LEFT JOIN employees e ON d.id = e.department_id WHERE d.dept_name LIKE '%HR%'
  ```
- **Result**: Returns the `HR` department with its employees (`John` and `Jane`) in a single query, thanks to `JOIN FETCH`.

---

### **Key Points**

- **Why `mappedBy = "department"`?**:
  - It refers to the `department` field in the `Employee` entity, which defines the `@ManyToOne` relationship.
  - It ensures JPA uses the `department_id` foreign key in the `employees` table, as specified by `@JoinColumn(name = "department_id")`.
- **Foreign Key (`department_id`)**:
  - The `department_id` column is user-defined in the `@JoinColumn` annotation, not predefined. It could be named anything (e.g., `dept_id`), as long as it matches the database schema.
- **Join Queries**:
  - JPQL queries use entity and field names (e.g., `d.employees`), abstracting away the foreign key column.
  - Native SQL queries require the exact column name (`department_id`) in the `ON` clause.
- **Bidirectional Relationship**:
  - The `addEmployee` helper method synchronizes both sides of the relationship to maintain consistency.
- **Not Related to `deptName`**:
  - The `deptName` field (e.g., "HR") is a business attribute of `Department`, unrelated to `mappedBy` or the relationship.

---

If you need another example with different entities, a specific join query, or clarification on any part of this setup, let me know!
</details>
