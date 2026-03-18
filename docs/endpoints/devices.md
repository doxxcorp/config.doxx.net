# Devices

Device management endpoints. Devices represent physical or virtual machines that have connected through the VPN. Each device is identified by a unique hash and can have multiple tunnels associated with it.

---

## `device_list_unified`

Returns all devices for the authenticated account, including their online status, seat status, tunnel count, and subscription summary. For plan owners, also returns guest account devices from seat grants.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `device_id_hash` | No | Current device's hash (marks it as `is_current` in the response) |
| `device_model` | No | Device model string (auto-synced for the calling device) |
| `os_type` | No | Operating system type (auto-synced for the calling device) |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "device_list_unified=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "my_devices": [
    {
      "device_hash": "a1b2c3d4e5f6...",
      "device_name": "iPhone 15 Pro",
      "device_model": "iPhone15,2",
      "os_type": "ios",
      "device_type": "mobile",
      "is_current": true,
      "is_online": true,
      "last_seen": "2026-03-18 10:00:00",
      "tunnel_count": 1,
      "has_seat": true,
      "can_remove": true,
      "can_rename": true,
      "can_delete": false,
      "last_active_at": "2026-03-18 09:00:00"
    }
  ],
  "guest_accounts": [
    {
      "guest_user_id": 42,
      "nickname": "Alex",
      "profile_name": "Alex's Account",
      "devices": [
        {
          "device_hash": "f6e5d4c3...",
          "device_name": "iPad Air",
          "device_model": "iPad14,1",
          "os_type": "ios",
          "is_online": false,
          "last_seen": "2026-03-17 18:00:00",
          "has_seat": true,
          "can_remove": true
        }
      ]
    }
  ],
  "subscription": {
    "exists": true,
    "tier": "pro",
    "status": "active",
    "device_count": 3,
    "max_devices": 6,
    "is_account_owner": true
  }
}
```

### Response Fields: `my_devices`

| Field | Type | Description |
|-------|------|-------------|
| `device_hash` | string | Unique device identifier |
| `device_name` | string | Display name (user-customizable via `device_rename`) |
| `device_model` | string | Hardware model identifier |
| `os_type` | string | Operating system (`ios`, `android`, `macos`, `windows`, `linux`) |
| `device_type` | string | `mobile`, `desktop`, `server`, or `web` |
| `is_current` | bool | True if this is the calling device |
| `is_online` | bool | True if the device has an active tunnel connection |
| `tunnel_count` | int | Number of tunnels associated with this device |
| `has_seat` | bool | Whether this device occupies a subscription seat |
| `can_remove` | bool | Whether the calling user can remove this device's seat |
| `can_rename` | bool | Whether the calling user can rename this device |
| `can_delete` | bool | Whether the calling user can delete this device. The current device cannot be deleted. |

### Response Fields: `guest_accounts`

Only populated for subscription owners. Shows devices belonging to accounts that have been granted seats.

### Response Fields: `subscription`

| Field | Type | Description |
|-------|------|-------------|
| `exists` | bool | Whether the account has a subscription |
| `tier` | string | Plan name (e.g. `pro`) |
| `status` | string | Subscription status (`active`, `past_due`, etc.) |
| `device_count` | int | Number of seated devices |
| `max_devices` | int | Maximum devices allowed by the plan |
| `is_account_owner` | bool | Whether this account is the subscription owner (vs. a guest) |

---

## `device_delete`

Permanently deletes a device and all its associated tunnels. The calling device cannot delete itself.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `target_device_hash` | Yes | Device hash to delete |
| `device_id_hash` | No | Calling device's hash (to prevent self-deletion) |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "device_delete=1&token=$TOKEN&target_device_hash=a1b2c3d4e5f6..." | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Device deleted"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `target_device_hash required` | Missing parameter |
| 400 | `Cannot delete your current device` | The target device is the calling device |
| 404 | `Device not found or not yours` | Device does not exist or belongs to another account |

---

## `device_rename`

Changes the display name of a device. Also updates the names of all tunnels associated with this device.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `device_hash` | Yes | Device hash (full 32-character hash or 8-character prefix) |
| `device_name` | Yes | New display name |
| `device_icon` | No | Icon identifier for UI rendering |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "device_rename=1&token=$TOKEN&device_hash=a1b2c3d4&device_name=Work+Laptop" | jq .
```

### Response

```json
{
  "status": "success",
  "renamed": true,
  "device_name": "Work Laptop",
  "tunnels_updated": 2
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `renamed` | bool | Whether the rename succeeded |
| `device_name` | string | The validated device name that was applied |
| `tunnels_updated` | int | Number of tunnel names that were updated to reflect the new device name |

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `device_hash required` | Missing device hash |
| 400 | `Invalid device name` | Name validation failed |
| 403 | `Device not found or not yours` | Device belongs to another account |
| 404 | `Device not found` | No device matches the provided hash or prefix |
