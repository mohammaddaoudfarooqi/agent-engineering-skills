# Edge Case Catalog

Use this catalog during Phase 2 (specification) to systematically identify edge
cases for your feature. Check each category and add relevant cases to the
Edge Cases section of requirements.md.

## Input Validation

| Category | Test Inputs | Expected Handling |
|----------|------------|-------------------|
| Empty/null | "", null, undefined, NaN | Validation error or sensible default |
| Whitespace only | "   ", "\t\n" | Trim then validate; reject if empty |
| Max length | String of max+1 chars | Truncate or reject with error |
| Min length | String of min-1 chars | Reject with error |
| Special characters | `<script>`, `'; DROP TABLE`, `../../../` | Sanitize/escape, never execute |
| Unicode | Emoji, CJK, RTL text, Zalgo | Handle gracefully, no crashes |
| Numeric bounds | 0, -1, -0, MAX_INT+1, NaN, Infinity | Validate range, reject out-of-bounds |
| Format violations | Invalid email, URL, date, phone | Show specific format error |
| Type mismatch | String where number expected | Type error or coerce with warning |

## Boundary Values

| Category | Conditions to Test |
|----------|-------------------|
| At minimum | value == min (accept) |
| At maximum | value == max (accept) |
| Below minimum | value == min - 1 (reject) |
| Above maximum | value == max + 1 (reject) |
| Empty collection | 0 items (empty state, no off-by-one) |
| Single element | 1 item (no off-by-one) |
| Collection at capacity | max items (pagination / rejection) |
| Negative where positive expected | Reject or use absolute value |

## State and Timing

| Category | Scenario |
|----------|----------|
| Race condition | Two simultaneous writes to same resource |
| Stale data | Data changed since user's last read |
| Session timeout | Action after session expires |
| Double submit | Form/action triggered twice rapidly |
| Interrupted operation | Network drops mid-operation |
| Back button | User navigates back after submit |
| Concurrent users | Same resource edited by 2 users |
| Out-of-order events | Events arrive in unexpected sequence |

## Data Edge Cases

| Category | Scenario |
|----------|----------|
| Missing required fields | Required field absent from payload |
| Extra/unknown fields | Unexpected fields in input |
| Circular references | Self-referencing data structures |
| Large payload | Request body exceeds expected size |
| Non-UTF8 encoding | Binary or non-standard encoding |
| Date edge cases | Leap year (Feb 29), DST transitions, timezone boundaries |
| Floating point | 0.1 + 0.2 precision, currency calculations |
| Empty nested objects | { "items": [] }, { "user": null } |

## Permission and Auth

| Category | Scenario |
|----------|----------|
| Unauthenticated | No token/session (expect 401) |
| Expired token | Token past expiry (expect 401 + refresh) |
| Wrong role | Valid user, insufficient permission (expect 403) |
| Privilege escalation | User accessing another user's data (expect 403) |
| Deleted account | Token for removed user (expect 401) |
| Modified token | Tampered JWT or session (expect 401) |

## Environment and Infrastructure

| Category | Scenario |
|----------|----------|
| No network | Offline or DNS failure |
| Slow network | High latency (>5s) responses |
| Disk full | No storage space available |
| Database down | DB connection refused/timeout |
| Memory pressure | Near OOM conditions |
| Dependency unavailable | External API returns 503 |
| Clock skew | Server time different from client time |

## How to Use This Catalog

During specification (Phase 2):

1. Scan each category above
2. For each category relevant to your feature, add an EARS requirement:
   ```
   IF [edge condition from catalog] THEN THE SYSTEM SHALL [handling behavior]
   ```
3. Add the most critical edge cases to the Edge Cases section of requirements.md
4. During test planning (Phase 3), ensure each edge case has a test

Not every category applies to every feature. Focus on categories that match
your feature's domain (e.g., skip "Permission and Auth" for a pure utility function).
