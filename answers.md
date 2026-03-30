# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

| Field           | Type          | Action                | Justification                                                             |
| --------------- | ------------- | --------------------- | ------------------------------------------------------------------------- |
| full_name       | Direct PII    | Drop                  | Directly identifies an individual; not needed for analysis.               |
| email           | Direct PII    | Drop                  | Unique identifier; high risk if exposed and no analytical value.          |
| date_of_birth   | Indirect PII  | Mask (keep only year) | Exact DOB can re-identify individuals; year is sufficient for analysis.   |
| zip_code        | Indirect PII  | Mask (first 3 digits) | Full ZIP code can enable re-identification; partial ZIP preserves trends. |
| job_title       | Indirect PII  | Keep / Generalize     | Low risk alone; can be generalized if needed.                             |
| diagnosis_notes | Sensitive PII | Pseudonymize + Clean  | Contains health data and possible identifiers; must remove personal info. |

---

## Task 2 — Audit the API Script for Ethical Compliance

### Violation 1: Hardcoded API Key

**Problem:**
The API key is directly written in the code. This is insecure and may violate the API provider’s Terms of Service, as others can misuse the key.

**Fix:** Use environment variables to store the API key securely.

```python
import requests
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])
```

---

### Violation 2: Excessive Data Collection & Storing Raw PII

**Problem:**
The script collects a large amount of data (100 pages) without limitation and stores raw sensitive data permanently. This violates data minimization principles and may breach API usage policies.

**Fix:** Limit data collection and store only cleaned, non-sensitive data.

```python
records = []
MAX_PAGES = 10  # limit API usage

for page in range(1, MAX_PAGES + 1):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()

    for record in data["results"]:
        cleaned_record = {
            "birth_year": record["date_of_birth"][:4],
            "zip_prefix": record["zip_code"][:3],
            "job_title": record["job_title"],
            "diagnosis": record["diagnosis_notes"]
        }
        records.append(cleaned_record)

save_to_database(records)
```

---

Final Notes
Follow data minimization: collect only necessary data
Protect PII using masking, dropping, or pseudonymization
Never expose API keys in code
Respect API rate limits and Terms of Service

