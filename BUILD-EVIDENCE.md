# BUILD-EVIDENCE.md — MM-Epic-1-Story-1A
Story: Customer Photo API for mCollect
Branch: feat/MM-Epic-1_CustomerPhotoAPI (customer-service)
Generated: 2026-06-25
Author: Aryan Maurya

---

## Phase 3 — TDD Red ✅

Commit: be09e48
Test file: test/customerPhoto.test.ts
Result: 15 tests failing as expected (3 TypeScript compile errors — implementation files absent)

Errors confirmed:
- TS2724: 'CustomerPhotoControllerInstance' not exported from '../src/controllers'
- TS2724: 'CustomerPhotoServiceInstance' not exported from '../src/controllers'
- TS2307: Cannot find module '../src/services/customerPhotoService'

---

## Phase 4 — TDD Green ✅

Commits: d5aa180, a3c2ddf, 6a0b437

Files created:
- src/services/customerPhotoService.ts
- src/controllers/customerPhotoController.ts
- src/validations/customerPhotoValidator.ts

Files modified:
- src/services/documentClientService.ts  (+getPresignedUrl method)
- src/controllers/index.ts               (+CustomerPhotoServiceInstance, CustomerPhotoControllerInstance)
- src/routes/index.ts                    (+GET /v1/customer/photo/:loanId)
- config/default.json                    (+customerPhotoApi.presignedUrlTtlSeconds: 3600)

---

## Phase 5 — Quality Gates ✅

### Tests: 15 / 15 passing
Run: NODE_ENV=preprod npm run test -- --testPathPattern="customerPhoto"

| Test | Result |
|------|--------|
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
| getCustomerPhotos loan not found → 404 shape | ✅ |
| getCustomerPhotos null dynamo response → 404 shape | ✅ |
| getPresignedUrl called with correct FILE_KEY and bucket inc-vkyc | ✅ |
| urlExpiresAt = Date.now() + (ttlSeconds * 1000) | ✅ |

### Lint: 0 errors ✅
Run: npx eslint src/controllers/customerPhotoController.ts src/services/customerPhotoService.ts src/validations/customerPhotoValidator.ts --ext .ts

Result: 0 errors, 1 warning (no-console — pre-existing pattern in codebase)

### Coverage (new files only) ✅
| File | Stmts | Branch | Funcs | Lines |
|------|-------|--------|-------|-------|
| customerPhotoController.ts | 59.3% | 54% | 84% | 83% |
| customerPhotoService.ts | 73.4% | 73% | 87% | 91.9% |
| customerPhotoValidator.ts | 100% | 100% | 100% | 100% |

No coverage threshold defined in PLAN.md. All critical paths (happy path, 404, 400, partial success, DOC_TYPE filter) are exercised.

### p95 Latency
Not measured — requires deployed service against live dynamo-svc and document-service.
Target: <3000ms for 2-customer loan (per PLAN.md Section 3).
Architecture supports it: 1 SQL query (~100ms) + parallel artifact fetches (~100ms) + parallel presigned URL calls (~200ms) = ~400ms expected.

### Regression ✅
Pre-existing test failures in test/index.test.ts: 21 (unchanged — pre-existing env config issue unrelated to this change).
No new failures introduced.

---

## document-service patch ✅

Repo: document-service
Branch: feat/MM-Epic-1_CustomerPhotoAPI
Commit: 6ce9d7a

Files modified:
- api/utils/schema.py       — expires_in optional Integer field added to GetSignedUrlSchema
- api/service/file_service.py — ExpiresIn uses request.json.get("expires_in") or max_expiry fallback

Existing callers of POST /v1/file/get_signed_url unaffected (expires_in is optional, load_default=None).

---

## Gate 2 — Peer Review Required

| Reviewer | Role | Status |
|----------|------|--------|
| Sunil Kumar | Tech Lead — implementation sign-off | ⏳ Pending |
