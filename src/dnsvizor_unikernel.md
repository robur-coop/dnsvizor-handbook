The DNSvizor unikernel provides a recursive DNS resolver, and a DHCP server. Its
configuration is close to dnsmasq.

## Installation

You can download the unikernel binary from [our reproducible build infrastructure](https://builds.robur.coop/job/dnsvizor/build/latest). Download the `bin/dnsvizor.hvt` artifact.
You need as well `solo5-hvt` which [our reproducible build infrastructure](https://builds.robur.coop/job/solo5) builds for select platforms.
If we don't build for your platform you need to [build it yourself](#building-solo5-from-source-alternative).
If you did all of that, skip to ["DNSvizor Configuration"](#dnsvizor-configuration).

## Building from source (alternative)

Here we document how to build the unikernel from source.

### Prerequisites

First, make sure to have ["opam"](https://opam.ocaml.org) and
["mirage"](https://mirage.io) installed on your system.

### Git repository

Do a `git clone https://github.com/robur-coop/dnsvizor.git` to retrieve the
source code of DNSvizor.

### Building

Inside of the cloned repository, execute `mirage configure` (other targets are
available, please check the mirage documentation):

```sh
$ cd dnsvizor/mirage
$ mirage configure -t hvt
$ make
```

The result is a binary, `dist/dnsvizor.hvt`, which we will use later.

## Building solo5 from source (alternative)

See the instructions in [doc/building.md](https://github.com/Solo5/solo5/blob/master/docs/building.md) in the Solo5 source tree.

## DNSvizor Configuration

The configuration is passed via command-line arguments. If you're lacking
configuration options here, please open an issue at the
[dnsvizor](https://github.com/robur-coop/dnsvizor) repository.

### Network configuration for the unikernel

All you need is a tap interface to run the unikernel on. You also need your
unikernel to be reachable from the outside (on the listening port), and be able
to communicate to the outside. There are multiple approaches to achieve this,
we will focus on setting up your firewall for this:

```sh
$ sysctl -w net.ipv4.ip_forward=1
$ ip route list default | cut -d' ' -f5
eth0

# allow the server to communicate to the outside
$ sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE

# redirect port 80, 443 to the unikernel
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT \
  --to-destination 10.0.0.2:80
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT \
  --to-destination 10.0.0.2:443
```

Setting up the network interface:

```sh
$ sudo ip tuntap add mode tap tap0
$ sudo ip link set dev tap0 up
$ sudo ip addr add tap0 10.0.0.1/24
```

We're all set now: the unikernel is allowed to communicate to the outside,
ports 80 and 443 are forwarded to the unikernel IP address, and a tap0 interface
exists where the host system has the IP address 10.0.0.1 configured.

## Launching DNSvizor

To launch the unikernel, you need a solo5 tender (that the Building section
already installed).

```sh
$ solo5-hvt --mem=96--net:service=tap0 --net-mac:service=00:80:41:ad:30:5e -- \
    dnsvizor.hvt --ipv4=10.0.0.2/24 --ipv4-gateway=10.0.0.1 --name=dnsvizor \
    --ca-seed=my-random-seed --password='my password'
```

### Solo5-hvt options

The solo5-hvt arguments follow the overall pattern `$ solo5-hvt <solo5-options> -- <kernel> <dnsvizor-arguments>`.
The relevant solo5-options are:
- `--mem 96` which allocates 96 MB of memory to dnsvizor. It can be omitted with a default allocation of 512 MB.
- `--net:service=tap0` tells `solo5-hvt` to use the TAP interface `tap0` for the unikernel network `service`. This is required for DNSvizor as it expects exactly one network named `service`.
- `--net-mac:service=00:80:41:ad:30:5e` tells `solo5-hvt` to assign the MAC address `00:80:41:ad:30:5e` to the unikernel network `service`. This is optional; if omitted a random MAC address is generated. Note that this may cause issues with ARP tables in the network.

### DNSvizor options

In the above example DNSvizor gets the arguments `--ipv4`, `--ipv4-gateway`, `--name`, `--ca-seed` and `--password`.
For more information about DNSvizor arguments see [DNSvizor options](./dnsvizor_options.md).
