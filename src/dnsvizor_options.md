# DNSvizor options

The options we pass to DNSvizor can be put into three overall categories:
- [MirageOS options](#mirageos-options),
- [DNSvizor options](#dnsvizor-options),
- [dnsmasq options](./dnsvizor_dnsmasq_compatibility.md)

All options with a description can be listed by running DNSvizor with `--help` as argument.
Note that you still need to pass a network device to `solo5-hvt`: `solo5-hvt --net:service=tapX -- dnsvizor.hvt --help`.

## MirageOS options
These options relate to [MirageOS][https://mirageos.org], the library operating system used to build DNSvizor.

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
DNSvizor strives for a degree of compatibility with [Dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html). See the [Dnsmasq Compatibility Guide](./dnsvizor_dnsmasq_compatibility.md) for details on those options.
Custom DNSvizor options include:

- `--hostname=<VALUE>`: which would be equivalent of the hostname of a Dnsmasq setup - that is, the hostname you would find in /etc/hostname on a \*NIX system.
- `--https-port=<port>`: the port for the DNSvizor https web interface.
- `--dns-cache=<size>`: the size of the DNS cache.
- `--password=<VAL>`: the password to authenticate for sensitive web interface endpoints (like configuration uploads or blocklist management). If not provided, these endpoints are not available.

### Blocklist
You can add specific hostnames to the DNS block list via the command line. You can also specify a URL to a blocklist:

- `--dns-block=<hostname>`: adds a specific `hostname` to the DNS block list. May be repeated.
- `--dns-blocklist-url=<url>`: retrieves `url` and uses it as a DNS block list.

### DNS resolver mode

DNSvizor is by default configured to do recursive DNS queries. It can be configured to ask a DNS resolver instead, and thus being a stub resolver:

- `--dns-upstream=<IP>`: DNS queries are forwarded to the specified upstream IP address.

### Privacy Features

DNSvizor implements modern RFCs to enhance privacy and security:

- `--qname-minimisation`: Enables QNAME minimisation [RFC 9156](https://www.rfc-editor.org/rfc/rfc9156.html). This improves privacy by sending only the minimal necessary data to upstream authoritative nameservers.
- `--opportunistic-tls-authoritative`: Enables opportunistic TLS when contacting authoritative nameservers [RFC 9539](https://www.rfc-editor.org/rfc/rfc9539.html).

### TLS support

For DNS-over-TLS, DNS-over-HTTP, and the web interface, DNSvizor generates its own CA and certificates. If you do not need these features, you can disable it with the option:

- `--no-tls`: This option disables the web interface, DoT, and DoH.

The `--ca-seed` argument determines the private key generation. It accepts a base64-encoded string, optionally prefixed to specify the key type:

* **RSA (Default):** `rsa:BITS:BASE64_SEED` or just `BASE64_SEED` (defaults to 4096 bits).
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
DNSvizor supports integration with [tlstunnel](https://github.com/robur-coop/tlstunnel) for updating SNI proxy rules dynamically.

- `--tlstunnel-server=<IP>`: The IP address of the TLSTUNNEL server.
- `--tlstunnel-key=<VAL>`: The shared secret used to authenticate updates with the TLSTUNNEL server.