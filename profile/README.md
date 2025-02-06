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
 
The **Range Strategy**: In a range-based distribution, the broker (the message producer or coordinator) assigns tasks (messages) to consumers based on predefined ranges of partitions. Each consumer is responsible for a specific range of partitions. When a new consumer joins, the partitions are redistributed, but the range assigned to each consumer remains static unless the system rebalances. (see [Partitioning in Distributed Systems]([https://docs.intersystems.com/irislatest/csp/docbook/DocBook.UI.Page.cls?KEY=GHA_failover](https://medium.com/@roopa.kushtagi/partitioning-in-distributed-systems-ade2fd0cc3ed)) for more details).

- **How it Works**: The broker assigns partitions to consumers based on a range. For example, if there are 10 partitions and 3 consumers, each consumer is responsible for a consecutive range of partitions (e.g., Consumer 1 handles partitions 1-3, Consumer 2 handles partitions 4-6, and Consumer 3 handles partitions 7-10).

- Pros:
   - Simple and efficient distribution of partitions across consumers.
   - Works well when there are fewer partitions than consumers.
- Cons:
   - Consumers at the beginning of the list may get more partitions, causing uneven load distribution.
   - Does not dynamically rebalance partitions efficiently when new consumers join.
  
### Round-Robin
The **Round-Robin Strategy**: ensures equitable distribution of tasks by assigning tasks sequentially to all consumers in a circular order. This strategy is widely used in scenarios requiring balanced load distribution across consumers.

- **How it Works**: The broker assigns tasks to consumers one by one in a fixed rotation. For example, if there are three consumers (Consumer 1, Consumer 2, and Consumer 3), Task 1 will go to Consumer 1, Task 2 to Consumer 2, Task 3 to Consumer 3, and Task 4 will return to Consumer 1.

- Pros:
   - Simple and ensures even task distribution across all consumers.
   - Avoids overloading a single consumer in normal scenarios.
- Cons:
   - Does not consider the current load or processing speed of individual consumers.
   - May result in inefficiency if tasks have varying resource requirements.

### Sticky
The **Sticky Strategy**: ensures task consistency by assigning tasks with the same key to the same consumer. This approach is commonly used when tasks require maintaining state or context.

- **How it Works**: The broker uses a key (e.g., user ID or session ID) to map tasks to a specific consumer. For example, all tasks related to User A may always be routed to Consumer 1, while tasks for User B go to Consumer 2.

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
