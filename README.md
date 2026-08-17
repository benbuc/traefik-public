# Using Traefik as Reverse Proxy

Some configuration I regularly used to setup Traefik as a reverse proxy.
Strongly inspired by https://dockerswarm.rocks/traefik/

## Preparation

- Populate the `.env` file by copying and adjusting to your needs.

    ```bash
    cp .env.example .env
    ```

- Create the shared Traefik network
    
    ```bash
    docker network create traefik-public
    ```

## Start the Traefik Proxy
```bash
docker compose up -d
```

## Usage in other services
Now that Traefik is running, you can use it to route traffic to your services.

In the other service's `docker-compose.yml` file you need to add `labels` and `networks`.
The following is a sample configuration to expose a service on the domain `example.com`:

Note that, by default, all requests to HTTP will be redirected to HTTPS.
You would need to change this in the `traefik.yml` of the reverse proxy.
With the default configuration it is not possible to access `my-service` via HTTP.
You have to specify the TLS configuration for the router though.

```yaml
services:
    my-service:
        [... other configuration...]

        networks:
            - default
            - traefik-public
        labels:
            - traefik.enable=true
            - traefik.docker.network=traefik-public
            - traefik.constraint-label=traefik-public
            - traefik.http.routers.my-service.rule=Host(`example.com`)
            - traefik.http.routers.my-service.tls=true
            - traefik.http.routers.my-service.tls.certresolver=le

networks:
    traefik-public:
        external: true
```

## Monitoring (optional)

`docker-compose.monitoring.yml` is an optional mixin that adds Prometheus + Grafana,
pre-wired with a dashboard for Traefik's own metrics. Since Traefik exposes per-router
and per-service metrics for everything it proxies, this dashboard gives you visibility
into every connected app's traffic (request rate, latency, response codes) without any
changes to those apps' own configuration.

Traefik's metrics are served on an internal-only entrypoint (`:8082`, not published to
the host), reachable only from other containers on the `traefik-public` network.

- Add the Grafana settings to your `.env` (see `.env.example`):
    ```
    GRAFANA_DOMAIN=grafana.example.com
    GRAFANA_ADMIN_USER=admin
    GRAFANA_ADMIN_PASSWORD=changeme
    ```
- Start Traefik together with the monitoring stack:
    ```bash
    docker compose -f docker-compose.yml -f docker-compose.monitoring.yml up -d
    ```
- Open `https://<GRAFANA_DOMAIN>` and log in with the credentials above. The
  "Traefik Official Standalone Dashboard" is provisioned automatically — no manual
  datasource or dashboard import needed.