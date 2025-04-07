# docker-dns-gen


## How it works

docker-dns-gen sets up a container running [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) and [docker-gen](https://github.com/nginx-proxy/docker-gen).
docker-gen generates a configuration for dnsmasq and reloads it when containers
are started and stopped. dnsmasq resolves only these containers, to all ifaces.

You must include a volume mapping /var/run/docker.sock into the container in the
same spot. It may (should) be read-only.

If, like me, you have multiple docker hosts whose containers must resolve, pass
NEXT_DNS environment variable to container. Do not create a loop by passing your
last resolver to your first resolver.

It will provide thoses domain:
- `container_name.docker`
- `container_name.network_name.docker`
- `docker-composer_service.docker-composer_project.docker`
- `docker-composer_service.docker-composer_project.network_name.docker`


## Rationale

It is meant to supplement an existing local DNS server, such as AdGuard, PiHole,
or your router. Use it to resolve container IPs from other containers/hosts on the
same network.

Example: You have two docker hosts on your LAN, and all of the containers on these
hosts is on the same VLAN network bound into Docker which we'll call docker-vlan,
via ipvlan or macvlan. Say, Sonarr/Radarr/Emby/Sabnzbd/Tdarr on one host, and Home
Assistant and Mealie on another host. Caddy or other reverse proxies run on a third
host would need static IP entries for every container to reach them. Adding a
docker-dns-gen container to each host and adding it to your DNS list will enable
container lookup by name instead.

I understand that probably nobody else will use this. I needed it.
If you also need it, feel free to thank me somehow, I'll appreciate it.
