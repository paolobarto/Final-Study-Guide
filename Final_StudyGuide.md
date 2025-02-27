# Final Study Guide
---
List of things to study from recording:
1. Outer join (Symmertric difference through symmetric difference)
2. SQL Set opertaions
   1. Aggrigate handling null
   2. Nested Querry
   3. Integrity constraints
3. ERD Design Cardinality
4. JDBC
   1. Update table through jdbc with pre-written code. 
5. Writing Triggers to include some integrity constraint (PL/SQL)
6. OLAP
   1. Most likely test cube
7. Normalization decomposition algorithm
   1. BCNF
   2. 3NF 
   3. (He says Memorization is not important but practice is the most important)
8. Functional dependencies
   1. Find primary key/canidate keys
   2. 3 armstrong axioms
9. Depenedency preservation / Losslessness 
10. Storage
    1.  Formulas
        1.  Cost of accessing data
        2.  Seeking time
        3.  rotational time
        4.  buffer within memory
    2. RAID 
 11. Index types
     1.  B+ tree index
         1.  How to operate on B+ tree
     2.  Hash index
     3.  Sparse / dense index
     4.  Primary / secondary index
     5.  Advantages or disadvantages 
 12. Querry processing 
     1.  Is the right algorithm used for the right situation 
     2.  Be able to connect formuals to real life connection
11. Recovery algorithm 
12. Async properties of transactions
13. Stable storage / checkpoints 
14. Concurrency 
    1.  Two-phase locking
    2.  Granularity
    3.  Deadlock
15. Phantoms 
16. Distributed and parallel querry processing

---
<!-- TODO add data for all joins -->
## 1 Outer join 

Outer join can be used for symmetric difference between the opposing side. Natually biasing the side in the description of 

* Left outer join
* Right outer join
* Full outer join

`natural left outer join` is the correct syntax of outer natual join

With this syntax, the side of the comparison that is used in the name will exist as the full table, and each of the other comparisons will have null when not populated.

---
## 2 SQL Set Operations

### Aggrigate Handling null

Things like `not(1<null)` would evaulate to true 

* **and**: The result of true **and** unknown is unknown, false **and** unknown is false, while unknown **and** unknown is unknown.
* **or**: The result of true **or** unknown is true, false **or** unknown is unknown, while unknown **or** unknown is unknown.
* **not**: The result of **not** unknown is unknown.

Any situation where the the outcome of the logical expression is explicity given by the other term, the solution will be unknown. 

In a aggrigation querry, null values are considered to be ignored in all situations except `count(*)`

### Nested Querry

Nested subquerries are tools used for performing set based operations by creating additional groups within the where clause of another querry. 

Using commands such as `in` and `not in` subquerries can be used for comparison

```sql
select distinct course id
from section
where semester = 'Fall' and year= 2017 and
course id not in (select course id
                from section
                where semester = 'Spring' and year= 2018);
```

Methods such as 
* all 
* some >1
* exists >=1
* not exists
* except 
* unique
---
## 2.5 Integrity Constraints 
Integrity constraints ensure changes made to the datbase by authorized users do not result in a loss of data consistency. Thus, integrity constraints guard against accidental damage to the datbase.

**Basic forms of integrity constraints**
* not null
* unique
* check(\<predicate\>)

``` sql
name varchar(20) not null
budget numeric(12,2) not null
```

**Check Predicate**
```sql
create table section
    (course id varchar (8),
    sec id varchar (8),
    semester varchar (6),
    year numeric (4,0),
    building varchar (15),
    room number varchar (7),
    time slot id varchar (4),
    primary key (course id, sec id, semester, year),
    check (semester in ('Fall', 'Winter', 'Spring', 'Summer')));
```
---
## 3 ERD Design
### Cardinality

* One to one
  * An entity in A is associated with at most one entity in B, and an entity
in B is associated with at most one entity in A.

    ![](./images/One-to-one.png)
* One to many
  * An entity in A is associated with any number (zero or more) of entities in B. An entity in B, however, can be associated with at most one entity in A.
  * 
    ![](images/One-to-many.png)
* Many to one
  * An entity in A is associated with at most one entity in B. An entity
in B, however, can be associated with any number (zero or more) of entities in A

    ![](./images/Many-to-one.png)
* Many to many
  * An entity in A is associated with any number (zero or more) of
entities in B, and an entity in B is associated with any number (zero or more) of
entities in A

    ![](./images/Many-to-many.png)

---

## 4. JDBC 

### Updating Table in JDBC 

