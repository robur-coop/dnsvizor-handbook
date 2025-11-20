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

- You can upload a Dnsmasq configuration file (txt file), and the contents of this file will be automatically loaded in the textarea where you can make modifications if necessary. When you are satisfied with the configuration, you can click on the "Save configuration" button to effect the changes.

![Dnsvizor configuration web interface](../images/config.png)

## Blocklist Management

DNSvizor allows managing blocked domains via the web interface. All endpoints related to the blocklist require [authentication](#authentication)

![Dnsvizor blocklist web interface](../images/blocklist.png)

The block list page displays information about blocked domains.

- At the top are the domains "manually" blocked either through boot arguments (see [DNSvizor options](./dnsvizor_options.md)) or via the web interface.

- Each blocked domain can be unblocked by the "delete" button.

- Below that is another list of remote block lists. There you find what block lists are used as well as the number of domains blocked through that block list source.

- Finally, at the top you find a text input field where you can type in domains you want to block.

- Next to the input field is a refresh button ("⟳") that can be clicked to refresh the remote domain block lists. This operation may take some time.

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