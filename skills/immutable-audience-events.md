---
name: immutable-audience-events
description: Send player events to Immutable Audience CDP from a backend, manage per-identity tracking consent, and handle erasure requests.
api: immutable-audience-api
operations:
  - IngestMessages
  - GetTrackingConsent
  - UpdateTrackingConsent
  - DeleteAudienceData
generated: '2026-08-23'
method: generated
source: openapi/immutable-audience-openapi.json, https://docs.immutable.com/docs/products/audience/analytics/rest-api
---

# Ingest player events into Audience CDP

Server-side event tracking. No browser or client library required. Base host `https://api.immutable.com`; auth is `x-immutable-api-key`.

## Steps

1. **Check consent first** — `GetTrackingConsent`, `GET /v1/audience/tracking-consent`.
   Consent is enforceable state on the API, not just a UI toggle. Read it before you emit events for an identity.

2. **Record consent changes** — `UpdateTrackingConsent`, `PUT /v1/audience/tracking-consent`.
   Last-write-wins, so this is freely reversible: call it again with the previous value.

3. **Send events** — `IngestMessages`, `POST /v1/audience/messages`.
   Segment-style envelope. `type` is one of `identify`, `track`, `page`, `screen`. Each message needs a `messageId` (a real UUID), an RFC 3339 `eventTimestamp`, and `userId` or `anonymousId`. Identify a player before you emit their events, or attribution has nothing to attach to.

4. **Honour erasure** — `DeleteAudienceData`, `DELETE /v1/audience/data`.
   Supply either `anonymousId` or `userId`, **not both**.

## Rules

- **A 200 does not mean every message was accepted.** Read `accepted`, `rejected` and `rejections[]` on every response. A 400 can return the same body shape when the entire batch fails validation — so branch on the body, not the status code.
- Rejection codes are an **open set**: `MISSING_REQUIRED_FIELD`, `INVALID_FORMAT`, `INVALID_VALUE`, `INVALID_ENUM`, `UNSUPPORTED_TYPE`, `DUPLICATE_MESSAGE_ID` today, more later. Handle the ones you know and fall back to `message`.
- `DELETE /v1/audience/data` requires a **secret** key. A publishable key returns 401.
- **Erasure is irreversible.** It resolves every linked identity through stored alias mappings and queues an async erasure. There is no cancel, no restore and no published grace period. An autonomous agent must not call it without explicit human confirmation — this is the highest-consequence operation Immutable publishes.
- On 500, retry with exponential back-off. Deduplicate on `messageId`; duplicates within a batch are rejected outright.

## Events arriving back

Audience webhooks (`audience_joined`, `audience_account_linked`, `audience_account_unlinked`) are delivered as **signed AWS SNS notifications with `Content-Type: text/plain`** — configure a raw text body parser, verify with `sns-validator`, and check the `TopicArn`. There is no shared secret and no signature header. Failed deliveries retry for at most 60 minutes and are then lost; there is no manual redelivery.

## References

- `asyncapi/immutable-webhooks.yml`
- `errors/immutable-problem-types.yml`
- https://docs.immutable.com/docs/products/audience/analytics/data-dictionary
