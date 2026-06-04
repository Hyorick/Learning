# SQL
**Table of Content**
- Directives (Common) ⌛
- Jointure ⌛
- Sous-Requêtes
- Table Modification Commands ✅
- Commons Aggregate Functions ✅
- Utility / Formatting Functions ⌛
- Fonctions mathématiques / numérique ⌛
- Fonctions de chaînes de caractères ✅
- Common Date Functions ✅

**Spring Persistence** :  Spring Boot App -> DB, full Communication flow
- **SQL** Databases
  - Spring Boot + **JpaRepository** + Hibernate
   > Spring Boot App → JpaRepository → Spring Data JPA → JPA → Hibernate → JDBC Driver → SQL DB
  - Spring Boot + **JpaRepository** + EclipseLink/OpenJPA
  >Spring Boot App → JpaRepository → Spring Data JPA → JPA → EclipseLink/OpenJPA → JDBC Driver → SQL DB
  - Spring Boot + `EntityManager` + Hibernate
  >Spring Boot App → EntityManager(JPQL) → JPA → Hibernate(HQL) → JDBC Driver → SQL DB
  - Spring Boot + JDBC only
  >Spring Boot App → JdbcTemplate(Pure SQL)/JDBC API → JDBC Driver → SQL DB
- **NoSQL** (Ex: MongoDB)
  - Spring Boot + **MongoRepository**
  >Spring Boot App → MongoRepository → Spring Data MongoDB → MongoDB Java Driver → MongoDB
  - Spring Boot + `MongoTemplate`
  >Spring Boot App → MongoTemplate → Spring Data MongoDB → MongoDB Java Driver → MongoDB
  - Spring Boot + MongoDB Driver only
  >Spring Boot App → MongoDB Java Driver → MongoDB

\- **Hibernate**

\- Difference between BDD types

SQL databases can absolutely be grouped into families because many share:
- The same relational model,
- Similar SQL syntax,
- Similar storage engines,
- or a common code ancestry/design philosophy.

But they differ in:
- Scalability,
- SQL Dialects,
- Concurrency model,
- Extensions,
- Architecture (Row-oriented databases or Column-oriented databases),
- and target use cases.

<pre>
                    SQL Databases
                           |
        ---------------------------------------
        |                 |                  |
 Traditional         Distributed         Embedded
    RDBMS               SQL                SQL
        |
   ----------------
   |              |
PostgreSQL     MySQL
 Family         Family
</pre>

| Use Case                      | Common Choice         |
| ----------------------------- | --------------------- |
| General backend app           | PostgreSQL            |
| Simple web hosting            | MySQL/MariaDB         |
| Enterprise systems            | Oracle / Microsoft SQL Server   |
| Mobile/local app              | SQLite                |
| Massive distributed cloud app | CockroachDB / Spanner |
| Analytics                     | ClickHouse / Redshift |



## Directives
* `EXPLAIN` ou `EXPLAIN ANALYZE`
* `SELECT` → choose columns
* `FROM` : choose table

Rows
* `WHERE` condition : **filter** **rows** (**before** grouping)
* `ORDER BY` column *DESC* or *ASC* (on peut avoir plusieurs column) : **sort** results
* `LIMIT`	**Restrict number** of **returned rows**
* `OFFSET`	**Skip rows** before returning results 📋
* `BETWEEN {lower_val} AND {upper_val}`	**Range** filter **rows** of a column

Grouping
* `GROUP BY` column (on peut avoir plusieurs column) : **group rows**
* `HAVING` condition : filter **groups** (**after** grouping)

Jointure
* `JOIN`	**Combine tables**
* `INNER JOIN ON`	Return matching **rows** only 📋
* `LEFT JOIN ON`	Keep all left-table **rows** 📋
* `RIGHT JOIN ON`	Keep all right-table **rows** 📋
* `FULL OUTER JOIN ON`	Keep all **rows** from both tables 📋
* `CROSS JOIN ON`	Cartesian product 📋
* `SELF JOIN `	 📋
* `NATURAL JOIN `	 📋

Union
* `UNION`	**Combine** ***unique*** **rows** 📋
* `UNION ALL`	Combine **rows** **including** ***duplicates*** 📋

Conditional logic
* `CASE`	Conditional logic

Match in List ⌛
* `IN`	Match against list 📋


