# Workflow — End-to-end Detailed Flow

This document describes the full workflow of the website from user registration through invoice generation, including data flows, background processes, and actor responsibilities.

Actors and roles
- Customer (or patient): browses products, uploads prescriptions, places orders.
- Guest: allowed to browse; checkout may require signup or guest-checkout with verification.
- Pharmacist: verifies prescriptions, approves/declines prescription orders, manages fulfillment.
- Admin / Manager: manages catalogs, pricing, invoices, reports, integrations.
- Courier/Delivery Partner: receives orders for pickup and delivery updates.

High-level sequence
1. Registration & onboarding
   - Customer registers with email/phone and password or via OAuth (Google/Apple).
   - System sends email or SMS verification link/OTP.
   - Optional: KYC or address verification for regulated products.
   - Data saved: `users` table (id, name, contact, verified, roles, addresses).

2. Profile & prescription setup
   - User adds personal details, default address, payment methods.
   - Upload prescription via image/PDF using multipart upload or direct S3 presigned URL for large files.
   - Prescription metadata stored (uploader id, timestamp, file url, OCR text if available, status).

3. Catalog browsing & search
   - Customer searches catalog with filters (category, brand, price, generic name).
   - Search powered by DB indexes and optional search engine (e.g., Elasticsearch) for full-text and fuzzy matches.
   - API endpoints: `GET /products`, `GET /products/:id`, `GET /search?q=`.

4. Add to cart & prescription association
   - Customer adds items to cart; prescription can be associated to specific cart items or the whole order.
   - Cart persisted in DB and optionally cached in Redis for speed.

5. Checkout
   - User confirms address, selects shipping option, and chooses payment method.
   - If prescription required, system ensures an approved prescription is attached or prompts for upload.
   - Prices, discounts, tax, shipping costs are calculated server-side (idempotency token generated here).
   - API: `POST /checkout` returns payment intent and order draft.

6. Payment
   - Use PCI-compliant gateway (Stripe/PayPal/Local PSP) or tokenized payment flow.
   - On success, generate order record and transition draft → `PLACED`.
   - Use webhooks from gateway to update payment status reliably (idempotent webhook handlers).

7. Order intake and pharmacist verification
   - Order placed enters `PENDING_VERIFICATION` if prescription required.
   - Pharmacist receives task in dashboard: `orders` queue or SSE/WS notification.
   - Pharmacist reviews prescription image, validates, approves or requests clarification.
   - On approval, transition `VERIFIED` → `READY_FOR_FULFILLMENT`.

8. Inventory reservation & fulfillment
   - On order placement (or at verification), reserve stock (DB transaction or distributed lock).
   - Decrement `available_quantity`, create `reservation` record with TTL; release on timeout or cancellation.
   - Fulfillment team/picking app prints pick lists, packs, and hands to courier.

9. Shipping & tracking
   - Courier integration via API or CSV: `POST /shipments` to create shipment and receive tracking number.
   - System updates order status: `SHIPPED` and sends tracking info to the customer.

10. Delivery and completion
   - Courier updates delivery status; on `DELIVERED`, set order `COMPLETED` and trigger invoice finalization.
   - For failed deliveries, trigger `RETURN` flow and support refunds/attempts.

11. Invoice generation
   - Invoice is generated once payment is captured and order is fulfilled, or upon order completion depending on business rules.
   - Invoice engine compiles order lines, taxes, discounts, shipping, and payment references.
   - Generate PDF (using a templating renderer like wkhtmltopdf or headless browser) and store its S3 URL.
   - Persist invoice metadata: `invoices` table (id, order_id, pdf_url, amount, tax_breakdown, generated_at).
   - Send invoice email with PDF link and/or make downloadable from account.

12. Post-order operations
   - Returns: user initiates return; pharmacist/admin approves; inventory is restocked and refunds issued.
   - Analytics: sales and inventory reports updated via event-streams into data warehouse.
   - Accounting sync: Batch or real-time transfer of invoices to accounting systems.

Data flow & architecture notes
- Use event-driven architecture for decoupling: order events (ORDER_PLACED, PRESCRIPTION_APPROVED, PAYMENT_CAPTURED, ORDER_SHIPPED, ORDER_DELIVERED) published to message bus (RabbitMQ/Kafka) and consumed by fulfillment, notification, analytics services.
- Sensitive files (prescriptions) stored encrypted, with fine-grained access control (only assigned pharmacists + auditors can access).
- Background jobs: invoice PDF generation, OCR processing for prescriptions, sending reminders for unverified prescriptions, periodic stock reconciliation.

Failure and retry handling
- Use idempotency keys for checkout/payment operations.
- Webhooks and external calls retried with exponential backoff; events persisted in a dead-letter queue after threshold.
- Distributed locking/optimistic concurrency used for inventory updates (optimistic release on conflict).

Security & compliance in workflow
- TLS everywhere, secure upload (pre-signed URLs), encryption-at-rest for files and PII.
- Audit logs capturing who accessed/approved prescriptions and invoices.
- Role-based access control (RBAC) and strong authentication (MFA for pharmacists/admins recommended).

Operational considerations
- Monitoring: track queue lengths, processing latencies, errors in verification pipeline, payment failure rates, and invoice generation latency.
- SLOs: set targets for order placement success rates and time-to-fulfillment.

Sequence summary (concise)
1. User registers and uploads prescription.
2. Browse → Cart → Checkout (payment intent created).
3. Payment captured → Order created.
4. Pharmacist verifies prescription.
5. Inventory reserved → Fulfillment picks and hands to courier.
6. Shipment tracking → Delivery → Invoice generated and stored.
