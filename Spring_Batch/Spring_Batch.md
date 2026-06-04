# Spring Batch
Youtube *JavaTechie* : Spring Batch for Beginners 

**Table of Content**
- 💡Architecture
- Dans la classe de Config
    - Job
    - Step
    - Chunk Processing
    - ItemReader«T»
    - ItemProcessor<I, O>
    - ItemWriter«T»
- Debug
- Exemple
- Real Production Improvements
- 💡Relation entre **Spring Batch** et **Hibernate JDBC Batch**

## Architecture
![Spring batch architecture](images\Spring_batch_architecture.png)

![Mental model](images\Mental_Model.png)


### Dans le Controller

**Job launcher**:

est chargé d'exécuté les **Job**, en paralèlle, il fait appel aussi au **Job Repository**.


### Dans la classe de Config
@EnableBatchProcessing 
- Spring Boot 3+ usually unnecessary, juste *@Configuration* suffit
- Older Spring / custom setup still used

On a déclaré en @Bean
#### Job
A Job is the complete batch process.

In real enterprise applications, teams often do: **One Config** file per **Job**.
- CustomerImportJobConfig
- InvoiceExportJobConfig
- CleanupJobConfig

Example:
- Importing a CSV file into a database
- Processing daily bank transactions

A Job contains one or more **Steps**.

Spring (Since **Batch 5+ version**) to create **Job** recommends directly using **JobBuilder** with 
- JobRepository
- PlatformTransactionManager

>new JobBuilder("job", jobRepository)

#### Step
A Step is one phase of the job.

Example:
- Step 1 → Read file (Reader)
- Step 2 → Validate data (Processor)
- Step 3 → Save into DB (Writer)

Most common type:

Chunk-oriented Step

Spring (Since **Batch 5+ version**) to create **Step** recommends directly using **StepBuilder** with 
- JobRepository
- PlatformTransactionManager

>new StepBuilder("step1", jobRepository)

Un **Step** peut avoir plusieurs reader, processor ou writer ce n'est pas limité à 1 pour chaque.

En général, chaque **Step** a 3 phases (objets): 
- un **ItemReader** (only one),
- un **ItemProcessor** (multiple is possible via **CompositeItemProcessor**) 
- et un **ItemWriter** (multiple is possible via **CompositeItemWriter**)

#### Chunk Processing

Spring Batch ***usually works with chunks***.

Example:
>- Read 100 records
>- Process 100 records
>- Write 100 records
>- Commit transaction

Then repeat.

This improves:
- performance
- memory usage
- transaction handling

#### ItemReader«T» :
**Purpose**:

Reads data from a source.
>Think:“Where does the data come from?”

Examples:
- CSV file
- Database
- JSON/XML
- Kafka
- REST API

![Reader Flow and implementations](images\ItemReader_Flow_and_Common_Implementations.png)

#### ItemProcessor<I, O>
**Purpose**

Transforms or validates data.
> A cette étape, on peut ne rien faire (aucne transformation) sur les objets reçus du Reader si on veut, avant de les passer au Writer. 

> Think: “What should happen to the data?”

Examples:
- convert names to uppercase
- calculate totals
- filter invalid records
- map DTO → Entity

##### Flow
>Input Object (**Dto** ou **Record**) --> ItemProcessor --> Modified Object

##### Important Detail
If processor returns:
>- **object** → item continues
>- *null* → item is filtered/skipped

![Processor implementations](images\ItemProcessor_Common_Implementations.png)

#### ItemWriter«T» 

**Purpose**

Writes data somewhere.

>Think: “Where does the processed data go?”

Examples:
- database
- file
- API
- Kafka

Flow
>Processed Objects --> ItemWriter --> DB/File/API

![Writer implementations](images\ItemWriter_Common_Implementations.png)

Pour écrire dans la **BDD**:
- **JpaItemWriter**
    - writes directly using JPA (uses JPA/Hibernate entities)
    - Slower for huge volumes
    - Less optimized than JDBC batch
    - More object-oriented
    - Higher-level
- **JdbcBatchItemWriter** :  
    - uses SQL queries
    - plus performant
    - more memory efficient
    - Lower-level
- **RepositoryItemWriter** :
    - Uses Spring Data repository (Mandatory @Repository)
    - speed : fine (less than JpaItemWriter)


### Exécution
- Par **default** (most common), le batch processing est exécuté de manière ***synchrone ou sequentiel*** (des chunks de données). Donc ici les entité crée dans la BDD appraîtront par ID croissant.

