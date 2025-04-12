# docker-dns-gen


Docker proves intra-host DNS resolution by itself. You can reach your docker
container from another container simply by using its container name as hostname
when they're on the same network in almost every case. I needed container-to-IP
resolution from OUTSIDE the Docker host. This is a very niche requirement, and
thus a niche container package, so I'm expecting if you've found it, you know
what you're doing. If you need guidance, ask. If you appreciate my work, let
me know.


## How it works

docker-dns-gen sets up a container running [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) and [docker-gen](https://github.com/nginx-proxy/docker-gen).
docker-gen generates a configuration for dnsmasq and reloads it when containers
are started and stopped. dnsmasq resolves only these containers, to all ifaces,
optionally using a configured upstream DNS.

You must either:
- include a volume mapping /var/run/docker.sock into the container in the
same spot. It may (should) be read-only.
- provide a DOCKER_HOST environment variable set to an appropriate value
to reach your Docker host, ie exposed TCP port

If, like me, you have multiple docker hosts whose containers must resolve, pass
NEXT_DNS environment variable to container. Do not create a loop by passing your
last resolver to your first resolver. If not provided NEXT_DNS the container will
consider itself authoratative and NXDOMAIN unmatched requests. If you wish to use
this as your primary DNS server to resolve all hosts, pass your upstream DNS as
NEXT_DNS. This is not an intended use of the container but there's theoretically
nothing but configuration stopping you.

It will resolve these domains, adding an entry for each IP, on each network, when valid:
- `container_name.docker`
- `container_name.network_name.docker`
- `docker-composer_service.docker-composer_project.docker`
- `docker-composer_service.docker-composer_project.network_name.docker`

For example, a container named `nginx` on networks `lan` at 192.168.0.1, `internal_services` at 172.16.0.1, and
`external_services` at 172.16.1.1 will provide DNS entries for:
- `nginx.docker`  192.168.0.1, 172.16.0.1, 172.16.1.1
- `nginx.lan.docker`  192.168.0.1
- `nginx.internal_services.docker`  172.16.0.1
- `nginx.external_services.docker`  172.16.1.1


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
