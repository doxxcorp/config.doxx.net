# Proxy

Proxy endpoints configure geo-spoofing for tunnels. When enabled, HTTP requests from the tunnel are routed through a proxy that makes them appear to originate from a different geographic location. This affects the apparent location reported by websites and services that use IP geolocation.

---

## `get_proxy_config`

Returns the current proxy configuration for a tunnel.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |

### Example

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "get_proxy_config=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN" | jq .
```

### Response

```json
{
  "status": "success",
  "config": {
    "tunnel_token": "abc...",
    "assigned_ip": "10.1.0.226",
    "assigned_v6": "2602:f5c1:1::1c0:8916",
    "enabled": false,
    "location": "newyork-us",
    "browser": null,
    "custom_lat": null,
    "custom_lon": null
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `enabled` | bool | Whether the proxy is currently active |
| `location` | string | Location ID determining the apparent geographic origin of proxied requests |
| `browser` | string or null | Browser fingerprint identifier for HTTP header spoofing |
| `custom_lat` | float or null | Custom latitude for geolocation spoofing (overrides location-based coordinates) |
| `custom_lon` | float or null | Custom longitude for geolocation spoofing (overrides location-based coordinates) |

---

## `update_proxy_config`

Updates the proxy configuration for a tunnel. Only provided parameters are changed.

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | Auth token |
| `tunnel_token` | Yes | Tunnel token |
| `enabled` | No | `1` to enable the proxy, `0` to disable |
| `location` | No | Location ID (e.g. `newyork-us`, `london-uk`, `tokyo-jp`) |
| `browser` | No | Browser fingerprint string for HTTP header spoofing |

### Example: Enable proxy with New York location

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_proxy_config=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&enabled=1&location=newyork-us" | jq .
```

### Example: Change location

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_proxy_config=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&location=london-uk" | jq .
```

### Example: Disable proxy

```bash
curl -s -X POST https://config.doxx.net/v1/ \
  -d "update_proxy_config=1&token=$TOKEN&tunnel_token=TUNNEL_TOKEN&enabled=0" | jq .
```

### Response

```json
{
  "status": "success",
  "message": "Proxy configuration updated"
}
```

### Geo-Spoofing Details

When the proxy is enabled:

1. **IP Geolocation:** HTTP requests appear to originate from the configured location, affecting IP-based geolocation services.
2. **Browser Fingerprint:** If a `browser` value is set, HTTP headers (User-Agent, Accept-Language, etc.) are modified to match the specified browser profile.
3. **Custom Coordinates:** When `custom_lat` and `custom_lon` are set, they override the location's default coordinates for services that check GPS-level geolocation via JavaScript APIs.

### Notes

- The proxy only affects HTTP/HTTPS traffic, not raw IP connections.
- Enabling the proxy requires the tunnel to be connected.
- Proxy is a Pro subscription feature.
