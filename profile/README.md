# Message Queue
This is the overview of our GitHub organization. Our message queue system is designed to handle distributed messaging with high availability, utilizing Apache Ratis for replication and Raft consensus. The organization contains three main repositories:

- **MessageQueue**: Implements the core message queue functionality and brokers.
- **MessageQueueProxy**: Acts as a proxy and cluster manager for message queue brokers, exposing RESTful APIs.
- **MessageQueueDashboard**: A React-based frontend for monitoring and managing the message queue system.

---

## Setup & Workflow for Running Our MQ Service
### Prerequisites
To run the message queue system, ensure you have the following installed:
- **Docker**: [Installation Guide](https://docs.docker.com/engine/install/)
- **Java Corretto 17 JDK**: [Download Here](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html)

### Running the System
1. **Start Message Queue Brokers** (first, build and run brokers):
```sh
   cd MessageQueue
   mvn clean install
   docker build -t message-queue .
   docker-compose up
```
This creates:
- 3 broker instances as Raft peers.
- mq-network: A docker network for the system.

2.	Start the Proxy Service:
```sh
cd MessageQueueProxy
./mvnw clean install
docker build -t mq-proxy .
docker compose up
```
- This launches the message queue proxy at http://localhost:9090.
- Use http://localhost:8080/swagger-ui/index.html to access API documentation.

3.	Start the Dashboard:
```sh
cd MessageQueueDashboard
npm install
npm start
```
- Visit http://localhost:3000 for the MessageQueueDashboard.
- If running locally, copy src/local.config.json to src/config.json before starting.

## Conceptual MQ

### Detailed & Holistic Scenario

A message queue enables asynchronous communication between services. Consider a coffee shop example:
- Customers place orders at the counter → Producers send messages to MQ.
- Baristas behind the counter take the orders → Consumers process messages from MQ.
- This prevents duplicate or lost orders.

More Examples:
- Topic: Categories for messages (e.g., “Latte Orders”).
- Producers: Order takers placing coffee orders.
- Consumers: Baristas making coffee.
- Subscriptions: Baristas subscribe to a coffee type.

## Pros & Cons of Partition Strategies

Partitioning enables scalability and parallel processing. We support four partition assigners:

### Failover

The **Failover Strategy**: Broker (the message producer or coordinator) sends tasks (messages) to a consumer.
If the consumer that was assigned the task fails (e.g., crashes or becomes unreachable), the failover strategy ensures that the task is not lost.
The task will then be sent to another available consumer to continue processing, ensuring high availability and resilience of the system. (see [Intersystems Failover Documentation](https://docs.intersystems.com/irislatest/csp/docbook/DocBook.UI.Page.cls?KEY=GHA_failover) for more details).

- **How it Works**: If a consumer fails, the system detects the failure and reassigns the task to another available consumer.

- Pros:
      - Ensures high availability and reliability by assigning all partitions to a single consumer until failure occurs.
      - Reduces partition reassignment complexity.
- Cons:
      - Can lead to load imbalance if one consumer handles too many partitions.
      - Recovery from failure can introduce latency.

### Range
- Pros:
  - Simple and efficient distribution of partitions across consumers.
  - Works well when there are fewer partitions than consumers.
- Cons:
  - Consumers at the beginning of the list may get more partitions, causing uneven load distribution.
  - Does not dynamically rebalance partitions efficiently when new consumers join.
### Round-Robin
- Pros:
  - Distributes partitions evenly, preventing load imbalances.
  - Helps in cases where consumers subscribe and unsubscribe frequently.
- Cons:
  - Frequent reassignment of partitions can lead to inefficient processing and state loss.
  - Does not prioritize data locality or message order.
### Sticky
- Pros:
  - Minimizes partition movement when consumers leave, preserving efficiency.
  - Ensures fairness in distribution without frequent reassignments.
- Cons:
  - More complex to implement compared to other strategies.
  - Requires careful handling of edge cases during reassignment.

# References:
- Apache Kafka Partition Assignor Strategies: Kafka Docs
- Apache Ratis Raft-based Replication: Apache Ratis
- Message Queue Architectures and Use Cases: Distributed Messaging

## Architecture MQ

### Activity Diagram & Components
- MessageQueue: Implements core MQ features (topics, partitions, message production/consumption).
- MessageQueueProxy: Exposes REST APIs for external clients.
- MessageQueueDashboard: Provides UI for monitoring MQ operations.

### High-Level Components:
- Producers: Send messages to MQ.
- Consumers: Subscribe to topics and process messages.
- Broker: Manages topics, partitions, and ensures message durability.
- Partition Assigner: Allocates partitions to consumers.

For further design details, refer to design-decisions.md in the MessageQueue repository.

## Resources
- Postman Collection: message-queue.postman_collection.json
- API Docs: http://localhost:8080/swagger-ui/index.html
- Apache Ratis (Raft Consensus for Replication): Official Repo
- gRPC: Used for efficient communication.
