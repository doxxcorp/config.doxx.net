# DNS Records

DNS record management for domains you own. Supports standard record types with per-type validation rules.

## Supported Record Types

| Type | Content Format | Notes |
|------|---------------|-------|
| `A` | IPv4 address (e.g. `1.2.3.4`) | Must be a valid IPv4 address |
| `AAAA` | IPv6 address (e.g. `2001:db8::1`) | Must be a valid IPv6 address |
| `CNAME` | Hostname (e.g. `other.example.com.`) | Must end with a dot. Cannot coexist with other record types at the same name. |
| `MX` | Mail server hostname (e.g. `mail.example.com.`) | Requires `prio` parameter for priority |
| `TXT` | Arbitrary text (e.g. `v=spf1 include:...`) | Maximum 255 characters per string. Multiple strings allowed. |
| `NS` | Nameserver hostname (e.g. `ns1.example.com.`) | Cannot modify the zone apex NS records |
| `SRV` | Uses separate fields: `srv_priority`, `srv_weight`, `srv_port`, `srv_target` | See SRV parameters below |
| `PTR` | Hostname (e.g. `host.example.com.`) | Reverse DNS pointer |

---

## `list_dns`

Returns all DNS records for a domain.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_dns=1&token=$TOKEN&domain=mysite.doxx" | jq .
```

### Response

```json
{
  "status": "success",
  "domain": "mysite.doxx",
  "records": [
    {
      "name": "mysite.doxx",
      "type": "A",
      "content": "1.2.3.4",
      "ttl": 300,
      "prio": 0
    },
    {
      "name": "mysite.doxx",
      "type": "SOA",
      "content": "ns.doxx. hostmaster.doxx. 2026020801 10800 3600 604800 3600",
      "ttl": 3600,
      "prio": 0
    },
    {
      "name": "mysite.doxx",
      "type": "NS",
      "content": "ns.doxx.",
      "ttl": 3600,
      "prio": 0
    }
  ]
}
```

---

## `create_dns_record`

Creates a new DNS record.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name |
| `name` | Yes | Record name (FQDN like `sub.mysite.doxx`, or `@` for the zone apex) |
| `type` | Yes | Record type (`A`, `AAAA`, `CNAME`, `MX`, `TXT`, `NS`, `SRV`, `PTR`) |
| `content` | Yes | Record value (see type-specific format above) |
| `ttl` | No | Time to live in seconds. Default: `3600`. Minimum: `60`. |
| `prio` | No | Priority value for MX records |

### SRV Record Parameters

For SRV records, use these parameters instead of `content`:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `srv_priority` | Yes | Service priority (lower = preferred) |
| `srv_weight` | Yes | Relative weight for same-priority records |
| `srv_port` | Yes | TCP/UDP port of the service |
| `srv_target` | Yes | Hostname providing the service |

### Examples

**A record:**

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_dns_record=1&token=$TOKEN&domain=mysite.doxx&name=mysite.doxx&type=A&content=1.2.3.4&ttl=300" | jq .
```

**Wildcard A record:**

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_dns_record=1&token=$TOKEN&domain=mysite.doxx&name=*.mysite.doxx&type=A&content=1.2.3.4&ttl=300" | jq .
```

**MX record:**

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_dns_record=1&token=$TOKEN&domain=mysite.doxx&name=mysite.doxx&type=MX&content=mail.mysite.doxx.&prio=10&ttl=3600" | jq .
```

**SRV record:**

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_dns_record=1&token=$TOKEN&domain=mysite.doxx&name=_minecraft._tcp.mysite.doxx&type=SRV&srv_priority=0&srv_weight=5&srv_port=25565&srv_target=play.mysite.doxx.&ttl=3600" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "DNS record created successfully"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `Invalid record type` | Unsupported record type |
| 400 | `Invalid IPv4 address` | A record content is not a valid IPv4 |
| 400 | `Invalid IPv6 address` | AAAA record content is not a valid IPv6 |
| 403 | `Domain not found or access denied` | Domain does not belong to this account |
| 409 | `Record already exists` | A record with the same name, type, and content already exists |

---

## `update_dns_record`

Updates an existing DNS record. You identify the existing record by its old name, type, and content, then provide the new values.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name |
| `old_name` | Yes | Current record name |
| `old_type` | Yes | Current record type |
| `old_content` | Yes | Current record content |
| `name` | Yes | New record name |
| `content` | Yes | New record content |
| `ttl` | Yes | New TTL value |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_dns_record=1&token=$TOKEN&domain=mysite.doxx&old_name=mysite.doxx&old_type=A&old_content=1.2.3.4&name=mysite.doxx&content=5.6.7.8&ttl=300" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "DNS record updated successfully"
}
```

---

## `delete_dns_record`

Deletes a specific DNS record identified by name, type, and content.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain name |
| `name` | Yes | Record name |
| `type` | Yes | Record type |
| `content` | Yes | Record content (must match exactly) |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "delete_dns_record=1&token=$TOKEN&domain=mysite.doxx&name=mysite.doxx&type=A&content=1.2.3.4" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "DNS record deleted successfully"
}
```

---

## `sign_certificate`

Signs a Certificate Signing Request (CSR) with the doxx.net root CA. The certificate is automatically upgraded to include both the base domain and a wildcard (`*.domain`). Returns raw PEM data, not JSON.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `domain` | Yes | Domain you own |
| `csr` | Yes | PEM-encoded CSR (use `--data-urlencode` for curl) |

### Example

```bash
openssl ecparam -genkey -name prime256v1 -out mysite.key
openssl req -new -key mysite.key -out mysite.csr -subj "/CN=mysite.doxx"

curl -s -X POST https://config.doxx.net/v1/ \
  -d "sign_certificate=1&token=$TOKEN&domain=mysite.doxx" \
  --data-urlencode "csr=$(cat mysite.csr)" \
  -o mysite.crt
```

### Response

Raw PEM certificate. The certificate includes `SAN: DNS:*.mysite.doxx, DNS:mysite.doxx`.

### Certificate Details

| Property | Value |
|----------|-------|
| Signing CA | doxx.net root CA |
| Validity | 365 days |
| SAN | Wildcard + base domain (automatic) |
| Key Types | RSA, ECDSA (prime256v1 recommended) |

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `domain and csr required` | Missing parameters |
| 400 | `Invalid CSR` | CSR could not be parsed |
| 403 | `Domain not found or access denied` | Domain does not belong to this account |

### Verifying the Certificate

```bash
openssl x509 -in mysite.crt -noout -subject -ext subjectAltName
```

### Installing the Root CA

Clients connecting to services using doxx.net-signed certificates need the doxx.net root CA:

```bash
curl -o doxx-root-ca.crt https://a0x13.doxx.net/assets/doxx-root-ca.crt
```
