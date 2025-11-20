# Web Interface & API

DNSvizor exposes a web-based management interface and a DNS-over-HTTPS (DoH) resolver.

## Authentication
Access to sensitive endpoints (like the blocklist or configuration) requires authentication.

- **Password Setup:** You must pass the `--password` argument at startup. If omitted, endpoints requiring authentication will not be accessible.
- **Method:** The interface uses [HTTP Basic Auth](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication).

![Dnsvizor web authentication](../images/auth.png)

Any username will work, but the password must the be same password you used to startup the unikernel.

## Dashboard Endpoints

The web server listens on the configured HTTPS port (default 443).

- `/` or `/dashboard` 
    - **method**: `GET` 
    - **description**: Displays statistics such as query rates etc. 
    - **authenticated**: false

![Dnsvizor dashboard web interface](../images/dashboard.png)

- `/querylog` 
    - **method**: `GET`
    - **description**: Displays a log of recent DNS queries processed by the resolver. 
    - **authenticated**: false

![Dnsvizor qeury log web interface](../images/query_log.png)

- `/configuration` 
    - **method**: `GET`
    - **description**: Displays the current configuration state. 
    - **authenticated**: true

- `/configuration/upload`
    - **method**: `POST` 
    - **description**: Allows uploading a new `dnsmasq` configuration file (multipart form data).
    - **authenticated**: true

![Dnsvizor configuration web interface](../images/config.png)

## Blocklist Management

DNSvizor allows managing blocked domains via the web interface. All endpoints related to the blocklist require [authentication](#authentication)

![Dnsvizor blocklist web interface](../images/blocklist.png)

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/blocklist` | `GET` | Lists all currently blocked domains. |
| `/blocklist/add` | `POST` | Adds a domain to the blocklist. Expects multipart form data with field `domain`. |
| `/blocklist/delete/<domain>`| `POST` | Removes `<domain>` from the blocklist. |
| `/blocklist/update` | `POST` | Triggers an immediate update of the blocklists from the configured URLs. |

## DNS-over-HTTPS (DoH)

The unikernel implements a compliant DNS-over-HTTPS resolver [RFC 8484](https://www.rfc-editor.org/rfc/rfc8484.html).

- **Endpoint:** `/dns-query`
- **Supported Methods:**
    - `GET`: Expects a base64url-encoded DNS query in the `?dns=` query parameter.
    - `POST`: Expects the raw DNS query in the body with `content-type: application/dns-message`.