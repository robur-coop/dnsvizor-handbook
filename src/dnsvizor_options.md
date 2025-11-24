# DNSvizor options

The options we pass to DNSvizor can be put into three overall categories:
- [Mirage options](#mirage-options),
- [DNSvizor options](#dnsvizor-options),

All options with a description can be listed by running DNSvizor with `--help` as argument.
The options are then printed to stdout.
Note that you still need to pass a network device to `solo5-hvt`.

## Mirage options
These options relate to [MirageOS][mirage], the library operating system used to build DNSvizor.
Important options are `--help`, network options and log options:

### Mirage network options
Notable options are:
- `--ipv4=<PREFIX>`
- `--ipv4-gateway=<IP>`
- `--ipv4-only={true|false}`
There are as well IPv6 options.
See the `--help` output for more information.

## DNSvizor options
DNSvizor strives for a degree of compatibility with Dnsmasq.
Thus most options are Dnsmasq-compatible.
See the [Dnsmasq Compatibility Guide](./dnsvizor_dnsmasq_compatibility.md) for information on those.
The non-dnsmasq-compatible options are:

- `--name=<name>` which would be equivalent of the hostname of a Dnsmasq setup - that is, the hostname you would find in /etc/hostname on a \*NIX system.
- `--https-port=<port>` the port for the DNSvizor https web interface.
- `--ca-seed=<base64-seed>` is the seed to generate the self-signed certificate from for the DNSvizor web interface.
- `--dns-block=<hostname>` adds `hostname` to the DNS block list.
- `--dns-blocklist-url=<url>` adds `url` to the list of DNS block list sources to fetch.
- `--dns-cache=<size>` the size of the DNS cache.

## Advanced Configuration

Beyond the basic setup, DNSvizor supports advanced features for privacy, upstreaming dns queries, and security.

### DNS Resolution Modes

By default, DNSvizor acts as a **recursive resolver**. However, you can configure it to act as a **stub resolver** (forwarder).

- `--dns-upstream=<IP>`: If specified, DNSvizor disables its internal recursive resolver and forwards all queries to the specified upstream IP address.

### Privacy and Standards

DNSvizor implements modern RFCs to enhance privacy and security:

- `--qname-minimisation`: Enables QNAME minimisation [RFC 9156](https://www.rfc-editor.org/rfc/rfc9156.html). This improves privacy by sending only the minimal necessary data to upstream authoritative nameservers.
- `--opportunistic-tls-authoritative`: Enables opportunistic TLS when contacting authoritative nameservers [RFC 9539](https://www.rfc-editor.org/rfc/rfc9539.html), encrypting traffic where supported.

### TLS and Certificates

The unikernel generates its own CA and certificates for HTTPS and DoH.

- `--no-tls`: This disables web interface and dns-over-https.

### Customizing the CA Seed
The `--ca-seed` argument determines the private key generation. It accepts a base64-encoded string, optionally prefixed to specify the key type:

* **RSA (Default):** `rsa:4096:BASE64_SEED` or just `BASE64_SEED` (defaults to 4096 bits).
* **Ed25519:** `ed25519:BASE64_SEED`.

To generate a random base64-encoded 32-byte seed once and reuse it you can use `urandom`:

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
