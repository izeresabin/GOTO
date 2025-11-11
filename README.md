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

## PL/SQL Records
```sql
DECLARE
  -- User-defined record
  TYPE emp_record IS RECORD (
    fname employees.first_name%TYPE,
    lname employees.last_name%TYPE,
    sal   employees.salary%TYPE
  );
  v_rec emp_record;

  -- %ROWTYPE record
  v_row employees%ROWTYPE;
BEGIN
  -- Fill custom record
  SELECT first_name, last_name, salary
  INTO v_rec
  FROM employees
  WHERE employee_id = 1;

  DBMS_OUTPUT.PUT_LINE('--- USER-DEFINED RECORD OUTPUT ---');
  DBMS_OUTPUT.PUT_LINE(v_rec.fname || ' ' || v_rec.lname || ' earns ' || v_rec.sal);

  -- Fill %ROWTYPE record
  SELECT *
  INTO v_row
  FROM employees
  WHERE employee_id = 2;

  DBMS_OUTPUT.PUT_LINE(CHR(10) || '--- %ROWTYPE RECORD OUTPUT ---');
  DBMS_OUTPUT.PUT_LINE(v_row.first_name || ' ' || v_row.last_name || ' earns ' || v_row.salary);
END;
/
```
<img width="1920" height="1080" alt="PLSQL Records" src="https://github.com/user-attachments/assets/fc455bf0-43a6-44fd-a336-e52152b08e16" />

## GOTO Statement
```sql
BEGIN
  DBMS_OUTPUT.PUT_LINE('--- GOTO STATEMENT OUTPUT ---');

  FOR emp IN (SELECT first_name, salary FROM employees) LOOP
    IF emp.salary <= 0 THEN
      GOTO skip_salary;
    END IF;

    DBMS_OUTPUT.PUT_LINE(emp.first_name || ' has valid salary: ' || emp.salary);
    CONTINUE;

    <<skip_salary>>
    DBMS_OUTPUT.PUT_LINE(emp.first_name || ' skipped due to invalid salary.');
  END LOOP;

END;
/
```
<img width="1920" height="1080" alt="GOTO Statement" src="https://github.com/user-attachments/assets/a6641e8c-b150-4ff2-913f-2ac6e668eedb" />

# PL/SQL Program output
```sql
SET SERVEROUTPUT ON;
DECLARE
  TYPE dept_stat_rec IS RECORD (
    emp_count NUMBER,
    total_salary NUMBER
  );
  TYPE dept_stat_tab IS TABLE OF dept_stat_rec INDEX BY PLS_INTEGER;
  v_dept_stats dept_stat_tab;

  TYPE salary_nt IS TABLE OF NUMBER;
  v_salaries salary_nt := salary_nt();

  TYPE top5_varray IS VARRAY(5) OF NUMBER;
  v_top5 top5_varray := top5_varray();

  TYPE employee_rec IS RECORD (
    emp_id      employees.employee_id%TYPE,
    first_name  employees.first_name%TYPE,
    last_name   employees.last_name%TYPE,
    salary      employees.salary%TYPE,
    dept_id     employees.department_id%TYPE
  );
  v_emp employee_rec;

  CURSOR c_emps IS
    SELECT employee_id, first_name, last_name, salary, department_id
    FROM employees;

  c_emp_rec c_emps%ROWTYPE;

BEGIN
  OPEN c_emps;
  LOOP
    FETCH c_emps INTO c_emp_rec;
    EXIT WHEN c_emps%NOTFOUND;

    v_emp.emp_id := c_emp_rec.employee_id;
    v_emp.first_name := c_emp_rec.first_name;
    v_emp.last_name := c_emp_rec.last_name;
    v_emp.salary := c_emp_rec.salary;
    v_emp.dept_id := c_emp_rec.department_id;

    IF v_emp.salary <= 0 THEN
      DBMS_OUTPUT.PUT_LINE('Skipping (invalid salary): ' || v_emp.first_name);
      GOTO skip_employee;
    END IF;

    v_salaries.EXTEND;
    v_salaries(v_salaries.COUNT) := v_emp.salary;

    IF v_dept_stats.EXISTS(v_emp.dept_id) THEN
      v_dept_stats(v_emp.dept_id).emp_count := v_dept_stats(v_emp.dept_id).emp_count + 1;
      v_dept_stats(v_emp.dept_id).total_salary := v_dept_stats(v_emp.dept_id).total_salary + v_emp.salary;
    ELSE
      v_dept_stats(v_emp.dept_id).emp_count := 1;
      v_dept_stats(v_emp.dept_id).total_salary := v_emp.salary;
    END IF;

    <<skip_employee>>
    NULL;
  END LOOP;
  CLOSE c_emps;

  DBMS_OUTPUT.PUT_LINE(CHR(10) || '--- DEPARTMENT SUMMARY ---');
  DECLARE
    i PLS_INTEGER;
  BEGIN
    i := v_dept_stats.FIRST;
    WHILE i IS NOT NULL LOOP
      DBMS_OUTPUT.PUT_LINE('Dept '||i||': Count='||v_dept_stats(i).emp_count||' Total='||v_dept_stats(i).total_salary);
      i := v_dept_stats.NEXT(i);
    END LOOP;
  END;

END;
/
```
<img width="1920" height="1080" alt="OUTPUT" src="https://github.com/user-attachments/assets/c26ef976-abed-4c11-b03c-dfd74a24efc8" />

 

