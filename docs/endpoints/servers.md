# Servers

Server discovery endpoints. These do not require authentication.

## `servers`

Returns the list of tunnel locations. Each entry is a doxx.net cluster on owned hardware (active-active nodes with per-packet failover) addressed by one hostname. There is no per-machine view: a location is the unit you connect to, and the cluster behind it is doxx.net's to run. `cluster_key_count` is the live number of concurrently valid server identities that cluster answers for (rotating derived keys held in RAM), which is why one location behaves like that many individual servers. Results are cached for 5 minutes.

**Authentication:** None required.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `servers` | Yes | Set to `1` |
| `type` | No | Filter by tunnel type (e.g. `wireguard`) |

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
      "server_name": "wireguard.zrh.eu.doxx.net",
      "hostname": "wireguard.zrh.eu.doxx.net",
      "location": "Zurich, Switzerland",
      "description": "WireGuard Server for doxx.net",
      "type": "wireguard",
      "public_key": "",
      "best_for": "Swiss privacy laws (world's strongest), Banking and financial privacy, Neutral jurisdiction.",
      "operator": "Doxx Communications Europe GmbH of Zurich Switzerland",
      "bg_image": "wireguard.zrh.eu.doxx.net",
      "flag_image": "ch",
      "continent": "Europe",
      "created_at": "2026-09-07T00:00:00Z",
      "cluster_key_count": 96
    }
  ]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `server_name` | string | The location's hostname. Pass this to `create_tunnel` as the `server` parameter and to `lease_public_ipv4` / `create_saved_profile` as `server`. |
| `hostname` | string | Mirrors `server_name`. A location is a cluster, so there is no single physical hostname to expose. |
| `location` | string | Human-readable city plus region or country |
| `description` | string | Static label, not location-specific |
| `type` | string | Tunnel type (currently always `wireguard`) |
| `public_key` | string | Always empty. Location keys are not published: a location is a cluster serving a pool of rotating server identities, and `create_tunnel` / `wireguard` hand each tunnel its own server key. The field exists only so older clients keep decoding. |
| `best_for` | string | Suggested use case or geographic affinity |
| `operator` | string | Legal entity operating the location |
| `bg_image` | string | Background image filename for UI rendering |
| `flag_image` | string | Lowercase country code for the flag asset (e.g. `us`, `ch`) |
| `continent` | string | Continent name as displayed (`North America`, `Europe`, `Asia Pacific`, ...). Apps build their continent filter from this value. |
| `created_at` | string | RFC 3339 timestamp of the location's earliest record |
| `cluster_key_count` | int | Live count of concurrently valid server identities behind this location, read from the key pool at request time (rotating monthly epochs). Not a fixed number; it moves as epochs rotate. |

---

## `list_tlds`

Returns all available top-level domains for domain registration, across categories including crypto, hacking, tech, gaming, and single-letter domains. The response `count` is the live total.

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