```java
  public static void main(String[] args) {
    try (
         Connection con=DriverManager.getConnection("jdbc:oracle:thin:@edgar1.cse.lehigh.edu:1521:cse241", user_name, new String(pwd));
         Statement s=con.createStatement();
         ) {
           String q;
           ResultSet result;
           int i;
           q = "create table enemies2 (dog_name varchar(20), weight numeric(9,2))";
           i = s.executeUpdate(q);
           System.out.println ("value returned: " + i);
         } catch(Exception e){e.printStackTrace();}
  }
```
---

## 5 Writing Triggers to include integrity contraints


**Trigger** is a statement that the system executes automatically as a side effect of a modification to the database 

1. Specify when a trigger is to be executed. Broken up into an event  and condition
2. Specify the actions to be taken when the trigger executes

Triggers can be used to implement integrity constaints that are not possible through nomral sql commands 

Example of trigger update
```sql 
    /*From In-class examples */
    create table TC (
    id number(7),
    name varchar(20)
    );

    create table tmp (
    id number(7),
    semester varchar(5),
    name varchar(20)
    );

    -- add new tuple to tmp when an new tuple is inserted into TC
    create or replace trigger add_tc after insert on TC
    referencing new as new old as old
    for each row
    begin
    insert into tmp (id, semester, name) values (:new.id, 'Fall', :new.name);
    end;

    insert into TC (id, name) values (10109,'abk');

    create trigger timeslot_check1 after insert on section
    referencing new row as nrow
    for each row
    when (nrow.time_slot_id not in (
            select time slot_id
            from time_slot)) /* time slot id not present in time slot */
    begin
        rollback
    end;

    create trigger timeslot_check2 after delete on timeslot
    referencing old row as orow
    for each row
    when (orow.time_slot_id not in (
            select time slot_id
            from time_slot) /* last tuple for time slot id deleted from time slot */
        and orow.time_slot_id in (
            select time slot_id
            from section)) /* and time slot id still referenced from section*/
    begin
        rollback
    end;
```
In a more simplistic fashion

```sql
create or replace trigger trigger_name [after/before] [insert/update/delete] of table_name on column_name

referencing old row as orow, new row as nrow
for each row
when /*predicate, if needed*/

 declare /*if needed*/
 variable_name type :=0

 begin
    /*process to be completed on rows*/
    insert into tmp (id, semester, name) values (:nrow.id, 'Fall', :nrow.name);

    select count(*) into variable_name
    from table_name
    if variable_name<1 then 
        raise_application_error(-20000,:nrow.dept_name||' not in dept_budget'); 
    end if;

    /*the : is used when referring to values in current row*/
    if :nrow.salary<0 or :nrow.salary is null then
    :nrow.salary := 0;
  end if;
end;
```
---
## 6 OLAP 

### Ranking

```sql
select ID, rank() over (order by GPA desc) as s_rank
from student_grades
```

### Cube

Will create groupings based on every possible combination of included sets

```sql
select item name, color, clothes size, sum(quantity)
from sales
group by cube(item name, color, clothes size);

{ (item name, color, clothes size), (item name, color), (item name, clothes size),
(color, clothes size), (item name), (color), (clothes size), () }

```

TODO insert formula for cube row number

## 7 Normalization Decomposition

### 1. BCNF
A given table is in BCNF if and only if it is in 3NF and for each non-trivial FD X->Y, X is a super key (i.e. X<sup>+</sup> covers all attributes in table). In other words, it is 3NF form but the first option/condition for 3NF is necessary.
BCNF doesn't ensure dependency preservation (unlike 3NF), but it guarantees a lossless join.

![Image](./images/bcnf.png)

### 2. 3NF
A table is in 3NF iff for each non-trivial FD at least one of the following holds:
   1. LHS is super key
   2. RHS is prime attribute
(i.e. is in 2NF and no transitive dependencies for non-prime attributes. NPA -> NPA not allowed). 3NF ensures dependency preservation but there is a possibility for redundancy.

![Image](./images/3nf.png)


---
## 8 Functional Dependencies

### 1. Find Candidate Keys
The candidate key is a minimal set of attributes that can determine every attribute in the table. This can be seen by finding the closure of candidate key (CK<sup>+</sup>). A method for finding a candidate key is to start with the closure of all attributes (Ex: ABCDEF+), then remove attributes incrementally that can be found from the remaining attributes' FDs.

### 2. Armstrong Axioms

## 9 Dependency Preservation and Losslessness

### 1. Dependency Preservation
A decomposition is dependency preserving if the closure of (F<sub>1</sub> U F<sub>2</sub> U ... F<sub>n</sub>) is equivalent to the closure of F. F<sub>1</sub>,F<sub>2</sub>.. being the set of FD's that include only attributes in their respective tables.. . In other words, can the decomposed tables' FDs be rewritten (using Armstrong axioms) to hold the same logic of original set of FDs, F?