Pattern matching
* `LIKE`	**Pattern matching**
* `IS NULL`	Check null values

Sous-Requêtes
* `EXISTS`	Test subquery existence 📋
* `NULLIF()` :	Return NULL if equal
* `IFNULL()` :	Replace NULL
* `WITH`	Common Table Expression (CTE) 📋

>### EXPLAIN

Mot clé à placer avant une requête "SELECT" pour avoir des **détails sur la façon dont la requête est exécutée** et sur les **index utilisés.**

* 💡**Sert à comprendre** comment le **SGBD exécute une requête SQL** et **quels index sont utilisés.**

* **EXPLAIN ANALYZE** → **exécute réellement la requête** et donne **les temps réels**

>### GROUP BY

1. Core idea of **GROUP BY** (simple mental model)

👉 GROUP BY means:
“***Put identical values together into buckets***, then calculate something per bucket”

2. Start with a simple table ***Orders***
<pre>
customer country amount
Alice	FR	100
Alice	FR	200
Bob	FR	50
Bob	US	300
Carol	US	400
</pre>

3. >GROUP BY **one column**
<pre>
SELECT country, SUM(amount)
FROM Orders
GROUP BY country;
</pre>

What happens internally:
<pre>
**Group FR**
customer amount
Alice	100
Alice	200
Bob	50
</pre>
→ SUM = 350
<pre>
**Group US**
customer amount
Bob	300
Carol	400
</pre>
→ SUM = 700

Result:
<pre>
country	total
FR	350
US	700
</pre>
4. >Now GROUP BY **2 columns** (this is where confusion starts)
<pre>
SELECT customer, country, SUM(amount)
FROM Orders
GROUP BY customer, country;
</pre>

Key idea:
👉 SQL now creates pairs (customer, country)

So groups become:
<pre>
Group 1: (Alice, FR)

amount
100
200
→ 300
</pre>

<pre>
Group 2: (Bob, FR)

amount
50
→ 50
</pre>
<pre>
Group 3: (Bob, US)

amount
300
→ 300
</pre>
<pre>
Group 4: (Carol, US)

amount
400
→ 400
</pre>
Result:
<pre>
customer country sum
Alice	FR	300
Bob	FR	50
Bob	US	300
Carol	US	400
</pre>

5. VERY IMPORTANT RULE

When you do: GROUP BY A, B

👉 It means: each unique combination of (A, B) is a group, 
NOT separately A then B.

>### HAVING

6. Now **HAVING** (the part that confuses most people)

Rule:
- `WHERE` = **before grouping**,
- `HAVING` = **after** **grouping**

Example:
<pre>
SELECT customer, country, SUM(amount) as total
FROM Orders
GROUP BY customer, country
HAVING SUM(amount) > 100;
</pre>
### Step-by-step thinking:

- Step 1: GROUP BY **forms buckets**

We already got:
<pre>
customer country total
Alice	FR	300
Bob	FR	50
Bob	US	300
Carol	US	400
</pre>
- Step 2: HAVING **filters groups**

Condition: SUM(amount) > 100

So we REMOVE:
Bob, FR → 50 ❌

Keep:
<pre>
Alice, FR → 300
Bob, US → 300
Carol, US → 400
</pre>

Final result:
<pre>
customer country total
Alice	FR	300
Bob	US	300
Carol	US	400
</pre>

>### UNION
#### UNION
#### UNION ALL

## Jointure ⌛
>### JOIN
#### INNER JOIN
#### LEFT JOIN 
#### RIGHT JOIN 
#### FULL OUTER JOIN 
#### CROSS JOIN
#### SELF JOIN 📋
#### NATURAL JOIN 📋



## Sous-Requêtes
* `EXISTS (sql_query)`	Test subquery existence 📋

## Table Modification Commands
* `INSERT INTO` :  Add rows (POST)
* `UPDATE` :  Modify rows (PUT, PATCH)
* `DELETE` :	Remove selected rows (DELETE Request)
* `CREATE TABLE` :	Create table
* `ALTER TABLE` :  Modify structure
* `DROP TABLE` :	Deletes table entirely
* `TRUNCATE TABLE` :	Removes all rows, faster


