---
name: Create and dispatch an order
description: Create a project and daily order, accept it, then assign drivers and send the jobs.
api: openapi/treadio-horizon-openapi.json
operations: [post-v1-projects, post-v1-companies-company-id-orders, put-v1-companies-company-id-orders-order-id-accept, get-v1-companies-company-id-jobs, put-v1-companies-company-id-jobs-bulk_assign_drivers, put-v1-companies-company-id-jobs-bulk_send]
---

# Create and dispatch an Order (Tread Horizon)

The dispatch hierarchy is Project -> Order -> Job -> Load. Most write endpoints are scoped to
a company via the URL (`/v1/companies/{company-id}/...`).

## Steps

1. **Create a project** (optional if one exists). `post-v1-projects` (`POST /v1/projects`).
2. **Create the daily order.** `post-v1-companies-company-id-orders`
   (`POST /v1/companies/{company-id}/orders`) referencing the project, material, sites, and truck count.
3. **Accept the order.** `put-v1-companies-company-id-orders-order-id-accept`
   (`PUT /v1/companies/{company-id}/orders/{order-id}/accept`).
4. **List the resulting jobs.** `get-v1-companies-company-id-jobs`
   (`GET /v1/companies/{company-id}/jobs`) — one Job is one driver's work for the order.
5. **Assign drivers.** `put-v1-companies-company-id-jobs-bulk_assign_drivers`
   (`PUT /v1/companies/{company-id}/jobs/bulk_assign_drivers`).
6. **Send to phones.** `put-v1-companies-company-id-jobs-bulk_send`
   (`PUT /v1/companies/{company-id}/jobs/bulk_send`) pushes assignments to the driver app.

## Rules

- State transitions are enforced by a state machine; an out-of-order action returns
  `409 conflict` with an `errors[]` array (model/field/state/event). See `errors/`.
- No idempotency-key exists — do not blindly retry a `POST` on a network error; re-list
  first to check whether the resource was created (see `conventions/`).
- Subscribe to `Order::*`, `Job::*`, and `Ticket::*` webhooks to track progress instead of
  polling (see `asyncapi/treadio-webhooks.yml`).
