# Tunnels

Tunnel endpoints manage VPN connections. Each tunnel gets a unique token, IP addresses (IPv4 and IPv6), and WireGuard keys. Creating tunnels requires a subscription with available seats.

## Subscription and Seat Limits

Tunnel creation is subject to seat limits based on your subscription plan. Each tunnel occupies one seat. When all seats are in use, additional tunnel creation is blocked until a tunnel is deleted or a seat is freed.

---

## `create_tunnel`

Creates a new WireGuard tunnel on the specified server.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `server` | Yes | Server hostname from the `servers` endpoint (e.g. `wireguard.mia.us.doxx.net`) |
| `name` | No | Display name for the tunnel |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "create_tunnel=1&token=$TOKEN&name=My+Laptop&server=wireguard.mia.us.doxx.net" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Tunnel created successfully"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `server required` | Missing `server` parameter |
| 401 | `Invalid or expired auth token` | Bad or missing token |
| 409 | `Seat limit reached` | All subscription seats are in use |

---

## `list_tunnels`

Returns all tunnels for the authenticated account with their configuration, connection status, and assigned addresses.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "list_tunnels=1&token=$TOKEN" | jq '.tunnels[] | {tunnel_token, name, assigned_ip, server}'
```

### Response

```json
{
  "status": "success",
  "tunnels": [
    {
      "tunnel_token": "Eh1xwlLd...",
      "name": "My Laptop",
      "server": "wireguard.mia.us.doxx.net",
      "assigned_ip": "10.1.0.226/31",
      "assigned_v6": "2602:f5c1:1::1c0:8916/127",
      "public_key": "abc...",
      "private_key": "xyz...",
      "type": "wireguard",
      "device_hash": "",
      "device_type": "",
      "created_at": "2025-06-01T12:00:00Z",
      "block_bad_dns": 1,
      "firewall": 1,
      "ipv6_enabled": 1,
      "onion_enabled": 0,
      "proxy_enabled": 0,
      "is_connected": true,
      "connection_status": "connected"
    }
  ]
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `tunnel_token` | string | Unique identifier for this tunnel. Used in all tunnel-specific API calls. |
| `name` | string | Display name |
| `server` | string | Server hostname this tunnel is on |
| `assigned_ip` | string | Private IPv4 address with CIDR mask |
| `assigned_v6` | string | IPv6 address with prefix length |
| `public_key` | string | WireGuard public key |
| `private_key` | string | WireGuard private key |
| `type` | string | Tunnel type (`wireguard`) |
| `device_hash` | string | Device identifier (mobile clients) |
| `device_type` | string | `mobile`, `desktop`, `server`, or `web` |
| `block_bad_dns` | int | 1 if DNS leak protection is enabled |
| `firewall` | int | 1 if firewall is enabled |
| `ipv6_enabled` | int | 1 if IPv6 is enabled |
| `onion_enabled` | int | 1 if Tor routing is enabled |
| `proxy_enabled` | int | 1 if geo-spoofing proxy is enabled |
| `is_connected` | bool | Whether the tunnel is currently connected |
| `connection_status` | string | `connected`, `disconnected`, or `sleeping` |

---

## `update_tunnel`

Updates tunnel settings. Only the provided parameters are changed.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel to update |
| `name` | No | New display name |
| `server` | No | New server hostname (migrates tunnel) |
| `firewall` | No | `1` to enable, `0` to disable |
| `ipv6_enabled` | No | `1` to enable, `0` to disable |
| `block_bad_dns` | No | `1` to enable DNS leak protection, `0` to disable |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_tunnel=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&name=Work+Server&firewall=1" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Tunnel updated successfully"
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `tunnel_token required` | Missing tunnel token |
| 404 | `Tunnel not found` | Tunnel does not exist or is not yours |

---

## `delete_tunnel`

Permanently deletes a tunnel and releases its IP addresses.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel to delete |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "delete_tunnel=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Tunnel deleted successfully"
}
```

---

## `wireguard`

Returns the full WireGuard configuration for a tunnel, including private key, addresses, DNS, peer public key, and endpoint. Use this to generate a `.conf` file.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "wireguard=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN" | jq .config
```

### Response

```json
{
  "status": "success",
  "config": {
    "interface": {
      "private_key": "your_private_key",
      "address": "10.1.0.227/31, 2602:f5c1:1::1c0:8917/128",
      "dns": "10.10.10.10,fd53::"
    },
    "peer": {
      "public_key": "server_public_key",
      "allowed_ips": "0.0.0.0/0, ::/0",
      "endpoint": "wireguard.mia.us.doxx.net:51820",
      "persistent_keepalive": 25
    }
  }
}
```

### Building a .conf File

```bash
CONFIG=$(curl -s -X POST $API -d "wireguard=1&token=$TOKEN&tunnel_token=$TUNNEL")

cat > /etc/wireguard/doxx.conf << EOF
[Interface]
PrivateKey = $(echo $CONFIG | jq -r '.config.interface.private_key')
Address = $(echo $CONFIG | jq -r '.config.interface.address')
DNS = $(echo $CONFIG | jq -r '.config.interface.dns')

[Peer]
PublicKey = $(echo $CONFIG | jq -r '.config.peer.public_key')
AllowedIPs = $(echo $CONFIG | jq -r '.config.peer.allowed_ips')
Endpoint = $(echo $CONFIG | jq -r '.config.peer.endpoint')
PersistentKeepalive = 25
EOF

sudo wg-quick up doxx
```

---

## `get_mobile_options`

Returns mobile app settings for the authenticated account.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Response

```json
{
  "status": "success",
  "mobile_options": {
    "connect_on_startup": 0,
    "kill_switch": 0,
    "transport": "wireguard",
    "proxy_enabled": 0,
    "onion_enabled": 0,
    "port": null
  }
}
```

---

## `set_mobile_options`

Updates mobile app settings. Only provided parameters are changed.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `connect_on_startup` | No | `1` to auto-connect on app launch, `0` to disable |
| `kill_switch` | No | `1` to block all traffic when VPN disconnects, `0` to allow |
| `proxy_enabled` | No | `1` to enable geo-spoofing proxy, `0` to disable |
| `onion_enabled` | No | `1` to route traffic through Tor, `0` to disable |

### Response

```json
{
  "status": "success",
  "message": "Mobile options updated"
}
```