## Commons Aggregate Functions 
De manière générale, les fonctions d'Aggregation **ignore** les lignes **NULL** dans leur calcul respectif, et opère uniquement sur les valeurs **non NULL** des lignes d'une colonne.
* `AVG(column)` : calcule la moyenne des valeurs uniquement d'une colonne numérique (nombre)
* `SUM(column)` : Total
* `COUNT(*)` ou COUNT(column) : count number of rows
* `MIN(column)` : minimum
* `MAX(column)` : maximum


## Utility / Formatting Functions
* `AS` : 	Rename column/table alias, column AS alias\_name
* `CAST()` :	Convert data types
* `COALESCE()` :	First non-null value
* `NULLIF()` :	Return NULL if equal
* `IFNULL()` :	Replace NULL

## Fonctions mathématiques / numérique
* `ROUND()` :	**Round decimal** values
* `TRUNCATE()` :	**Cut decimals** without rounding
* `RAND()` : 

## Fonctions de chaînes de caractères
* `CONCAT()` :	**Join strings**
* `LENGTH()` :	String length
* `UPPER()` :	Uppercase
* `LOWER()` :	Lowercase
* `SUBSTRING()` :	**Extract text**
* `TRIM()` :	**Remove spaces**

## Common Date Functions
Function	Purpose
* `NOW()` :	Current timestamp
* `CURDATE()` : Current date
* `YEAR(date)` :	Extract year
* `MONTH(date)` :	Extract month
* `DAY(date)` :	Extract day
* `DATEDIFF()` :	Difference between dates

## SQL (3) 

20. Bonne Réponse **✅** 
<pre>
  "SELECT AVG(rating) 
   FROM ratings" 
  et 
  "SELECT SUM(rating) / COUNT(rating)
   FROM Ratings;"
</pre>

---
21. Bonne Réponse **✅Il est possible de** créer des **unit test** pour les BDD : "Il n'existe pas d'outils spécifiques pour cela, mais **il est toujours possible d'utiliser un framework de test unitaire générique"**.

On peut utiliser
* JUnit + DB test
* TestContainers
* DBUnit

Et tester via:
* des requetes SQL
* des assertions sur le résultat

---
22. Faux ❌: bonne réponse **EXPLAIN** (plan estimé)

Mot clé à placer avant une requête "SELECT" pour avoir des **détails sur la façon dont la requête est exécutée** et sur les **index utilisés.**

* °|°**sert à comprendre** comment le **SGBD exécute une requête SQL** et **quels index sont utilisés.**

* **EXPLAIN ANALYZE** → **exécute réellement la requête** et donne **les temps réels**

---
23. Bonne Réponse **✅La principale difficulté** lors d'un export de **NoSQL vers une BDD SQL** c'est de **"Mapper les structures flexibles ou imbriquées** des bases **NoSQL** dans le **schéma rigide des bases SQL**".

Exemple : exportation de données NoSQL vers une BDD SQL

**NoSQL document** (MongoDB-style)
<pre>
{
 "id": 1,
 "name": "Alice",
 "addresses": [
   {
    "city": "Paris",
    "country": "France"
    },
    {
      "city": "Lyon",
      "country": "France"
    }
   ]
}
</pre>

en convertissant pour compatibilité **SQL Database**

2 tables
* *users* table : column id et name
* *addresses* table : column id, user\_id, city, country


**NoSQL** databases requires:
>* **schema-less** or **flexible** schema
>* **document**-oriented / **key-value** / **graph-based**
>* capable of **storing** **nested or inconsistent data**

**SQL** databases requires:
>* **fixed** tables
>* **predefined** columns
>* **strict data types**
>* normalized relationships


So **migration requires**:
>* flattening nested structures
>* creating relationships (**FOREIGN KEY**)
>* splitting one document into multiple tables

---
24. Bonne reponse **✅** pour compter le nombre de produits par catégorie
<pre>
SELECT name as product_category, COUNT(product_id) as number_of_product
FROM product
GROUP BY product_category (ou name);
</pre>

Signification : groups product by "name" and counts how many "product\_id" are in each "product\_category" (name)

25. solution **✅**:
<pre>
SELECT first_name, last_name, ROUND(AVG(score), 2) AS avg_score
FROM students
GROUP BY first_name, last_name
HAVING AVG(score) >= 0.9
ORDER BY avg_score DESC, first_name ASC;
</pre>

