# Lead Vendor Disposition API

Returns the policies sold from calls to QIL's numbers: which number was
dialed, the caller's phone number, the invoiced annual premium, and the sale
date.

## Endpoint

```
GET https://api.goeterna.com/api/v1/qil/dispositions
```

## Authentication

Send your API key in the `X-API-Key` header on every request. The key is
sent to you separately; keep it server-side and don't share it.

```
X-API-Key: <your key>
```

## Query parameters

All are optional.

| Param | Format | Default | Notes |
|---|---|---|---|
| `crm_sold_date_from` | `YYYY-MM-DD` | none | Sale date, inclusive |
| `crm_sold_date_to` | `YYYY-MM-DD` | none | Sale date, inclusive |
| `limit` | integer | `1000` | Rows per page, max `5000` |
| `cursor` | string | none | `next_cursor` from the previous page; omit for the first page |

## Response

```json
{
  "count": 2,
  "results": [
    {
      "to_number": "+19498687709",
      "source": "QIL - SOCIAL",
      "contact_phone": "+17145550142",
      "invoiced_annual_premium": 1026.00,
      "crm_sold_date": "2026-09-02"
    },
    {
      "to_number": "+19497732434",
      "source": "QIL - TV FEX",
      "contact_phone": "+19095550187",
      "invoiced_annual_premium": 684.36,
      "crm_sold_date": "2026-09-02"
    }
  ],
  "has_more": true,
  "next_cursor": "MjAyNi0wOS0wMnw0ODIxMw"
}
```

| Field | Description |
|---|---|
| `to_number` | The QIL number the caller dialed |
| `source` | That number's label: `QIL - SOCIAL` (+19498687709) or `QIL - TV FEX` (+19497732434) |
| `contact_phone` | The caller's phone number |
| `invoiced_annual_premium` | Annual premium invoiced for the policy, in USD |
| `crm_sold_date` | Date the policy was sold, `YYYY-MM-DD` |

Each row is one sold policy, so a caller who bought more than one policy
appears more than once. Rows are ordered by `crm_sold_date`, oldest first.

## Pagination

When `has_more` is `true`, request the next page with the same date filters
and `cursor` set to `next_cursor`. Repeat until `has_more` is `false`;
`next_cursor` is left out on the last page.

```python
import requests

URL = "https://api.goeterna.com/api/v1/qil/dispositions"
params = {"crm_sold_date_from": "2026-09-01", "crm_sold_date_to": "2026-09-30"}
headers = {"X-API-Key": "<your key>"}

rows = []
while True:
    page = requests.get(URL, params=params, headers=headers, timeout=30).json()
    rows.extend(page["results"])
    if not page["has_more"]:
        break
    params["cursor"] = page["next_cursor"]
```

A range with no sales returns `{"count": 0, "results": [], "has_more": false}`.

## Errors

| Status | Body | Cause |
|---|---|---|
| 401 | `{"error": "Unauthorized", "message": "X-API-Key header required"}` | No `X-API-Key` header |
| 401 | `{"error": "Unauthorized", "message": "Invalid API key"}` | Wrong key |
| 400 | `{"error": "Invalid crm_sold_date_from, expected YYYY-MM-DD"}` | Bad date (same for `crm_sold_date_to`) |
| 400 | `{"error": "crm_sold_date_from must not be after crm_sold_date_to"}` | Dates reversed |
| 400 | `{"error": "Invalid cursor"}` | `cursor` wasn't a `next_cursor` from this API |
| 500 | `{"error": "Failed to list dispositions"}` | Server error; retry later |

## Example

```bash
curl "https://api.goeterna.com/api/v1/qil/dispositions?crm_sold_date_from=2026-09-01&crm_sold_date_to=2026-09-30" \
  -H "X-API-Key: <your key>"
```
