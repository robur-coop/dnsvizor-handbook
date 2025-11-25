# Dnsmasq Compatibility Guide

DNSvizor tries to parse `dnsmasq` style configuration files and arguments. However, not all options are fully implemented. This section details which options are safe to use. If you use Dnsmasq and are missing any options in DNSvizor please [reach out to us][https://robur.coop/Contact].

## Supported Options
These options are fully implemented and affect the unikernel's behavior:

* `--dhcp-range`: Sets the IP range for the DHCP server.
    *Note:* Currently supports `start,end,lease_time` or `start,lease_time` formats.
* `--domain`: Sets the local domain name for the network.
* `--no-hosts`: Prevents reading local hostnames (only uses the `--name` argument).
* `--dnssec`: Enables DNSSEC validation.
* `--dhcp-host`: Adds a static host entry.
* `--dhcp-option`: Adds custom DHCP options, currently only "log-server" is supported.

## Ignored Options
The following options are parsed (so they won't break your existing config scripts) but are explicitly **ignored** by DNSvizor. They have no effect on the unikernel:

* `--interface`
* `--except-interface`
* `--listen-address`
* `--bind-interfaces`
* `--no-dhcp-interface`
