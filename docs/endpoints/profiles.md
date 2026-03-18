# Profiles

Saved connection profiles store a snapshot of tunnel settings (DNS blocking, firewall, proxy, transport, etc.) that can be loaded onto any tunnel. Profiles act as reusable configuration templates.

## Profile Types

Profiles have a `profile_type` field that determines which tunnels they can be loaded onto:

| Type | Description |
|------|-------------|
| `ios` | iOS/Android mobile app profiles. Created from mobile tunnels. |
| `wireguard` | Desktop/server WireGuard profiles. Created from WireGuard tunnels. |

A profile can only be loaded onto a tunnel that matches its type. Attempting to load an `ios` profile onto a WireGuard tunnel (or vice versa) returns an error.

## Profile Limits

Each account can have up to 20 saved profiles. Attempting to create more returns HTTP 409.

## Profile Locking

Profiles support two independent lock types:

| Lock Type | Effect |
|-----------|--------|
| `ip` | Prevents IP address changes on the profile (no assign, release, rotate, or lease) |
| `settings` | Prevents settings changes on the profile |

When both locks are active, the profile is fully frozen. A profile that is currently in use by another device cannot be locked or unlocked.

---

## `list_saved_profiles`

Returns all saved profiles for the account, including their settings, lock state, and whether they are currently in use by a connected device.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_saved_profiles=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "profiles": [
    {
      "profile_id": 1,
      "profile_name": "Privacy Mode",
      "profile_icon": "shield",
      "profile_notes": "",
      "profile_type": "ios",
      "preferred_server": "wireguard.mia.us.doxx.net",
      "bandwidth_stats": 1,
      "security_stats": 1,
      "block_bad_dns": 1,
      "block_doh_dot": 0,
      "keep_established_ssh": 0,
      "kill_default_route": 0,
      "auto_reconnect": 1,
      "enable_routing": 1,
      "snarf_dns": 1,
      "firewall": 1,
      "ipv6_enabled": 1,
      "onion_enabled": 0,
      "mtu_spoofing": 0,
      "kill_switch": 0,
      "connect_on_startup": 1,
      "ipv4_public_enabled": 0,
      "proxy_enabled": 0,
      "connection_profile": "security",
      "transport": "normal",
      "locked": 0,
      "ip_locked": 0,
      "settings_locked": 0,
      "source_tunnel_name": "iPhone - Miami",
      "created_at": "2026-01-15 10:00:00",
      "updated_at": "2026-03-01 14:00:00",
      "domain_name": "server1.mysite.doxx",
      "in_use": true,
      "in_use_by": "iPhone 15 Pro",
      "in_use_tunnel_token": "abc...",
      "in_use_device_icon": "iphone"
    }
  ]
}
```

### Key Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `profile_id` | int | Unique identifier within the account |
| `profile_type` | string | `ios` or `wireguard` |
| `preferred_server` | string | Server hostname this profile is designed for |
| `locked` | int | Legacy lock flag (1 = locked) |
| `ip_locked` | int | 1 if IP changes are locked |
| `settings_locked` | int | 1 if settings changes are locked |
| `in_use` | bool | Whether this profile is loaded on a connected device |
| `in_use_by` | string | Device name using this profile (only present when `in_use` is true) |
| `in_use_tunnel_token` | string | Tunnel token using this profile (only present when `in_use` is true) |
| `domain_name` | string | Linked domain FQDN, if any |
| `requires_pro` | array | List of Pro features needed by this profile (e.g. `["onion"]`) |

---

## `save_profile`

Snapshots a tunnel's current settings into a new saved profile. The profile captures all DNS, firewall, proxy, and connection settings from the tunnel at the time of the call.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel to snapshot |
| `profile_name` | Yes | Name for the profile (max 128 characters) |
| `profile_icon` | No | Icon identifier |
| `profile_notes` | No | Freeform notes |
| `profile_type` | No | `ios` or `wireguard`. Defaults to `ios`. |
| `preferred_server` | No | Override the server stored in the profile |
| `save_preferred_server` | No | `1` to save the tunnel's current server as the preferred server |
| `lock_after_save` | No | `1` to lock the profile immediately after saving |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "save_profile=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&profile_name=Privacy+Mode&profile_type=ios&save_preferred_server=1" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile saved",
  "profile_id": 3
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `tunnel_token and profile_name required` | Missing parameters |
| 400 | `Profile name must be 128 characters or less` | Name too long |
| 403 | `Access denied` | Tunnel belongs to another account |
| 404 | `Tunnel not found` | Invalid tunnel token |
| 409 | `Profile limit reached` | Already at 20 profiles |

---

## `create_saved_profile`

Creates a new empty profile with default settings, assigned to a specific server location. Unlike `save_profile`, this does not snapshot an existing tunnel.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_name` | Yes | Name for the profile (max 128 characters) |
| `server` | Yes | Server hostname for the profile's location |
| `profile_icon` | No | Icon identifier |
| `profile_type` | No | `ios` or `wireguard`. Defaults to `wireguard`. |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_saved_profile=1&token=$TOKEN&profile_name=New+Server&server=wireguard.mia.us.doxx.net&profile_type=wireguard" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile created",
  "profile_id": 4
}
```

---

## `load_profile`

Applies a saved profile's settings to a tunnel. The tunnel's DNS subscriptions, firewall settings, proxy config, and all other options are overwritten with the profile's stored values. For WireGuard profiles, public IPv4 allocations are also bound to the tunnel.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | Yes | Profile to load |
| `tunnel_token` | Yes | Tunnel to apply the profile to |
| `source` | No | Caller type (`ios`, `wireguard`). If provided, must match the profile's type. |
| `lock_after_load` | No | `1` to lock the profile onto this tunnel after loading |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "load_profile=1&token=$TOKEN&profile_id=3&tunnel_token=TUNNEL_TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile loaded",
  "profile_id": 3,
  "profile_name": "Privacy Mode",
  "mode": "security"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `profile_id and tunnel_token required` | Missing parameters |
| 400 | `Cannot apply a ios profile to a wireguard tunnel` | Profile type mismatch |
| 400 | `Profile location does not match tunnel location` | Site mismatch between profile and tunnel |
| 402 | `Pro subscription required for this profile` | Profile uses features that require a paid plan |
| 404 | `Tunnel not found or access denied` | Invalid tunnel token |
| 404 | `Profile not found` | Invalid profile_id |
| 409 | `This profile is already in use on "iPhone 15 Pro"...` | Profile is loaded on another connected device |

---

## `update_saved_profile`

Updates a profile's metadata (name, icon, notes) or re-snapshots settings from a tunnel.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | Yes | Profile to update |
| `profile_name` | No | New name (max 128 characters) |
| `profile_icon` | No | New icon identifier |
| `profile_notes` | No | New notes |
| `re_snapshot` | No | `1` to re-capture settings from a tunnel |
| `tunnel_token` | No | Required when `re_snapshot=1`. Tunnel to snapshot from. |

### Example: Rename

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_saved_profile=1&token=$TOKEN&profile_id=3&profile_name=Maximum+Privacy" | jq .
```

