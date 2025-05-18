
# Azure Service Bus — AZ-204 Exam Prep Summary

Azure Service Bus is a fully managed enterprise message broker service used to decouple applications and services, ensuring reliable message delivery.

---

## 📌 Key Benefits:
- Load balancing work across competing workers.
- Safe routing of data and control across service/application boundaries.
- High-reliability transactional work coordination.

---

## 📚 Messaging Scenarios:
- **Messaging**: Transfer business data like orders or logs.
- **Decoupling Applications**: Producer and consumer operate asynchronously.
- **Load Balancing**: Competing consumers read from queues safely.
- **Topics & Subscriptions**: 1:n pub/sub pattern.
- **Transactions**: Multiple operations within atomic scope.
- **Message Sessions**: Enable strict message ordering and deferral.

---

## 📝 Core Concepts:

### Queues:
- Point-to-point communication.
- Messages stored until consumed.
- FIFO (First In, First Out) processing model.

### Topics & Subscriptions:
- Pub/Sub model.
- Topics: Accept messages from publishers.
- Subscriptions: Virtual queues that receive topic messages.
- Subscribers can filter and receive message subsets.

### Namespaces:
- Scoping container for messaging resources (queues, topics, subscriptions).
- Provides a unique FQDN (e.g., `mycompany.servicebus.windows.net`).
- Security boundary and access control.

---

## 📊 Visual Overview:
**Queues**
```
[Producer] --> [Queue] --> [Consumer]
```

**Topics**
```
[Publisher] --> [Topic] --> [Subscription A]
                               [Subscription B]
```

---

## ✅ Exam Tips — Things to Remember:
- Know when to use **queues** vs **topics/subscriptions**:
  - **Queue** = one-to-one.
  - **Topic** = one-to-many.
- Understand **message sessions** for ordered processing.
- Transactions can span multiple queues and operations.
- Familiarize yourself with namespace-level configuration.
- Be clear on **competing consumers** pattern.
- Service Bus supports **dead-lettering** (unprocessed message handling).
- Know the difference between **Standard** and **Premium tiers**.
- Service Bus messages can have properties for filtering and routing.

---


# Azure Service Bus — Performance Best Practices

## 📌 Messaging Performance Optimization

|Area|Best Practice|
|:--|:--|
|**Connection Management**|Reuse `ServiceBusClient` and `ServiceBusSender/Receiver` instances instead of creating new ones.|
|**Batching Messages**|Send and receive messages in batches to reduce network calls and improve throughput.|
|**Prefetching**|Enable `PrefetchCount` to load multiple messages in a single call, reducing latency.|
|**Concurrent Processing**|Use `MaxConcurrentCalls` or `ProcessMessageAsync` with parallelism for faster message processing.|
|**Message Size Optimization**|Keep message size below **256 KB (Standard)** or **1 MB (Premium)**. Use **Azure Blob Storage** for larger payloads with message body containing just a reference URI.|
|**Dead-lettering**|Use dead-letter queues (DLQ) for unprocessable messages to avoid blocking the queue.|
|**Auto-complete Disable**|Disable `AutoCompleteMessages` and complete messages manually after processing for better control and reliability.|
|**Transactions**|Use transactions only when necessary as they add overhead.|
|**Connection Pooling**|Share Service Bus clients and connections to avoid connection throttling and excessive resource usage.|

---

## 📌 Reliability and Resilience Best Practices

- **Retry Policies**: Implement exponential backoff and retries for transient faults.
- **Duplicate Detection**: Use **duplicate detection** (via `MessageId`) to avoid processing the same message multiple times.
- **Session-based Messaging**: For ordered or stateful workflows, use message sessions.
- **Dead-letter Handling**: Monitor and process dead-letter queues proactively.
- **Error Logging and Monitoring**: Integrate with Azure Monitor and set up alerts for message counts, DLQ counts, and connection metrics.

---

## 📌 Security Best Practices

- Use **Managed Identity** or **Azure AD authentication** where possible.
- Apply **RBAC (Role-Based Access Control)** for namespace access.
- Restrict access via **Shared Access Policies** with minimal required permissions.

---

## 📌 Cost Optimization Tips

- Use **Standard vs Premium tiers** appropriately:
  - **Standard**: Suitable for most scenarios.
  - **Premium**: When you need lower latency, predictable performance, and higher message sizes.
- Batch messages and prefetch to reduce transaction and operation costs.

---

## 📌 AZ-204 Exam Tips:

- Know what **PrefetchCount** and **MaxConcurrentCalls** do.
- Understand **dead-lettering** behavior.
- Recognize when to use **message sessions**.
- Be able to choose between **Standard and Premium tiers** for different workload scenarios.
- Expect questions on **retry policies** and handling transient faults.

---

## 📎 Docs References:
- [Service Bus performance tips](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements)
- [Azure messaging best practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/messaging)


## 📎 Official Documentation Links:
- [Azure Service Bus Overview](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- [Queue-based Load Leveling Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling)
- [Competing Consumers Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)
- [Publisher/Subscriber Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/publisher-subscriber)

---
