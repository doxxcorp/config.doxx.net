# Authentication

## Overview

The doxx.net Config API uses token-based authentication. There are no usernames, passwords, or email addresses. All API calls authenticate using one of three token types, passed as a form parameter or encrypted header.

## Token Types

| Token | Format | Purpose | How to Obtain |
|-------|--------|---------|---------------|
| **Auth Token** | ~43 character base64 string | Identifies your account. Used for all authenticated API calls. | Created by a human at [a0x13.doxx.net](https://a0x13.doxx.net) via proof-of-work challenge |
| **Tunnel Token** | ~43 character base64 string | Identifies a specific VPN tunnel within your account. | Returned by `create_tunnel` or `list_tunnels` |
| **POW Token** | Variable-length string | One-time human verification token. Proves a real person created the account. | Returned by completing the DOXX POW challenge |

**You cannot create accounts via API.** A human must visit [a0x13.doxx.net](https://a0x13.doxx.net), complete the proof-of-work challenge, and accept the Terms of Service. The auth token from that process is then used for all subsequent API calls.

## Passing Your Token

The API accepts authentication credentials through four methods, checked in priority order:

### 1. X-Auth Encrypted Header (Recommended)

The `X-Auth` header carries an AES-256-GCM encrypted payload containing the token and a timestamp. This prevents token exposure in server logs and provides replay protection.

**Payload format before encryption:** `token|unix_timestamp`

**Encryption:** AES-256-GCM with a 12-byte random nonce. The encrypted output is `nonce(12 bytes) || ciphertext || tag(16 bytes)`, then base64url-encoded.

**Timestamp validation:** The timestamp must be within 300 seconds (5 minutes) of the server's clock. Requests outside this window are rejected with an expiration error.

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -H "X-Auth: <base64url-encoded-encrypted-payload>" \
  -d "list_tunnels=1"
```

### 2. x-auth Query Parameter

Same encryption format as the `X-Auth` header, passed as a query parameter. Useful for GET requests or contexts where setting headers is difficult.

```
https://config.doxx.net/v1/?x-auth=<base64url-encoded-encrypted-payload>&list_tunnels=1
```

### 3. doxx_token Cookie

A plaintext auth token stored in a cookie named `doxx_token`. Used by the web portal.

### 4. token Form Parameter (Legacy)

The token passed as a standard form parameter. This is the simplest method and is used in all curl examples throughout this documentation.

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_tunnels=1&token=YOUR_AUTH_TOKEN"
```

## Subscription Requirements

Some API features require an active subscription (Pro plan). When a feature requires a subscription and the account does not have one, the API returns HTTP 402 or 403 with details about which feature is required.

Free accounts can:
- Create tunnels (limited seats)
- Register domains
- Manage DNS records
- Use basic DNS blocking
- Configure firewall rules

Pro accounts additionally get:
- Dedicated public IPv4 addresses
- Additional tunnel seats
- Family sharing (guest seat grants)
- Advanced DNS features
- Tor onion routing
- Geo-spoofing proxy

## Error Responses

### Invalid or Missing Token

```json
{
  "status": "error",
  "message": "Invalid or expired auth token"
}
```
HTTP 401

### Expired X-Auth Timestamp

```json
{
  "status": "error",
  "message": "X-Auth header: X-Auth token expired (age: 450s)"
}
```
HTTP 401

### Feature Requires Subscription

```json
{
  "status": "error",
  "message": "Pro subscription required for dedicated public IPv4",
  "feature_required": "dedicated_ip"
}
```
HTTP 403

### Seat Limit Reached

When the account has used all available device seats:

```json
{
  "status": "error",
  "message": "Seat limit reached"
}
```
HTTP 409

## Regional Endpoints

The Config API is available at multiple regional endpoints. If the primary endpoint is unavailable, clients should fail over to a regional endpoint.

| Endpoint | Region |
|----------|--------|
| `https://config.doxx.net/v1/` | Primary (anycast) |
| `https://config-us-east.doxx.net/v1/` | US East |
| `https://config-us-west.doxx.net/v1/` | US West |
| `https://config-eu-central.doxx.net/v1/` | EU Central |

## Request Format

All Config API requests use `POST` with `application/x-www-form-urlencoded` content type. Endpoints are selected by setting `endpoint_name=1` as a form parameter.

```bash
TOKEN="your_auth_token"
API="https://config.doxx.net/v1/"

curl -s -X POST $API -d "servers=1"
curl -s -X POST $API -d "list_tunnels=1&token=$TOKEN"
```
