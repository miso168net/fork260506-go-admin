# Contract: `GET /api/v1/captcha`

> **This contract documents existing behaviour for learning purposes.**
> Source of truth: `app/other/router/gen_router.go:18` (route registration)
> + `app/admin/apis/captcha.go:18` (handler).

## Endpoint

- **Method**: `GET`
- **Path**: `/api/v1/captcha`
- **Auth required**: No (must be reachable before the user has a token)
- **Casbin enforcement**: No

## Request

No body, no required headers. May be called any number of times — each call
produces a fresh captcha state (UUID + image + answer) on the server side.

## Response — success (HTTP 200)

```json
{
  "code": 200,
  "data": "data:image/png;base64,iVBORw0KGgoAAAANSU...",
  "id":   "<UUID string>",
  "msg":  "success"
}
```

| Field | Type | Meaning |
|-------|------|---------|
| `code` | int | HTTP-style code; 200 on success |
| `data` | string | Base64-encoded PNG of the captcha image (data URI prefix included) |
| `id` | string | UUID identifying this captcha session — submit alongside the answer in `POST /api/v1/login` body's `uuid` field |
| `msg` | string | Always `"success"` on this branch |

## Response — failure (HTTP 500)

If `captcha.DriverDigitFunc()` errors (rare; usually a font-loading or
random-source failure during process startup):

```json
{
  "code": 500,
  "msg":  "验证码获取失败",
  ...
}
```

## Pairing with `POST /api/v1/login` (non-bypass path)

For a learner who wants to NOT use the dev-mode `"0/0"` bypass:

```bash
# Step 1: fetch a captcha
RESP=$(curl -s http://localhost:8000/api/v1/captcha)
UUID=$(echo "$RESP" | jq -r '.id')
B64=$(echo "$RESP" | jq -r '.data' | sed 's|^data:image/png;base64,||')

# Step 2: decode and view (or OCR) the digit string
echo "$B64" | base64 -d > /tmp/captcha.png
# Open /tmp/captcha.png, read the digits visually → assign to CODE

# Step 3: log in with the real captcha pair
curl -X POST http://localhost:8000/api/v1/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"admin\",\"password\":\"123456\",\"code\":\"$CODE\",\"uuid\":\"$UUID\"}"
```

Each captcha is single-use (`captcha.Verify(..., clear=true)` at
`common/middleware/handler/auth.go:88`). Reusing the same UUID + code pair
fails on the second login attempt.

## Server-side state

The captcha library stores the `(uuid → answer)` mapping in an in-process
cache (per `go-admin-core/sdk/pkg/captcha`). Restarting the container
invalidates all outstanding captchas.
