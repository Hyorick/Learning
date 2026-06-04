# Proxy
Simple idea

Instead of calling the real object directly:

Client ---> Real Object

you call a proxy:

Client ---> Proxy ---> Real Object

The proxy controls access to the real object and can do extra work before or after forwarding the call.

Real-world example

Imagine entering a secure office building.

You ---> Security Guard ---> Office

The security guard:

checks your identity
logs your entry
may deny access

The guard is the proxy.

Java Example
1. Common interface
public interface PaymentService {
    void processPayment();
}
2. Real object
public class PaymentServiceImpl implements PaymentService {

    @Override
    public void processPayment() {
        System.out.println("Processing payment...");
    }
}
3. Proxy object
public class PaymentServiceProxy implements PaymentService {

    private PaymentService paymentService;

    public PaymentServiceProxy() {
        this.paymentService = new PaymentServiceImpl();
    }

    @Override
    public void processPayment() {

        System.out.println("Checking permissions...");

        paymentService.processPayment();

        System.out.println("Logging transaction...");
    }
}
Usage
public class Main {

    public static void main(String[] args) {

        PaymentService service =
                new PaymentServiceProxy();

        service.processPayment();
    }
}

Output:

Checking permissions...
Processing payment...
Logging transaction...

The client thinks it's calling PaymentService, but the proxy is intercepting the call.

Why use a Proxy?

Common use cases:

1. Security
checkUserRole();
realObject.doWork();
2. Logging
logRequest();
realObject.doWork();
3. Caching
if(cacheExists)
   return cache;

return realObject.doWork();
4. Lazy Loading

Create expensive objects only when needed.

5. Remote Calls

Proxy makes network calls appear like local method calls.

How Spring Boot uses Proxy

This is one of the most important Spring concepts.

Many Spring features work because Spring creates a proxy around your bean.

Instead of:

Controller
    |
    v
Service Bean

Spring often creates:

Controller
    |
    v
Proxy Bean
    |
    v
Real Service Bean

The proxy intercepts method calls.

Example: @Transactional
@Service
public class OrderService {

    @Transactional
    public void createOrder() {
        // save order
    }
}

What actually happens:

Controller
    |
    v
Transaction Proxy
    |
    v
OrderService

When createOrder() is called:

Proxy starts transaction
Calls real method
Commits transaction
Rolls back if exception occurs

Conceptually:

proxy.createOrder() {

    beginTransaction();

    try {
        realObject.createOrder();
        commit();
    } catch(Exception e) {
        rollback();
    }
}

You never write this code; Spring generates the proxy.

Example: @Cacheable
@Cacheable("users")
public User getUser(Long id) {
    return repository.findById(id);
}

Proxy behavior:

if(cache.contains(id)) {
    return cache.get(id);
}

User user = realMethod();

cache.put(id, user);

return user;

Again, Spring uses a proxy.

Example: @Async
@Async
public void sendEmail() {
    // email logic
}

Proxy behavior:

executor.submit(() -> {
    realObject.sendEmail();
});

The caller immediately returns while the proxy runs the method in another thread.

Example: Spring Security
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser() {
}

Proxy behavior:

if(currentUser.hasRole("ADMIN")) {
    realMethod();
} else {
    throw new AccessDeniedException();
}
JDK Dynamic Proxy vs CGLIB Proxy

Spring typically creates proxies in two ways:

JDK Dynamic Proxy

Used when a bean implements an interface.

public interface UserService {}
public class UserServiceImpl implements UserService {}

Spring creates a proxy implementing the same interface.

CGLIB Proxy

Used when no interface exists.

@Service
public class UserService {
}

Spring creates a subclass at runtime:

class UserServiceProxy extends UserService {
}

and intercepts method calls.

Important Spring Interview Question

Why doesn't this work?

@Service
public class UserService {

    @Transactional
    public void methodA() {
        methodB();  // problem
    }

    @Transactional
    public void methodB() {
    }
}

Because:

External Call
    |
    v
Proxy
    |
    v
methodA()

But inside methodA():

methodB();

is a direct call on the same object (this.methodB()), bypassing the proxy.

methodA()
   |
   v
methodB()   (proxy NOT involved)

Therefore transaction, caching, async, security, etc. may not be applied.

This is known as Spring self-invocation bypassing the proxy.

One-sentence summary

