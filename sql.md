# SQL Basics 

Examples taken from `236363 - Databases` course in Technion - Israeli Institute of Technology. Credit to course staff. 

## Env

Use https://sqliteonline.com/ and PostgreSQL.

## Table Creation

Create the following tables:

```sql
create table Student(sid int, name varchar, year int);
insert into Student values
    (861, 'Alma', 2),
    (753, 'Amir', 1),
    (955, 'Ahuva', 2),
    (699, 'Adi', 2);
```

```sql
create table Enroll(sid int, course varchar, grade int);
insert into Enroll values
    (861, 'DB', 79),
    (861, 'PL', 82),
    (753, 'PL', 93),
    (955, 'DB', 99),
    (955, 'AI', 72),
    (699, 'DB', 95);
```

```sql
create table Course(name varchar, credit int);
insert into Course values
    ('DB', 3),
    ('PL', 2),
    ('AI', 3);
```

```sql
create table Employee(sid int, name varchar);
insert into Employee values
    (233, 'Alma'),
    (651, 'Avia'),
    (122, 'Avi');
```

## Basic Queries - SELECT FROM WHERE

- Basic selection - bag semantics
    ```sql
    SELECT name, course
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ```

- We use `DISTINCT` for set semantics.

    ```sql
    SELECT DISTINCT name
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ```

- Use `*` to project on all attributes
    ```sql 
    SELECT *
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ```

- We use `AS` for attribute renaming
    ```sql
    SELECT name AS student, course AS coursename
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ```

- We can use functions on attributes, and we can add columns with constant values (i.e. 'great'):

    ```sql
    SELECT sid,
    course,
    credit*grade AS cg,
    'great' as comment
    FROM Course, Enroll
    WHERE grade>69
    ```

- Use the same relation multiple times with different names:

    ```sql
    SELECT Student.sid, name
    FROM Student, Enroll E, Enroll F
    WHERE Student.sid = E.sid AND
    Student.sid = F.sid AND
    E.course='DB' AND
    F.course='PL'
    ```

- `WHERE` clause operators - https://www.w3schools.com/sql/sql_operators.asp
  
    ```sql
    SELECT *
    FROM Enroll
    WHERE grade between 70 AND 95 AND course <> 'PL'
    ```
    
    ```sql
    SELECT *
    FROM Enroll
    WHERE course IN ('PL', 'OS', 'AI')
    ```

## Set Operations

- Set semantics:
    ```sql
    (SELECT NAME FROM Student)
    UNION           
    (SELECT NAME FROM Employee)
    ```

- Bag semantics

    ```sql
    (SELECT name FROM Student)
    UNION ALL
    (SELECT name FROM Employee)
    ```

## Tuple order

- General `ORDER BY`
  
    ```sql
    SELECT *
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ORDER BY name, course
    ```

- Ordering direction - ascending vs descending:

    ```sql
    SELECT *
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ORDER BY name ASC, course DESC
    ```

- Ordering attributes don't have to be in the list of attributes we project, meaning, we first order (`ORDER BY`) then project (`SELECT`):

    ```sql
    SELECT Student.sid, course
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ORDER BY name, course
    ```

- We can limit the number of tuples in the result (Top K):

    ```sql
    SELECT *
    FROM Student, Enroll
    WHERE Student.sid = Enroll.sid
    ORDER BY name ASC, course DESC
    LIMIT 3
    ```

## Aggregations

- Basic aggregates

    ```sql
    SELECT COUNT(sid) as num,
    MAX(grade) as max
    FROM Enroll
    WHERE course='PL'
    ```

- Basic grouping - Average per course:

    ```sql
    SELECT course, ROUND(AVG(grade), 1) as avg
    FROM Enroll
    GROUP BY course
    ```

- Grouping with conditions - average of 2nd year students with at least 5 credit  points

    ```sql
    SELECT S.name,
    SUM(T.grade*C.credit)*1.0/SUM(C.credit) as average
    FROM Student S, Enroll T, Course C
    WHERE S.sid=T.sid AND T.course = C.name
    GROUP BY S.sid, S.name, S.year
    HAVING S.year=2 AND SUM(C.credit)>=5;
    ```

    or 

    ```sql
    SELECT S.name,
    SUM(T.grade*C.credit)*1.0/SUM(C.credit) as average
    FROM Student S, Enroll T, Course C
    WHERE S.sid=T.sid AND T.course = C.name AND S.year=2
    GROUP BY S.sid, S.name
    HAVING SUM(C.credit)>=5;
    ```

