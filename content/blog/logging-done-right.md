---
title: "Logging Done Right"
date: 2024-06-10
tags: ["software engineering"]
showTopProgressBar: true
ShowToc: false
---

Writing effective log messages is crucial for the overall observability of your application. In this guide, we are going to focus mainly on what to log, and how to write effective log messages.

The code examples are written in JavaScript for its simple syntax, but these guidelines are applicable to any programming language.

Let's get started!

## 1. All log messages should start with the class and function name

```javascript
logger.info('[MyClass.myFunction] The log message');
```
**Reason**: Quickly identifying where the log message comes from. It also adds uniqueness to the logs, ensuring that logs are not duplicated.

## 2. Add logs in all exception handling blocks

```javascript
try {
    // code that might throw an error
} catch (error) {
    logger.error('[OrderService.placeOrder] Order placement failed: ' + error.message);
}
```

**Reason**: Very crucial for troubleshooting.

## 3. Include context if useful

```javascript
logger.warn('[PaymentGateway.processPayment] Payment failed for UserID: 123, OrderID: 456, Error: Insufficient funds');
```

**Reason**: Context helps you understand the conditions under which the log was generated, aiding in replicating and fixing issues.

## 4. Add logs for auditing

```javascript
logger.info('[OrderService.placeOrder] Order ID: 12345 placed successfully');
```

**Reason**: Audit logs have many benefits, such as reconstructing the timeline of a system outage or a security breach, and also being able to detect them while happening.

## 5. Don't include large log messages if not necessary

**Bad:**

```javascript
logger.info('[SomeOrdersJob.processOrders] Finished processing orders chunk, index: ' + chunkIndex + 'orders: ' + JSON.stringify(orders));
```

**Good:**

```javascript
logger.info('[SomeOrdersJob.processOrders] Finished processing orders chunk, index: ' + chunkIndex + 'orders length: ' + orders.length);
```

**Reason**: Large log messages reduce readability, consume more space, and might introduce performance implications.

## 6. Don't log sensitive data

**Bad, too bad:**

```javascript
logger.warn('[AuthService.login] User login attempt failed, login data: ' + JSON.stringify(loginData));
```

```javascript
logger.warn('[PaymentGateway.registerPaymentCard] Card registration failed, card data: ' + JSON.stringify(cardData));
```

**Good:**

```javascript
logger.warn('[AuthService.login] User login attempt failed, username: ' + loginData.username);
```

**Reason**: Logging sensitive data can create security risks and violate privacy regulations such as GDPR. **Always sanitize data before logging.** It's very likely to overlook sanitizing your data when logging HTTP requests and responses, for example.

## 7. Use the correct log level

```javascript
logger.debug('[OrderService.placeOrder] Inside placeOrder function');
logger.info('[OrderService.placeOrder] Order ID: 12345 placed successfully');
logger.warn('[AuthService.login] User login attempt failed, username: ' + loginData.username);
logger.error('[OrderService.placeOrder] Order placement failed: ' + error.message);
logger.fatal('[Database.connect] Unable to connect to the database: ' + error.message);
```

**Reason**: Using log levels has many benefits, such as filtering logs based on their level, taking specific actions such as sending an alert for high-severity logs.

## 8. Timestamp your logs

```javascript
logger.info('[2024-06-08T12:00:00Z] [OrderService.placeOrder] Order ID: 12345 placed successfully');
```

**Reason**: Timestamps provide a chronological order to logs, making it easier to track and understand the sequence of actions. They also allow you to troubleshoot issues that happened at specific times.

## 9. Avoid logging in loops or recursive functions if not necessary

**Bad:**

```javascript
for (let i = 0; i < items.length; i++) {
    logger.info('[InvoiceCalculator.calculate] Calculating item: ' + i);
    // calculation logic
}
```

**Good:**

```javascript
logger.info('[InvoiceCalculator.calculate] Calculation started');
for (let i = 0; i < items.length; i++) {
    // calculation logic
}
logger.info('[InvoiceCalculator.calculate] Calculation finished');
```

**Reason**: Excessive log entries make it harder to find relevant information, consume more space, and can lead to performance issues.

## 10. Log the start and end of long-running operations

```javascript
logger.info('[DataImportService.importData] Data import started');

// long-running operation

logger.info('[DataImportService.importData] Data import completed');
```

**Reason**: It helps you monitor the progress and duration of these operations.

## 11. Ensure log messages are concise and clear

**Bad:**

```javascript
logger.info('[OrderService.placeOrder] The order placement function has successfully completed processing the order with ID 12345');
```

**Good:**

```javascript
logger.info('[OrderService.placeOrder] Order ID: 12345 placed successfully');
```

**Reason**: Concise and clear log messages are easier to read and understand.

## Conclusion

I hope you found this guide helpful. Please feel free to suggest other practices that you think are important, and I will be happy to include them.

## References and Further Reading

- https://www.dataset.com/blog/the-10-commandments-of-logging/
- https://medium.com/@squarecog/logging-101-d74ff92f8c91
- https://www.datadoghq.com/knowledge-center/audit-logging/
