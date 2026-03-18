# Domains

Domain management endpoints. doxx.net supports 196 top-level domains including `.doxx`, `.crypto`, `.eth`, `.onion`, `.vpn`, and more. You can also import external domains (`.com`, `.net`, `.org`) via TXT record verification.

---

## `list_domains`

Returns all domains registered to the authenticated account.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_domains=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "domains": [
    {
      "name": "mysite.doxx",
      "id": 1234
    },
    {
      "name": "cool.crypto",
      "id": 1235
    }
  ]
}
```

---

## `create_domain`

Registers a new domain under one of the 196 available TLDs. If no TLD is specified, `.doxx` is used as the default.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name (e.g. `mysite.doxx`, `cool.crypto`, or just `mysite` which defaults to `.doxx`) |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_domain=1&token=$TOKEN&domain=mysite.doxx" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Domain registered successfully"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `domain required` | Missing domain parameter |
| 400 | `Invalid TLD` | TLD is not in the supported list |
| 409 | `Domain already registered` | Another account owns this domain |

---

## `delete_domain`

Deletes a domain and all its associated DNS records.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name to delete |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "delete_domain=1&token=$TOKEN&domain=mysite.doxx" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Domain deleted successfully"
}
```

---

## `import_domain`

Imports an external domain (`.com`, `.net`, `.org`, etc.) after TXT record verification. You must first add a `_doxx-verify` TXT record at your current DNS provider with the verification code from `get_domain_validation`.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | External domain to import (e.g. `example.com`) |

### Prerequisites

1. Call `get_domain_validation` to get your verification code
2. At your current DNS provider, create a TXT record: `_doxx-verify.example.com` with the verification code as the value
3. Wait for DNS propagation (may take up to a few hours)
4. Call `import_domain`
5. After successful import, update your registrar's nameservers to the doxx.net authoritative servers

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "import_domain=1&token=$TOKEN&domain=example.com" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Domain imported successfully",
  "nameservers": [
    "a.root-dx.net",
    "a.root-dx.com",
    "a.root-dx.org"
  ],
  "note": "Update your domain registrar to use these nameservers"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `domain required` | Missing domain parameter |
| 403 | `TXT verification failed` | The `_doxx-verify` TXT record was not found or does not match |
| 409 | `Domain already registered` | Another account already imported this domain |

---

## `get_domain_validation`

Returns the TXT verification code needed to import an external domain. Set this as a TXT record at `_doxx-verify.yourdomain.com` before calling `import_domain`.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain you want to import |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "get_domain_validation=1&token=$TOKEN&domain=example.com" | jq .
```

### Response

```json
{
  "status": "success",
  "validation_code": "a1b2c3d4e5f6..."
}
```

---

## `link_profile_domain`

Links a connection profile to a domain by creating DNS A/AAAA records that point the hostname to the profile's assigned IP addresses. When the profile is loaded onto a tunnel, the DNS records automatically resolve to that tunnel's addresses.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Base domain you own (e.g. `mysite.doxx`) |
| `hostname` | Yes | Subdomain label (e.g. `server1`). Must be alphanumeric with hyphens. Cannot start or end with a hyphen. |
| `profile_id` | Yes | Profile ID to link |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "link_profile_domain=1&token=$TOKEN&domain=mysite.doxx&hostname=server1&profile_id=3" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "DNS linked: server1.mysite.doxx",
  "fqdn": "server1.mysite.doxx"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `Missing required parameters: domain, hostname, profile_id` | One or more parameters missing |
| 400 | `Hostname must be alphanumeric with hyphens...` | Invalid hostname format |
| 403 | `Domain not found or you do not have permission` | Domain does not belong to this account |
| 404 | `Profile not found` | Invalid profile_id |

---

## `unlink_profile_domain`

Removes the DNS records associated with a profile and clears its domain name link.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | Yes | Profile ID to unlink |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "unlink_profile_domain=1&token=$TOKEN&profile_id=3" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "DNS unlinked"
}
```
