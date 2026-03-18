# Addresses

Address management endpoints handle IP address allocation, assignment to profiles, rotation, and public IPv4 leasing. Addresses can be persistent (survive tunnel reconnects) and are tied to geographic locations (sites).

## Address Types

| Type | Description |
|------|-------------|
| `static_private` | Private IPv4 address from the 10.x.x.x range. Assigned per-site. Used for VPN tunnel connectivity. |
| `static_public` | Dedicated public IPv4 address. Requires a Pro subscription. Used for services that need a fixed internet-facing IP. |
| `static_ipv6` | IPv6 address. Assigned per-site alongside IPv4 allocations. |

## Site and Location

Every address is allocated at a specific site (geographic location). When assigning an address to a profile, the address's site must match the profile's preferred server location. A site mismatch returns HTTP 409 with details about both locations, allowing the client to either correct the mismatch or pass `update_location=1` to automatically update the profile's location.

## IP Lock Behavior

When a profile has `ip_locked=1`, address operations (assign, release, rotate, lease) are blocked with HTTP 423. Unlock the profile's IPs first.

---

## `list_ip_reservations`

Returns public IPv4 addresses that are reserved to profiles (dedicated IPs).

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_ip_reservations=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "reservations": [
    {
      "ip_address": "198.51.100.42",
      "server": "wireguard.mia.us.doxx.net",
      "profile_id": 3,
      "profile_name": "Production Server"
    }
  ],
  "slots_used": 1,
  "slots_max": 3
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `reservations` | array | Public IPs bound to profiles |
| `slots_used` | int | Number of dedicated IP slots in use |
| `slots_max` | int | Maximum slots available based on subscription |

---

## `release_ip_reservation`

Releases a public IPv4 reservation from a profile. The IP remains allocated to the account but is no longer bound to a specific profile.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `ip_address` | Yes | IP address to un-reserve |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "release_ip_reservation=1&token=$TOKEN&ip_address=198.51.100.42" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "IP reservation released"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `ip_address required` | Missing parameter |
| 400 | `This IP is not reserved to a profile` | IP exists but has no profile assignment |
| 404 | `IP reservation not found` | IP does not exist or is not yours |

---

## `list_addresses`

Returns all addresses (private IPv4, public IPv4, and IPv6) for the account. Includes assignment status, profile binding, location, and connection state.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | No | Filter to addresses used by a specific tunnel |
| `device_hash` | No | Filter to addresses used by a specific device's tunnels |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_addresses=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "addresses": [
    {
      "address": "10.1.0.227",
      "type": "static_private",
      "site_id": 1,
      "location": "Miami, FL",
      "persistent": 1,
      "connected": true,
      "profile_id": 3,
      "profile_name": "Privacy Mode",
      "device_name": "iPhone 15 Pro",
      "tunnel_name": "iPhone - Miami"
    },
    {
      "address": "198.51.100.42",
      "type": "static_public",
      "site_id": 1,
      "location": "Miami, FL",
      "persistent": 1,
      "connected": false,
      "profile_id": 5,
      "profile_name": "Production Server"
    },
    {
      "address": "2602:f5c1:1::1c0:8917",
      "type": "static_ipv6",
      "site_id": 1,
      "location": "Miami, FL",
      "persistent": 1,
      "connected": true,
      "profile_id": 3,
      "profile_name": "Privacy Mode"
    }
  ],
  "total_addresses": 3,
  "max_addresses": 25,
  "public_ipv4_used": 1,
  "public_ipv4_max": 3
}
```

### Response Fields per Address

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | IP address (host address, not CIDR) |
| `type` | string | `static_private`, `static_public`, or `static_ipv6` |
| `site_id` | int | Geographic site identifier |
| `location` | string | Human-readable location name |
| `persistent` | int | 1 if the address persists across reconnects |
| `connected` | bool | Whether the tunnel using this address is currently online |
| `profile_id` | int | Profile this address is assigned to (if any) |
| `profile_name` | string | Profile name (if assigned) |
| `device_name` | string | Device using this address (if any) |
| `tunnel_name` | string | Tunnel name (if any) |

---

## `assign_address`

Assigns an address to a profile, or unassigns it (by omitting `profile_id`). Each profile can have one address per type. Assigning a public IPv4 to a profile automatically releases any private IPv4 on that profile, and vice versa.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `address` | Yes | IP address to assign |
| `type` | Yes | `static_private`, `static_public`, or `static_ipv6` |
| `profile_id` | No | Profile to assign to. Omit to unassign. |
| `update_location` | No | `1` to automatically update the profile's location to match the address's site (resolves site mismatches) |

### Example: Assign to profile

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "assign_address=1&token=$TOKEN&address=10.1.0.227&type=static_private&profile_id=3" | jq .
```

