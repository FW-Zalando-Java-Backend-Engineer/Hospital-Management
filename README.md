# 🏥 Hospital Management REST API Assignment

## 📜 Overview

Build a **Spring Boot** REST API to manage a hospital's staff and patients.  
You'll design real-world entities, connect to PostgreSQL, perform CRUD, and write **advanced native SQL queries** with Spring Data JPA.

> 🎯 Goal: Practice full-stack Spring Boot development with custom SQL.

---

## 📌 Background

Our hospital has:

- **Staff**: doctors, nurses, admin staff — organized by department.
- **Patients**: each assigned to a doctor. Patients have diagnoses and bills.

We want a REST API to:

✅ Register staff & patients  
✅ Query assigned patients for a doctor  
✅ See billing statistics  
✅ Identify doctors with many patients  
✅ Report patients with bills higher than the average  

---

## ✅ Database Schema

### Staff Table

| Column   | Type      | Notes                        |
|----------|-----------|-----------------------------|
| id       | BIGSERIAL | Primary key                 |
| name     | TEXT      | Staff name                  |
| role     | TEXT      | doctor/nurse/admin          |
| department | TEXT    | e.g. Cardiology, ER         |

---

### Patients Table

| Column             | Type      | Notes                                  |
|---------------------|-----------|----------------------------------------|
| id                  | BIGSERIAL | Primary key                           |
| name                | TEXT      | Patient name                          |
| assigned_doctor_id  | BIGINT    | FK to Staff.id (doctor only)          |
| diagnosis           | TEXT      | Description of diagnosis              |
| bill_amount         | NUMERIC   | Total bill in euros                   |

✅ Use foreign key to link patients to their assigned doctor.

---

## ✅ Assignment Tasks

### Task 1️⃣: Set up Spring Boot Project

✅ Use Maven manually (no Spring Initializr)  
✅ Suggested name: `hospital-api`  
✅ Dependencies:
- spring-boot-starter-web
- spring-boot-starter-data-jpa
- postgresql driver

✅ Recommended package structure:
```

org.example.hospital
\|-- model
\|-- repo
\|-- sql
\|-- web

````

> 💡 *Hint*: Use IntelliJ IDEA Community Edition. Make sure `pom.xml` has the correct dependencies.

---

### Task 2️⃣: Configure PostgreSQL

✅ Create a new PostgreSQL database (e.g. `hospitaldb`)  
✅ Add application.properties:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/hospitaldb
spring.datasource.username=YOUR_USER
spring.datasource.password=YOUR_PASSWORD
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
````

> 💡 *Hint*: use `spring.jpa.hibernate.ddl-auto=update` for fast iteration.

---

### Task 3️⃣: Model Layer (JPA Entities)

#### Staff Entity

✅ Fields:

* id (auto-generated)
* name
* role (ENUM or String)
* department

✅ Annotations:

* @Entity
* @Id, @GeneratedValue
* @Column

---

#### Patient Entity

✅ Fields:

* id
* name
* diagnosis
* billAmount
* assignedDoctor (ManyToOne → Staff)

✅ Relationship:

* @ManyToOne
* @JoinColumn(name = "assigned\_doctor\_id")

> 💡 *Hint*: Annotate with @Entity, use standard Lombok or getters/setters.

---

### Task 4️⃣: Repository Layer

✅ Create `StaffRepository` and `PatientRepository` extending `CrudRepository`.

✅ Create a **Custom Repository Interface**:

```java
@Repository
public interface PatientRepositoryCustom {
    List<Patient> findPatientsForDoctor(Long doctorId);
    List<Object[]> averageBillPerDepartment();
    List<Object[]> doctorsWithPatientCount(int minCount);
    List<Patient> patientsAboveAvgBill();
}
```

✅ Implement it with EntityManager + native SQL.

---

### Task 5️⃣: SQL Queries

✅ Write SQL for custom methods:

* **a. Patients for a given doctor**

```sql
SELECT * FROM patients WHERE assigned_doctor_id = ?
```

* **b. Average bill per department**

```sql
SELECT s.department, AVG(p.bill_amount)
FROM patients p
JOIN staff s ON p.assigned_doctor_id = s.id
GROUP BY s.department
```

* **c. Doctors with more than X patients**

```sql
SELECT s.id, s.name, COUNT(p.id) as patient_count
FROM staff s
JOIN patients p ON s.id = p.assigned_doctor_id
WHERE s.role = 'doctor'
GROUP BY s.id, s.name
HAVING COUNT(p.id) > ?
```

* **d. Patients with bill above avg**

```sql
SELECT * FROM patients
WHERE bill_amount > (SELECT AVG(bill_amount) FROM patients)
```

✅ Store these in an `enum` (e.g. `PatientQuery`).

> 💡 *Hint*: use EntityManager's `createNativeQuery`.

---

### Task 6️⃣: Web Layer (Controller)

✅ Create `StaffController` and `PatientController`:

✅ Expose endpoints:

* `/patients`

  * GET all
  * POST new
  * GET /for-doctor?doctorId=
  * GET /above-avg-bill

* `/stats/avg-bill-per-department`

* `/stats/doctors-with-many-patients?minCount=`

> 💡 *Hint*: Use `@RequestParam` for dynamic inputs.

---

### Task 7️⃣: Data Initialization

✅ Write SQL (or Spring CommandLineRunner) to insert sample data:

* 3–5 staff (doctors/nurses/admin)
* 10+ patients

Example:

```sql
INSERT INTO staff (name, role, department) VALUES ('Dr. Smith', 'doctor', 'Cardiology');
INSERT INTO patients (name, diagnosis, bill_amount, assigned_doctor_id) VALUES ('John Doe', 'Arrhythmia', 4500, 1);
```

---

### Task 8️⃣: Testing

✅ Use Postman or curl:

* `GET /patients/for-doctor?doctorId=1`
* `GET /stats/avg-bill-per-department`
* `GET /stats/doctors-with-many-patients?minCount=3`
* `GET /patients/above-avg-bill`

> 💡 *Hint*: Inspect JSON responses carefully.

---

## ⭐ Hints & Resources

✅ [Baeldung: Spring Data JPA – Custom Queries](https://www.baeldung.com/spring-data-jpa-query)
✅ [Spring.io Data JPA Docs](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods)
✅ [PostgreSQL JOINs](https://www.w3schools.com/postgresql/postgresql_joins.php)
✅ [Spring @ManyToOne tutorial](https://www.baeldung.com/hibernate-one-to-many)
✅ [Hibernate Native Queries](https://thorben-janssen.com/jpa-native-queries/)

---

## 🎯 Expected Deliverables

✅ Clean Maven project with:

```
src/main/java/org/example/hospital/
  model/
  repo/
  sql/
  web/
```

✅ PostgreSQL schema and data scripts.
✅ README explaining setup instructions.
✅ Well-documented code (inline comments + Javadoc).

---

## 📈 Grading Rubric (Example)

| Criterion                          | Points  |
| ---------------------------------- | ------- |
| Correct entity modeling            | 20      |
| Repository CRUD and custom queries | 30      |
| Controller endpoints work          | 20      |
| Native SQL correctly written       | 20      |
| Clean structure and documentation  | 10      |
| **TOTAL**                          | **100** |

---

## 🗂️ Notes for Students

✅ Focus on clarity and maintainability.
✅ Use meaningful method names.
✅ Add inline comments explaining the queries.
✅ Follow good REST conventions.

---

> 💬 If you get stuck, check the resources above, or ask your instructor!

---

