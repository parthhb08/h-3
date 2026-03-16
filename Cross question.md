# Cross-Questions — Interview Prep (Easy → Hard)

This file contains questions an interviewer may ask about the website, organized by difficulty. For many questions, hints or short answers are provided to help you prepare concise responses.

---

**Easy — Concept & product-level**
1. What problem does this website solve? (Answer succinctly: online medicine ordering, secure prescriptions, automation)
2. Who are the primary users and stakeholders?
3. What are the main user journeys?
4. Which data is sensitive in this system and why?
5. How do you handle prescription uploads from the user?
6. How does the invoice generation work at a high level?
7. What are the main roles and permissions in the system?
8. Which third-party services would you integrate (payments, shipping, storage)?
9. How do you support multi-address delivery?
10. What is idempotency and why use it in checkout?

**Intermediate — Implementation & architecture**
11. How would you model orders and invoices in the database? (Tables: users, products, orders, order_lines, invoices, payments, prescriptions, inventory)
12. How do you store and protect prescription files?
    - Hint: use object storage (S3/GCS) with encryption, presigned URLs, limited access roles.
13. How would you implement prescription verification workflow?
14. How do you prevent double allocation of inventory? (pessimistic locking, optimistic locking with version, or reservation records)
15. Describe how payments and webhooks should be handled safely.
16. How do you design the notification system for order status updates?
17. How do you handle partial fulfillment and partial invoicing?
18. How do you ensure tax calculation correctness across regions?
19. Where and how would you generate PDFs for invoices? (background job + renderer)
20. How would you implement retries and dead-letter handling for background jobs?

**Advanced — Scalability, reliability & security**
21. How would you scale the search experience for thousands of SKUs? (Use search engine, sharding, caching, incremental indexing)
22. How to design for high order volume at peak times? (auto-scaling, rate-limiting, queueing)
23. How do you ensure transactional integrity across inventory, payment, and order services in a microservices architecture? (sagas, two-phase commit alternatives, compensation transactions)
24. How to design idempotent order creation across retries and duplicate webhook events?
25. How to minimize the time between order placement and fulfillment while preserving correctness?
26. How do you monitor SLAs for pharmacist verification workflows and escalate breaches?
27. How would you secure APIs and prevent abuse (bots, scraping)? (rate limits, CAPTCHAs, WAF)
28. How to implement audit logging for prescription approvals that is tamper-evident? (append-only logs, cryptographic signing, WORM storage)
29. Talk about PCI compliance for payment flows—what parts remain in-scope and how to minimize scope?
30. How would you perform database schema migrations safely in production with zero downtime?

**System-design style questions**
31. Design the invoice generation service. Consider latency, reliability, storage, and re-generation.
    - Hints: create async job generator; store invoice metadata in DB; generate PDF in worker pool; store in immutable object store; return link.
32. Design a prescription verification queue that guarantees each prescription is processed once.
    - Hints: use at-least-once delivery with dedup keys, consumer-side idempotency, and DLQ for manual triage.
33. Design a search system that supports synonyms and misspellings for medicine names.
34. Architect a multi-warehouse inventory to pick the optimal fulfillment location automatically.

**Database & queries**
35. Which indexes would you add for common product queries? (name, category, brand, price, prescription_required flag)
36. How do you avoid N+1 queries when rendering orders with line items?
37. Given a slow query on order list view, what steps would you take to optimize?
38. How would you model time-series stock movement for auditability and reconciliation?

**Security & compliance deep-dive**
39. How do you handle GDPR/CCPA requests for data deletion when invoices must be retained for accounting?
40. How to handle access control for prescription data across pharmacists and auditors?
41. What encryption algorithm and key management model would you choose for stored files?
42. How to implement secure file uploads to avoid malware and PII leakage?

**Integration & operations**
43. How do you reconcile payments, orders, and invoices nightly?
44. How would you roll out a change that alters invoice format? (migrations, versioned templates, opt-in rollout)
45. What metrics and dashboards would you track to detect problems early?
46. How to implement blue-green or canary deployments for the platform?

**Edge cases & tricky scenarios**
47. How do you handle fraud or chargebacks?
48. What if the pharmacist rejects a previously approved prescription after fulfillment begins?
49. How to handle partial refunds where some items are used and others returned?
50. How to support offline/poor-connectivity pharmacies on the network?

**Behavioral / product trade-offs (good to prepare)**
51. Trade-offs between synchronous vs asynchronous verification.
52. When to denormalize read models (e.g., product catalog) for performance vs normalized transactional model.
53. Decide between single monolith vs microservices for a small pharmacy chain.

**Sample answers / quick notes for common hard questions**
- On ACID vs eventual consistency: prefer ACID for single-service inventory updates, but use eventual consistency with compensating transactions for cross-service flows.
- For high availability: use multi-AZ deployments, redundant data stores with automated failover, and design for graceful degradation (read-only features on partial outages).
- On data retention vs deletion: keep invoice copies immutable for accounting, but redact or pseudonymize personal fields when fulfilling deletion requests.

**Coding/whiteboard prompts you might get**
54. Write an SQL query to get last 30 days of orders with total revenue by day.
55. Sketch the API contract for `POST /orders` (request payload, idempotency key, response codes).
56. How to implement optimistic locking for inventory: show pseudo-code or SQL clauses.

Preparation tips
- Have 2–3 concrete trade-offs and justifications for architecture choices.
- Be ready to sketch diagrams: user flow, event-flow for order lifecycle, DB schema for orders/invoices.
- Practice explaining how security and compliance shaped implementation decisions.
- Practice concise, structured answers (problem → approach → trade-offs → outcome).
