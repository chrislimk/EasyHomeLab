# This container updates your public IP to Cloudflare

## Compose or portainer stack
```
version: '3.8'
services:
  cloudflareddns:
    image: oznu/cloudflare-ddns:latest
    deploy:
      placement:
        constraints:
          - "node.role==worker"

    environment:
    - API_KEY=<api key>
    - PUID=1000
    - PGID=1000
    - PROXIED=true
    - ZONE=<domain> #eg. yourdomain.com

    restart: always
```
