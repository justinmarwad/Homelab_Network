
# Homelab Network Setup #  

This is a collection of docker compose services I use in my homelab. 

## Services 

These are the current services in the docker-compose. All you need to do is change domains/ips/etc and set a .env file with the required environment variables.

### Cloudflare DDNS

DDNS stands for dynamic DNS. Every time your variable public IP changes, this container will update an A record on Cloudflare. This essentially gives you all the benefits of a static IP without any of the cost. 

See for more information: https://github.com/oznu/docker-cloudflare-ddns/. 

### Portainer

This is just to be able to view all the containers on a nice looking webpage. Portainer is going to be a very helpful visualizer when I starting running containers on a Docker Swarm. 

### Caddy Docker Proxy

Caddy itself is a cool proxy that automatically integrates Letsencrypt to ensure that all your HTTP proxies are SSL-encrypted. Caddy Docker Proxy moves this entire process into a Docker container and also allows you to set labels on each Docker container to automatically setup the HTTPS proxy without needing to edit the Caddy file yourself. 

See more information here: https://github.com/lucaslorentz/caddy-docker-proxy. 

Make sure you created the the docker network "caddy": `docker network create caddy`. 

Once you have the original container setup, you can use this in other docker-compose configs to setup an HTTPS proxy connection to the greater internet: 

```yaml 
version: '3.7'
services:
  whoami:
    image: containous/whoami
    networks:
      - caddy
    labels:
      caddy: whoami.example.com
      caddy.reverse_proxy: "{{upstreams 80}}"

networks:
  caddy:
    external: true
```

Here's an example of using it with Swarm, the only difference is that you define the label at the service level: 

```yaml
services:
  foo:
    deploy:
      labels:
        caddy: service.example.com
        caddy.reverse_proxy: {{upstreams}}
``` 

### TODO: DNSmasq 

This is a placeholder as I have not yet implemented this service yet. The idea is to have an internal DNS running that all hosts use so that they know who each other are. 

Here's an example config that I currently have here (https://github.com/justinmarwad/CyberDefense): 

```yaml
  dnsmasq:
    container_name: dnsmasq
    image: tschaffter/dnsmasq:2.85    
    restart: unless-stopped
    volumes:
      - ./config/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf:ro
      - ./config/dnsmasq/my.exampledomain.com.conf:/etc/dnsmasq.d/my.exampledomain.com.conf:ro
    command: ["dnsmasq", "-k", "--server", "1.1.1.1", "--server", "1.0.0.1"]
    ports:
     - "53:53/udp"
     - "53:53/tcp"
```

### TODO: WireGuard

This is also a placeholder. WireGuard is a VPN that enables remote access to a network. Here's an example I have located here (https://github.com/justinmarwad/CyberDefense): 

```yaml
  wireguard:
    container_name: wireguard
    image: lscr.io/linuxserver/wireguard:v1.0.20210914-ls75
    restart: unless-stopped

    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    env_file:
      - ./config/wireguard/common.env
      - ./config/wireguard/secret.env
    environment:
      - SERVERURL=example.com
      - SERVERPORT=51820
      - PEERS=mattLaptop,mattPhone
    networks:
      - wireguard
    ports:
      - 51820:51820/udp
    volumes:
      - ./config/wireguard/modules:/lib/modules
      - ./config//wireguard/wireguard-config:/config
```
