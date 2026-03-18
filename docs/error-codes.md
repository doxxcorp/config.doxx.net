# Error Codes

## Response Format

All API responses are JSON with a `status` field. Successful responses use `"status": "success"`. Error responses use `"status": "error"` with a human-readable `message`.

### Standard Error Response

```json
{
  "status": "error",
  "message": "Description of what went wrong"
}
```

### Error Response with Context

Most endpoints include a `context` field that provides structured information for programmatic consumers. The context string contains the endpoint name, description, accepted parameters, and a suggested fix.

```json
{
  "status": "error",
  "message": "tunnel_token and profile_name required",
  "context": "save_profile: Snapshots tunnel settings into a saved profile | Params: tunnel_token, profile_name, profile_icon, profile_notes, profile_type | Error: tunnel_token and profile_name required | Fix: Provide the required parameters"
}
```

## HTTP Status Codes

| Code | Meaning | When It Happens |
|------|---------|-----------------|
| **200** | Success | Request succeeded. Always check the `status` field in the JSON body, as some 200 responses contain `"status": "error"`. |
| **400** | Bad Request | Missing or invalid parameters. The `message` field describes which parameter is wrong. |
| **401** | Unauthorized | Invalid, expired, or missing authentication token. Verify your auth token is correct. |
| **402** | Payment Required | Feature requires a paid subscription. The profile or feature needs a Pro plan. |
| **403** | Forbidden | Access denied. You do not own the requested resource, or the action requires proof-of-work verification. |
| **404** | Not Found | The requested resource (tunnel, domain, profile, device, address) does not exist or does not belong to your account. |
| **409** | Conflict | Resource conflict. Examples: domain already registered, profile already in use on another device, seat limit reached, profile limit reached, IP slot limit reached, site mismatch. |
| **422** | Unprocessable Entity | The request was well-formed but the data cannot be processed. Example: invalid DNS record content for the specified record type. |
| **423** | Locked | The resource is locked and cannot be modified. Occurs when a profile has `ip_locked=1` and you attempt an address operation. |
| **500** | Internal Server Error | An unexpected error occurred. Retry the request. If it persists, try a regional failover endpoint or contact support. |
| **503** | Service Unavailable | The service is in a degraded state. Try a different regional endpoint. |

## Extended Error Fields

Some error responses include additional structured fields beyond `status` and `message`:

### `feature_required`

Returned when an endpoint requires a subscription feature the account does not have.

```json
{
  "status": "error",
  "message": "Pro subscription required for dedicated public IPv4",
  "feature_required": "dedicated_ip"
}
```

### `seat_limit`

Returned when the account has reached its maximum number of device seats.

```json
{
  "status": "error",
  "message": "Seat limit reached",
  "device_count": 6,
  "max_devices": 6
}
```

### `site_mismatch`

Returned by address assignment when the address's geographic site does not match the profile's preferred server location. The response includes both locations so the client can display the conflict.

```json
{
  "status": "error",
  "error": "site_mismatch",
  "message": "This address is at Miami, FL but \"Privacy Mode\" uses Zurich, CH. Addresses can only be assigned to profiles at their home location.",
  "ip_location": "Miami, FL",
  "ip_site_id": 1,
  "profile_location": "Zurich, CH",
  "profile_site_id": 5,
  "profile_name": "Privacy Mode"
}
```

Pass `update_location=1` to automatically resolve the mismatch by updating the profile's location to match the address.

### Profile Locked

Returned when attempting to modify addresses on a profile that has `ip_locked=1`.

```json
{
  "status": "error",
  "message": "IPs are locked on this profile"
}
```
HTTP 423

### Profile In Use

Returned when attempting an operation (load, lock, re-snapshot, lease) on a profile that is currently loaded on another connected device.

```json
{
  "status": "error",
  "message": "This profile is already in use on \"iPhone 15 Pro\". Disconnect or switch profiles on that device first.",
  "in_use_by": "iPhone 15 Pro",
  "profile_id": 3
}
```
HTTP 409

## Important Notes

- **HTTP 200 can contain errors.** Always check the `status` field in the response body. Some legacy endpoints return HTTP 200 with `"status": "error"` in the JSON.
- **The `context` field is informational.** It is designed for AI agents and programmatic consumers to understand endpoint behavior without external documentation.
- **Regional failover.** When you receive 500 or 503 errors, retry against a different regional endpoint (`config-us-east`, `config-us-west`, `config-eu-central`).
