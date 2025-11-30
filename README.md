🚀 Project Titan: Distributed E-Commerce System

## 🌟 Overview
Project Titan is a system design blueprint for a high-availability, cloud-native e-commerce platform. The architecture is designed for immense scalability and low-latency global access, capable of sustaining a peak load of **1,000 requests per second (RPS)** across multiple geographic regions.

The design focuses on decoupling critical paths using **microservices** (Auth, Orders, Payments), achieving resilience through **horizontal scaling** and **read replicas**, and utilizing **asynchronous processing** via dedicated worker queues.

---

## 🗺️ System Design Diagram
A high-level view of the entire architecture, illustrating the request flow from the user through the primary node to the backend services and workers.

<img width="1395" height="769" alt="Screenshot 2025-11-30 175350" src="https://github.com/user-attachments/assets/d134b445-e7da-40c3-9d6e-8bbea4487228" />

*(Note: The full design rationale is available in the Design Document below.)*

---

## 🛠️ Key Architectural Highlights

The system incorporates several advanced design patterns for performance and reliability:

### 1. **Global Distribution & Performance**
* **Anycast Routing:** Directs global users to the nearest point of presence for initial connection, improving latency.
* **CDN Integration:** Utilizes a Content Delivery Network for caching static assets closer to the user, reducing load on the primary nodes.

### 2. **Scalability & Resilience**
* **Horizontal Scaling:** Implemented for the primary processing nodes and database replicas (READ Replicas) to distribute load and prevent single points of failure.
* **Rate Limiting:** A **Token Bucket** or **Leaky Bucket** algorithm is applied at the Load Balancer level (`10.2.3.7`) to protect downstream microservices from traffic spikes and Denial of Service (DoS) attempts.

### 3. **Decoupling & Asynchronous Processing**
* **Microservices:** The architecture is decoupled into dedicated services (`Auth`, `Orders`, `Payments`) communicating over an API Gateway.
* **Worker Queues:** Asynchronous processing using SQS-style queues allows non-critical tasks (Email, WhatsApp, Bulk Processing) to be handled by dedicated workers, ensuring the core payment/order flow remains fast.

---

## 📂 Repository Structure
This repository focuses on design documentation and architectural notes, serving as the definitive record of the system's blueprint.

---

## 💡 Inspiration & Resources
This system design architecture was inspired by common industry practices and modeled after a system design breakdown of a leading e-commerce platform.

A special thanks to Piyush Garg for their detailed system design video which provided a framework for the initial architecture sketch.

(https://www.youtube.com/watch?v=lFeYU31TnQ8)
