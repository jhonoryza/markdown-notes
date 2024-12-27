# Traefik Cloudflare DNS Challenge

## Example Docker Compose Configuration

```
traefik:
  image: "traefik:2.3.7"
  container_name: "traefik"
  restart: unless-stopped
  ports:
    - "80:80"
    - "443:443"
    #- "8080:8080"
  environment:
    - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
  volumes:
    - ./traefik/config:/etc/traefik
    - traefik-ssl-certs:/ssl-certs
    - /etc/localtime:/etc/localtime
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
  depends_on:
    - app
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.traefik.entrypoints=websecure"
    - "traefik.http.routers.traefik.tls.certresolver=myresolver"
    - "traefik.http.routers.traefik.rule=Host(`traefik.domain.id`)"
    - "traefik.http.routers.traefik.service=api@internal"
```

Replace traefik.domain.id with your actual domain.

Create a .env file and add the following:

```
CF_DNS_API_TOKEN=
```

This token can be obtained from Cloudflare.

### Steps to Create an API Token in Cloudflare with the Required Permissions to Manage DNS for Your Domain

#### Step 1: Log In to Your Cloudflare Account

1. Visit Cloudflare and log in with your credentials.

#### Step 2: Access the API Tokens Settings

1. Once logged in, click on your avatar or account name in the top-right corner.
2. Select My Profile from the dropdown menu.
3. On the profile page, navigate to the API Tokens tab.

#### Step 3: Create a New API Token

1. Click the Create Token button.
2. Select the Edit zone DNS template or click Create Custom Token if you want to
   customize the permissions.

#### Langkah 4: Mengatur Izin API Token

If choosing Create Custom Token:

### Token Configuration Steps

1. **Token Name**:\
   Give your token a name, such as `Traefik DNS Challenge`.

2. **Permissions**:
   - Click **Add permissions**.
   - Choose **Zone** as the service.
   - Select **DNS** as the resource.
   - Choose **Edit** as the action.

3. **Zone Resources**:
   - Click **Add Zone Resources**.
   - Select **Include**.
   - Choose **All Zones** or **Specific Zone** to restrict access to a
     particular zone.
     - If choosing **Specific Zone**, specify the domain, e.g., `example.com`.

4. **Client IP Address Filtering (optional)**:\
   Add specific IP addresses if you want to restrict token usage to certain IPs.

5. **TTL (optional)**:\
   Set a token expiration period if needed.

### Step 5: Create and Save the API Token

1. Click **Continue to summary**.
2. Review the token settings to ensure they are correct.
3. Click **Create Token**.

---

### Step 6: Save the API Token

1. Once the token is created, copy and save it in a secure location.
   > **Note**: This is the only time the token will be displayed, so ensure you
   > record it properly.

Create a traefik/config/traefik.yaml file:

```yaml
global:
  checkNewVersion: false
  sendAnonymousUsage: false

api:
  dashboard: true
  debug: true

accessLog: {}

entryPoints:
  web:
    address: :80

  websecure:
    address: :443

serversTransport:
  insecureSkipVerify: true

http:
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true

certificatesResolvers:
  myresolver:
    acme:
      email: xxx@gmail.com
      storage: /ssl-certs/acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
        delayBeforeCheck: 5

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    directory: /etc/traefik
    watch: true
```

Replace xxx@gmail.com with the email you use for Cloudflare.

## Running Traefik

Run the following command to start Traefik:

```bash
docker compose up -d
```

Then, access the dashboard via http://traefik.domain.id or the domain you
configured earlier.

## Debugging the DNS Challenge Process

To debug the DNS challenge, inspect the file with the following command:

```bash
cat /ssl-certs/acme.json
```

However, you need to access the Traefik container first:

```bash
docker exec -it traefik sh
```
