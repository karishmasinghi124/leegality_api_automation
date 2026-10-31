# Reservation Hub – API Automation (Leegality QA Assignment)

A maintainable API regression suite for the public Restful-booker sandbox, implemented with Python, `requests`, and `pytest`. The suite covers booking CRUD, authentication/authorization, response contracts, and negative/boundary inputs.

## Requirements and setup

- Python 3.10+
- Network access to `https://restful-booker.herokuapp.com`

```bash
python -m venv .venv
# macOS/Linux: source .venv/bin/activate
# Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

The sandbox credentials in `.env.example` are the public credentials supplied in the assignment. Override them through environment variables when needed. Do not commit `.env` or real credentials.

## Run the suite

```bash
pytest
```

The HTML report is written to `reports/api-test-report.html`; a JUnit XML report can also be generated with:

```bash
pytest --junitxml=reports/junit.xml
```

Run by feature with `pytest -m auth`, `pytest -m negative`, or `pytest -m smoke`.

## Strategy and design choices

- **Risk first:** prioritize the production incident class (negative/zero price, invalid dates, checkout before check-in), then core booking journeys and auth boundaries.
- **Isolation:** each CRUD test creates its own booking and uses the returned ID; no test depends on the ten seeded records or another test's state. Fixtures perform best-effort cleanup. The remote reset is treated as an external event, not as test setup.
- **Reusable layers:** `src/config.py` owns environment configuration; `src/client.py` owns HTTP behavior; `src/data.py` provides fresh payload factories; `tests/` contains readable feature-focused cases.
- **Assertions:** validate response codes, response bodies, key field types, and read-after-write behavior. Negative tests assert the expected client validation response rather than accepting a successful response as correct.
- **Cold start:** only the idempotent health GET uses bounded retry for transport failures and transient 502/503/504 responses. Writes are not retried automatically because a timeout after a POST could otherwise create duplicate bookings.

## Scope and deliberate exclusions

Included: `/ping`, booking list and filters, get by ID, create, PUT, PATCH, DELETE, token authentication, invalid credentials, and missing/invalid authorization. Negative data covers missing/empty fields, invalid types, negative/zero prices, malformed/impossible dates, and reversed date ranges.

Not included: load testing, exhaustive fuzzing, long-running soak tests, partner-specific authorization roles, and a full OpenAPI-generated schema validator. These would add noise or require requirements/contracts not supplied in the exercise. The current contract checks focus on the documented booking fields and their types.

## Limitations / interpretation

Restful-booker is a free shared service that resets and may be slow. Status codes and behavior can vary if the service is unavailable. The tests intentionally flag acceptance of invalid booking payloads as failures: these are product defects to investigate, not conditions to normalize in the tests. `BUGS.md` distinguishes candidate defects from confirmed observations; run the suite against the live sandbox and attach the resulting HTML report before final submission.

## Project layout

```text
src/                 configuration, HTTP client, payload factories
tests/               feature-based pytest cases
reports/             generated HTML/JUnit reports
BUGS.md              reproducible defect candidates and verification status
.github/workflows/   CI workflow
```
