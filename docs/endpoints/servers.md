# Servers

Server discovery endpoints. These do not require authentication.

## `servers`

Returns the list of available VPN servers with their locations, public keys, and metadata.

**Authentication:** None required.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `servers` | Yes | Set to `1` |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ -d "servers=1" | jq .
```

### Response

```json
{
  "status": "success",
  "servers": [
    {
      "server_name": "wireguard.mia.us.doxx.net",
      "location": "Miami, FL",
      "description": "US Southeast",
      "type": "wireguard",
      "public_key": "abc123...",
      "best_for": "US East Coast",
      "operator": "doxx.net",
      "bg_image": "miami.jpg",
      "flag_image": "us.svg",
      "continent": "NA"
    }
  ]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `server_name` | string | Hostname used when creating tunnels. Pass this to `create_tunnel` as the `server` parameter. |
| `location` | string | Human-readable city and region |
| `description` | string | Short description of the server's coverage area |
| `type` | string | Server type (currently always `wireguard`) |
| `public_key` | string | WireGuard public key for this server |
| `best_for` | string | Suggested use case or geographic affinity |
| `operator` | string | Entity operating the server |
| `bg_image` | string | Background image filename for UI rendering |
| `flag_image` | string | Country flag image filename |
| `continent` | string | Two-letter continent code (`NA`, `EU`, `AS`, etc.) |

---

## `list_tlds`

Returns all available top-level domains for domain registration. 196 TLDs are available across categories including crypto, hacking, tech, gaming, and single-letter domains.

**Authentication:** None required.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `list_tlds` | Yes | Set to `1` |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ -d "list_tlds=1" | jq .
```

### Response

```json
{
  "status": "success",
  "tlds": [
    {
      "tld": "doxx",
      "category": "gaming"
    },
    {
      "tld": "crypto",
      "category": "crypto"
    },
    {
      "tld": "onion",
      "category": "hacking"
    }
  ],
  "count": 196
}
```

### TLD Categories

| Category | Examples |
|----------|----------|
| Single Letters | `.b`, `.c`, `.x`, `.z` |
| Numbers | `.8`, `.404`, `.1337`, `.31337` |
| Crypto & Web3 | `.btc`, `.crypto`, `.eth`, `.dao`, `.wallet` |
| Hacking & Security | `.cyber`, `.exploit`, `.onion`, `.tor`, `.pwnd` |
| Tech & Infrastructure | `.api`, `.dns`, `.json`, `.wireguard`, `.sql` |
| Gaming & Culture | `.doxx`, `.gamer`, `.gta6`, `.vpn`, `.vibe` |

---

## `dns_get_options`

Returns all available DNS blocklists with metadata. Used to populate blocklist selection UI and to see available subscription names for `dns_set_subscription`.

**Authentication:** None required.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `dns_get_options` | Yes | Set to `1` |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ -d "dns_get_options=1" | jq .
```

### Response

```json
{
  "status": "success",
  "options": [
    {
      "name": "ads",
      "display_name": "Advertising",
      "description": "Block ad networks and trackers",
      "category": "privacy",
      "icon": "ad-icon",
      "domain_count": 150000,
      "default_enabled": true,
      "user_toggleable": true,
      "is_base_safety": false
    },
    {
      "name": "malware",
      "display_name": "Malware",
      "description": "Block known malware domains",
      "category": "security",
      "icon": "shield-icon",
      "domain_count": 85000,
      "default_enabled": true,
      "user_toggleable": false,
      "is_base_safety": true
    }
  ]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Machine-readable blocklist identifier. Use this as the `subscription` value in `dns_set_subscription`. |
| `display_name` | string | Human-readable name for UI display |
| `description` | string | What this blocklist blocks |
| `category` | string | Grouping category (`privacy`, `security`, `social`, etc.) |
| `icon` | string | Icon identifier for UI rendering |
| `domain_count` | int | Number of domains in this blocklist |
| `default_enabled` | bool | Whether this blocklist is enabled by default on new tunnels |
| `user_toggleable` | bool | Whether users can enable/disable this blocklist. Base safety lists cannot be toggled off. |
| `is_base_safety` | bool | If true, this is a mandatory protection list (malware, phishing) that cannot be disabled |
