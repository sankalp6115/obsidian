## 1. Introduction to Heartbeat

In distributed systems and networking, a heartbeat is a signal or message sent at regular intervals to indicate that a node, device, or component is functioning correctly and is still alive.

Heartbeats are a fundamental mechanism for monitoring the health and availability of resources in a distributed environment.

They help systems:
- Monitor the health and availability of nodes.
- Detect failures and unresponsive components.
- Reroute requests to healthy nodes.
- Initiate failover procedures.
- Maintain system reliability and availability.

## 2. How Heartbeat Works
The operation of a heartbeat mechanism depends on the context and system in which it is used. Generally, it works by sending periodic signals or messages between nodes or components in a distributed system.

![[Screenshot 2026-10-15 at 01.21.56.png]]
### 2.1 Periodic Heartbeat Messages

1. Each node or server periodically sends a heartbeat message to its peers.
2. Other nodes listen for incoming heartbeat messages.
3. When a node receives a heartbeat from a peer, it can update its record of that peer's status.
4. If a node does not receive a heartbeat within the expected time frame, it may assume that the peer has failed or become unresponsive.

Servers employ a communication strategy that can be thought of as a "knock-knock" system. Every now and then, servers give a little knock to their neighbours, letting them know they are still awake and ready to work.

This quick check-in helps with two major tasks: routing and failure detection.

### 2.2 Routing

When a new task arises, a server needs to identify the most suitable teammate to handle it.

Servers can consult the collected heartbeat data to determine the appropriate destination for incoming requests. Rather than conducting a time-consuming search, servers can use recent heartbeat information to identify an appropriate candidate.

### 2.3 Failure Detection

Failure detection relies on the absence of expected heartbeat messages.

- If a node does not receive a heartbeat from a peer within the configured timeout period, it may assume that the peer has failed or is unresponsive.
- When a server's heartbeat stops entirely, other servers can detect the missing pulse.
- They can then take necessary actions, such as rerouting requests or initiating failover procedures.
    
This collaborative effort prevents disruptions and maintains system reliability, similar to team members stepping up when a colleague encounters a challenge.

## 3. Initialization of a Heartbeat Mechanism

In a distributed system, multiple nodes or components need to monitor each other's health and availability.

The heartbeat mechanism is initialized by configuring settings such as:
- The frequency of heartbeat messages.
- The addresses of the nodes with which communication must take place.
- The timeout duration for detecting missing heartbeats.

Nodes or components continuously listen for incoming heartbeat messages from their peers.
When a node receives a heartbeat message from a peer, it can acknowledge the receipt to confirm that the sender is alive and responsive.

If no heartbeat is received within the configured timeout duration, the system can trigger actions such as:
- Rerouting traffic to other healthy nodes.
- Initiating a failover procedure.

## 4. Heartbeat with Acknowledgement

In this mechanism, nodes send heartbeat signals, and recipients are expected to send an acknowledgement (ACK) message back to the sender.
This ensures not only that the node is operational but also that the communication channel is working in both directions.

### Advantages
- Can reduce false positives because acknowledgements provide additional evidence of successful communication.
- Helps verify bidirectional communication.

### Disadvantages
- Higher network overhead: Additional acknowledgement messages increase network traffic.
- Increased complexity: Implementing acknowledgements adds complexity to the heartbeat system.

## 5. Types of Heartbeat Detection Mechanisms

### 5.1 Push-Based Model

In a push-based model, nodes or components actively send heartbeat signals at regular intervals to designated recipients, such as monitoring systems or other nodes.

Advantages:
- Potential failures can be identified quickly.
- Particularly useful for monitoring systems with strict failure-detection requirements.

Disadvantages:
- Each node generates periodic heartbeat traffic.
- Network overhead can increase, potentially contributing to congestion.

### 5.2 Pull-Based Model
In a pull-based model, a central monitoring system or designated node periodically sends an "Are you alive?" request to other nodes.
The nodes respond with a heartbeat message to confirm their availability.
In this model, nodes do not send unsolicited heartbeat messages.

Advantages:
- Reduces unsolicited heartbeat traffic.
- The central monitoring entity has a clear overview of system health.

Disadvantages:
- Failure detection depends on the polling interval of the central monitor.
- Failures may be detected with a delay.

### 5.3 Heartbeat with Quorum
Heartbeat with quorum is used in highly distributed systems where a majority vote is required to confirm availability.
Nodes send heartbeat signals to multiple peers, and a node is considered alive only if it receives a quorum, or majority, of acknowledgement responses from its peers.

Advantages:
- Reduces the risk of a single network issue or node failure causing incorrect failure detection.
- Suitable for highly distributed systems requiring fault tolerance.

Disadvantages:
- Significant network overhead: Heartbeats and acknowledgements are sent to multiple nodes, increasing traffic.
- Coordination overhead: Managing and reaching quorum adds complexity.

## 6. Heartbeat Messages
Heartbeat messages are typically lightweight, low-overhead signals or packets that convey a node's status or presence.
Their content varies according to system-specific requirements.
A heartbeat message can be as simple as a periodic "I'm alive" signal, or it can contain additional information about the sender, such as:
- Current status.
- Load.
- Capabilities.

### 6.1 Redundancy and Failover
In systems that prioritize high availability, multiple nodes or components often send and receive heartbeat messages.
Redundancy ensures that if one node or component fails or becomes unresponsive, others can take over its tasks or responsibilities.
Failover mechanisms, such as promoting a standby node to an active state, may be triggered when a failure is detected through the heartbeat mechanism.