- Dans le cas d'une **exécution concurentiel** (des chunks de données) (***asynchrone***) pour **augmenter plus encore performance d'execution**. 
    - Ajouter **TaskExecutor** au **StepBuilder**
    - **Reader** must be thread-safe (très peu le sont)
    - **Writer** must be thread-safe
    - **Incovénients**: Ordre insertion pas garanti

- les **Steps** d'un **Job** sont exécutés **par defaut** en **sequentiel**, on peut aussi les executer en **parallèle** (**concurentielles**) avec *split() + TaskExecutor*

### Debug:
Le Spring Batch processing, créer des tables supplémentaires dans la BDD qui servent à monitor, debug les jobs executés par le Batch Processing.

### Exemple:
So When Do We Use Multiple Steps?

Multiple steps are used when phases are truly independent.

Example:
- Step 1 -> Download file from SFTP: download file
- Step 2 -> Process CSV: CSV -> DB (Reader, Processor, Writer)
- Step 3 -> Generate report: Generate summary
- Step 4 -> Send email: Notify users

======

On veut Importer des clients (leur informations) d'un fichier .csv pour les sauvegarder dans la BDD en utilisant le Batch processing.
Flow: “CSV → Process → Database”

- resources/customers.csv (Exemple de localisation, dans la réalité on fournit le fichier depuis le endpoint)

- domain/entity/Customer.java

- domain/dto/CustomerCsv.java
![Exemple part 1](images\Exemple_part_1.png)


- config/CustomerBatchConfig.java
![Exemple batch config part 1](images\Exemple_config_part_1.png)
![Exemple batch config part 2](images\Exemple_config_part_2.png)

- controller/JobController
![Exemple batch controller](images\Exemple_controller_part.png)

- repository/CustomerRepository (**facultatif** à cause de **JpaItemWriter** qui n'a pas besoin de Sprind Data Repository)

### Real Production Improvements
Usually you would also add:
- Validation
- Skip invalid rows
- Retry mechanism
- Logging (Ex: invalid rows with SkipListener<CustomerDto, Customer> on Step.Skip with Step.listener)
- Parallel processing
- Job listeners
- Dynamic CSV upload
- Multi-step jobs
- Restartability

### Relation entre Spring Batch et Hibernate JDBC Batch
1. ***Spring Batch*** **chunk(size)**

Example:
<pre>.stepBuilderFactory.get("step")
    .«User, User»chunk(100)
</pre>

👉 This means:

>Process 100 items per transaction

Flow:
<pre>
READ 100 items
PROCESS 100 items
WRITE 100 items
COMMIT transaction 
</pre>

After commit:
- persistence context cleared
- next chunk starts
2. ***Hibernate*** **jdbc.batch_size**

Example:

hibernate.**jdbc.batch_size=20**

👉 This means:

>Hibernate groups SQL statements in batches of 20

inside the transaction.

#### Combined Example

Suppose:
Spring Batch
<pre>
chunk(100)
</pre>
Hibernate
<pre>
hibernate.jdbc.batch_size=20
</pre>
Writing 100 entities
<pre>
for each item:
    entityManager.persist(user)
</pre>
What Actually Happens

💡**Spring Batch** opens:
<pre>
ONE transaction for 100 items
</pre>
💡Inside that transaction, **Hibernate** sends INSERTs in groups of 20.

Database Requests
<pre>
Batch Request 1 -> inserts 1-20
Batch Request 2 -> inserts 21-40
Batch Request 3 -> inserts 41-60
Batch Request 4 -> inserts 61-80
Batch Request 5 -> inserts 81-100
</pre>
Then:

COMMIT

#### Important Difference

"chunk(100)" 
>controls: **Transaction boundary**

"batch_size=20"

>controls: SQL grouping inside transaction

Visual Flow
<pre>
Chunk transaction (100 items)
│
├── JDBC batch 1 (20 inserts)
├── JDBC batch 2 (20 inserts)
├── JDBC batch 3 (20 inserts)
├── JDBC batch 4 (20 inserts)
└── JDBC batch 5 (20 inserts)

COMMIT
</pre>

donc 
<pre>
- chunk(100) -> 100 items processed in one transaction

- Hibernate creates (with hibernate.jdbc.batch_size=20):

100/20 = 5 JDBC batches, it means it will communicate with DB 5 times

-if **hibernate.jdbc.batch_size is 0 ** (disbaled) then we will 1 communication with DB per 1 request (!!🚩which is very bad for performance for large datasets)

JDBC batches.
</pre>




