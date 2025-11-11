# PL/SQL Project: Collections, Records, and GOTO

## Overview
This project demonstrates the use of:
- **PL/SQL Collections**
  - Associative Array
  - Nested Table
  - VARRAY
- **PL/SQL Records**
  - User-Defined RECORD Type
  - `%ROWTYPE` (table-based record)
- **GOTO Statement**
  - Used to skip processing of invalid data in a controlled way

## 1. CREATING TABLE AND INSERTING DATA
<img width="1920" height="1080" alt="creating table" src="https://github.com/user-attachments/assets/c52cf11f-305c-49c9-a969-ccea9b3c3062" />
<img width="1920" height="1080" alt="Inserting" src="https://github.com/user-attachments/assets/bfa4f8a8-6f05-4239-84a5-90f94b4608d0" />

## PL/SQL Collections
```sql
DECLARE
  TYPE salary_list IS TABLE OF NUMBER;
  v_salaries salary_list := salary_list();
  v_total NUMBER := 0;
BEGIN
  -- Fill nested table with salary values
  FOR emp IN (SELECT salary FROM employees WHERE salary > 0) LOOP
    v_salaries.EXTEND;
    v_salaries(v_salaries.COUNT) := emp.salary;
  END LOOP;

  -- Display data
  DBMS_OUTPUT.PUT_LINE('--- PL/SQL COLLECTION (NESTED TABLE) OUTPUT ---');
  FOR i IN 1 .. v_salaries.COUNT LOOP
    DBMS_OUTPUT.PUT_LINE('Salary['||i||'] = ' || v_salaries(i));
    v_total := v_total + v_salaries(i);
  END LOOP;

  DBMS_OUTPUT.PUT_LINE('Total Salary = ' || v_total);
END;
/
```
<img width="1920" height="1080" alt="PLSQL Collection" src="https://github.com/user-attachments/assets/a68147a8-708e-47cc-85e0-951847f5ce75" />