### 6.2 Monitoring and Alerts
Monitoring tools can use information collected from heartbeat messages to assess the health and status of nodes or components.
Alerts or notifications can be generated when failures are detected, enabling proactive intervention.

## 7. Common Heartbeat Implementations
### 7.1 Simple Ping
Servers ping each other at regular intervals and expect a response to confirm that they are alive.
This is similar to giving a neighbour a quick knock and waiting for a response.

### 7.2 Gossip Protocols
Gossip protocols are procedures for peer-to-peer communication in which servers share heartbeat information with their immediate neighbours, who then propagate it further.
This ensures that information spreads throughout the system.

#### Example: Content Delivery Network (CDN)
Imagine you are trying to access a popular news website. A gossip protocol can help distribute information about new content across a CDN.

Process:
1. The CDN's central server receives a new update from the news website, such as a breaking news story or a fresh article.
2. The central server gossips about the update to a few nearby servers, such as Server A and Server B. This gossip may simply notify them that a new update exists.
3. Server A and Server B share the information with their immediate neighbours. For example, Server A informs Server C, while Server B informs Server D.
4. This process continues, with each server informing a few of its neighbours.
5. Servers farther away from the central server receive the information a little later.
6. When you try to access the news website, a local DNS server directs you to the nearest CDN server, which might be Server B.
7. Server B, having received the update notification, checks its local cache. It might already have downloaded the update from the central server based on the gossip or through a separate update mechanism.
8. If Server B has the update, it can serve the latest version of the website quickly.
    

This allows users to connect to a nearby server that has the most recent content.

### 7.3 Leader-Based Heartbeats

In a hierarchical system, leader-based heartbeats are used in which a designated leader collects and analyses heartbeat data from all servers, providing a centralized view of system health.

Heartbeats also form the foundation of the leader election process in distributed systems, where nodes must determine which node will act as the leader.

#### Leader Election

Leader election is the process of assigning one process or machine in a distributed system special powers.
These powers may include:

- Assigning work.
- Handling all requests in the system.

It is similar to electing a president for a committee.

#### Example: Delivery Drones
Imagine a fleet of delivery drones operating in a busy city. Each drone needs to know the location and status of other drones to avoid collisions and coordinate deliveries efficiently.

1. Before starting deliveries, the drones use a leader election algorithm to choose a single leader drone.
2. The leader may be selected based on factors such as battery life, processing power, or central location.
3. The leader drone periodically sends a heartbeat message containing its current location and status.
4. Other drones listen for these leader heartbeats.
5. When a drone receives a heartbeat, it updates its internal map with the leader's location and status.
    

## 8. Real-World Applications of Heartbeats

### 8.1 Web Services and High Availability

Multiple server instances are often deployed to handle incoming requests.

Load balancers distribute traffic among these servers, while heartbeat mechanisms monitor their health and availability.

Working:
1. Each server periodically sends heartbeat signals to the load balancer.
2. The load balancer monitors the heartbeat signals or checks whether the servers are responsive.
3. If a server stops sending heartbeats or becomes unresponsive, the load balancer identifies the problem.
4. Incoming traffic is redirected to other healthy servers.


This ensures that the service remains available even if one server fails, minimizing downtime for users.

![[Screenshot 2026-10-15 at 01.29.00.png]]

### 8.2 Networking: Routers and Switches
In networking, heartbeat-like messages, often called keepalives or hello packets, are exchanged between neighbouring routers or switches.
These messages confirm the liveness of network links and neighbouring devices.
If a router stops receiving keepalive messages from a neighbour, it can consider the link or neighbour down and reroute traffic accordingly.
This is crucial for maintaining network stability and routing efficiency.

## 9. Heartbeat Frequency and Timeout Configuration
The frequency of heartbeat messages is a balancing act.
- Too frequent: Network overhead increases.
- Too infrequent: Failures may not be detected quickly enough.

Timeout values must also be carefully tuned.
- Timeout too short: Temporary delays may cause false positives, making a healthy node appear to have failed.
- Timeout too long: Actual failures take longer to detect and handle.
    

### 9.1 Example: Large Online Store
Imagine managing a large online store with hundreds of servers handling customer requests across multiple data centres.

#### Scenario A: Heartbeat Every Second
If the heartbeat frequency is set to one second:

Advantages:
- Failures are detected quickly.
- Downtime can be minimized for customers.

Disadvantages:
- Every server sends and receives heartbeat messages every second.
- Constant communication increases network traffic and may cause congestion.
- Processing frequent heartbeat messages consumes CPU resources.
- Server resources may be diverted from handling customer requests efficiently.
    

The sheer volume of heartbeat messages could overload network infrastructure and cause delays in critical operations, such as processing customer transactions.

#### Scenario B: Heartbeat Every Minute
If the heartbeat frequency is set to one minute:

Advantages:
- Network overhead is reduced.
- The strain on the network and servers is lower.

Disadvantages:
- Up to a minute may pass before a failure is detected, depending on the timeout configuration.
- Customers may experience errors or delays while the system waits to detect the failure and reroute requests.
- If a server handling shopping carts fails, checkout operations may be disrupted.
- Delayed detection can contribute to lost orders or inconsistent data if the system does not handle failures correctly.    

### 9.2 Finding the Right Balance
The goal is to find the right balance between responsiveness and overhead.
Heartbeat frequency and timeout values should be selected according to the requirements and behaviour of the actual system.
Monitoring dashboards and tools can help identify an appropriate heartbeat frequency based on real system behaviour.