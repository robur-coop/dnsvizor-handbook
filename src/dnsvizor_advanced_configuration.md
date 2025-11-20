# Advanced Configuration

Beyond the basic setup, DNSvizor supports advanced features for privacy, upstream integration, and security.

## DNS Resolution Modes

By default, DNSvizor acts as a **recursive resolver**. However, you can configure it to act as a **stub resolver** (forwarder).

- `--dns-upstream=<IP>`: If specified, DNSvizor disables its internal recursive resolver and forwards all queries to the specified upstream IP address.

## Privacy and Standards

DNSvizor implements modern RFCs to enhance privacy and security:

- `--qname-minimisation`: Enables QNAME minimisation [RFC 9156](https://www.rfc-editor.org/rfc/rfc9156.html). This improves privacy by sending only the minimal necessary data to upstream authoritative nameservers.
- `--opportunistic-tls-authoritative`: Enables opportunistic TLS when contacting authoritative nameservers [RFC 9539](https://www.rfc-editor.org/rfc/rfc9539.html), encrypting traffic where supported.

## TLS and Certificates

The unikernel generates its own CA and certificates for HTTPS and DoH.

- `--no-tls`: **Critical**. Disables TLS entirely. This turns off the HTTPS management server, DNS-over-TLS, and DNS-over-HTTPS. Use this only if running behind a secure reverse proxy or in a trusted environment.

### Customizing the CA Seed
The `--ca-seed` argument determines the private key generation. It accepts a base64-encoded string, optionally prefixed to specify the key type:

* **RSA (Default):** `rsa:4096:BASE64_SEED` or just `BASE64_SEED` (defaults to 4096 bits).
* **Ed25519:** `ed25519:BASE64_SEED`.

To generate a random base64-encoded 32-byte seed once and reuse it you can use `urandom`:

    ```bash
    # Generate a 32-byte seed encoded in base64

    dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64
    ```

## Dynamic Updates (RFC 2136)

DNSvizor can update an external authoritative nameserver when a local DHCP client acquires a lease.

- `--dns-server=<IP>`: The IP address of the external authoritative nameserver to update.
- `--dns-key=<HOST:HASH:DATA>`: The TSIG key required to authenticate the update.
    * **Format:** `name:hash_alg:base64_secret` (e.g., `mykey:sha256:SGVsbG8=`).

When configured, if a DHCP client requests a hostname (via FQDN option), DNSvizor will send a signed update packet to the external server.