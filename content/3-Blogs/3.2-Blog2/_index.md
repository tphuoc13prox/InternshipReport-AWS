---
title: "Blog 2"
date: 2026-11-16
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# The Order Was Placed but the Warehouse Never Got the Event: How Does the Transactional Outbox Solve This on AWS?

by Huỳnh Ngọc Tuấn | November 16, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338993156865643/)

*A post by team member Huỳnh Ngọc Tuấn. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version, with the full links of the references.*

When designing a sales system on AWS, there is a rather dangerous situation that I think many people may not have noticed: the customer places an order successfully and the database stores it, but the warehouse system never receives the notification to prepare the goods.

At first glance this looks like a simple messaging bug. In reality, it is a classic problem of microservice architectures: the **Dual Write Problem**.

In this post I look at how to solve it with the Transactional Outbox Pattern, combined with Amazon DynamoDB Streams and Amazon EventBridge Pipes. This is one way to build event processing in an Event-Driven Architecture on AWS.

### 1. The problem: the database saved the order but the event was lost

Suppose we have an online store with this flow:

1. The customer sends an order request.
2. The Order Service saves the order to the database.
3. The Order Service sends an `OrderCreated` event to notify the Inventory Service.
4. The Inventory Service receives the event and updates the stock.

A typical implementation might look like this:

```python
def create_order(order):
    save_order_to_database(order)
    publish_event({
        "eventType": "OrderCreated",
        "orderId": order["orderId"]
    })
```

Nothing looks wrong at first. But imagine that:

- The database saves the order successfully.
- Right after that, the application crashes or loses its connection before sending the event.
- The Inventory Service never receives the notification.

The result: the order exists in the database, but the warehouse system does not know it was just created.

The opposite case is just as dangerous: the event is sent successfully but saving to the database fails. The warehouse system may then process an order that does not exist.

This is the Dual Write Problem: the application has to write to two independent systems, but there is no shared transaction guaranteeing that both operations succeed or fail together.

### 2. How does the Transactional Outbox Pattern solve it?

The idea of the Transactional Outbox is quite simple: instead of writing to the database and then sending the event directly, we store both the business data and the event to be published in the same database transaction.

For example, when a customer places an order, the system writes two items:

The Order item:

```json
{
  "PK": "ORDER#1001",
  "SK": "DETAIL",
  "status": "CREATED",
  "total": 1500000
}
```

The Outbox Event item:

```json
{
  "PK": "OUTBOX#evt-1001",
  "SK": "EVENT",
  "eventType": "OrderCreated",
  "orderId": "1001",
  "status": "PENDING"
}
```

Both writes are performed in the same DynamoDB transaction with `TransactWriteItems`.

What matters is that:

- If writing the Order succeeds but writing the Outbox fails, the whole transaction is rolled back.
- If the transaction succeeds, both the Order and the Outbox Event are stored.
- If the transaction fails, no valid event is created from that order attempt.

The application no longer has to perform two independent operations, writing to the database and sending a message, in the same processing flow. Instead, the event is published asynchronously after the transaction has committed.

### 3. Implementing it on AWS with 7 components

To illustrate, I use an architecture made of the following services:

- **Amazon API Gateway:** receives the order request.
- **AWS Lambda – Order Service:** handles the business logic and creates the transaction.
- **Amazon DynamoDB:** stores the Order and the Outbox Event.
- **DynamoDB Streams:** captures the changes in DynamoDB.
- **Amazon EventBridge Pipes:** reads, filters, and forwards events from the stream.
- **Amazon EventBridge:** routes events to the right consumers.
- **Amazon SQS and Lambda:** receive and process work asynchronously.

The flow in detail:

**(1) The customer sends a request.** The client sends an HTTP POST with the order details to API Gateway.

**(2) Lambda handles the request and writes the transaction.** API Gateway invokes the Order Service Lambda, which uses `TransactWriteItems` to write the Order and the Outbox Event to DynamoDB at the same time.

**(3) DynamoDB Streams captures the changes.** After the transaction succeeds, the item changes are recorded in DynamoDB Streams. The stream provides an ordered sequence of changes so that downstream components can read and process the new data.

**(4) EventBridge Pipes picks up the event.** EventBridge Pipes connects directly to DynamoDB Streams. We can configure Filter Criteria so that only the Outbox items that need processing are forwarded, instead of sending every change in the table to the event bus. Pipes can also transform the data before sending it to the target.

**(5) The event is sent to EventBridge.** EventBridge receives the event and uses Event Rules to route it to the interested services. For example:

- `OrderCreated` → Inventory Service.
- `OrderPaid` → Notification Service.
- `OrderCancelled` → Order Management Service.

**(6) SQS takes in the work.** Matching events are sent to an SQS queue. SQS acts as a buffer so that consumers can process work independently of the rate at which customers send requests.

