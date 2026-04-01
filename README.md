# ghcr.io/dbosoft/license-proxy

A local proxy server container that handles license validation for dbosoft products.

For non-container and Windows downloads, please refer to the following link: https://www.dbosoft.eu/toolchain/license-proxy


## Quick Start

```bash
docker pull ghcr.io/dbosoft/license-proxy
docker run -d -p 5080:5080 ghcr.io/dbosoft/license-proxy
```
The proxy listens on port **5080** (HTTP) by default.

## Configuration

All settings can be configured via environment variables.

### Environment Variables

| Variable | Description | Default |
|---|---|---|
| `LICENSE_PROXY_URLS` | Listening URLs (e.g. `https://+:5443;http://+:5080`) | `http://+:5080` |
| `LICENSE_PROXY_CERT_FILE` | Path to PFX certificate file for HTTPS | - |
| `LICENSE_PROXY_CERT_PASSWORD` | Password for the PFX certificate file | - |
| `LICENSE_PROXY_UPSTREAM_CA_FILE` | Path to PEM/CER file of a custom CA to trust for upstream connections | - |
| `LICENSE_PROXY_PROXY_ADDRESS` | HTTP proxy for upstream connections (e.g. `http://proxy:8080`) | - |
| `LICENSE_PROXY_PROXY_USERNAME` | Username for proxy authentication | - |
| `LICENSE_PROXY_PROXY_PASSWORD` | Password for proxy authentication | - |
| `LICENSE_PROXY_ACCEPT_ANY_CERT` | Accept any upstream HTTPS certificate (`true`/`false`). **Dev only.** | `false` |

### Volumes

| Path | Description |
|---|---|
| `/app/certs` | Mount SSL certificates here |
| `/usr/share/dbosoft/LicenseProxy` | Configuration and data directory (mount an `appsettings.json` here for file-based config) |

## Examples

### HTTP only

```bash
docker run -d \
  -p 5080:5080 \
  ghcr.io/dbosoft/license-proxy
```

### HTTPS with PFX certificate

```bash
docker run -d \
  -p 5443:5443 \
  -v ./certs:/app/certs:ro \
  -e LICENSE_PROXY_URLS="https://+:5443" \
  -e LICENSE_PROXY_CERT_FILE=/app/certs/server.pfx \
  -e LICENSE_PROXY_CERT_PASSWORD=changeit \
  ghcr.io/dbosoft/license-proxy
```

### Behind a corporate proxy with custom CA

```bash
docker run -d \
  -p 5080:5080 \
  -v ./certs:/app/certs:ro \
  -e LICENSE_PROXY_PROXY_ADDRESS=http://proxy.corp:8080 \
  -e LICENSE_PROXY_UPSTREAM_CA_FILE=/app/certs/corp-ca.pem \
  ghcr.io/dbosoft/license-proxy
```

### File-based configuration

Mount an `appsettings.json` into the data directory:

```bash
docker run -d \
  -p 5080:5080 \
  -v ./config:/usr/share/dbosoft/LicenseProxy:ro \
  ghcr.io/dbosoft/license-proxy
```

Example `appsettings.json`:

```json
{
  "LicenseProxy": {
    "ProxyAddress": "http://proxy.corp:8080"
  },
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://+:5080"
      }
    }
  }
}
```

### Docker Compose

```yaml
services:
  license-proxy:
    image: ghcr.io/dbosoft/license-proxy
    ports:
      - "5080:5080"
    volumes:
      - proxy-data:/usr/share/dbosoft/LicenseProxy
    restart: unless-stopped

volumes:
  proxy-data:
```

## Health Check

The proxy exposes a health endpoint at `/health` that returns JSON with upstream connectivity status.

```bash
curl http://localhost:5080/health
```

## Tags

- `latest` - latest stable release
- `x.y.z` - specific version (e.g. `1.2.0`)
- `x.y` - latest patch of a minor version (e.g. `1.2`)
