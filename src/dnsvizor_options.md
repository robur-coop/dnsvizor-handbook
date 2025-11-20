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
