# Pritunl-Traefik

[Pritunl in docker](https://github.com/jippi/docker-pritunl) compose with embedded MongoDB behind [Traefik v2](https://github.com/Bald1nh0/Traefik-v2-docker-compose)

## Setup

### Run this commands before docker compose up -d

```sh
data_dir=$(pwd)/data

mkdir -p $(data_dir)/pritunl $(data_dir)/mongodb
touch $(data_dir)/pritunl.conf
```
and then the following `docker-compose.yaml` file in `$(pwd)` followed by `docker-compose up -d`

### Docker compose file

I would recommend using a Docker `volume` or `bind` mount for persistent data like shown in the examples below:

```yaml
version: '3.9'

networks:
  web:
    external: true

services:

  pritunl:
    container_name: pritunl
    image: ghcr.io/jippi/docker-pritunl
    restart: always
    privileged: true
    ports:
      - "15173:15173/udp"
    volumes:
      - './data/pritunl.conf:/etc/pritunl.conf'
      - './data/pritunl:/var/lib/pritunl'
      - './data/mongodb:/var/lib/mongodb'
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=web"
      - "traefik.http.services.pritunl-service.loadbalancer.server.port=80"
      - "traefik.http.routers.pritunl-http.entrypoints=http"
      - "traefik.http.routers.pritunl-http.service=pritunl-service"
      - "traefik.http.routers.pritunl-http.rule=Host(`${FQDN_VALUE}`)"
      - "traefik.http.routers.pritunl-https.tls.certresolver=letsencryptresolver"
      - "traefik.http.middlewares.pritunl-redirect.redirectscheme.scheme=https"
      - "traefik.http.routers.pritunl-http.middlewares=pritunl-redirect"
      - "traefik.http.routers.pritunl-https.entrypoints=https"
      - "traefik.http.routers.pritunl-https.tls=true"
      - "traefik.http.routers.pritunl-https.rule=Host(`${FQDN_VALUE}`)"
      - "traefik.http.routers.pritunl-https.service=pritunl-service"
    networks:
      - web
```
Change ports if necessary.

### To [disable](https://docs.pritunl.com/docs/load-balancing) redirect to `https` in pritunl:

```bash
docker exec -it [container_name] pritunl set app.reverse_proxy true
docker exec -it [container_name] pritunl set app.redirect_server false
docker exec -it [container_name] pritunl set app.server_ssl false
docker exec -it [container_name] pritunl set app.server_port 80
```

Ex:

```bash
docker exec -it pritunl pritunl set app.reverse_proxy true
docker exec -it pritunl pritunl set app.redirect_server false
docker exec -it pritunl pritunl set app.server_ssl false
docker exec -it pritunl pritunl set app.server_port 80
```

## Default user and password

Run the following command to obtain the default login username and password:

```sh
docker exec -it [container_name] pritunl default-password
```

Ex:

```sh
docker exec -it pritunl pritunl default-password
```

## Further help and docs

For any help specific to Pritunl please have a look at http://pritunl.com and https://github.com/pritunl/pritunl