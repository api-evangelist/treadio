---
name: Collect tickets and approve loads
description: Pull a job's loads and tickets from the field and approve completed loads for billing.
api: openapi/treadio-horizon-openapi.json
operations: [get-v1-companies-company-id-jobs, get-v1-jobs-job-id-loads, get-v1-jobs-job-id-tickets, put-v1-loads-id-approve, get-v1-tickets]
---

# Collect tickets and approve loads (Tread Horizon)

A Load is one pickup-to-drop-off cycle; a Ticket is the proof-of-work record for a Load.
Approving loads is what makes work billable/payable in a Settlement.

## Steps

1. **List jobs for the company.** `get-v1-companies-company-id-jobs`
   (`GET /v1/companies/{company-id}/jobs`).
2. **List loads for a job.** `get-v1-jobs-job-id-loads` (`GET /v1/jobs/{job-id}/loads`).
3. **List tickets for a job.** `get-v1-jobs-job-id-tickets` (`GET /v1/jobs/{job-id}/tickets`) —
   drivers capture scale tickets by photo/OCR in the mobile app.
4. **Approve each completed load.** `put-v1-loads-id-approve`
   (`PUT /v1/loads/{load-id}/approve`).
5. **Reconcile company-wide.** `get-v1-tickets` (`GET /v1/tickets`) with `page[limit]` and the
   `Link`-header cursor to sweep all tickets.

## Rules

- Approving a load already in a non-approvable state returns `409 conflict`; check the
  `errors[]` state/event before retrying.
- Paginate with `page[limit]` (max 100) and follow the `Link` `rel="next"` URL.
- Ticket webhooks (`Ticket::create`, `Ticket::state_change`) let you react the moment a ticket
  lands rather than polling (see `asyncapi/treadio-webhooks.yml`).
