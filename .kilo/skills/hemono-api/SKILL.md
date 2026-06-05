---
name: hemono-api
description: Hemono (荷物账本) REST API skill for managing shared expense-splitting ledgers, transactions, members, and invitations via /api/v1/ endpoints. Use this skill when the user wants to interact with Hemono programmatically — create ledgers, record expenses, manage members, view statistics, or handle invitation codes.
---

# Hemono API Skill

Hemono (荷物账本) is a shared expense-splitting ledger application. This skill documents the `/api/v1/` REST API, which uses **API token authentication**.

## Base URL

```
http://localhost:8090
```

## Authentication

All `/api/v1/*` endpoints require a Bearer token:

```
Authorization: Bearer hmn_<40-hex-chars>
```

Tokens are 44 characters: `hmn_` + 40 hex chars. Create via `POST /api/tokens` (session auth) or web UI.

## Amount Convention

All amounts are **integer cents (分)**. `12850` = ¥128.50. Divide by 100 for display.

## Quick Reference

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/ledgers` | List user's ledgers |
| `POST` | `/api/v1/ledgers` | Create a new ledger |
| `GET` | `/api/v1/ledgers/{id}` | Get ledger detail |
| `DELETE` | `/api/v1/ledgers/{id}` | Delete ledger (owner only) |
| `GET` | `/api/v1/ledgers/{id}/members` | List ledger members |
| `GET` | `/api/v1/ledgers/{id}/transactions` | List transactions |
| `POST` | `/api/v1/ledgers/{id}/transactions` | Create transaction |
| `DELETE` | `/api/v1/transactions/{id}` | Delete transaction (payer only) |
| `GET` | `/api/v1/ledgers/{id}/stats` | Get ledger statistics |
| `GET` | `/api/v1/ledgers/{id}/invitation` | Get active invitation |
| `POST` | `/api/v1/ledgers/{id}/invitation` | Generate invitation code |
| `DELETE` | `/api/v1/invitations/{id}` | Delete invitation (owner only) |
| `POST` | `/api/v1/invitations/join` | Join ledger via invitation code |

## Key Endpoints

### POST /api/v1/ledgers/{id}/transactions

Create a transaction. The authenticated user becomes the `payer`.

```json
{
  "amount": 12850,
  "type": "AA",
  "direction": "EXPENSE",
  "beneficiary": "",
  "note": "晚餐",
  "date": "2026-05-31"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | int | Yes | Cents, must be > 0 |
| `type` | string | Yes | `AA` (split equally) or `SINGLE` (one beneficiary) |
| `direction` | string | Yes | `EXPENSE` or `INCOME` |
| `beneficiary` | string | Conditional | Required when `type` = `SINGLE`. User ID |
| `note` | string | No | Description |
| `date` | string | No | `YYYY-MM-DD`, defaults to today |

### GET /api/v1/ledgers/{id}/stats

Returns statistical summary. Optional `?month=YYYY-MM` query param.

```json
{
  "totalExpense": 50000,
  "totalBenefit": 25000,
  "totalIncome": 10000,
  "totalIncomeShare": 5000,
  "monthlyExpense": 20000,
  "monthlyIncome": 5000,
  "last7DaysExpense": 8000,
  "last7DaysIncome": 2000,
  "memberStats": [
    {
      "userId": "user_id",
      "name": "张三",
      "email": "zhangsan@example.com",
      "avatar": "https://...",
      "totalExpense": 30000,
      "totalBenefit": 15000,
      "totalIncome": 5000,
      "incomeShare": 2500,
      "balance": 7500,
      "percentage": 100
    }
  ]
}
```

Balance = `totalExpense - totalIncome - totalBenefit + incomeShare`. Positive = owed money, negative = owes money.

### POST /api/v1/invitations/join

Join a ledger via invitation code.

```json
{ "code": "ABC-123456" }
```

Returns `{ "success": true, "ledgerId": "...", "ledgerName": "..." }`.

## Error Format

```json
{ "code": <http_status>, "message": "<chinese_error_description>" }
```

| Code | Meaning |
|------|---------|
| 400 | Bad request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not found |
| 409 | Conflict |
| 410 | Gone (expired) |
| 500 | Internal error |

## Common Workflows

### Record an AA expense

```bash
curl -X POST http://localhost:8090/api/v1/ledgers/{id}/transactions \
  -H "Authorization: Bearer hmn_xxxx" \
  -H "Content-Type: application/json" \
  -d '{"amount": 12850, "type": "AA", "direction": "EXPENSE", "note": "晚餐"}'
```

### Invite someone

```bash
# Generate code (owner only)
curl -X POST http://localhost:8090/api/v1/ledgers/{id}/invitation \
  -H "Authorization: Bearer hmn_xxxx" \
  -H "Content-Type: application/json" \
  -d '{"max_uses": 5}'

# Other person joins
curl -X POST http://localhost:8090/api/v1/invitations/join \
  -H "Authorization: Bearer hmn_yyyy" \
  -H "Content-Type: application/json" \
  -d '{"code": "ABC-123456"}'
```

### View stats

```bash
curl -H "Authorization: Bearer hmn_xxxx" \
  "http://localhost:8090/api/v1/ledgers/{id}/stats?month=2026-05"
```

## Detailed Documentation

- [API Endpoints](../../../hemono-api-skill/api-endpoints.md) — Full request/response specs
- [Data Models](../../../hemono-api-skill/data-models.md) — Collection schemas
- [Examples](../../../hemono-api-skill/examples.md) — curl, Python, JavaScript examples
