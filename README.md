# Features of the Website

This file lists all major features of the website with detailed explanations and implementation notes useful for interview discussions.

1. Authentication & Authorization
   - Email/phone + password, OAuth options, and optional social login.
   - MFA for pharmacist/admin roles.
   - RBAC: roles `customer`, `pharmacist`, `admin`, `courier` with least privilege.
   - Implementation notes: JWT for session tokens, refresh tokens stored with rotation, revoke list for compromised tokens.

2. Registration, Profile & Addresses
   - Manage multiple addresses, default shipping, and address verification.
   - KYC flows for regulated products (ID upload, manual verification).

3. Product Catalog & Inventory
   - Product details, category hierarchy, brands, images, SKU management, batch and expiry metadata.
   - Inventory model: available_qty, reserved_qty, incoming_qty; support for multiple warehouses.
   - Inventory sync adapters for POS/legacy systems.

4. Search & Filtering
   - Full-text and faceted search (category, price, brand, prescription-required flag).
   - Autocomplete, fuzzy matching, synonyms, and spelling tolerance using a search engine.

5. Prescription Upload & OCR
   - Image/PDF upload with presigned URLs (S3/GCS) and virus scanning.
   - Optional OCR (for quick pharmacist triage and autocomplete) with human-in-the-loop validation.

6. Cart, Checkout & Price Calculation
   - Centralized server-side price calc (prices, discounts, coupons, taxes) and idempotent checkout.
   - Multiple payment methods, saved payment tokens, guest checkout support.

7. Payment Integration
   - Third-party PSPs with secure tokenized flows.
   - Webhook handlers for asynchronous payment events; idempotent handlers to avoid double charges.

8. Pharmacist Dashboard & Prescription Management
   - Work queues, priority sorting, approve/decline, request clarification flows, notes, and audit trail.
   - SLA enforcement and escalations for unhandled verification tasks.

9. Order Management & Fulfillment
   - Status lifecycle (PLACED, PENDING_VERIFICATION, VERIFIED, READY_FOR_FULFILLMENT, SHIPPED, DELIVERED, RETURNED, CANCELLED).
   - Reservation mechanism for inventory; pick-list printing and packing support.

10. Shipping & Courier Integrations
   - Multi-carrier integrations; shipment creation, tracking, pickup scheduling.
   - Shipment cost calculation and address validation.

11. Invoice & Billing
   - Auto-generate PDF invoices with breakdown: items, taxes, discounts, shipping, payment references.
   - Store invoice metadata and provide downloadable copies; batch sync to accounting systems.

12. Notifications & Communications
   - Email, SMS, in-app notifications for order events, verification requests, promos.
   - Template system with localization support.

13. Returns, Refunds & Support
   - Return request workflow with eligibility checks (time window, item condition).
   - Refund processing (auto or manual) and accounting adjustments.

14. Admin & Reporting
   - Sales dashboards, inventory health, low-stock alerts, pharmacy performance metrics.
   - Auditable logs for prescription approvals, invoice changes, user role modifications.

15. Security & Compliance Features
   - Data encryption at rest and in transit, role-based access, logging and SIEM integration.
   - Data retention policies and secure deletion for PII and prescriptions.

16. Observability & Reliability
   - Metrics (Prometheus), tracing (OpenTelemetry), centralized logs (ELK/Opensearch).
   - Health checks, readiness/liveness probes, circuit breakers for external dependencies.

17. Scalability & Performance
   - Caching (Redis) for session and product reads, CDN for static assets, horizontal scaling for API services.
   - Background worker pools for heavy tasks (PDF generation, OCR, reconciliation).

18. Internationalization & Localization
   - Multi-currency, multi-locale formatting for addresses, taxes, and user-facing content.

19. Accessibility & UX
   - WCAG-compliant UI elements, keyboard navigation, descriptive labels for prescription uploads.

20. Testing & CI/CD
   - Unit, integration, end-to-end tests; contract tests for external APIs.
   - CI pipelines with linting, static analysis, and gated deployments.

Implementation tips (concise)
- Prefer event-driven decoupling (message bus) between orders, fulfillment, invoices, and analytics.
- Keep payment and refund flows idempotent and safe to retry.
- Treat prescription files as high-sensitivity assets (encrypt and limit full-file download windows).
