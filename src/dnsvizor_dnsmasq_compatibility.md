# Dnsmasq Compatibility Guide

DNSvizor tries to parse `dnsmasq` style configuration files and arguments. However, not all options are fully implemented. This section details which options are safe to use and which may cause the unikernel to exit.
If you use Dnsmasq and are missing any options in DNSvizor please [reach out to us][contact].

## Supported Options
These options are fully implemented and affect the unikernel's behavior:

* `--dhcp-range`: Sets the IP range for the DHCP server.
    *Note:* Currently supports `start,end,lease_time` or `start,lease_time` formats.
* `--domain`: Sets the local domain name for the network.
* `--no-hosts`: Prevents reading local hostnames (only uses the `--name` argument).
* `--dnssec`: Enables DNSSEC validation.

## Ignored Options
The following options are parsed (so they won't break your existing config scripts) but are explicitly **ignored** by DNSvizor. They have no effect on the unikernel:

* `--interface`
* `--except-interface`
* `--listen-address`
* `--bind-interfaces`
* `--no-dhcp-interface`

## Unsupported Options
The following options are recognized by the parser but are **not yet implemented**. Using these will currently result in an error or startup failure:

* `--dhcp-host`: Using this will trigger an "Unhandled" error during configuration processing.
* `--dhcp-option`: Most tags are currently unhandled. The parser specifically checks for `Log_servers`, but usage is experimental.

[contact]: https://robur.coop/Contact