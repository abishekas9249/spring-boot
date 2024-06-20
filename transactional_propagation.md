# Transactional Propagation 
Transactional propagation in Spring Boot refers to how transactions are managed and propagated when a method annotated with @Transactional calls another transactional method. Spring provides several propagation behaviors to handle different transaction scenarios. Here are the main types of propagation with examples:

## REQUIRED:

### Behavior: 
If a transaction exists, the current method will run within that transaction. If there is no existing transaction, a new one will be created.
### Example:
```java
@Service
public class UserService {
    @Transactional(propagation = Propagation.REQUIRED)
    public void createUser(User user) {
        userRepository.save(user);
        // Other operations that should be part of the same transaction
    }
}
```
## REQUIRES_NEW:

### Behavior: 
A new transaction will always be created, suspending any existing transaction.

### Example:
```java
@Service
public class EmailService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendWelcomeEmail(User user) {
        emailRepository.save(new Email(user.getEmail(), "Welcome!"));
        // Operations here are in a new transaction
    }
}

@Service
public class UserService {
    @Autowired
    private EmailService emailService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
        // The emailService method runs in a new, separate transaction
    }
}
```
## MANDATORY:

### Behavior: 
Must run within an existing transaction; throws an exception if no transaction is present.
### Example:
```java
@Service
public class PaymentService {
    @Transactional(propagation = Propagation.MANDATORY)
    public void processPayment(Payment payment) {
        paymentRepository.save(payment);
        // Operations here must be part of an existing transaction
    }
}

@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder(Order order) {
        orderRepository.save(order);
        paymentService.processPayment(order.getPayment());
        // The processPayment method must be called within an existing transaction
    }
}
```
## NEVER:

### Behavior: 
Must not run within a transaction; throws an exception if a transaction exists.
### Example:
```java
@Service
public class LoggingService {
    @Transactional(propagation = Propagation.NEVER)
    public void logAction(String action) {
        logRepository.save(new Log(action));
        // Operations here must not be part of a transaction
    }
}

@Service
public class UserService {
    @Autowired
    private LoggingService loggingService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void createUser(User user) {
        userRepository.save(user);
        loggingService.logAction("Created user " + user.getName());
        // The logAction method must be called without an existing transaction
    }
}
```
## SUPPORTS:

### Behavior: 
If a transaction exists, it will run within that transaction. If there is no transaction, it will run non-transactionally.
### Example:
```java
@Service
public class NotificationService {
    @Transactional(propagation = Propagation.SUPPORTS)
    public void notifyUser(User user) {
        // This method will join an existing transaction if present, or run non-transactionally otherwise
        notificationRepository.save(new Notification(user.getEmail(), "Notification"));
    }
}
```
## NOT_SUPPORTED:

### Behavior: 
Always runs non-transactionally, suspending any existing transaction.
### Example:
```java
@Service
public class AuditService {
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void auditAction(String action) {
        auditRepository.save(new Audit(action));
        // Operations here are non-transactional
    }
}

@Service
public class OrderService {
    @Autowired
    private AuditService auditService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder(Order order) {
        orderRepository.save(order);
        auditService.auditAction("Order placed for " + order.getId());
        // The auditAction method runs non-transactionally
    }
}
```
## NESTED:

### Behavior:
If a transaction exists, it will create a nested transaction within the existing one. If there is no existing transaction, it behaves like REQUIRED.
### Example:
```java
@Service
public class ReviewService {
    @Transactional(propagation = Propagation.NESTED)
    public void addReview(Review review) {
        reviewRepository.save(review);
        // Operations here are in a nested transaction if there is an existing one
    }
}

@Service
public class ProductService {
    @Autowired
    private ReviewService reviewService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void addProduct(Product product) {
        productRepository.save(product);
        reviewService.addReview(product.getReview());
        // The addReview method runs in a nested transaction if there's an existing one
    }
}
```
Each propagation type serves different use cases and helps manage transaction boundaries effectively in a Spring Boot application.


EAGER. By default, @OneToMany and @ManyToMany associations use the FetchType. LAZY strategy while the @OneToOne and @ManyToOne use the FetchType.