## Tutorial 

Start from scratch at https://sqliteonline.com/ and PostgreSQL.

### Create Library tables

1. `Customers`
```sql
create table Customers(CustId int, CustName varchar, Faculty varchar, primary key (CustId));

insert into Customers values
    (12345, 'Moshe Cohen', 'CS'),
    (23456, 'Avi Barak', 'EE'),
    (34567, 'Avi Barak', 'MED'),
    (45678, 'Lior Edri', 'EE'),
    (56789, 'Moshe Cohen', 'EE'),
    (67890, 'Moshe Cohen', 'EE');
```

2. `Books`

```sql
create table Books(BookId int, BookName varchar, Year int, MaxTime int, Pages int, Faculty varchar, primary key (BookId));

insert into Books values
    (1111, 'Database Systems', 1998, 7, 348, 'CS'),
    (1112, 'Database Systems', 1998, 14, 348, 'CS'),
    (1113, 'Database Systems', 2001, 7, 424, 'CS'),
    (2222, 'Database And Knowledge', 1998, 1, 390, 'CS'),
    (2223, 'Database And Knowledge', 1998, 7, 390, 'EE'),
    (3333, 'Electronic Circuits', 1998, 21, 180, 'EE'),
    (4444, 'Genes 7', 1985, 7, 580, 'MED'),
    (5555, 'Anatomy', 1988, 7, 450, 'MED');

```

3. `Ordered`: Note the use of ForeignKey via `references` and `on delete cascade`
```sql
create table Ordered(CustId int references Customers on delete cascade, BookId int references Books on delete cascade, OrderDate Timestamp, primary key (CustId, BookId));

insert into Ordered values 
    (12345, 1111, '2002-10-14 00:00:00'),
    (45678, 1112, '2002-10-24 00:00:00'),
    (12345, 1113, '2002-10-30 00:00:00'),
    (45678, 2222, '2002-10-12 00:00:00');
```

4. `Borrowed`: note the `NULL` during the insert.
```sql
create table Borrowed(BookId int references Books on delete cascade, CustId int references Customers on delete cascade, FromDate Timestamp, ToDate Timestamp, primary key (BookId, CustId));

insert into Borrowed values
    (5555, 56789, '2002-10-13 00:00:00');

```

### Create Table from Query

```sql
create table CSBooks as 
select BookId, BookName 
from Books
where Faculty = 'CS';
```

or 

```sql
create table CSBooks2(Id, Name) as 
select BookId, BookName 
from Books
where Faculty = 'CS';
```

### Views

```sql
create view CsBooksView as
select BookId, BookName, MaxTime
from Books
where Faculty = 'CS';
```

Note that you can query `CsBooksView` and this is under the section of `views`.
However if you create a `MATERIALIZED VIEW`:

```sql
drop view CsBooksView;

create materialized view CsBooksView as
select BookId, BookName, MaxTime
from Books
where Faculty = 'CS';
```

it won't show under views but it will still exist:

```sql
select * from CsBooksView
```

### Insert 

Insert via a query:

```sql
create table Readers(CustId int references Customers on delete cascade);

insert into Readers
select CustId from Ordered;
```

### Update

- Update everything:
    ```sql
    update Books
    set MaxTime = 7
    ```
- Update with a filter:
    ```sql
    update Books
    set Faculty = 'GEN'
    WHERE Faculty = 'MED'
    ```
- Set an expression:
    ```sql
    update Books
    set MaxTime = MaxTime+1
    ```

### Delete


- Delete with a filter:
    ```sql
    delete from Ordered
    where CustId = 12345
    ```
- Delete everything:
    ```sql 
    delete from Ordered
    ```

Note the table still exists, only tuples are deleted. You can delete the table  using:

```sql
drop table Ordered
```