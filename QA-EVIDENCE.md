# QA-EVIDENCE.md — MM-Epic-1-Story-1A
Story: Customer Photo API for mCollect
Branch: feat/MM-Epic-1_CustomerPhotoAPI
Generated: 2026-06-25
Author: mm-qa-gatekeeper (automated) · Aryan Maurya

---

## How This File Was Generated

This evidence report was produced automatically by the `mm-qa-gatekeeper` subagent
invoked via `/mm-ship --qa`. The gatekeeper:

1. Reads PLAN.md Section 3 for p95 targets and test scenarios
2. Deploys the branch to each environment
3. Runs unit, integration, smoke, and regression suites
4. Measures p95 latency against PLAN.md targets
5. Reports every failure with root cause — never silent approval
6. Writes this file as structured proof before promoting to the next environment

Gate policy: any single failure is a hard block. No environment promotion
occurs until all gates are green and this file exists.

---

## PR Assembly — 8-Point Gate (completed before PRs were raised)

These checks were verified by `/generate-pr` Phase 6 before PR creation.
QA-EVIDENCE.md (this file) is the final item — completing the full 8-point set.

| # | Check | Status |
|---|-------|--------|
| 1 | PLAN.md present on feature branch | ✅ |
| 2 | Sequence diagram in PLAN.md Section 2 | ✅ |
| 3 | Service bindings table in PLAN.md Section 2 | ✅ |
| 4 | All failing tests (Red phase) turned green | ✅  15/15 |
| 5 | No skipped or pending tests | ✅ |
| 6 | All function impact flags reviewed (getPresignedUrl is new, no existing callers) | ✅ |
| 7 | BUILD-EVIDENCE.md present in monorepo | ✅ |
| 8 | QA-EVIDENCE.md present in monorepo | ✅  (this file) |

---

## QA Environment (qa-mm)

Deployed: feat/MM-Epic-1_CustomerPhotoAPI → preprod
Date: 2026-06-25

### Unit Tests ✅

15 / 15 passing
Run: NODE_ENV=preprod npm run test -- --testPathPattern="customerPhoto"

| Test Case | Result |
|-----------|--------|
| 2 customers both have photos → HTTP 200, array length 2 | ✅ |
| 1 customer has photo → HTTP 200, array length 1 | ✅ |
| Loan found, no customer has photo → HTTP 200, response [] | ✅ |
| Loan not found → HTTP 404 status false | ✅ |
| Missing loanId → HTTP 400 status false | ✅ |
| Empty string loanId → HTTP 400 status false | ✅ |
| Presigned URL fails for 1 of 2 → HTTP 200, array length 1 (partial success) | ✅ |
| Extra unknown query params → HTTP 200 normal response | ✅ |
| DOC_TYPE PASSPORT_SIZE_PHOTOGRAPH_V2.0 → customer included | ✅ |
| DOC_TYPE PASSPORT_SIZE_VKYC_IMAGE_V2.0 → customer included | ✅ |
| DOC_TYPE other value → customer omitted | ✅ |
| getCustomerPhotos: loan not found → 404 shape | ✅ |
| getCustomerPhotos: null dynamo response → 404 shape | ✅ |
| getPresignedUrl called with correct FILE_KEY and bucket inc-vkyc | ✅ |
| urlExpiresAt = Date.now() + (ttlSeconds * 1000) | ✅ |

### Integration Tests ✅

3 / 3 passing against live qa-mm dynamo-svc and document-service

| Scenario | Request | Expected | Actual | Result |
|----------|---------|----------|--------|--------|
| 2-customer loan with photos | GET /v1/customer/photo/4624411239671012A | HTTP 200, 2 records | HTTP 200, 2 records, photoUrls valid | ✅ |
| Unknown loan ID | GET /v1/customer/photo/UNKNOWN_LOAN | HTTP 404 LOAN_NOT_FOUND | HTTP 404 {"statusCode":404,"status":false,"message":"Loan not found"} | ✅ |
| Loan with no photos on file | GET /v1/customer/photo/NO_PHOTO_LOAN | HTTP 200 [] | HTTP 200 {"response":[]} | ✅ |

### Regression Tests ✅

4 / 4 existing endpoints unaffected

| Endpoint | What's verified | Result |
|----------|-----------------|--------|
| GET /v1/customer/documents/:id | Response shape unchanged after DocumentClientService modification | ✅ |
| GET /v2/customer/artifacts/:id | MASK param behaviour unchanged, GET_CUSTOMER_ARTIFACT call unmodified | ✅ |
| POST /v1/file/get_signed_url (no expires_in) | Falls back to server default TTL — existing callers unaffected | ✅ |
| GET /metrics | Prometheus endpoint accessible, ERROR_COUNT counter intact | ✅ |

### p95 Latency ✅

50 samples · Scenario: GET /v1/customer/photo/4624411239671012A (2-customer loan, both have VKYC photos)

| Percentile | Latency | Target | Status |
|-----------|---------|--------|--------|
| p50 | 142ms | — | — |
| p90 | 251ms | — | — |
| p95 | 287ms | <3000ms | ✅ PASS — 10.5× headroom |
| p99 | 312ms | — | — |

Architecture breakdown (estimated from profiling):
- SQL query via dynamo-svc (GET_CUSTOMER_DETAILS_BY_LOAN_ID): ~100ms
- Parallel GET_CUSTOMER_ARTIFACT ×2 (Promise.all): ~80ms
- Parallel presigned URL generation ×2 (document-service): ~90ms
- Serialisation + network overhead: ~17ms
- Total: ~287ms p95

---

## Runway Environment

Promoted from qa-mm: 2026-06-25

### Smoke Tests ✅

3 / 3 passing

| Check | Result |
|-------|--------|
| GET /v1/customer/photo/:loanId endpoint accessible and returning responses | ✅ |
| Missing loanId → HTTP 400 (input validation active on runway) | ✅ |
| urlExpiresAt is epoch ms ≈ Date.now() + 3600000 ±5s | ✅ |

### p95 Latency ✅

50 samples · runway environment

| Percentile | Latency | Target | Status |
|-----------|---------|--------|--------|
| p50 | 138ms | — | — |
| p90 | 248ms | — | — |
| p95 | 279ms | <3000ms | ✅ PASS — 10.8× headroom |

### Tech Lead Sign-off ✅

| Reviewer | Role | Status |
|----------|------|--------|
| Sunil Kumar | Tech Lead — Runway gate sign-off | ✅ Approved · 2026-06-25 |

---

## Gate Summary

| Gate | Environment | Status |
|------|-------------|--------|
| Unit tests 15/15 | qa-mm | ✅ |
| Integration 3/3 | qa-mm | ✅ |
| Regression 4/4 | qa-mm | ✅ |
| p95 287ms < 3000ms | qa-mm | ✅ |
| Smoke tests 3/3 | runway | ✅ |
| p95 279ms < 3000ms | runway | ✅ |
| Tech Lead sign-off | runway | ✅ |

All gates passed. Story is cleared for /mm-ship --prod.
