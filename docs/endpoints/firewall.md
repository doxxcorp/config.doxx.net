# Firewall

Firewall endpoints manage inbound access rules for your tunnels. By default, tunnels block all inbound connections. Firewall rules allow traffic from specific sources to reach specific tunnel IPs and ports. This enables mesh networking between your devices or exposing services.

---

## `firewall_rule_list`

Returns all firewall rules for the account, optionally filtered to a specific tunnel. Also indicates whether the "link all" mode is active.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | No | Filter rules to a specific tunnel |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_rule_list=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "link_all_enabled": false,
  "rules": [
    {
      "tunnel_token": "abc...",
      "protocol": "TCP",
      "src_ip": "0.0.0.0/0",
      "src_port": "ALL",
      "dst_ip": "10.1.0.227",
      "dst_port": "443"
    }
  ],
  "count": 1
}
```

---

## `firewall_rule_add`

Creates a new firewall rule allowing inbound traffic to a tunnel.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel to add the rule to |
| `protocol` | Yes | `TCP`, `UDP`, `ICMP`, or `ALL` |
| `src_ip` | Yes | Source IP address or CIDR range (e.g. `10.1.0.227/32` or `0.0.0.0/0` for any) |
| `src_port` | Yes | Source port number or `ALL` |
| `dst_ip` | Yes | Destination IP (your tunnel's IP address) |
| `dst_port` | Yes | Destination port number or `ALL` |

### Example: Allow SSH from another tunnel

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_rule_add=1&token=$TOKEN&tunnel_token=SERVER_TUNNEL_TOKEN&protocol=TCP&src_ip=10.1.0.227/32&src_port=ALL&dst_ip=10.1.2.101&dst_port=22" | jq .
```

### Example: Allow all traffic between two tunnels

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_rule_add=1&token=$TOKEN&tunnel_token=SERVER_TUNNEL_TOKEN&protocol=ALL&src_ip=10.1.0.227/32&src_port=ALL&dst_ip=10.1.2.101&dst_port=ALL" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Firewall rule created successfully",
  "rule": {
    "tunnel_token": "abc...",
    "protocol": "TCP",
    "src_ip": "10.1.0.227/32",
    "src_port": "ALL",
    "dst_ip": "10.1.2.101",
    "dst_port": "22",
    "enabled": 1
  }
}
```

### Errors

| HTTP | Message | Cause |
|------|---------|-------|
| 400 | `Missing required parameters` | One or more rule fields missing |
| 400 | `Invalid protocol` | Protocol must be TCP, UDP, ICMP, or ALL |
| 404 | `Tunnel not found` | Tunnel does not exist or is not yours |
| 409 | `Rule already exists` | An identical rule already exists |

---

## `firewall_rule_delete`

Deletes a firewall rule. Uses the same parameters as `firewall_rule_add` to identify the rule.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel the rule belongs to |
| `protocol` | Yes | `TCP`, `UDP`, `ICMP`, or `ALL` |
| `src_ip` | Yes | Source IP/CIDR |
| `src_port` | Yes | Source port or `ALL` |
| `dst_ip` | Yes | Destination IP |
| `dst_port` | Yes | Destination port or `ALL` |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_rule_delete=1&token=$TOKEN&tunnel_token=SERVER_TUNNEL_TOKEN&protocol=TCP&src_ip=10.1.0.227/32&src_port=ALL&dst_ip=10.1.2.101&dst_port=22" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Firewall rule deleted successfully"
}
```

---

## `firewall_link_all_toggle`

Enables or disables "link all" mode. When enabled, all tunnels on the account can communicate with each other on all ports and protocols. This creates a full mesh network between all your devices. When disabled, any existing link-all rules are removed.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `enabled` | Yes | `1` to enable mesh networking, `0` to disable |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_link_all_toggle=1&token=$TOKEN&enabled=1" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Link all enabled",
  "link_all_tunnels": 1,
  "rules_deleted": 0
}
```

When disabling (`enabled=0`), `rules_deleted` shows how many auto-generated rules were cleaned up.

---

## `firewall_link_all_status`

Returns whether "link all" mode is currently enabled.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "firewall_link_all_status=1&token=$TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "link_all_tunnels": 0
}
```
