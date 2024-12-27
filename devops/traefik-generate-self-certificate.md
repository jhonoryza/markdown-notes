## Traefik Using Owned Certificate

Example docker-compose.yml Configuration for Traefik with SSL Certificate
Below is an example of a docker-compose.yml configuration for Traefik that uses your existing SSL certificate. In this example, we will utilize pre-existing certificate and private key files.

Create the docker-compose.yml File

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v2.9
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./certs:/certs
      - /var/run/docker.sock:/var/run/docker.sock
    command:
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --providers.docker=true
      - --providers.docker.network=web
      - --api.dashboard=true
      - --log.level=INFO
      - --certificatesresolvers.myresolver.acme.tlschallenge=true
      - --certificatesresolvers.myresolver.acme.email=your-email@example.com
      - --certificatesresolvers.myresolver.acme.storage=/acme.json
      - --tls.certificates.0.certfile=/certs/your-certificate.crt
      - --tls.certificates.0.keyfile=/certs/your-private-key.key
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`traefik.yourdomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.tls.certresolver=myresolver"
    networks:
      - web

networks:
  web:
    external: false
```

Explanation

1. command:

* --entrypoints.web.address=:80: Defines the HTTP entrypoint on port 80.
* --entrypoints.websecure.address=:443: Defines the HTTPS entrypoint on port 443.
* --providers.docker=true: Enables Docker as the service provider.
* --providers.docker.network=web: Uses the web network for Docker services.
* --api.dashboard=true: Enables the Traefik dashboard.
* --log.level=INFO: Sets the log level to INFO.
* --certificatesresolvers.myresolver.acme.tlschallenge=true: Enables ACME with the TLS-ALPN-01 challenge (optional, can be commented out if not used).
* --certificatesresolvers.myresolver.acme.email=your-email@example.com: Email for ACME (optional, can be commented out if not used).
* --certificatesresolvers.myresolver.acme.storage=/acme.json: Specifies the ACME storage location (optional, can be commented out if not used).
* --tls.certificates.0.certfile=/certs/your-certificate.crt: Path to your certificate file.
* --tls.certificates.0.keyfile=/certs/your-private-key.key: Path to your private key file.

2. labels:

* traefik.enable=true: Enables Traefik for this service.
* traefik.http.routers.api.rule=Host(traefik.yourdomain.com): Defines a rule for the API router.
* traefik.http.routers.api.service=api@internal: Directs the API router to Traefik's internal service.
* traefik.http.routers.api.tls=true: Enables TLS for the API router.
* traefik.http.routers.api.tls.certresolver=myresolver: Uses the defined certificate resolver.

Ensure Your Folder Structure is as Follows:

```
.
├── docker-compose.yml
└── certs
    ├── your-certificate.crt
    └── your-private-key.key
```

Start Traefik
Once you have all the required files, start Traefik using the following command:

```
docker-compose up -d
```

Traefik will now run and use your SSL certificate with the configuration provided through the command and labels.