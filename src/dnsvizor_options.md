# DNSvizor options

The options we pass to DNSvizor can be put into two overall categories:
- [Mirage options](#mirage-options),
- [DNSvizor options](#dnsvizor-options),

All options with a description can be listed by running DNSvizor with `--help` as argument.
The options are then printed to stdout.
Note that you still need to pass a network device to `solo5-hvt`.

## Mirage options
These options relate to [MirageOS][mirage], the library operating system used to build DNSvizor.

### Network Options
These flags configure the underlying network interface and IP stack.

- `--ipv4=<PREFIX>`: The IPv4 network address and prefix length for the unikernel (e.g., `10.0.0.2/24`).
- `--ipv4-gateway=<IP>`: The IPv4 gateway address.
- `--ipv4-only=<BOOL>`: Restrict the unikernel to IPv4 only (default: `false`).
- `--ipv6=<PREFIX>`: The IPv6 network address and prefix length.
- `--ipv6-gateway=<IP>`: The IPv6 gateway address.
- `--ipv6-only=<BOOL>`: Restrict the unikernel to IPv6 only (default: `false`).
- `--accept-router-advertisements=<BOOL>`: Controls whether the unikernel accepts IPv6 Router Advertisements (default: `true`).

### Logging Options
- `-l <LEVEL>` or `--logs=<LEVEL>`: Set the log verbosity level.
    - **Levels:** `quiet`, `app`, `error`, `warning`, `info`, `debug`.

### Common Options
- `--name=<VAL>`: The **runtime name** of the unikernel. This is used primarily for syslog identity. If omitted, it defaults to the configuration-time name.
- `--delay=<INT>`: Delays the startup by `n` seconds (default: `0`).

## DNSvizor options
DNSvizor strives for a degree of compatibility with Dnsmasq. See the [Dnsmasq Compatibility Guide](./dnsvizor_dnsmasq_compatibility.md) for details on those options.
The non-dnsmasq-compatible options are:

- `--hostname=<VALUE>`: which would be equivalent of the hostname of a Dnsmasq setup - that is, the hostname you would find in /etc/hostname on a \*NIX system.
- `--https-port=<port>`: the port for the DNSvizor https web interface.
- `--ca-seed=<base64-seed>`: the seed used to generate the self-signed Certificate Authority (CA). See [Customizing the CA Seed](#customizing-the-ca-seed) below.
- `--dns-block=<hostname>`: adds a specific `hostname` to the DNS block list at startup.
- `--dns-blocklist-url=<url>`: adds `url` to the list of DNS block list sources to fetch.
- `--dns-cache=<size>`: the size of the DNS cache.
- `--password=<VAL>`: the password required to authenticate for sensitive web interface endpoints (like configuration uploads or blocklist management).

## Advanced Configuration

Beyond the basic setup, DNSvizor supports advanced features for privacy and security.

### DNS Resolution Modes

By default, DNSvizor acts as a **recursive resolver**. However, you can configure it to act as a **stub resolver** (forwarder).

- `--dns-upstream=<IP>`: If specified, DNSvizor disables its internal recursive resolver and forwards all queries to the specified upstream IP address.

### Privacy and Standards

DNSvizor implements modern RFCs to enhance privacy and security:

- `--qname-minimisation`: Enables QNAME minimisation [RFC 9156](https://www.rfc-editor.org/rfc/rfc9156.html). This improves privacy by sending only the minimal necessary data to upstream authoritative nameservers.
- `--opportunistic-tls-authoritative`: Enables opportunistic TLS when contacting authoritative nameservers [RFC 9539](https://www.rfc-editor.org/rfc/rfc9539.html), encrypting traffic where supported.

### TLS and Certificates

The unikernel generates its own CA and certificates for HTTPS and DoH.

- `--no-tls`: This option disables the web interface.
    * **Consequence:** The HTTPS management dashboard, the Blocklist API, and DNS-over-HTTPS endpoints will **not be started**.
    * **Usage:** Use this if you only require standard DNS and DHCP services and do not need the web dashboard.

### Customizing the CA Seed
The `--ca-seed` argument determines the private key generation. It accepts a base64-encoded string, optionally prefixed to specify the key type:

* **RSA (Default):** `rsa:4096:BASE64_SEED` or just `BASE64_SEED` (defaults to 4096 bits).
* **Ed25519:** `ed25519:BASE64_SEED`.

To generate a secure, random base64-encoded 32-byte seed once and reuse it, you can use `urandom`:

```bash
# Generate a 32-byte seed encoded in base64
dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64
```
### Dynamic Updates [RFC 2136](https://www.rfc-editor.org/rfc/rfc2136.html)

DNSvizor can update an external authoritative nameserver and requests a `client_fqdn` option [RFC 4702](https://datatracker.ietf.org/doc/html/rfc4702) when a local DHCP client acquires a lease.

- `--dns-server=<IP>`: The IP address of the external authoritative nameserver to update.
- `--dns-key=<HOST:HASH:DATA>`: The TSIG key required to authenticate the update.
    * **Format:** `name:hash_alg:base64_secret` (e.g., `mykey:sha256:SGVsbG8=`).

When configured, if a DHCP client requests a hostname (via FQDN option), DNSvizor sends a TSIG-signed Dynamic DNS update packet (defined in [RFC 2136](https://datatracker.ietf.org/doc/html/rfc2136)) to the external server.

### TLSTUNNEL Integration
DNSvizor supports integration with tlstunnel for updating SNI proxy rules dynamically.

- `--tlstunnel-server=<IP>`: The IP address of the TLSTUNNEL server.

- `--tlstunnel-key=<VAL>`: The shared secret used to authenticate updates with the TLSTUNNEL server.