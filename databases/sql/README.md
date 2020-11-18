# SQL

## How to paginate results

```sql
SELECT * FROM this_table LIMIT 0, 20
```
### ORACLE version
```sql
SELECT *
FROM user
ORDER BY first_name
OFFSET 5 ROWS FETCH NEXT 10 ROWS ONLY;

SELECT *
FROM user
ORDER BY first_name
FETCH FIRST 5 ROWS ONLY
```

Detailed syntax:
```
[ OFFSET offset { ROW | ROWS } ]
[ FETCH { FIRST | NEXT } [ { rowcount | percent PERCENT } ]
    { ROW | ROWS } { ONLY | WITH TIES } ]
```
- Using the ONLY clause limits the number of rows returned to the exact number requested.
- Using the WITH TIES clause may result in more rows being returned if multiple rows match the value of the Nth row

Resource: [https://oracle-base.com/articles/12c/row-limiting-clause-for-top-n-queries-12cr1]

## How to query recursive records

### PostgreSql
```sql
WITH RECURSIVE menu_tree 
  (id, name, url, level, parent_node_id, seq) 
AS ( 
  SELECT
    id, 
    name, 
    '' || url_path, 
    0, 
    parent_node_id, 
    1 
  FROM menu_node 
  WHERE parent_node_id is NULL
 
  UNION ALL
  SELECT
    mn.id, 
    mn.name, 
    mt.url || '/' || mn.url_path, 
    mt.level + 1, 
    mt.id, 
    mn.seq 
  FROM menu_node mn, menu_tree mt 
  WHERE mn.parent_node_id = mt.id 
) 
SELECT * FROM menu_tree 
WHERE level > 0 
ORDER BY level, parent_node_id, seq;
```
Resource: [https://learnsql.com/blog/do-it-in-sql-recursive-tree-traversal/]

### Oracle
1) as postgres without RECURSIVE keyword
2) only for oracle above version 11g release 2:
```sql
SELECT
  id, 
  name,
  SYS_CONNECT_BY_PATH(url_path, '/') AS url, 
  LEVEL, 
  parent_node_id, 
  seq 
FROM menu_node 
START WITH id IN
  (SELECT id FROM menu_node WHERE parent_node_id IS NULL) 
CONNECT BY PRIOR id = parent_node_id;
```
Resource: [https://learnsql.com/blog/do-it-in-sql-recursive-tree-traversal/]

### SQL Server
```sql
USE AdventureWorks;
GO
WITH DirectReports (ManagerID, EmployeeID, Title, DeptID, Level)
AS
(
-- Anchor member definition
    SELECT e.ManagerID, e.EmployeeID, e.Title, edh.DepartmentID, 
        0 AS Level
    FROM HumanResources.Employee AS e
    INNER JOIN HumanResources.EmployeeDepartmentHistory AS edh
        ON e.EmployeeID = edh.EmployeeID AND edh.EndDate IS NULL
    WHERE ManagerID IS NULL
    UNION ALL
-- Recursive member definition
    SELECT e.ManagerID, e.EmployeeID, e.Title, edh.DepartmentID,
        Level + 1
    FROM HumanResources.Employee AS e
    INNER JOIN HumanResources.EmployeeDepartmentHistory AS edh
        ON e.EmployeeID = edh.EmployeeID AND edh.EndDate IS NULL
    INNER JOIN DirectReports AS d
        ON e.ManagerID = d.EmployeeID
)
-- Statement that executes the CTE
SELECT ManagerID, EmployeeID, Title, Level
FROM DirectReports
INNER JOIN HumanResources.Department AS dp
    ON DirectReports.DeptID = dp.DepartmentID
WHERE dp.GroupName = N'Research and Development' OR Level = 0;
GO
```
Resource: [https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008/ms186243(v=sql.100)]

## CTE

A common table expression (CTE) can be thought of as a temporary result set that is defined within the execution scope of a single SELECT, INSERT, UPDATE, DELETE, or CREATE VIEW statement. A CTE is similar to a derived table in that it is not stored as an object and lasts only for the duration of the query. Unlike a derived table, a CTE can be self-referencing and can be referenced multiple times in the same query.

A CTE can be used to:

- Create a recursive query. For more information, see Recursive Queries Using Common Table Expressions.

- Substitute for a view when the general use of a view is not required; that is, you do not have to store the definition in metadata.

- Enable grouping by a column that is derived from a scalar subselect, or a function that is either not deterministic or has external access.

- Reference the resulting table multiple times in the same statement.

Using a CTE offers the advantages of improved readability and ease in maintenance of complex queries. The query can be divided into separate, simple, logical building blocks. These simple blocks can then be used to build more complex, interim CTEs until the final result set is generated.

CTEs can be defined in user-defined routines, such as functions, stored procedures, triggers, or views.

### STRUCTURE

A CTE is made up of an expression name representing the CTE, an optional column list, and a query defining the CTE. After a CTE is defined, it can be referenced like a table or view can in a SELECT, INSERT, UPDATE, or DELETE statement. A CTE can also be used in a CREATE VIEW statement as part of its defining SELECT statement.

The basic syntax structure for a CTE is:

WITH expression_name [ ( column_name [,...n] ) ]

AS

( CTE_query_definition )

The list of column names is optional only if distinct names for all resulting columns are supplied in the query definition.

The statement to run the CTE is:

SELECT <column_list>

FROM expression_name;

### EXAMPLE:
```sql
USE AdventureWorks;
GO
WITH Sales_CTE (SalesPersonID, NumberOfOrders, MaxDate)
AS
(
    SELECT SalesPersonID, COUNT(*), MAX(OrderDate)
    FROM Sales.SalesOrderHeader
    GROUP BY SalesPersonID
)
SELECT E.EmployeeID, OS.NumberOfOrders, OS.MaxDate,
    E.ManagerID, OM.NumberOfOrders, OM.MaxDate
FROM HumanResources.Employee AS E
    JOIN Sales_CTE AS OS
    ON E.EmployeeID = OS.SalesPersonID
    LEFT OUTER JOIN Sales_CTE AS OM
    ON E.ManagerID = OM.SalesPersonID
ORDER BY E.EmployeeID;
GO
```
Resource: [https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008/ms186243(v=sql.100)]

# Managing Hierarchical Data
## The Adjacency List Model
In the adjacency list model, each item in the table contains a pointer to its parent. The topmost element has a NULL value for its parent. The adjacency list model has the advantage of being quite simple. 
While the adjacency list model can be dealt with fairly easily in client-side code, working with the model can be more problematic in pure SQL.
### RETRIEVING A FULL TREE
```sql
SELECT t1.name AS lev1, t2.name as lev2, t3.name as lev3, t4.name as lev4
FROM category AS t1
LEFT JOIN category AS t2 ON t2.parent = t1.category_id
LEFT JOIN category AS t3 ON t3.parent = t2.category_id
LEFT JOIN category AS t4 ON t4.parent = t3.category_id
WHERE t1.name = 'ELECTRONICS';
```
### FINDING ALL THE LEAF NODES
```sql
SELECT t1.name FROM
category AS t1 LEFT JOIN category as t2
ON t1.category_id = t2.parent
WHERE t2.category_id IS NULL;
```
### RETRIEVING A SINGLE PATH
```sql
SELECT t1.name AS lev1, t2.name as lev2, t3.name as lev3, t4.name as lev4
FROM category AS t1
LEFT JOIN category AS t2 ON t2.parent = t1.category_id
LEFT JOIN category AS t3 ON t3.parent = t2.category_id
LEFT JOIN category AS t4 ON t4.parent = t3.category_id
WHERE t1.name = 'ELECTRONICS' AND t4.name = 'FLASH';
```
### LIMITATIONS OF THE ADJACENCY LIST MODEL
Working with the adjacency list model in pure SQL can be difficult at best. Before being able to see the full path of a category we have to know the level at which it resides. In addition, special care must be taken when deleting nodes because of the potential for orphaning an entire sub-tree in the process (delete the portable electronics category and all of its children are orphaned). Some of these limitations can be addressed through the use of client-side code or stored procedures. With a procedural language we can start at the bottom of the tree and iterate upwards to return the full tree or a single path. We can also use procedural programming to delete nodes without orphaning entire sub-trees by promoting one child element and re-ordering the remaining children to point to the new parent.

## The Nested Set Model
 In the Nested Set Model, we can look at our hierarchy in a new way, not as nodes and lines, but as nested containers.
 We represent this form of hierarchy in a table through the use of left and right values to represent the nesting of our nodes:
```sql
 CREATE TABLE nested_category (
        category_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(20) NOT NULL,
        lft INT NOT NULL,
        rgt INT NOT NULL
);
```
When working with a tree, we work from left to right, one layer at a time, descending to each node’s children before assigning a right-hand number and moving on to the right. This approach is called the modified preorder tree traversal algorithm.
### RETRIEVING A FULL TREE
```sql
SELECT node.name
FROM nested_category AS node,
        nested_category AS parent
WHERE node.lft BETWEEN parent.lft AND parent.rgt
        AND parent.name = 'ELECTRONICS'
ORDER BY node.lft;
```
### FINDING ALL THE LEAF NODES
```sql
SELECT name
FROM nested_category
WHERE rgt = lft + 1;
```
### RETRIEVING A SINGLE PATH
```sql
SELECT parent.name
FROM nested_category AS node,
        nested_category AS parent
WHERE node.lft BETWEEN parent.lft AND parent.rgt
        AND node.name = 'FLASH'
ORDER BY parent.lft;
```

Resource: [http://mikehillyer.com/articles/managing-hierarchical-data-in-mysql/]