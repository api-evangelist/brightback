---
name: Create a Brightback pre-cancel session
description: >-
  Initiate a Brightback (Chargebee Retention) cancel/save session for a churning
  subscriber and obtain the Cancel Session URL to redirect them into the hosted
  retention experience.
api: openapi/brightback-retention-openapi.yml
operations:
  - createPrecancelSession
---

# Create a Brightback pre-cancel (save) session

Use this skill when a subscriber clicks "cancel" and you want to route them into
Brightback / Chargebee Retention's hosted save experience.

## Prerequisites
- Your Brightback `app_id`.
- For server-to-server calls: your Brightback-issued **private** and **public**
  keys for request signing.

## Steps

1. **Build the request body** for `createPrecancelSession`
   (`POST https://app.brightback.com/precancel`, or
   `https://app.retention.chargebee.com/precancel`). Required:
   - `app_id`
   - `account.internal_id`
   - one of `email` or `subscription_id`
   Recommended: `save_return_url`, `cancel_confirmation_url`, and account
   context (`plan`, `plan_term`, `company_name`, `value`, `created_at`).

2. **Sign the request (server-side only).** Add an ISO8601 timestamp, compute an
   HMAC-SHA-512 signature over the payload using your private key, then send:
   - `Signature: <hmac-sha-512 signature>`
   - `Signature-Public-Key: <your public key>`

3. **POST** the JSON body to `/precancel`.

4. **Handle the response.** On `200` you get `{ "valid": true, "url": "https://app.brightback.com/<company>/cancel/<sessionId>" }`
   (the session URL may appear as `url` or `message`). Redirect the end user to
   that URL. On `400`/`401` read `message` and fix the request — see
   `errors/brightback-problem-types.yml`.

## Rules
- Auth model and headers: `conventions/brightback-conventions.yml`.
- No idempotency key is supported; do not retry blindly on ambiguous failures.
- `custom` accepts a flat object of business attributes (no arrays).
