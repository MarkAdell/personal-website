---
title: "Writing Testable Code: Common Anti-Patterns and How to Fix Them"
date: 2026-04-26
tags: ["software engineering"]
ShowToc: false
showTopProgressBar: true
---

When code is hard to test, it is usually a design problem, not a testing problem. Code becomes difficult to test for the same reasons it becomes difficult to maintain. This article looks at eight common anti-patterns that make code harder to test and how to improve them. There are other anti-patterns, but in my experience writing and reviewing code, these are the most common.

Please note that these anti-patterns mostly hurt unit testing, where the goal is to test pieces of business logic in isolation. Other types of testing, such as integration and end-to-end testing, may be less affected because they test larger parts of the system together.

The advice in this guide is aimed at production codebases that will be maintained over time. Applying it to one-time scripts or throwaway prototypes would be overkill.

## Table of contents

- [Prerequisite: Terms used in this guide](#prerequisite-terms-used-in-this-guide)
- [1. Hard-coded dependencies](#1-hard-coded-dependencies)
- [2. Hidden time and randomness](#2-hidden-time-and-randomness)
- [3. Global mutable state](#3-global-mutable-state)
- [4. Static calls to side-effecting code](#4-static-calls-to-side-effecting-code)
- [5. Mixing business logic with I/O](#5-mixing-business-logic-with-io)
- [6. Catching and swallowing exceptions](#6-catching-and-swallowing-exceptions)
- [7. Business logic trapped inside framework code](#7-business-logic-trapped-inside-framework-code)
- [8. Business logic hidden inside a large workflow](#8-business-logic-hidden-inside-a-large-workflow)
- [When it is not necessary to inject dependencies](#when-it-is-not-necessary-to-inject-dependencies)
- [When to use an interface and when not to](#when-to-use-an-interface-and-when-not-to)
- [Where integration tests fit](#where-integration-tests-fit)
- [Testability checklist](#testability-checklist)
- [Conclusion](#conclusion)

## Prerequisite: Terms used in this guide

- **Business logic**: Business logic or Domain logic are the real-world rules that define how application data can be created, stored, changed, and used, separate from infrastructure or UI details.
- **Infrastructure**: Code that talks to the outside world, such as databases, file storage, external APIs, queues, and caches.
- **Dependency**: Something a class or method needs in order to do its work.
- **Hard-coded dependency**: A dependency created directly inside the code, such as with `new`.
- **Dependency injection**: Passing a dependency into a class or a method instead of hard-coding it.
- **Mock**: A test object that can replace a real dependency, return prepared values, and verify that expected calls happened.
- **Fake**: A simple working implementation used in tests instead of a real dependency, such as an in-memory repository.
- **Side effect**: Anything a method does beyond returning a value, such as saving data, sending email, changing state, or making an HTTP request.
- **Deterministic/Non-deterministic code**: Deterministic code gives the same output for the same input; non-deterministic code can give different results for the same input.

Now let's look at the common anti-patterns that make code harder to test and how to fix them.

## 1. Hard-coded dependencies

```java
class OrderService {
  public void placeOrder(Order order) {
    OrderRepository orderRepo = new OrderRepository();
    PaymentGateway paymentGateway = new PaymentGateway();

    paymentGateway.charge(order.getCustomerId(), order.getTotal());
    orderRepo.save(order);
  }
}
```

### Why this is hard to test

The method creates its own dependencies with `new`. That means a test for `placeOrder` always uses the real repository and the real payment gateway. There is no way to substitute a fake or a mock because the class or the method does not accept those dependencies as inputs.

To test this class, you either need a real database and a real payment service running somewhere, or you need specialized tooling to replace what `new` returns.

### The fix

Inject the dependencies. Let the caller decide which implementations to use.

```java
class OrderService {
  private final OrderRepository orderRepo;
  private final PaymentGateway paymentGateway;

  OrderService(OrderRepository orderRepo, PaymentGateway paymentGateway) {
    this.orderRepo = orderRepo;
    this.paymentGateway = paymentGateway;
  }

  public void placeOrder(Order order) {
    paymentGateway.charge(order.getCustomerId(), order.getTotal());
    orderRepo.save(order);
  }
}
```

### Why it is now easy to test

Tests can pass a fake or mock repository and payment gateway. Production code passes the real ones. The class does not care which, because the dependencies are no longer hard-coded.

Injection does not automatically mean introducing an interface. There is a section at the end of the article that covers [when I think an interface is worth introducing](#when-to-use-an-interface-and-when-not-to).

Also, [not every dependency necessarily needs to be injected](#when-it-is-not-necessary-to-inject-dependencies).

## 2. Hidden time and randomness

```java
class TokenService {
  public Token issue(String userId) {
    LocalDateTime issuedAt = LocalDateTime.now();
    String id = "T-" + new Random().nextInt(1_000_000);
    return new Token(id, userId, issuedAt);
  }
}
```

### Why this is hard to test

The method depends on two things that change on every call: the current time and a random number. Because the output is different every time, tests can only check weak things like "token is not null" or "ID starts with `T-`". Those assertions pass even when the code is broken.

This is sometimes called *non-determinism*: given the same input, the function gives you a different result on each call. Non-deterministic code is hard to test.

### The fix

Inject the clock and the random provider, so the caller decides what they return:

```java
class TokenService {
  private final Clock clock;
  private final RandomProvider random;

  TokenService(Clock clock, RandomProvider random) {
    this.clock = clock;
    this.random = random;
  }

  public Token issue(String userId) {
    return new Token(
      "T-" + this.random.nextInt(1_000_000),
      userId,
      LocalDateTime.now(this.clock)
    );
  }
}
```

### Why it is now easy to test

A test can pass a fixed clock and a fake `RandomProvider` that always returns a fixed value like `123456`. The token now has the same value every time, so the test can check every token field exactly. Nothing hidden, nothing flaky.

## 3. Global mutable state

```java
class AppConfig {
  public static boolean DISCOUNT_ENABLED = true;
}

class PricingService {
  public double finalPrice(double basePrice) {
    return AppConfig.DISCOUNT_ENABLED ? basePrice * 0.9 : basePrice;
  }
}
```

### Why this is hard to test

The behavior depends on a global mutable flag. Any piece of code, anywhere in the program, can change it. Worse, tests can affect each other: one test updates the flag, the next test runs with the updated value, and results start depending on the order the tests are run in.

### The fix

Avoid global mutable state. Instead, make configuration immutable and inject it into the service:

```java
class AppConfig {
  public final boolean discountEnabled;

  AppConfig(boolean discountEnabled) {
    this.discountEnabled = discountEnabled;
  }
}

class PricingService {
  private final AppConfig config;

  PricingService(AppConfig config) {
    this.config = config;
  }

  public double finalPrice(double basePrice) {
    return config.discountEnabled ? basePrice * 0.9 : basePrice;
  }
}
```

### Why it is now easy to test

Each test creates its own config and passes it in. The configuration is immutable, so it cannot be changed accidentally by another test or another part of the program.

## 4. Static calls to side-effecting code

```java
class EmailSender {
  public static void send(String to, String message) {
    // send email
  }
}

class PasswordResetService {
  public void sendResetLink(String email, String link) {
    String message = "Reset your password: " + link;
    EmailSender.send(email, message);
  }
}
```

### Why this is hard to test

The problem here is that `PasswordResetService` depends directly on a static method that performs I/O. Because the call is hard-coded, a test cannot easily replace it with a mock or fake implementation. Instead, the test is forced either to invoke the real email-sending code or to rely on heavier tooling to intercept the static call.

### The fix

Instead of calling the email-sending code statically, inject an `EmailSender` dependency and call it through the instance:

```java
class EmailSender {
  public void send(String to, String message) {
    // send email
  }
}

class PasswordResetService {
  private final EmailSender emailSender;

  PasswordResetService(EmailSender emailSender) {
    this.emailSender = emailSender;
  }

  public void sendResetLink(String email, String link) {
    String message = "Reset your password: " + link;
    emailSender.send(email, message);
  }
}
```

### Why it is now easy to test

Each test can pass in a mocked `EmailSender` and verify that the correct email would have been sent, without invoking the real email-sending code.

Please note that pure static helpers are usually not a problem. Static calls become painful when they include side-effecting code.

## 5. Mixing business logic with I/O

```java
class PricingService {
  public double getDiscountedPrice(String userId, double price) throws IOException {
    URL url = new URL("https://api.example.com/users/" + userId);
    HttpURLConnection conn = (HttpURLConnection) url.openConnection();

    boolean isVip = conn.getResponseCode() == 200;

    double discount;
    if (isVip && price >= 200) {
      discount = 0.25;
    } else if (isVip) {
      discount = 0.10;
    } else {
      discount = 0.0;
    }

    return price * (1 - discount);
  }
}
```

### Why this is hard to test

The method mixes an HTTP call with a pricing rule. The rule has several branches that deserve their own tests, but you cannot exercise any of them without making a real network request.

### The fix

Separate the HTTP call from the pricing logic:

```java
class UserClient {
  public boolean isVip(String userId) throws IOException {
    URL url = new URL("https://api.example.com/users/" + userId);
    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    return conn.getResponseCode() == 200;
  }
}

class PricingService {
  private final UserClient userClient;

  PricingService(UserClient userClient) {
    this.userClient = userClient;
  }

  public double getDiscountedPrice(String userId, double price) throws IOException {
    boolean isVip = userClient.isVip(userId);

    double discount;
    if (isVip && price >= 200) {
      discount = 0.25;
    } else if (isVip) {
      discount = 0.10;
    } else {
      discount = 0.0;
    }

    return price * (1 - discount);
  }
}
```

### Why it is now easy to test

The pricing rule is now separate from the HTTP call, so each branch can be tested without making network requests. `PricingService` can be tested with a mock `UserClient`, while `UserClient` can be covered separately with an integration test if needed.

## 6. Catching and swallowing exceptions

```java
class UserService {
  private final EmailSender emailSender;

  UserService(EmailSender emailSender) {
    this.emailSender = emailSender;
  }

  public void sendWelcomeEmail(User user) {
    try {
      emailSender.send(user.getEmail(), "Welcome!");
    } catch (Exception e) {
      // exception is ignored or just logged
    }
  }
}
```

### Why this is hard to test

The method hides the failure. If sending the email fails, the code ignores the exception.

There is no clear way for the test to determine whether this method succeeded or failed. The deeper issue is that the method's contract is dishonest: it claims to send a welcome email but silently does nothing on failure. Hard-to-test is the symptom.

### The fix

Make the failure part of the method's contract. For example, let the exception stop the flow:

```java
class UserService {
  private final EmailSender emailSender;

  UserService(EmailSender emailSender) {
    this.emailSender = emailSender;
  }

  public void sendWelcomeEmail(User user) throws EmailException {
    emailSender.send(user.getEmail(), "Welcome!");
  }
}
```

Or return an explicit result:

```java
class UserService {
  private final EmailSender emailSender;

  UserService(EmailSender emailSender) {
    this.emailSender = emailSender;
  }

  public WelcomeResult sendWelcomeEmail(User user) {
    try {
      emailSender.send(user.getEmail(), "Welcome!");
      return WelcomeResult.success();
    } catch (EmailException e) {
      return WelcomeResult.emailFailed();
    }
  }
}
```

### Why it is now easy to test

The failure is now visible to the caller, so the test has something explicit to assert.

In the first version, a test can assert that an email failure threw an exception. In the second version, a test can assert that the result is `emailFailed`.

## 7. Business logic trapped inside framework code

```java
@RestController
@RequestMapping("/invoices")
class InvoiceController {
  private final OrderRepository orders;

  InvoiceController(OrderRepository orders) {
    this.orders = orders;
  }

  @GetMapping("/{orderId}")
  public ResponseEntity<Map<String, Object>> calculateInvoice(
    @PathVariable long orderId,
    @RequestParam boolean includeTax
  ) {
    Optional<Order> optionalOrder = orders.findWithItems(orderId);

    if (optionalOrder.isEmpty()) {
      return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(Map.of("error", "Order not found"));
    }

    Order order = optionalOrder.get();

    BigDecimal subtotal = BigDecimal.ZERO;

    for (OrderItem item : order.getItems()) {
      BigDecimal lineTotal = item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity()));
      subtotal = subtotal.add(lineTotal);
    }

    if (includeTax) {
      BigDecimal tax = subtotal.multiply(new BigDecimal("0.14"));
      subtotal = subtotal.add(tax);
    }

    return ResponseEntity.ok(Map.of("total", subtotal));
  }
}
```

### Why this is hard to test

The controller mixes invoice calculation with Spring-specific details. A test for the total is no longer just "given these order items, expect this total".

Instead, the test has to deal with request parameters, `ResponseEntity`, HTTP status codes, and response body shape.

Most of that setup and assertion is about Spring details, not invoice calculation.

### The fix

Keep the controller focused on the HTTP boundary, and move the invoice calculation into application code:

```java
class InvoiceService {
  private final OrderRepository orders;

  InvoiceService(OrderRepository orders) {
    this.orders = orders;
  }

  public BigDecimal calculateInvoice(long orderId, boolean includeTax) {
    Order order = orders.findWithItems(orderId)
      .orElseThrow(OrderNotFoundException::new);

    BigDecimal subtotal = BigDecimal.ZERO;

    for (OrderItem item : order.getItems()) {
      BigDecimal lineTotal = item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity()));
      subtotal = subtotal.add(lineTotal);
    }

    if (includeTax) {
      BigDecimal tax = subtotal.multiply(new BigDecimal("0.14"));
      return subtotal.add(tax);
    }

    return subtotal;
  }
}

@RestController
@RequestMapping("/invoices")
class InvoiceController {
  private final InvoiceService invoiceService;

  InvoiceController(InvoiceService invoiceService) {
    this.invoiceService = invoiceService;
  }

  @GetMapping("/{orderId}")
  public ResponseEntity<Map<String, Object>> calculateInvoice(
    @PathVariable long orderId,
    @RequestParam boolean includeTax
  ) {
    try {
      BigDecimal total = invoiceService.calculateInvoice(orderId, includeTax);

      return ResponseEntity.ok(Map.of("total", total));
    } catch (OrderNotFoundException e) {
      return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(Map.of("error", "Order not found"));
    }
  }
}
```

### Why it is now easy to test

`InvoiceService` can be tested without preparing an HTTP request or inspecting a `ResponseEntity`. A test can inject a fake order repository, call `calculateInvoice`, and assert the returned total directly.

Business logic stays in application code. Request handling and responses stay in the framework layer.

## 8. Business logic hidden inside a large workflow

Private methods are not automatically a problem. In most cases, they should be tested through the public behavior of the class.

The problem appears when a public method does many unrelated things, and an important business rule is buried inside it. Testing that rule now requires setting up the whole workflow.

```java
class CheckoutService {
  private final InventoryService inventory;
  private final PaymentGateway paymentGateway;
  private final EmailSender emailSender;

  CheckoutService(
    InventoryService inventory,
    PaymentGateway paymentGateway,
    EmailSender emailSender
  ) {
    this.inventory = inventory;
    this.paymentGateway = paymentGateway;
    this.emailSender = emailSender;
  }

  public Receipt checkout(Cart cart, Customer customer) {
    inventory.reserve(cart.items());

    int subtotal = 0;

    for (CartItem item : cart.items()) {
      subtotal += item.price() * item.quantity();
    }

    int discount = calculateDiscount(customer, subtotal);
    int total = subtotal - discount;

    paymentGateway.charge(customer.id(), total);
    emailSender.send(customer.email(), "Thanks for your order");

    return new Receipt(total);
  }

  private int calculateDiscount(Customer customer, int subtotal) {
    if (customer.isVip() && subtotal >= 10_000) {
      return 2_000;
    }

    if (customer.isVip()) {
      return 1_000;
    }

    return 0;
  }
}
```

### Why this is hard to test

The discount rule is simple, but it is trapped inside `checkout`.

To test the VIP discount, the test has to create a cart, prepare inventory reservation, avoid a real payment charge, avoid sending a real email, call `checkout`, and then inspect the receipt.

Most of that setup has nothing to do with the discount rule.

### The fix

Move the independent business rule into a small class with clear inputs and outputs:

```java
class DiscountPolicy {
  public int discountFor(Customer customer, int subtotal) {
    if (customer.isVip() && subtotal >= 10_000) {
      return 2_000;
    }

    if (customer.isVip()) {
      return 1_000;
    }

    return 0;
  }
}
```

### Why it is now easy to test

The discount rule can be tested directly, without inventory, payment, email, or a checkout workflow. Moving it out gives the rule a smaller testing surface and keeps `CheckoutService` focused on orchestration:

```java
DiscountPolicy policy = new DiscountPolicy();

int discount = policy.discountFor(vipCustomer, 12_000);

assertEquals(2_000, discount);
```

## When it is not necessary to inject dependencies

Not every dependency needs to be injected. A useful rule of thumb is: **inject infrastructure, not pure business logic.**

Infrastructure is the code that talks to the outside world, such as databases, payment gateways, email services, external APIs, file storage, queues, and caches. You want to be able to swap these in tests.

Pure business logic is the code that does calculations and decisions, such as discount rules, tax math, and validation. You rarely need to replace these in tests. Writing `new DiscountCalculator()` inside a method is usually fine, because there is no good reason to swap it out. If the calculator has a bug, the test catches it. If the calculator is slow or unreliable, that is already a bigger problem.

## When to use an interface and when not to

This is a controversial topic, and the following is my current point of view.

An interface earns its place when you genuinely expect more than one implementation, not just because testing requires it.

Payment gateways are the clearest example. Even if you only have one implementation today, there is a good chance you will have another later, either replacing the current one or running alongside it. That is a real need for polymorphism, so an interface makes sense.

In my experience, database repositories usually do not qualify. It is rare to have multiple implementations of your data layer, and if that does happen, the missing interface will be the least of your problems. The real challenge will be data mapping and migration.

A better rule than "every dependency needs an interface" is this: any dependency that must be replaceable should provide a clear way to replace it.

## Where integration tests fit

Writing testable code does not mean every behavior should be tested only with unit tests.

Unit tests are good for checking business logic in isolation. Integration tests are still needed to verify real interactions between modules, databases, APIs, and other external systems.

Relying only on unit tests is an anti-pattern because they cannot catch failures in how components work together.

At the same time, integration tests are slower, harder to debug, and more complex, so they should not replace unit tests.

A good balance is:

- Many unit tests for fast, precise validation of logic
- A smaller number of integration tests to verify real-world wiring and behavior

The first two sections of [this article](https://blog.codepipes.com/testing/software-testing-antipatterns.html) explore this topic in more detail.

## Testability checklist

Before writing a unit test, ask:

- Can I control the dependencies?
- Can I control time and randomness?
- Can I avoid shared mutable state between tests?
- Are static calls limited to pure, predictable behavior without side effects?
- Can I test the business logic without real I/O?
- Can I observe success and failure clearly?
- Is business logic separate from framework code?
- Can I test key business rules without running the whole workflow?

## Conclusion

Testable code tends to be easier to read, change, debug, and maintain for the same reasons it is easier to test: fewer hidden dependencies, more predictable behavior, and business logic that is isolated from infrastructure. That is why testability is worth treating as a design goal, not just a testing concern.
