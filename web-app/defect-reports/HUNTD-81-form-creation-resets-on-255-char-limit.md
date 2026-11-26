# HUNTD-81 — Experience Entry Creation form on Candidate Experience Page resets to empty state without error when input exceeds 255 character limit

**Severity:** Critical  
**Priority:** High

---

## Environment

| | |
|---|---|
| Browser | Microsoft Edge 148.0.3967.70 (64-bit), Google Chrome 148.0.7778.168 (64-bit) |
| OS | Windows 10 Pro |

---

## Preconditions

User is logged in as Candidate.

---

## Steps to Reproduce

1. Navigate to [Experience](https://huntd.tech/profile/candidate/experience)
2. Click `[+ Add]` to open the new experience form
3. Enter 256+ characters in the "Role" or "Company Name" field
4. Fill remaining required fields with valid data
5. Click `[Save]`

---

## Expected Result

Validation message displayed informing the user of the 255 character limit. Form retains all entered valid values and allows the user to correct the invalid input.

---

## Actual Result

- No error message displayed
- Form remains open but resets to a completely empty state
- All entered values lost, including valid ones
- Subsequent attempt to create an experience entry with valid data in the same form also fails silently due to aborted transaction state (`25P02`)

---

## Root Cause

The `INSERT` transaction is aborted by PostgreSQL when the `varchar(255)` constraint is violated. The server returns error code `25P02` — `current transaction is aborted, commands ignored until end of transaction block` — meaning all subsequent queries in the same transaction are also rejected until the block ends. The UI resets the form on failure without surfacing the error, and the aborted transaction state prevents any further saves in the same session without a page reload.

```json
{
  "errors": [{
    "message": "current transaction is aborted, commands ignored until end of transaction block",
    "locations": [{"line": 2, "column": 3}],
    "path": ["createWorkPlace"],
    "extensions": {
      "code": "INTERNAL_SERVER_ERROR",
      "exception": {
        "name": "SequelizeDatabaseError",
        "parent": {
          "code": "25P02",
          "routine": "exec_simple_query",
          "sql": "INSERT INTO \"candidate_profile_work_places\" (...,'1','TESTNAME...256chars',...)"
        }
      }
    }
  }],
  "data": null
}
```

---

## Additional Notes

- Related to [HUNTD-79](./HUNTD-79-role-company-name-fields-silently-discard-input-exceeding-255-chars.md) (edit flow — silent discard) and [HUNTD-80](./HUNTD-80-recruiter-company-details-mutation-discards-all-values-on-255-limit.md) (mutation discards all values) — same `varchar(255)` root cause, different failure modes across creation and edit flows
- The `25P02` aborted transaction state is a compounding failure: not only is the current save lost, but the user is silently blocked from saving any valid data until they reload the page