### 2. Losslessness
A decomposition is lossless if the union of all tables ( ∀ R<sub>i</sub>), equals the original table R. This is held if every table is connected through a foreign and primary key, which then can determine all attributes in each R<sub>i</sub>.


---

## 10. Storage
### 1.  Formulas
#### 1.  Cost of accessing data
#### 2.  Seeking time
#### 3.  rotational time
#### 4.  buffer within memory
### 2. RAID 
#### RAID 0 - Block striping; non-redundant.
Blocks are sequentially distributed to each disk in RAID 0.
Parallelism makes RAID 0 high-performance, but data loss is common. RAID 0 doesnt prevent data loss.
Example: 
  Blocks of data are denoted as letters
  File ABCDEF
  Disk1(ACE)
  Disk2(BDF)

#### RAID 1 - Mirrored disks with block striping
Essentially RAID 0, but copied to another set of disks.
Offers best write performance and is data loss preventative.
Example:
  File ABCDEF
  Disk1(ACE)
  Disk2(BDF)
  Disk3(ACE)
  Disk4(BDF)
#### RAID 5 - Block-Interleaved Distributed Parity
Parity blocks help reconstruct data between each parity.
Block writes occur in parallel for multiple disk setups -> high write performance
![Image](./images/raid5.png)

## 11 Index Types

### 1.  B+ tree index
B+ trees have multilevel indexing, where nonleaf levels of the tree form a hierarchy of sparse indices. The leaf nodes are dense indices connected by sequential pointers. Each node can hold up to n pointers. A leaf node has between ceiling((n-1)/2) and n-1 values.
![Image](./images/b+node.png)

### 2.  How to operate on B+ tree
Insertion: Traverse down tree until you find leaf node where key value should appear after. Insert the key and pointer of new insertion in the node if it has less than n-1 key-values. Else split leafs and parents according to the B+ rules involving the specified limits of pointers and values a given node can have.
Splitting example:
![Image](./images/splitting.png)
Find Query: find(v)
Finds a given value v, returns null if there is no record v that exists. Finds the smallest i where K<sub>i</sub> in the current node is greater than or equal to v. Then go to the the P<sub>i+1</sub> if v ==  K<sub>i</sub>. Else go to P<sub>i</sub>. Once a leaf node K is reached return K if equal to v. Else return null.

