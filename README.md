# Microservices Modernization: Workflow Reliability and Eventual Consistency

A software architecture case study exploring the modernization of a legacy PHP monolith into domain-oriented microservices, with a focus on distributed workflows, read-model consistency, payment recovery, and failure isolation.

## Overview

Modernizing a monolith involves more than splitting an application into smaller services. It requires explicit service boundaries, coordination across independently managed data stores, resilient communication, and recovery mechanisms for failures that cross service boundaries.

This case study describes architectural patterns used during a modernization spanning orders, carts, promotions, rewards, payments, subscriptions, catalogs, inventory, notifications, and related business domains.

## Technology Stack

- **Languages:** PHP, Ruby, JavaScript, TypeScript, Java
- **Application frameworks:** Ruby on Rails, React
- **Data and search:** SQL databases, Elasticsearch
- **Communication:** Synchronous calls, asynchronous processing, domain events, queues, and batch processing

The stack varied across services and domains.

## Architecture at a Glance

The architecture separates business responsibilities across services, with service-owned databases and different communication patterns based on workflow requirements.

Key design concerns include:

- Coordinating multi-service workflows through state machines and domain events.
- Separating database-backed writes from Elasticsearch-backed query models.
- Recovering from missing or duplicate payment events through retries and reconciliation.
- Isolating secondary notification processing from successful checkout completion.
- Supporting independent evolution of services and technology choices.

See [`docs/architecture.md`](docs/architecture.md) and [`diagrams/architecture.mmd`](diagrams/architecture.mmd).

## 1. Distributed Workflows and Saga

Business processes often span multiple services, making a single atomic database transaction impractical.

Workflow state machines and domain events coordinate the next actions. Depending on the workflow outcome, the system can continue fulfillment or initiate compensating actions.

This approach makes distributed business processes explicit instead of treating a sequence of service calls as one indivisible transaction.

**Engineering considerations**
- Define valid workflow states and transitions.
- Make repeated event handling safe where required.
- Specify which business actions can be compensated.
- Define recovery behavior for failed or incomplete compensation.
- Monitor workflows that remain incomplete.

The exact transitions and compensation rules depend on the individual workflow.

## 2. CQRS and Elasticsearch Read Models

Database tables support write operations, while Elasticsearch provides a query-oriented read system.

This separation allows query needs to evolve independently from write-side responsibilities. It also introduces eventual consistency: search results may lag behind the latest committed database state.

**Engineering considerations**
- Identify which data source is authoritative.
- Determine which reads can tolerate stale data.
- Plan how read models are refreshed or updated.
- Account for indexing delays and failed updates.
- Avoid relying on search indexes for operations that require authoritative, current state.

## 3. Payment Recovery and Reconciliation

Payment-related workflows combine event-driven processing with retries, idempotency, monitoring, and manual recovery.

Batch processing and reconciliation help identify discrepancies involving duplicate or missing payment events and provide a recovery path beyond normal event processing.

Financial reconciliation requires particular care. The authoritative transaction source, matching criteria, duplicate-prevention rules, and safeguards for financial corrections must be defined by the actual implementation.

## 4. Checkout Failure Isolation

One concrete improvement separated notification processing from checkout completion.

After checkout publishes an event, notification processing can proceed independently. A notification failure can be retried without unnecessarily rolling back an otherwise successful checkout.

This creates clearer ownership between the core purchase workflow and secondary notification work.

The broader modernization also allowed domains such as promotions and reporting to retain distinct technology choices and evolve with clearer responsibilities.

## Engineering Lessons

1. **Model distributed workflows explicitly.** State transitions, business outcomes, and compensating actions need clear definitions.
2. **Make consistency trade-offs visible.** Query performance and read-model flexibility must be balanced against data freshness.
3. **Design for recovery.** Retries, idempotency, monitoring, and reconciliation help handle failures that inevitably occur across service boundaries.
4. **Align service boundaries with business responsibilities.** Independent services are valuable when ownership and interactions are clear.
5. **Treat migration as an engineering process, not just a deployment.** Architecture decisions, implementation, code reviews, production support, and cross-team coordination all matter.

## My Contributions

My responsibilities included architecture decisions, implementation, migration, code reviews, production support, and cross-team coordination.

The work involved evaluating communication patterns, clarifying service responsibilities, and improving reliability across distributed business workflows.

## Further Reading

- [Architecture Diagram](mermain-diagram.png)

---

*This case study describes architectural patterns and engineering lessons. Detailed workflow transitions, infrastructure choices, quantitative outcomes, and implementation-specific safeguards should be added only when verified against the actual system.*
