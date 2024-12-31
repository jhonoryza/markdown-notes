# Self-Hosted Media Server with Jellyfin

This guide will walk you through setting up a self-hosted media server using
Jellyfin and Docker. We will use Docker Compose to simplify the setup process.

## Prerequisites

- Docker installed on your system
- Docker Compose installed on your system

## Step 1: Create Docker Compose Configuration

Create a file named `docker-compose.yml` with the following content:

```yaml
volumes:
    config:
    cache:

services:
    jellyfin:
        image: jellyfin/jellyfin
        container_name: jellyfin
        user: 1000:1000
        volumes:
            - config:/config
            - cache:/cache
            - /path/to/your/media:/media
        restart: "unless-stopped"
        environment:
            - JELLYFIN_PublishedServerUrl=http://yourdomain.com
        ports:
            - 8096:8096
            - 8920:8920 # optional
            - 7359:7359/udp # optional
            - 1900:1900/udp # optional
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.jellyfin.entrypoints=web"
            - "traefik.http.routers.jellyfin.rule=Host(`media.yourdomain.com`)"
            - "traefik.http.services.jellyfin.loadbalancer.server.port=8096"
```

Replace `/path/to/your/media` with the actual path to your media files and
`yourdomain.com` with your actual domain.

## Step 2: Start Jellyfin

Navigate to the directory containing your `docker-compose.yml` file and run the
following command to start Jellyfin:

```bash
docker-compose up -d
```

This command will download the Jellyfin image and start the container in
detached mode.

## Step 3: Access Jellyfin

Open your web browser and navigate to `http://yourdomain.com:8096` (replace
`yourdomain.com` with your actual domain). You should see the Jellyfin setup
screen.

## Step 4: Setup Jellyfin

Follow the on-screen instructions to complete the Jellyfin setup. You will need
to create an admin account and configure basic settings.

## Step 5: Add Media to Jellyfin

Once you have completed the setup, you can add media to your Jellyfin library:

1. Log in to your Jellyfin server using the admin account you created.
2. Go to the "Dashboard" from the menu.
3. Navigate to "Libraries" and click "Add Media Library".
4. Choose the type of media (e.g., Movies, TV Shows, Music).
5. Enter a display name for the library.
6. Click "Folders" and browse to the `/media` directory (or the path you
   specified in the Docker Compose file).
7. Click "OK" and then "Save".

Jellyfin will now scan the specified directory and add your media files to the
library.

## Conclusion

You have successfully set up a self-hosted media server using Jellyfin and
Docker. You can now enjoy your media collection from any device with a web
browser.

## Notes

You can force the setupwizard to show up by going to the following link:
[http://localhost:8096/web/index.html#!/wizardstart.html](http://localhost:8096/web/index.html#!/wizardstart.html)
