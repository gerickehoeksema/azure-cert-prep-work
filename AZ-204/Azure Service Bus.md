
https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters

https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview

https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sequencing

https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements

https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements?tabs=net-standard-sdk

## Azure Service Bus Fundamentals

### Namespaces

A Service Bus namespace is a container for all messaging components (queues, topics, subscriptions). Each namespace provides a unique scoping container, in which your messaging resources reside. A namespace is essentially the root of your Service Bus address.

Think of a namespace like a domain for your messaging infrastructure. It provides:

- A unique fully-qualified domain name (e.g., `mycompany.servicebus.windows.net`)
- Access control points
- A security boundary for your messaging resources

### Topics and Subscriptions

Topics and subscriptions provide a publish/subscribe (pub/sub) messaging model. Unlike a queue where each message is processed by a single consumer, topics and subscriptions allow multiple subscribers to receive the same message.

Here's how they work together:

1. **Topics**: A topic can receive messages from multiple independent publishers and make those messages available to multiple subscriptions. Messages are sent to a topic and delivered to one or more associated subscriptions.
2. **Subscriptions**: A subscription represents a virtual queue that receives copies of messages sent to the topic. Subscribers can receive messages from a subscription, similar to how they would from a queue. Each subscription can have different filtering rules that restrict which messages it receives.

## Key Exam Concepts for AZ-204

### Namespace Details

For the exam, understand that:

- A namespace is a container for all your messaging components
- You can have multiple queues and topics within a single namespace
- Azure provides three different tiers for Service Bus: Basic, Standard, and Premium
- Topics/subscriptions are only available in Standard and Premium tiers, not in Basic

### Topic and Subscription Rules

The AZ-204 exam often tests your understanding of subscription filters. These are rules that determine which messages from a topic get copied to a specific subscription. There are three types of filters:

- Boolean filters: `TrueFilter` and `FalseFilter`
- SQL filters: Allow complex conditions
- Correlation filters: Match against message properties

### Programming Model

For the exam, you should understand how to:

- Create namespaces, topics, and subscriptions programmatically
- Send messages to topics
- Receive messages from subscriptions
- Handle message sessions and deadletter queues

## Practical Example

```csharp
// Creating a client for a specific namespace
var client = new ServiceBusClient("mycompany.servicebus.windows.net", new DefaultAzureCredential());

// Creating a sender for a specific topic
var sender = client.CreateSender("orders-topic");

// Sending a message to the topic
await sender.SendMessageAsync(new ServiceBusMessage
{
    Body = BinaryData.FromString(JsonSerializer.Serialize(order)),
    Subject = "New Order",
    ApplicationProperties =
    {
        { "Region", "Europe" },
        { "Category", "Books" }
    }
});

// Creating a receiver for a specific subscription
var receiver = client.CreateReceiver("orders-topic", "europe-orders-subscription");

// Receiving messages
var message = await receiver.ReceiveMessageAsync();
```
