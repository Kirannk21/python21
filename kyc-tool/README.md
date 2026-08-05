# KYC Document Collection Tool (Prototype)

A simple, mobile-friendly tool that **only collects and stores** KYC documents
submitted by customers. It does **not** verify, approve, reject, or
authenticate documents automatically — the internal team reviews every
submission separately.

> **Prototype note:** this is a front-end prototype using **sample data
> only**. Submissions are stored in the browser's `localStorage` so the
> customer flow and admin dashboard can demonstrate the complete workflow
> without a backend. The [Production architecture](#production-architecture)
> section describes how each security requirement is implemented in a real
> deployment.

## Files

| File | Purpose |
|---|---|
| `index.html` | Customer upload flow (mobile-first) |
| `admin.html` | Staff admin dashboard |

Open `index.html` in a browser to try the customer flow, then open
`admin.html` **in the same browser** to see the submission appear in the
dashboard (prototype credentials: `admin` / `kyc@demo`).

## Customer flow

1. Customer receives a WhatsApp message with a secure upload link
   (the admin dashboard's *"Send upload link again"* template produces it).
2. Customer opens the link and sees a **privacy notice**, then enters:
   full name, mobile number, email (optional), and customer/application ID
   (the ID pre-fills from the link's `?cid=` parameter).
3. Customer selects a document type: Aadhaar Card, PAN Card, Passport,
   Driving Licence, Voter ID, Address Proof, Business Registration
   Document, or Other.
4. Customer uploads a clear image (JPG/PNG) or PDF, up to 5 MB per file.
   **Front and back sides** are required for Aadhaar, Driving Licence and
   Voter ID; the back side is optional for other card-type documents.
5. Uploaded files are shown for confirmation together with the entered
   details (mobile number masked).
6. Customer must tick an explicit **consent** checkbox before submitting;
   the consent text, timestamp and version are recorded with the submission.
7. A **reference number** (`KYC-YYYYMMDD-NNNN`) is generated and shown with
   the message: *"Your documents have been received and are subject to
   review."* — the tool never displays "KYC verified".
8. Where WhatsApp integration is available, a confirmation message with the
   reference number is sent (in the prototype, the admin dashboard's
   *"Confirm documents received"* template).

Languages: **English** (default), **ಕನ್ನಡ (Kannada)** and **हिंदी (Hindi)**
via the selector in the header. A progress indicator shows Details →
Documents → Confirm → Done.

## Admin dashboard (`admin.html`)

- **Staff login** (prototype credentials above; production uses SSO/2FA).
- **View all submissions** with status counts at a glance.
- **Search** by name, mobile number, customer ID or reference number, plus
  a status filter.
- **View and download documents** — every view/download is written to the
  activity log.
- **Add internal notes** (never visible to the customer).
- **Change status**: Documents Received · Under Review · Additional
  Document Required · Accepted · Rejected.
- **WhatsApp messages** from templates: request a missing document, request
  a clearer copy, resend the upload link, or confirm receipt. Opens
  `wa.me` in the prototype; production uses the WhatsApp Business API.
- **Activity log** records date, time, user and action for every event
  (submission, status change, note, view, download, WhatsApp message,
  reveal of a masked number).
- **Retention delete** — permanent deletion of a submission and its
  documents in line with the company retention policy, recorded in the
  audit trail.
- **Export CSV** of all submissions (mobile numbers masked in the export);
  the CSV opens directly in Excel.
- Mobile numbers are **masked** everywhere by default (`98XXXXX345`); a
  "Reveal" action exists for authorised staff and is logged.

## WhatsApp invitation message

> Hello [Customer Name], please use the secure link below to upload your
> KYC documents. This link is intended only for document collection. Your
> documents will be reviewed separately after submission.
>
> Secure upload link: [Link]
>
> Please do not send your KYC documents directly in the WhatsApp chat.

## Production architecture

The prototype demonstrates the workflow; a production deployment implements
the security requirements as follows:

| Requirement | Production implementation |
|---|---|
| Encrypt in transfer & storage | HTTPS/TLS 1.2+ everywhere; documents stored in encrypted object storage (e.g. S3 with KMS-managed keys or equivalent), encrypted database fields for PII |
| Restrict access to authorised staff | Server-side authentication (SSO + 2FA), role-based access control, short-lived sessions |
| Secure upload links | Single-use, tokenised, expiring links tied to the customer/application ID |
| No public document links | Documents served only through authenticated, short-lived signed URLs; storage buckets private |
| Mask ID numbers | Masked by default in every list, detail view and export; unmasking is role-gated and audit-logged |
| Record consent | Consent text version, timestamp, IP and user agent stored with each submission |
| Privacy notice | Shown before collection begins, in the customer's chosen language |
| Retention & deletion | Scheduled deletion jobs per policy (e.g. 5 years after account closure) plus on-request erasure workflow; all deletions audit-logged |
| No AI training | Contractual and technical controls: documents are excluded from any analytics or model-training pipelines |
| Indian data protection | Aligned with the Digital Personal Data Protection Act, 2023 (purpose limitation, consent, erasure) and applicable RBI KYC master directions; data hosted in India |
| WhatsApp integration | WhatsApp Business API with approved message templates for invitations and confirmations |
