# PolyglotBot: Your Multilingual Assistant

## Introduction

In our increasingly interconnected world, the demand for tools that can bridge language barriers and enhance learning experiences has never been higher. **PolyglotBot** stands at the forefront of this innovation, offering non-English speakers a powerful tool for creating presentations, essays, and for reading content aloud in a multitude of languages. This document explores PolyglotBot's functionality, its technological backbone, and how it leverages cutting-edge technologies to provide a seamless user experience.

## Workflow and Enhancements

### UML Diagram
![UML Diagram](https://iyvhmpdfrnznxgyvvkvx.supabase.co/storage/v1/object/public/articles/whiteboard_exported_image.png "Micro-Service Structure")

### User Engagement

PolyglotBot greets users with a simple yet engaging interface, allowing them to specify requirements for presentations, essays, or any text-based tasks. Its multilingual capabilities ensure broad accessibility, catering to a diverse global audience.

### Content Synthesis

Utilizing advanced AI, machine learning models, and natural language processing (NLP) techniques, PolyglotBot generates structured, coherent content. For presentations, it automatically selects appropriate templates and populates them with AI-generated text, creating professional and visually appealing PowerPoint slides.

### Chat Generate and User Behavior Analysis 
The Chat Service interacts with the OpenAI API to generate essays, implementing a circuit breaker pattern using Resilience4J to manage high traffic volumes and prevent system overload. In case of failures, a fallback function is in place to handle exceptions gracefully. After generating the essay, the Chat Service notifies the Recommendation Service via message queue, which analyzes the user's chat history and provides personalized recommendations based on the insights derived from this analysis.

### Payment 
The User Service will update the user's prime membership expiration date and then use a message queue to notify the Subscription Service to complete the payment process. The Subscription Service will then finalize the payment and record the transaction details.

## Technological Foundations and Deployment

### Microservices Framework

PolyglotBot is built on a microservices architecture, enhancing scalability, flexibility, and the independence of its components. This approach facilitates easy updates and the addition of new features and languages with minimal impact on the existing system.

- **API Gateway**: Features authentication capabilities as well as routing.
- **User Service**: Leverages Supabase's auth component for secure user access.
- **Discovery Server**: Uses Eureka Server for service registration.
- **Essay Creation Service**: Automates the process of essay writing.
- **Text-to-Speech Service**: Converts text into high-quality audio in various languages.

### Docker Compose and Microservices Overview

PolyglotBot’s architecture is enhanced with Docker Compose, which facilitates easy management and orchestration of its microservices. Below is an overview of the core components and services:

- **Zookeeper and Kafka**: Central to handling distributed messaging within the microservices architecture, with Zookeeper providing coordination services for Kafka, which is used for event streaming and communication between services.

- **MySQL and PostgreSQL Databases**: Two relational databases, MySQL and PostgreSQL, store and manage data for different services depending on their needs.

- **Spring Boot Microservices**:
  - **Chat Service**: Handles user chat functionalities and interacts with PostgreSQL. It also communicates with the OpenAI API for generating essays and uses Resilience4J for fault tolerance.
  - **Recommendation Service**: Provides user recommendations based on data analyzed from Kafka events, interacting with the PostgreSQL database.
  - **Subscription Service**: Manages subscription payments, interacting with Kafka and using MySQL for data persistence.
  - **User Service**: Handles user data management, storing information in a MySQL database.
  - **Gateway**: Acts as a central entry point for external requests, routing them to the appropriate microservices.

### Kubernetes (K8s)

Kubernetes orchestrates PolyglotBot's containerized microservices, ensuring high availability, effective load balancing, and automatic scaling. This maintains optimal performance across fluctuating usage patterns.

### Redis

A key component of PolyglotBot's architecture, Redis serves multiple purposes:

- **Caching**: Improves response times by storing frequently accessed data.
- **Session Management**: Manages user sessions to preserve the context of interactions.
- **Message Brokerage**: Facilitates communication between microservices for streamlined operations.

### Docker

By containerizing each microservice with Docker, PolyglotBot achieves a consistent operating environment that simplifies deployment and scaling. Docker's benefits are maximized in conjunction with Kubernetes, enhancing the reliability and efficiency of the system.

### Docker Compose File Overview

The Docker Compose setup reflects the microservices architecture, with several interconnected services:

- **Zookeeper Service**:
  - **Image**: `confluentinc/cp-zookeeper:latest`
  - **Ports**: Exposes port `2181` for Zookeeper clients.
  - **Environment Variables**: Configures the client port and tick time.

- **Kafka Service**:
  - **Image**: `confluentinc/cp-kafka:latest`
  - **Depends On**: Zookeeper.
  - **Ports**: Exposes port `9092` for Kafka clients.
  - **Environment Variables**: Includes broker ID, Zookeeper connection, and replication factor configurations.

- **MySQL Database Service**:
  - **Image**: `mysql:latest`
  - **Ports**: Exposes port `3306`.
  - **Volumes**: Uses `mysql-data` for data persistence.
  - **Environment Variables**: Configures the root password and database name.

- **PostgreSQL Database Service**:
  - **Image**: `postgres:latest`
  - **Ports**: Exposes port `5432`.
  - **Volumes**: Uses `postgres-data` for data persistence.
  - **Environment Variables**: Configures username, password, and database name.

- **Chat Service**:
  - **Build Context**: `./chat-service`
  - **Depends On**: Kafka, PostgreSQL.
  - **Ports**: Exposes port `8085`.
  - **Environment Variables**: Configures Spring profiles, data source details, and Kafka connection.
  - **Additional Functionality**: Utilizes Resilience4J for resilience and sends requests to OpenAI for generating essays.

- **Recommendation Service**:
  - **Build Context**: `./recommendation-service`
  - **Depends On**: Kafka, PostgreSQL.
  - **Ports**: Exposes port `8084`.
  - **Environment Variables**: Configures Spring profiles, data source details, and Kafka connection.
  - **Additional Functionality**: Analyzes user behavior by processing Kafka events.

- **Subscription Service**:
  - **Build Context**: `./subscription-service`
  - **Depends On**: Kafka, MySQL.
  - **Ports**: Exposes port `8082`.
  - **Environment Variables**: Configures Spring profiles, data source details, Kafka connection, and Stripe API key for handling subscription payments.
  - **Additional Functionality**: Manages subscription payments and interacts with Kafka for event-driven processing.

- **User Service**:
  - **Build Context**: `./user-service`
  - **Depends On**: MySQL, Kafka.
  - **Ports**: Exposes port `8081`.
  - **Environment Variables**: Configures Spring profiles, data source details, and Kafka connection.
  - **Additional Functionality**: Manages user data stored in the MySQL database.

- **Gateway Service**:
  - **Build Context**: `./gateway`
  - **Depends On**: All microservices.
  - **Ports**: Exposes port `8080`.
  - **Environment Variables**: Configures Spring profiles and service URIs for routing requests to other microservices.
  - **Additional Functionality**: Centralized routing of requests, including interaction with a JWT token-based authentication server.

### Networking and Volumes

- **Networks**:
  - **Backend Network**: All services are connected to a single `backend` network, allowing secure communication between microservices within a Virtual Private Cloud (VPC) and a subnet.

- **Volumes**:
  - **mysql-data**: Volume for persisting MySQL data.
  - **postgres-data**: Volume for persisting PostgreSQL data.

### Service Dependencies and Interaction

- **Gateway Service**: 
  - Receives requests from external actors and routes them to appropriate microservices.
  - Interacts with the JWT token-based authentication server for securing communications.
  
- **Chat Service**: 
  - Manages chat functionalities, sends requests to OpenAI for generating essays, and publishes events to Kafka for analyzing user behavior.

- **Recommendation Service**: 
  - Processes Kafka events to analyze user behavior and generate recommendations.

- **Subscription Service**: 
  - Handles payment processing for subscriptions and interacts with Kafka for event-driven data processing.

- **User Service**: 
  - Manages user-related data and interacts with other services through Kafka events.

## Conclusion

PolyglotBot exemplifies a robust and scalable architecture designed to provide high-quality multilingual content generation and delivery. By leveraging a microservices architecture, Kubernetes, Redis, and Docker, PolyglotBot ensures high availability, flexibility, and a seamless user experience across various languages and content types. The integrated use of advanced AI technologies further enhances its capabilities, making it an invaluable tool for users worldwide.
