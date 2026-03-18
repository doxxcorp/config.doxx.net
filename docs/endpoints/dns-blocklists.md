# DNS Blocklists

DNS blocking endpoints manage per-tunnel blocklist subscriptions, custom whitelist/blacklist entries, and Secure DNS (DoH/DoT) sharing.

---

## `dns_blocklist_stats`

Returns blocklist statistics for the authenticated account, including which lists are available and how many domains each contains.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_blocklist_stats=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "total_domains": 500000,
  "count": 12,
  "lists": [
    {
      "name": "ads",
      "display_name": "Advertising",
      "domain_count": 150000,
      "category": "privacy",
      "is_base_safety": false,
      "default_enabled": true,
      "enabled": true
    }
  ]
}
```

---

## `dns_get_tunnel_config`

Returns the complete DNS blocking configuration for a specific tunnel, including enabled blocklists, custom whitelist entries, and custom blacklist entries.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_get_tunnel_config=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "tunnel_token": "abc...",
  "dns_blocking_enabled": true,
  "base_protections": ["malware", "phishing"],
  "subscriptions": [
    {"blocklist_name": "ads", "enabled": 1}
  ],
  "whitelists": [
    {"domain": "example.com", "reason": null}
  ],
  "blacklists": [
    {"domain": "evil.com", "reason": "manual block"}
  ]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `dns_blocking_enabled` | bool | Whether DNS blocking is active on this tunnel |
| `base_protections` | array | Mandatory protection lists (always active, cannot be disabled) |
| `subscriptions` | array | User-toggleable blocklist subscriptions and their enabled state |
| `whitelists` | array | Custom domains that bypass all blocking |
| `blacklists` | array | Custom domains that are always blocked |

---

## `dns_set_subscription`

Enables or disables a blocklist subscription on a tunnel.

### Single Tunnel Mode

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `subscription` | Yes | Blocklist name (from `dns_get_options`) |
| `enabled` | Yes | `1` to enable, `0` to disable |

### Batch Mode (Apply to All Tunnels)

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Any tunnel token (used as reference) |
| `subscription` | Yes | Blocklist name |
| `enabled` | Yes | `1` to enable, `0` to disable |
| `apply_to_all` | Yes | Set to `1` to apply the change to all tunnels on the account |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_set_subscription=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&subscription=ads&enabled=1" | jq .
```

### Batch Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_set_subscription=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&subscription=ads&enabled=1&apply_to_all=1" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Subscription updated",
  "blocklist": "ads",
  "enabled": true,
  "tunnels_updated": 1
}
```

When `apply_to_all=1`, `tunnels_updated` reflects the number of tunnels that were updated.

---

## `dns_add_whitelist`

Adds a domain to the tunnel's custom whitelist. Whitelisted domains bypass all blocklist filtering.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `domain` | Yes | Domain to whitelist (e.g. `example.com`) |
| `apply_to_all` | No | `1` to add to all tunnels |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_add_whitelist=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&domain=example.com" | jq .
```

---

## `dns_remove_whitelist`

Removes a domain from the tunnel's custom whitelist.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `domain` | Yes | Domain to remove |
| `apply_to_all` | No | `1` to remove from all tunnels |

---

## `dns_add_blacklist`

Adds a domain to the tunnel's custom blacklist. Blacklisted domains are always blocked regardless of blocklist subscriptions.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `domain` | Yes | Domain to blacklist |
| `apply_to_all` | No | `1` to add to all tunnels |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "dns_add_blacklist=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&domain=evil.com" | jq .
```

---

## `dns_remove_blacklist`

Removes a domain from the tunnel's custom blacklist.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `domain` | Yes | Domain to remove |
| `apply_to_all` | No | `1` to remove from all tunnels |

---

## Secure DNS (DoH/DoT) Sharing

These endpoints create personalized DNS-over-HTTPS and DNS-over-TLS endpoints that mirror a tunnel's DNS blocking configuration. This lets you use your blocking settings on devices that are not connected to the VPN.

### `public_dns_list_hashes`

Lists all Secure DNS hashes for the account.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

```json
{
  "status": "success",
  "count": 1,
  "hashes": [
    {
      "host_hash": "gl6nqcbyhsau",
      "tunnel_token": "abc...",
      "label": "",
      "created_at": "2025-12-01 10:00:00",
      "tunnel_name": "My Laptop",
      "tunnel_server": "wireguard.mia.us.doxx.net",
      "doh_url": "https://gl6nqcbyhsau.sdns.doxx.net/dns-query",
      "dot_host": "gl6nqcbyhsau.sdns.doxx.net"
    }
  ]
}
```

### `public_dns_create_hash`

Creates a new Secure DNS hash for a tunnel.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel whose DNS config to share |

#### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "public_dns_create_hash=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN" | jq .
```

#### Response

```json
{
  "status": "success",
  "host_hash": "gl6nqcbyhsau",
  "tunnel_token": "abc...",
  "doh_url": "https://gl6nqcbyhsau.sdns.doxx.net/dns-query",
  "dot_host": "gl6nqcbyhsau.sdns.doxx.net"
}
```

Configure the returned URLs on any device:
- **DoH:** `https://gl6nqcbyhsau.sdns.doxx.net/dns-query` (browsers, iOS, macOS)
- **DoT:** `gl6nqcbyhsau.sdns.doxx.net` on port 853 (Android Private DNS)

### `public_dns_delete_hash`

Deletes a Secure DNS hash, disabling the shared DoH/DoT endpoint.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `host_hash` | Yes | The hash to delete |

#### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "public_dns_delete_hash=1&token=$TOKEN&host_hash=gl6nqcbyhsau" | jq .
```