On utilise HAVING à la place du WHERE (car le WHERE est censé s'executer avant la fonction d'aggregtion AVG du ROUND juste après le  GROUP BY)



## Hibernate (4)

16. Faussé ❌ **Bonne reponse** : `CascadeType.REMOVE` et `CascadeType.ALL` (les types qui cause des possible propagations de suppression) utiliser No Cascade (Best)

`CascadeType.ALL` à eviter si on veut que la suppression d'"User" n'entraîne pas la suppression de "Role"

Dedans:
* `CascadeType.PERSIST` : (Save parent → saves children),

* `CascadeType.MERGE` : (Update parent → updates children),

* `CascadeType.REMOVE` : (Delete parent → deletes children) le plus dangereux,

* `CascadeType.REFRESH` If parent is refreshed from DB. children are also refreshed,

* `CascadeType.DETACH` If parent is detached from DB: children are also detached

>👉 In Many-to-Many:

- Deleting one side only removes the relationship

- It does NOT delete the other side

>👉 In One-to-Many (Department-> Employee):

- If Department ❌ deleted then All Employees ❌ also deleted (because cascade ALL)

- If Employee deleted, Department not Affected
`
>👉 In Many-to-One (Employee -> Department):

**Without cascade:**  If Department ❌ deleted then Employees ❌ still exist -> !!bad design


---
17.  Faussé ❌ Réponse ~~"18 Requêtes car Hibernate ne batch pas jamais les" "DELETE" (J2EE entityManager.remove(user)) request. Chaque suppression est envoyée individuellement.~~ "3 requêtes (communication vers la BDD). Un SELECT, 10 suppressions dans un batch, puis 7 suppressions."

18.
`org.hibernate.LazyInitializationException : could not initialize proxy - no Session`.

signifie généralement :

**Hibernate** essaie de charger une relation LAZY
alors que la **Session Hibernate est déjà fermée**.

>**Cause typique**, supposons :
<pre>
@Entity
public class User {

    @OneToMany(fetch = FetchType.LAZY)
    private List<Order> orders;
}
</pre>

Hibernate ne charge PAS orders immédiatement. (Les données ne sont chargées qu'au moment d'accès.)

Il crée un proxy/lazy collection.

Puis plus tard :

La **connexion DB** est déjà **fermée**.

🚫 Donc Hibernate **ne peut plus récupérer** les données.

19. Faussé ❌ Bonne réponse `5 requêtes` (**persist()** + **flush()**) sinon `0` (si **persist()** seul), 
>par *défaut* la config ***Hibernate*** de **batching** est **hibernate.jdbc.batch_size = 0** (means **JDBC Batching** is **disabled**) et donc chaque INSERT ici est fait séparément. 

> On modifie la valeur dans le **application.properties**

<pre>
for (int i=0; i<5; i++) {
    EntityManager.persist(new User("User n'"+ id, i+1));
} 
  User {
    @Id private Integer id;
    private String name;
 
    public User(String name, Integer id) {
         this.id = id;
         this.name = name;
 	}
}
</pre>



### Important Detail: When SQL Actually Executes

**persist()** itself does NOT necessarily execute SQL immediately.

>***Hibernate*** stores entities in the persistence context first.

Actual SQL execution usually happens during:
- flush()
- transaction commit
- query execution that triggers flush 

<pre> ⚠️Only Applies to INSERT query, 🚨 Batch not disabled for the others (UPDATE, DELETE etc)</pre>


| Id creation Strategy   | Inserts Delayed? | Batch Friendly? | SQL Requests for 5 Entities             |
| ---------- 		| ---------------- | --------------- | --------------------------------------- |
| Manual IDs (only @Id) | Yes              | Yes             | 5 INSERTs                               |
| IDENTITY   		| No               | No (disable batching)| 5 immediate INSERTs                     |
| SEQUENCE   		| Yes              | Yes             | 5 INSERTs (+ possible sequence fetches) |
| AUTO (@GeneratedValue)| Depends on DB    | Depends         | Depends                                 |

Pour AUTO, Common Hibernate Defaults by Database (can depend also on dialect
, capabilities)
| Database   | Typical AUTO Strategy |
| ---------- | --------------------- |
| MySQL      | IDENTITY              |
| PostgreSQL | SEQUENCE              |
| Oracle     | SEQUENCE              |
| H2         | SEQUENCE or IDENTITY  |



