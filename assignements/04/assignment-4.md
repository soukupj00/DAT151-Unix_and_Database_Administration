# DAT151: Assignment 4 Report

**Group:** 5

**Group Members:** Soukup Jan, Fabienne Feilke

**Date:** March ..., 2026

---

## Task 1: Triggers

*Objective: As a normal database user (i.e. not root), create triggers on table TheTable so that all changes of TheTable will be logged in LogTable. Make sure all values before and after a change are logged.*

**Requirements:**
* For DELETE, you must log the action DELETE, and the row before values.
* For INSERT, you must log the action INSERT, and the row after values.
* For UPDATE, you must log the action UPDATE, and both the row before and after values.
    * The LogTable must clearly identify the old and the new values.

**Delivery:**
* The DDL of the triggers
* The DDL of the tables
* Output that demonstrates the working of the triggers

---

## Task 2: Temporal database

*Objective: As a normal database user (i.e. not root), create a table AnotherTable as a System-Versioned table. Insert some data, then do some DELETEs, UPDATEs and INSERTs.*

**Requirements:**
* Do not manually create a ROW_START or ROW_END column.
* After all modifications are done, use queries with “FOR SYSTEM_TIME AS OF” to list the before and after values after each modification of the table.
* Every result set should show the state of the table at a specific point in time. Each result sets should include a row only once, showing the state of the row at that specific point in time.

---

## Task 3: Integrity Constraints

*Objective: As a normal database user (i.e. not root), create a teacher table, with numeric attributes for salary, bonus and total.*

**1. Implement the following integrity constraints:**
i. The attribute salary must be between 1 000 and 100 000.
ii. The attribute total must be the sum of salary and bonus, and calculated automatically.

*Insert some test data to verify whether your solution works. Document the output.*

**2. Is the table 3NF?**

---

## Task 4: Order of triggers

**1. Test the following, and give necessary explanations:**
a) Create three tables T1, T2, and T3.
b) Create a trigger tr12 before insert on T1, insert into T2 a row.
c) Create a trigger tr23 after insert on T2, insert into T3 a row.
d) Create a trigger tr13 after insert on T1, insert into T3 a row.
e) Insert a row into T1, in which order are the triggers fired? Why?

**2. Can there be any unexpected result or deadlock regarding the order of triggers?**

---

## Task 5: Pendant DELETE

*Objective: Implement the following as a normal database user (i.e. not root):*

1. Create two tables with a one-to-many relation, and implement referential integrity. (Create foreign keys on the “many”-table).
2. Add some rows to each table. The child table must have several rows referencing each row of the parent rows.
3. Use triggers to implement the rule known as pendant DELETE (Mullins page 435): A row in the parent table should be deleted if no rows in the child table refer to it any more (when you delete the last row in the child table referencing it).

**Explain the point of doing this and why it is difficult compared with other rules on p.435.**

---

## Task 6: Concurrency

*Objective: An application uses a DBMS to store data about events and participants. Events can have many participants, but the number of spaces should still be limited.*

**6.1 Normalization and Data Loading**

1. Make a normalized, 3NF logical for the data provided in `data.txt` and implement the model in a MariaDB database using InnoDB tables. The model should have an attribute showing the total number of spaces at each event.
*(The fastest, and probably also easiest solution is to create a table that temporarily holds all the data. From this table, load data into the normalized tables and afterwards delete the auxiliary table.)*

**6.2 Validation Queries**

2. Before continuing, you must confirm that your model is working, and that the data got correctly loaded into the database. Formulate the queries below and test your database.

i. Find the maximum number of events that a single participant will attend. (Avoid ORDER BY)
ii. List names of the participants that attend the largest number of events. (Do not use LIMIT, Avoid ORDER BY)
iii. What events are attended by Ludvig Rustad?
iv. List participants that attend exactly 3 events.
v. What events do they attend, the participants that attend 3 events? (Your query should not refer explicitly to any events, nor eventId equal to 5)
vi. Are there any events that is not attended by anybody that attends three events? (Your query should not refer explicitly any of the events)
vii. Are there any events that is attended only by the participants that attend only one event? (Your query should not refer explicitly any of the events)

**6.3 Stored Procedure takeSpace**

3. Make a stored procedure `takeSpace` that will book a space for an existing participant at an event.
* If there are available spaces at the event, book a space for the participant.
* If the event is sold out, do nothing.
* The procedure should be accessible by concurrent users, i.e. locking is required with a 3NF model.
* The stored procedure should not modify column totSpaces of the table Event.

**6.4 Profiling**

4. Start profiling and check the time consumption used by the stored procedure when booking a space at an event. What takes most time?
* Discuss how multiple concurrent bookings for the same event would influence the time consumption.

**6.5 Denormalisation**

5. Can you advice a denormalisation scheme that would reduce the time consumption?
* Implement the new model, and update `takeSpace` to take the new model into account. Try to avoid the need for explicit locks in the updated version of `takeSpace`.
* Use profiling and measure the time consumption when booking a space at an event with the new version `takeSpace`.
