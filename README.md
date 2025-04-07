# Docker DNS-gen

dns-gen sets up a container running Dnsmasq and [docker-gen].
docker-gen generates a configuration for Dnsmasq and reloads it when containers are
started and stopped.

It will provide thoses domain:
- `container_name.docker`
- `container_name.network_name.docker`
- `docker-composer_service.docker-composer_project.docker`
- `docker-composer_service.docker-composer_project.network_name.docker`

## How it works

The container `dns-gen` expose a standard dnsmasq service. It returns IPs of
known containers. It does NOT resolve any other addresses. It is meant to
supplement an existing local DNS server, such as AdGuard, PiHole, or your router.
Use it to resolve container IPs from other containers/hosts on the same network.
I use it so my edge router reverse proxy can find my externally-exposed services
across hosts by container name instead of manually tracked IPs. It is intended to
be bound to its own IP (preferably static) on a network using ipvlan or macvlan.