A proxy is a stand-in object that intercepts method calls and adds behavior before or after delegating to the real object. Spring Boot heavily relies on proxies to implement features such as @Transactional, @Cacheable, @Async, and method-level security without requiring you to write the surrounding infrastructure code yourself

## Spring AOP

Spring AOP and the Proxy Pattern are very closely related.

In fact, Spring AOP is implemented using proxies in most cases.

The connection

When you write:

@Service
public class PaymentService {

    public void processPayment() {
        System.out.println("Processing payment");
    }
}

and add an aspect:

@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void log() {
        System.out.println("Method called");
    }
}

you might imagine:

Controller
    |
    v
PaymentService

But Spring actually creates:

Controller
    |
    v
Proxy
    |
    v
PaymentService

The proxy intercepts the method call and executes the aspect.

What happens internally?

Suppose:

paymentService.processPayment();

is called.

The call flow becomes:

proxy.processPayment() {

    loggingAspect.log();   // @Before advice

    target.processPayment();
}

Output:

Method called
Processing payment

The aspect code executes because the proxy intercepted the call.

AOP terminology mapped to proxies
Target Object

The actual bean:

PaymentService
Proxy

Object created by Spring:

PaymentServiceProxy

or a JDK dynamic proxy.

Advice

The code you want to execute:

@Before
@After
@Around

Example:

@Before(...)
public void log() {}
Join Point

A point where the advice can run.

In Spring AOP, this is usually:

method execution

Example:

processPayment()
Aspect

A collection of advices.

@Aspect
public class LoggingAspect {
}
Visualizing it

Without AOP:

Client
   |
   v
PaymentService

With AOP:

Client
   |
   v
Spring Proxy
   |
   +--> Logging Advice
   |
   +--> Security Advice
   |
   +--> Transaction Advice
   |
   v
PaymentService

The proxy is the mechanism that applies the aspects.

Why @Transactional is considered AOP

Consider:

@Transactional
public void createOrder() {
}

This is essentially equivalent to:

proxy.createOrder() {

    beginTransaction();

    try {
        target.createOrder();
        commit();
    } catch(Exception e) {
        rollback();
    }
}

The transaction logic is a cross-cutting concern—something needed in many places but unrelated to the business logic itself.

AOP separates that concern from your service code.

Around Advice example
@Around("execution(* com.example.service.*.*(..))")
public Object logExecutionTime(
        ProceedingJoinPoint pjp) throws Throwable {

    long start = System.currentTimeMillis();

    Object result = pjp.proceed();

    long end = System.currentTimeMillis();

    System.out.println("Time: " + (end - start));

    return result;
}

Proxy behavior:

proxy.someMethod() {

    long start = now();

    Object result = target.someMethod();

    long end = now();

    log(end - start);

    return result;
}

This is almost exactly the Proxy Pattern in action.

Why Spring uses proxies instead of modifying your code

Spring wants to add behavior:

transactions
logging
security
caching
async execution

without changing your class:

public class OrderService {
}

By wrapping your bean with a proxy, Spring can inject behavior dynamically at runtime.

Interview-style answer

How are Spring AOP and the Proxy Pattern related?

Spring AOP is primarily implemented using the Proxy Pattern. Spring creates a proxy object around a bean, intercepts method calls on that bean, executes any configured advice (such as logging, transactions, security, or caching), and then delegates the call to the actual target object. Features like @Transactional, @Cacheable, @Async, and method-level security are all built

How to recognize them

Ask yourself:

Is Spring creating a proxy around my bean?
@Service
public class OrderService {
}

and method calls are intercepted?

→ AOP + Proxy Pattern

Examples:

@Transactional
@Cacheable
@Async
@PreAuthorize
@MyCustomAnnotation

with an Aspect.

Is validation happening during HTTP request binding?
@PostMapping
public void create(@Valid UserRequest request)

→ Bean Validation

Not AOP.

Is validation happening in the request pipeline before controller execution?
HandlerInterceptor
Filter

→ Chain of Responsibility

Not AOP.

Interview answer

If someone asks:

"I created a custom annotation to validate controller inputs. Is that AOP?"

The correct answer is:

Not necessarily. If the annotation is implemented using Bean Validation (ConstraintValidator), it is not AOP; Spring MVC invokes the validator during request binding. If the annotation is implemented using an @Aspect that intercepts annotated methods, then it is AOP, and Spring applies it through a proxy around the target bean.