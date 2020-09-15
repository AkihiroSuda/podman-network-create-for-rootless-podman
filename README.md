# UPDATE: This work has been merged to [Podman 2.1](https://github.com/containers/podman/pull/7460)

---

# `podman network create` for Rootless Podman

POC scripts for allowing an equivalent of `podman network create` with Rootless Podman.

## Usage

Prepare a sandbox container:
```console
$ podman build -t rootless-podman-network-sandbox rootless-podman-network-sandbox

$ podman run -d --name rootless-podman-network-sandbox --pid=host --privileged -v $(pwd)/etc_cni_net.d:/etc/cni/net.d:ro rootless-podman-network-sandbox
```

Create "apple" container with the network "foo" (10.88.2.0/24, defined in [`etc_cni_net.d/foo.conflist`](./etc_cni_net.d/foo.conflist)):
```console
$ podman exec rootless-podman-network-sandbox /alloc apple foo
{"ns":"/proc/46797/ns/net","dns":"10.88.2.1"}

$ podman run -it --network=ns:/proc/46797/ns/net --dns 10.88.2.1 --hostname apple --name apple alpine
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether ae:9b:c5:96:d4:84 brd ff:ff:ff:ff:ff:ff
    inet 10.88.2.2/24 brd 10.88.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ac9b:c5ff:fe96:d484/64 scope link 
       valid_lft forever preferred_lft forever
```

Create "banana" container with the same network:
```console
$ podman exec rootless-podman-network-sandbox /alloc banana foo
{"ns":"/proc/46902/ns/net","dns":"10.88.2.1"}

$ podman run -it --network=ns:/proc/46902/ns/net --dns 10.88.2.1 --hostname banana --name banana alpine
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 3e:91:ae:1a:52:cb brd ff:ff:ff:ff:ff:ff
    inet 10.88.2.3/24 brd 10.88.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::3c91:aeff:fe1a:52cb/64 scope link 
       valid_lft forever preferred_lft forever
/ # ping -c 1 apple.dns.podman
PING apple.dns.podman (10.88.2.2): 56 data bytes
64 bytes from 10.88.2.2: seq=0 ttl=64 time=0.053 ms

--- apple.dns.podman ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.053/0.053/0.053 ms
```

## Q. What about Rootless Docker?

[Rootless Docker](https://docs.docker.com/engine/security/rootless/) provides built-in support for `docker network create` since the beginning.
No configuration is needed, just run `docker network create`.