**(7) Lambda processes the event.** The Inventory Lambda reads messages from SQS and updates the stock. If processing fails repeatedly according to the retry configuration, the message can be moved to a Dead-Letter Queue for inspection and reprocessing.

![The Transactional Outbox architecture with DynamoDB Streams, EventBridge Pipes, and EventBridge](/images/3-Blog/blog2_figure1.jpg)

*Figure: The Transactional Outbox architecture using Amazon DynamoDB Streams, EventBridge Pipes, and Amazon EventBridge to deliver events to multiple consumers. Source: AWS Compute Blog*

### 4. What is special about EventBridge Pipes?

One thing I found interesting is that, in the past, if you wanted to read data from DynamoDB Streams and forward it to another system, you usually had to write an extra intermediate Lambda to poll the stream, process the data, and send the event.

With Amazon EventBridge Pipes, AWS provides a managed component that connects a source directly to a target. Some notable capabilities:

- **Filtering:** forward only the events that match a condition.
- **Transformation:** reshape the event before sending it to the target.
- **Retry:** configure retries when processing fails.
- **Dead-letter queue:** configure an SQS DLQ to keep the events that still cannot be processed after the retries.

This reduces the amount of glue code the development team has to manage.

However, Pipes does not turn the whole system into a single transaction. It only connects and forwards events between components.

### 5. With a Transactional Outbox, is each event delivered exactly once?

This is a very important detail. The answer is: **do not assume that an event is processed exactly once.**

DynamoDB Streams, EventBridge, and SQS each have their own retry or delivery mechanisms. In some situations the same event can be processed again. For example:

1. The Inventory Lambda receives an `OrderCreated` event.
2. The Lambda updates the stock successfully.
3. But before the system confirms that the message has been processed, an error occurs.
4. The message is delivered again.
5. The Lambda may run the stock update again.

Without protection, the stock could be decremented twice.

The solution is **idempotency**. Each event should have a unique `eventId`. The consumer records the events it has processed and checks the ID before running the business logic. For example:

```python
def process_event(event):
    event_id = event["eventId"]

    if already_processed(event_id):
        return

    process_business_logic(event)
    mark_as_processed(event_id)
```

This is only an illustration. In a real system, checking, running the business logic, and marking the event as processed must be designed carefully to avoid race conditions. With DynamoDB, a conditional write or a transaction can protect this operation.

Another point to keep in mind is that DynamoDB Streams keeps records for at most 24 hours. The system therefore needs to monitor processing progress and have a suitable recovery plan if a consumer is down for a long time.

### 6. When should you use this architecture?

A Transactional Outbox combined with DynamoDB Streams and EventBridge Pipes suits systems that:

- Have several microservices that need to be notified when data changes.
- Want to separate writing to the database from processing events.
- Need to reduce the risk of losing notifications when the application fails after saving data.
- Want to process business logic asynchronously.
- Need to add more consumers without changing the Order Service directly.

However, not every system needs all of these components. If the application is small, with a single database and one simple consumer, adding too many services can increase cost and operational complexity.

Also, if you use Amazon RDS or Aurora, you can implement an Outbox Table in the relational database and use a separate publisher that reads the outbox and sends messages to SQS.

The choice depends on the current database, the transaction requirements, the event volume, and the team's operational capacity.

### Conclusion

What I took away from studying the Transactional Outbox Pattern is that, in a distributed system, you should not assume that successfully saving data means the other services have been notified.

Instead of trying to perform two independent operations in the same request, we can store the business data and the event in one transaction, and then use a CDC mechanism to publish the event asynchronously.

On AWS, combining DynamoDB, DynamoDB Streams, EventBridge Pipes, EventBridge, and SQS gives a fairly flexible architecture for event-driven applications.

Of course, the Transactional Outbox does not remove every failure in the system. Duplicate delivery, retries, event ordering, and idempotency still need careful design. But it is a pattern worth learning if you want to build microservice systems that handle failures better.

### References – AWS Official

- [Transactional Outbox Pattern – AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
- [Implementing the Transactional Outbox Pattern with Amazon EventBridge Pipes – AWS Compute Blog](https://aws.amazon.com/blogs/compute/implementing-the-transactional-outbox-pattern-with-amazon-eventbridge-pipes/)
- [DynamoDB Transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [Amazon EventBridge Pipes](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-pipes.html)
- [Amazon SQS Dead-Letter Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [AWS Sample – Transactional Outbox Pattern](https://github.com/aws-samples/transactional-outbox-pattern)

#AWS #AmazonDynamoDB #DynamoDBStreams #AmazonEventBridge #EventBridgePipes #AmazonSQS #Microservices #EventDrivenArchitecture #CloudArchitecture #AWSStudyGroup #FCAJ