### Example: Re-snapshot from tunnel

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_saved_profile=1&token=$TOKEN&profile_id=3&re_snapshot=1&tunnel_token=TUNNEL_TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile updated",
  "profile_id": 3
}
```

---

## `delete_saved_profile`

Deletes a saved profile. If any tunnels are currently using this profile, their active_profile_id is cleared, their profile lock is released, and public IPv4 allocations are swapped to private addresses. IP reservations tied to the profile are also released.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | Yes | Profile to delete |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "delete_saved_profile=1&token=$TOKEN&profile_id=3" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile deleted"
}
```

---

## `lock_profile`

Locks a profile to prevent changes. You can lock IP addresses, settings, or both.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | No | Profile to lock (provide this or `tunnel_token`) |
| `tunnel_token` | No | Tunnel whose active profile should be locked |
| `lock_type` | No | `ip` (lock IPs only), `settings` (lock settings only), or omit for both |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "lock_profile=1&token=$TOKEN&profile_id=3&lock_type=ip" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile locked",
  "profile_id": 3,
  "lock_type": "ip"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `profile_id or tunnel_token required` | Neither identifier provided |
| 400 | `No profile is loaded on this tunnel` | Tunnel has no active profile (when using tunnel_token) |
| 404 | `Profile not found` | Invalid profile_id |
| 409 | `Profile is in use on another device` | Cannot lock while another device is using it |

---

## `unlock_profile`

Unlocks a previously locked profile.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | No | Profile to unlock (provide this or `tunnel_token`) |
| `tunnel_token` | No | Tunnel whose active profile should be unlocked |
| `lock_type` | No | `ip` (unlock IPs only), `settings` (unlock settings only), or omit for both |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "unlock_profile=1&token=$TOKEN&profile_id=3" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Profile unlocked",
  "profile_id": 3
}
```

---

## `apply_mode`

Applies a configuration mode directly to a tunnel without saving a profile. This is a "quick apply" that writes settings, DNS blocklists, and custom whitelist/blacklist entries to a tunnel in a single call.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel to apply to |
| `template_key` | No | Mode identifier (e.g. `security`, `privacy`, `gaming`) |
| `settings` | Yes | JSON object of settings to apply |
| `dns` | No | JSON object with `blocklists`, `custom_blacklist`, and `custom_whitelist` arrays |

### Settings JSON Fields

```json
{
  "bandwidth_stats": true,
  "security_stats": true,
  "block_bad_dns": true,
  "block_doh_dot": false,
  "keep_established_ssh": false,
  "kill_default_route": false,
  "auto_reconnect": true,
  "enable_routing": true,
  "snarf_dns": true,
  "firewall": true,
  "ipv6_enabled": true,
  "onion_enabled": false,
  "mtu_spoofing": false,
  "kill_switch": false,
  "connect_on_startup": true,
  "proxy_enabled": false,
  "connection_profile": "security",
  "transport": "normal",
  "port": null
}
```

### DNS JSON Fields

```json
{
  "blocklists": ["ads", "tracking", "malware"],
  "custom_blacklist": ["evil.com"],
  "custom_whitelist": ["example.com"]
}
```

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "apply_mode=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&template_key=security" \
  --data-urlencode 'settings={"block_bad_dns":true,"firewall":true,"kill_switch":true}' \
  --data-urlencode 'dns={"blocklists":["ads","malware","tracking"]}' | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Mode applied",
  "template_key": "security"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `tunnel_token and settings required` | Missing parameters |
| 400 | `Invalid settings JSON` | Settings could not be parsed |
| 404 | `Tunnel not found or access denied` | Invalid tunnel token |
