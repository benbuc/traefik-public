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