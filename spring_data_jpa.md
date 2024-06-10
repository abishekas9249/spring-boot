# Getting Started
- Spring Data JPA is a part of the larger Spring Data family, which aims to simplify the development of data access layers in Spring-based applications. 
- Spring Data JPA provides a repository abstraction over the JPA (Java Persistence API) and reduces the boilerplate code required for database operations. 
- Here's an in-depth look into Spring Data JPA:
## Repositories:

Spring Data JPA revolves around the concept of repositories, which are interfaces for data access.
Repositories extend the CrudRepository, JpaRepository, or PagingAndSortingRepository interfaces to provide CRUD operations and additional JPA-specific methods.
## Entities:

Entities are plain Java objects (POJOs) annotated with JPA annotations like @Entity, @Table, @Id, etc.
These annotations define how the POJOs map to the database tables.
## Repository Interfaces:

- CrudRepository<T, ID>: Provides CRUD operations.
- JpaRepository<T, ID>: Extends CrudRepository and PagingAndSortingRepository, providing JPA-specific methods and batch operations.
- PagingAndSortingRepository<T, ID>: Adds methods for pagination and sorting.

```JAVA
@Entity
class Person {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;
  private String name;
}

interface PersonRepository extends Repository<Person, Long> {

  Person save(Person person);
  Optional<Person> findById(long id);
}
Create the main application to run, as the following example shows:

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  CommandLineRunner runner(PersonRepository repository) {
    return args -> {

      Person person = new Person();
      person.setName("John");

      repository.save(person);
      Person saved = repository.findById(person.getId()).orElseThrow(NoSuchElementException::new);
    };
  }
}
```
Even in this simple example, there are a few notable things to point out:

- Repository instances are automatically implemented. When used as parameters of @Bean methods, these will be autowired without further need for annotations.

- The basic repository extends Repository. We suggest to consider how much API surface you want to expose towards your application. More complex repository interfaces are ListCrudRepository or JpaRepository.
## Transactional in Spring Data JPA

In Spring, the @Transactional annotation is used to manage transaction boundaries in an application. Transactions ensure that a series of operations on a database are executed as a single unit of work, either completely succeeding or completely failing. Here’s a detailed look at how @Transactional works and how to use it effectively in Spring applications:

## Key Concepts
- Atomicity: Ensures that all operations within a transaction are completed successfully. If any operation fails, the transaction is rolled back, undoing all operations.
- Consistency: Ensures that the database remains in a consistent state before and after the transaction.
- Isolation: Ensures that transactions are isolated from one another until they are complete, preventing data inconsistency due to concurrent access.
- Durability: Ensures that once a transaction is committed, the changes are permanent.

## Usage of @Transactional
Basic Annotation
```java
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
        // other operations
    }
}
```
## Propagation and Isolation
@Transactional has several attributes that can be configured:

1. Propagation: Determines how transactions relate to each other.
- REQUIRED (default): Join the existing transaction or create a new one if none exists.
- REQUIRES_NEW: Suspend the current transaction and create a new one.
- MANDATORY: Join the existing transaction, throw an exception if none exists.
- NESTED: Execute within a nested transaction if a current transaction exists.
- NOT_SUPPORTED: Execute non-transactionally, suspending the current transaction if one exists.
- NEVER: Execute non-transactionally, throwing an exception if a transaction exists.
- SUPPORTS: Execute within a transaction if one exists, otherwise execute non-transactionally.
- Isolation: Defines the isolation level for transactions.

2. DEFAULT (default): Use the default isolation level of the database.
- READ_UNCOMMITTED: Allows dirty reads, non-repeatable reads, and phantom reads.
- READ_COMMITTED: Prevents dirty reads but allows non-repeatable reads and phantom reads.
- REPEATABLE_READ: Prevents dirty and non-repeatable reads but allows phantom reads.
- SERIALIZABLE: Ensures full isolation by preventing dirty reads, non-repeatable reads, and phantom reads.

3. Read-Only Transactions:

Useful for optimizing read operations by avoiding unnecessary locks.
```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}
```
4. Timeout:

Specifies a timeout for the transaction, after which it will be rolled back if not completed.
```java
@Transactional(timeout = 5)  
public void performLongRunningOperation() {
    // implementation
}
```
5. Rollback Rules:

Define conditions under which the transaction should be rolled back
By default, transactions are rolled back on unchecked exceptions (subclasses of RuntimeException).
```java
@Transactional(rollbackFor = Exception.class)
public void updateUser(User user) throws Exception {
    userRepository.save(user);
    // implementation that might throw checked exception
}
```
### Declarative Transaction Management
Declarative transaction management uses annotations and AOP (Aspect-Oriented Programming) to manage transactions.
This approach separates transaction management from business logic, making it easier to manage and maintain.
```java
@Configuration
@EnableTransactionManagement
public class AppConfig {
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```
Programmatic Transaction Management
Allows more fine-grained control over transaction boundaries.
```java
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

@Service
public class UserService {

    @Autowired
    private PlatformTransactionManager transactionManager;

    public void createUser(User user) {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setName("createUserTransaction");
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        TransactionStatus status = transactionManager.getTransaction(def);
        try {
            userRepository.save(user);
            // other operations
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
### Transaction Management Best Practices
1. Keep Transactions Short:

- Keep the code within transactional methods as short and fast as possible to minimize the time locks are held on database resources.
2. Avoid Non-Transactional Operations:

- Avoid invoking non-transactional methods within a transactional context as they might break transactional guarantees.
3. Proper Exception Handling:

- Understand the types of exceptions that trigger rollbacks and handle them appropriately.
4. Database Constraints:

- Use database constraints (like foreign keys) in conjunction with transactions to ensure data integrity.
5. Monitoring and Logging:

- Implement monitoring and logging around transactions to understand their performance and troubleshoot issues.
6. Isolation Level Considerations:

- Choose the appropriate isolation level based on the consistency requirements and performance considerations of your application.
