# 🌌 Project Titan: Distributed E-Commerce System Design

**Author:** Shreyas Joshi
**Context:** High-level system architecture designed to meet the extreme scale and resilience demands of leading global e-commerce platforms.

---

## 1. Requirements and Goals

The architecture is built on a set of aggressive functional and non-functional requirements (NFRs) necessary for an "enterprise-scale" system.

### 1.1 Functional Requirements (What the System Does)
* **User Management:** Users can register, log in (Auth Service), and manage profiles.
* **Order Lifecycle:** Users can browse products, add items to a cart, and place an order (Orders Service).
* **Transaction Processing:** The system must securely process payments (Payments Service).
* **Notifications:** The system must send asynchronous order confirmations via email and SMS/WhatsApp.

### 1.2 Non-Functional Requirements (NFRs)
| Requirement | Target | Architectural Solution |
| :--- | :--- | :--- |
| **Availability** | $\ge 99.99\%$ (Four Nines) | Redundancy via Read Replicas, Horizontal Scaling, and Queue-based decoupling. |
| **Latency** | Order Placement $\le 200 \text{ ms}$ | CDN, Anycast Routing, In-memory caching (Redis), and ELB load distribution. |
| **Scalability** | Must handle massive, unpredictable peak traffic (e.g., flash sales). | Horizontal scaling for compute layer and asynchronous workers. |
| **Consistency** | **Strong** for Payments/Inventory; **Eventual** for User Notifications/Stats. | Database transaction handling for payments; queues for eventual consistency. |

---

## 2. Capacity Estimation and Sizing

While a specific target RPS is not explicitly required for this design, the architecture is sized to handle traffic typical of a major retailer experiencing high peak load.

### Target Scale Justification
The current architecture necessitates scaling components to handle peaks that could reach **1,000+ Requests Per Second (RPS)**. This number is derived from a typical capacity plan for a high-volume platform:

1.  **Assumed User Base:** 5 Million Daily Active Users (DAU).
2.  **Average RPS:** $\approx 1,157 \text{ RPS (Average)}$ across 24 hours.
3.  **Peak Load Factor:** Using a conservative peak factor of $3\text{x}$, the peak target is over $\mathbf{3,400 \text{ RPS}}$.

Designing for a foundation of **1,000 RPS (Average)** ensures the underlying components are robust enough to handle the sustained pressure leading up to peak events.

### Component Sizing Focus
* **Primary Node:** Designed for **Horizontal Scaling** (adding more machines) rather than Vertical Scaling to ensure elasticity and fault tolerance.
* **Database:** Requires **Read Replicas** to offload the $1,000+ \text{ RPS}$ read traffic from the Primary Node.

---

## 3. High-Level Architecture Walkthrough

### 3.1 Request Flow
1.  **Global Entry:** A user connects to **ecomm-corp.com**. This request is handled by **Anycast** routing to the closest regional entry point.
2.  **Edge Defense:** The request hits a **CDN** and then a **DNS** provider which points it to the nearest **Load Balancer (ELB)**.
3.  **Rate Limiting:** The ELB layer includes a **Rate Limiting** node (using a Token Bucket mechanism) to protect the Primary Compute layer by capping requests at a safe threshold.
4.  **Compute & Scaling:** The request is distributed across the **Primary Compute Nodes**. Authentication requests go to the **Auth Service**.
5.  **Microservices & API Gateway:** Requests requiring order or payment logic go through the **API Gateway** to the respective services.

### 3.2 Asynchronous Processing
* The **Payments Service** drops a message into a **Queue** (e.g., SQS) after confirming the transaction.
* Dedicated, non-critical workers (**Email Worker**, **SMS/WhatsApp Worker**, **Bulk Worker**) poll the Queue, pick up the message, and complete the respective high-latency tasks without blocking the main user order flow.

---

## 4. Deep Dive: Component Justification

### 4.1 Rate Limiting (Token Bucket)
* **Choice:** **Token Bucket** over Leaky Bucket.
* **Justification:** Token Bucket offers superior **burst control**. It allows pre-approved bursts of traffic (when tokens are available) up to the bucket capacity, which is crucial for genuine traffic spikes at the start of an e-commerce sale.

### 4.2 Database & Caching
* **Read Replicas:** This is mandatory at this scale; it ensures that 99% of read operations (e.g., product browsing) hit the replicas, allowing the Primary (WRITE) Node to focus solely on high-value transactions.
* **CDN:** Caches static assets globally, significantly reducing the load on the ELB and Primary Nodes, improving latency for all users.

### 4.3 Worker Decoupling
* **Bulk Worker:** Crucial for performance. Tasks like generating complex reports, running ML models for recommendations, or performing massive inventory updates are handled offline by the **Bulk Worker**. This ensures the main microservices are never blocked by heavy computational tasks.
