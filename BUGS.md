# Defect log – Reservation Hub / Restful-booker

> **Verification note:** The assignment sandbox could not be reached from the preparation environment (DNS/network access unavailable). Therefore the items below are high-priority defect candidates derived directly from the incident and required probes, **not claims of independently confirmed live observations**. Run the included negative tests against the sandbox and update each status with the actual response before submitting this file as a confirmed bug report.

## BUG-CAND-001 – Booking accepts a negative total price

- **Severity:** Critical – can create financially invalid reservations and incorrect partner/customer totals.
- **Status:** Pending live reproduction.
- **Steps to reproduce:**

```bash
curl -i -X POST 'https://restful-booker.herokuapp.com/booking' \
  -H 'Content-Type: application/json' \
  -d '{"firstname":"QA","lastname":"NegativePrice","totalprice":-100,"depositpaid":true,"bookingdates":{"checkin":"2027-05-10","checkout":"2027-05-12"},"additionalneeds":"Breakfast"}'
```

- **Expected:** HTTP 400 or 422; no booking is created; response explains that `totalprice` must be a non-negative/positive monetary value (confirm the product rule with the API owner).
- **Actual:** Not verified in this environment. Record the exact status/body and check `GET /booking/{bookingid}` if a booking ID is returned.

## BUG-CAND-002 – Booking accepts checkout earlier than check-in

- **Severity:** Critical – creates an impossible stay and can corrupt availability, pricing, and downstream partner workflows.
- **Status:** Pending live reproduction.
- **Steps to reproduce:**

```bash
curl -i -X POST 'https://restful-booker.herokuapp.com/booking' \
  -H 'Content-Type: application/json' \
  -d '{"firstname":"QA","lastname":"ReversedDates","totalprice":120,"depositpaid":false,"bookingdates":{"checkin":"2027-05-20","checkout":"2027-05-10"},"additionalneeds":""}'
```

- **Expected:** HTTP 400 or 422; booking is not persisted; validation message identifies the invalid date range.
- **Actual:** Not verified in this environment. Record exact status/body and verify persistence if an ID is returned.

## BUG-CAND-003 – Booking accepts malformed or impossible calendar dates

- **Severity:** High – invalid dates may break availability calculations, partner synchronization, or downstream date parsing.
- **Status:** Pending live reproduction.
- **Steps to reproduce:**

```bash
curl -i -X POST 'https://restful-booker.herokuapp.com/booking' \
  -H 'Content-Type: application/json' \
  -d '{"firstname":"QA","lastname":"BadDate","totalprice":120,"depositpaid":true,"bookingdates":{"checkin":"2027-02-30","checkout":"2027-03-02"},"additionalneeds":""}'
```

- **Expected:** HTTP 400 or 422; no booking is created because 30 February is not a valid calendar date.
- **Actual:** Not verified in this environment. Record exact status/body and verify persistence if an ID is returned.

## Verification procedure

Run `pytest -m negative -vv`. For each failing validation test, capture the request payload, HTTP status, response body, timestamp, and whether a booking was persisted. A failing test is evidence of a mismatch against the expected contract; do not mark a defect confirmed until the live response and persistence check are recorded. Clean up any created booking using an authenticated DELETE where possible.
