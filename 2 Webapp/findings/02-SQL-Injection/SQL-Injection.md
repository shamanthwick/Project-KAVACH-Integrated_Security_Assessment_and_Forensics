# SQL Injection

## Objective
Demonstrate how unsanitized user input can influence backend database queries.

## Case 1

### Normal Input
![Normal Input](images/Case-1/Normal-Input.png)

### Manipulated Input
![Payload 1](images/Case-1/Manipulated-Input-1.png)

![Payload 2](images/Case-1/Manipulated-Input-2.png)

### Observation
The application processed manipulated input and exhibited SQL-related behavior.

---

## Case 2

### Normal Input
![Normal Input](images/Case-2/Normal-Input.png)

### Manipulated Input
![Payload](images/Case-2/Manipulated-Input.png)

### Observation
Additional testing was performed to evaluate how the application handled unexpected user input.

## Impact
SQL Injection can lead to unauthorized access to sensitive information stored in the database.

## Mitigation
- Parameterized queries
- Input validation
- Principle of least privilege