Range Query: Perform find(lb), then iterate through the leaf nodes in the linked list until the upper bound is reached or the next node is null. i.e. find(lower bound), then linear search to ub.
[B+ Tree Video](https://youtu.be/aZjYr87r1b8?t=1639)
[Inserting](https://www.youtube.com/watch?v=47DUnfH1D0w)
[Deleting](https://www.youtube.com/watch?v=YZECPU-3iHs)

### 3.  Hash index
For a hash index we only care about putting, getting and deleting records by key. Hence, the hash index can be modeled as the following component with its parameters and performance characteristics (hash indexes provide terrible performance for range queries and full scans so will be omitted):
![Image](./images/hash.png)
 
### 4.  Sparse / dense index
Dense indexing: one-to-one relation with search-key values and records.
Sparse indexing: one-to-many relation with search-key values and records. i.e. contains index records for only some search-key values. Sparse indexing requires records to be sequentially ordered.

### 5.  Primary / secondary index
Clustering/primary index: in a sequentially ordered file, the indexing is done using a search key (typically the primary key/unique ID).
Nonclustering/secondary index: an index whose search key specifies an order different than the one on record. Secondary indices have to be dense. 
### 6.  Advantages or disadvantages 

B+ Trees: The tree height is no more than ceiling(log<sub>ceiling(n/2)</sub>K), K being the amount of search-key values in file. Searching is very efficient. 
* Example: if n = 100, 4 nodes would need to be accessed to find a key value within 1,000,000 search-key values.

Insertion and deltion to main file is efficient, as the index can be restructured in log time.


---

## 14 concurrency
### Two Phase Locking

* Exclusive (X) mode. Data item can be read and written
* Shared (S) mode can only be read

* Two phase locking protocol 
  * Phase 1: Growing phase
    * Transaction may obtain locks
    * Transaction may not release locks
  * Phase 2: Shrinking phase
    * Transaction may release locks
    * Transaction may not obtain locks

A schedule S is legal under locking protocol if it can be generated by a set of transactions that follow the protocol

A protocol such as 2PL ensures serializability that all legal schedules under that protocol are serializable

### Granularity

### Deadlocking

Deadlocking is the possibility that seperate X and S transactions may wait for eachother in the process of completing a transaction. 

Transactions will need to be rolled back to prevent these processes

---
## 15 Phantoms
    

Phantoms is basically just a missing tuple in the process of reading. Usually a phantom is created by a sitaution such as insert before a read has been completed.

There are different ways of handling transactions such as this.

### Index locking to prevent phantoms
* Index locking protocol to prevent phantoms
  * Every relation must have at least one index
  * A transaction can access tuples only after finding them through one or more indicies on the relation
  * A transaction Ti that performs a lookup must lock all the index leaf nodes that it accesses, in S-mode
    * Even if the leaf node does not contain any tuple satisfying the index lookup (e.g. for a range query, no tuple in a leaf is in the range)
  * A transaction Ti that inserts, updates or deletes a tuple ti in a relation r
    * Must update all indices to r
    * Must obtain exclusive locks on all index leaf nodes affected by the insert/update/delete
  * The rules of the two-phase locking protocol must be observed
* Guarantees that phantom phenomenon won’t occur

### Snapshot Isolation
Motivation: Decision support queries that read large amounts of data
have concurrency conflicts with OLTP transactions that update a few
rows
  * Poor performance results
* Solution 1: Use multiversion 2-phase locking
  * Give logical “snapshot” of database state to read only transaction
    * Reads performed on snapshot
  * Update (read-write) transactions use normal locking
  * Works well, but how does system know a transaction is read only?
* Solution 2 (partial): Give snapshot of database state to every transaction
  * Reads performed on snapshot
  * Use 2-phase locking on updated data items
  * Problem: variety of anomalies such as lost update can result
  * Better solution: snapshot isolation level (next slide)

---
## 16 Distributed and parallel querry processing
**I don't know what to cover here since he didn't explicitly say what**

* Homogeneous distributed databases
  * Same software/schema on all sites, data may be partitioned among
sites
  * Goal: provide a view of a single database, hiding details of distribution
* Heterogeneous distributed databases
  * Different software/schema on different sites
  * Goal: integrate existing databases to provide useful functionality

* Local transactions
  * Access/update data at only one database
* Global transactions
  * Access/update data at more than one database

### Two Phase Commit Protocol (2PC)
Let T be a transaction initiated at site S<sub>i</sub>, and let the transaction coordinator at S<sub>i</sub> be C<sub>i</sub>

* Coordinator asks all participants to prepare to commit transaction Ti
  * Ci adds the records **<prepare T\>** to the log and forces log to stable storage
  * sends prepare T messages to all sites at which T executed
* Upon receiving message, transaction manager at site determines if it can
commit the transaction
  * if not, add a record **<no T\>** to the log and send **abort** T message to Ci
  * if the transaction can be committed, then:
    * add the record **<ready T\>** to the log
    * force all records for T to stable storage
    * send **ready** T message to Ci


Transaction is now in ready state at the site


* T can be committed of Ci received a **ready** T message from all the
participating sites: otherwise T must be aborted.
* Coordinator adds a decision record, **<commit T\>** or **<abort T\>**, to the log
and forces record onto stable storage. Once the record stable storage it is
irrevocable (even if failures occur)
* Coordinator sends a message to each participant informing it of the
decision (commit or abort)
* Participants take appropriate action locally

![](images/Two_phase_commit.png)

**Handling of failures**
The handling of most failures is mostly intuituve, noting in all cases that if the logs contain:
* **<commit T\>**, then execute a redo(T)
* **<abort T\>**, then execute an undo(T)
* **<ready T\>**, then consult coordinator
  * same as prior, with commit and abort
* If there are no control records then undo(T)


### Concurrency control

### Single-Lock-Manager Approach
* In single lock-manager approach, lock manager runs on a single chosen site, say S<sub>i</sub>
  * All lock requests sent to central lock manager
* The transaction can read the data item from any one of the sites at which a replica of the data item resides.
* Writes must be performed on all replicas of a data item
* Advantages of scheme:
  * Simple implementation
  * Simple deadlock handling
* Disadvantages of scheme are:
  * Bottleneck: lock manager site becomes a bottleneck
  * Vulnerability: system is vulnerable to lock manager site failure.

#### Distributed-Lock_Manager Approach
* In the distributed lock-manager approach, functionality of locking is
implemented by lock managers at each site
  * Lock managers control access to local data items
  * Locking is performed separately on each site accessed by transaction
    * Every replica must be locked and updated
    * But special protocols may be used for replicas (more on this later)
* Advantage: work is distributed and can be made robust to failures
* Disadvantage:
  * Possibility of a global deadlock without local deadlock at any single site
  * Lock managers must cooperate for deadlock detection

There is a section here about local and wait-for graphs and pretty much all it indicates is when a cycle is detected between resources it the coordinator chooses who will be executed.

This is ususally called a **false-cycle** things like two-phase locking can prevent this.

A **Distributed deadlock-detection** approach is one that waiting for informations and checks for deadlocks. **Expensive and not used in practice**.