### Example: Unassign from profile

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "assign_address=1&token=$TOKEN&address=10.1.0.227&type=static_private" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Address assigned"
}
```

### Site Mismatch Response (HTTP 409)

When the address's site does not match the profile's preferred server location:

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

Pass `update_location=1` to automatically switch the profile's location to match the address.

### Auto-Replacement Behavior

When unassigning an IPv4 from a profile, the system automatically allocates a new private IPv4 at the same site for the profile, ensuring the profile always has at least one IPv4 address.

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `address and type required` | Missing parameters |
| 400 | `Invalid type` | Type is not `static_private`, `static_public`, or `static_ipv6` |
| 404 | `Address not found` | Address does not exist or is not yours |
| 404 | `Profile not found` | Invalid profile_id |
| 409 | `site_mismatch` | Address site does not match profile site (see above) |

---

## `release_address`

Releases an address back to the pool. For public IPv4, this deallocates the IP entirely. For private IPv4 and IPv6, this clears the profile binding and persistence flag.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `address` | Yes | IP address to release |
| `type` | Yes | `static_private`, `static_public`, or `static_ipv6` |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "release_address=1&token=$TOKEN&address=198.51.100.42&type=static_public" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Address released"
}
```

---

## `rotate_address`

Replaces a private IPv4 or IPv6 address with a new one at the same site. The old address is released and a new address is allocated from the same location's pool. Public IPv4 addresses cannot be rotated.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `address` | Yes | Current IP address to rotate |
| `type` | Yes | `static_private` or `static_ipv6` (public IPv4 cannot be rotated) |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "rotate_address=1&token=$TOKEN&address=10.1.0.227&type=static_private" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Address rotated",
  "old_address": "10.1.0.227",
  "new_address": "10.1.2.101"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `address and type required` | Missing parameters |
| 400 | `Can only rotate static_private or static_ipv6` | Attempted to rotate a public IPv4 |
| 404 | `Address not found or has no location` | Address does not exist or has no site assignment |

---

## `lease_public_ipv4`

Allocates a dedicated public IPv4 address. Operates in three modes depending on which parameters are provided. Public IPv4 requires a Pro subscription and available dedicated IP slots.

### Mode 1: Existing Profile

Leases a public IPv4 and attaches it to an existing profile.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_id` | Yes | Profile to attach the IP to |
| `include_ipv6` | No | `1` to also allocate an IPv6 address |

### Mode 2: New Profile + IP

Creates a new profile and leases a public IPv4 in one call.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `profile_name` | Yes | Name for the new profile (max 128 characters) |
| `server` | Yes | Server hostname for the profile's location |
| `profile_icon` | No | Icon identifier |
| `profile_type` | No | `ios` or `wireguard`. Defaults to `wireguard`. |
| `include_ipv6` | No | `1` to also allocate an IPv6 address |

### Mode 3: Pool Lease

Allocates an IP to the account pool without binding to a profile.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `server` | Yes | Server hostname for the site to allocate from |
| `ip_type` | No | Set to `ipv6` for IPv6-only pool lease |
| `include_ipv6` | No | `1` to also allocate an IPv6 address alongside IPv4 |

### Example: Lease for existing profile

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "lease_public_ipv4=1&token=$TOKEN&profile_id=3" | jq .
```

### Example: Create profile + lease

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "lease_public_ipv4=1&token=$TOKEN&profile_name=Production&server=wireguard.mia.us.doxx.net&profile_type=wireguard" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Public IPv4 leased",
  "ip_address": "198.51.100.42",
  "site_id": 1,
  "server": "wireguard.mia.us.doxx.net",
  "profile_id": 3,
  "requires_reconnect": true
}
```

When `include_ipv6=1`:

```json
{
  "status": "success",
  "message": "Public IPv4 + IPv6 leased",
  "ip_address": "198.51.100.42",
  "ipv6_address": "2602:f5c1:1::abc:1234/127",
  "site_id": 1,
  "server": "wireguard.mia.us.doxx.net",
  "profile_id": 3,
  "requires_reconnect": true
}
```

When a new profile was created:

```json
{
  "status": "success",
  "message": "Public IPv4 leased",
  "ip_address": "198.51.100.42",
  "profile_id": 5,
  "profile_created": true,
  "site_id": 1,
  "server": "wireguard.mia.us.doxx.net",
  "requires_reconnect": true
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `Provide profile_id, profile_name + server, or server alone for pool lease` | Invalid parameter combination |
| 400 | `Profile has no preferred location` | Profile exists but has no server set |
| 403 | `Pro subscription required for dedicated public IPv4` | No dedicated IP slots available |
| 404 | `Profile not found` | Invalid profile_id |
| 409 | `Profile already has a public IPv4: ...` | Profile already has a dedicated IP |
| 409 | `All dedicated IP slots in use...` | All subscription slots are occupied |
| 409 | `Profile limit reached` | Already at 20 profiles (mode 2 only) |
| 423 | `IPs are locked on this profile` | Profile has ip_locked=1 |
