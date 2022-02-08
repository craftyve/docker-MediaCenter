# docker-MediaCenter
<hr id="bkmrk-">
<h4 id="bkmrk-pr%C3%A9-requis-%3A">Pré-requis :</h4>
<p id="bkmrk-docker-d%27installer-s">docker d'installer sur votre machine.</p>
<hr id="bkmrk--0">
<h4 id="bkmrk-le-docker-compose-%3A">Le docker-compose :</h4>
<p id="bkmrk-ce-yaml-suivant-cont">Ce yaml suivant contient un ensemble de container pour votre solution de mediacenter :</p>
<ul id="bkmrk-deluge-vpn---%3E-clien">
<li style="list-style-type: none;">
<ul>
<li>Deluge VPN --&gt; client VPN</li>
<li>Jackett --&gt; un seul tracker pour tout vos tracker preferer</li>
<li>Netdata --&gt; Monitorer votre mediacenter</li>
<li>Plex --&gt; Solution de MediaCenter</li>
<li>Portainer --&gt; Container manadgement via webui</li>
<li>Radarr --&gt; automatiser la récuperation de vos films</li>
<li>Sonarr --&gt; automatiser la récuperation de vos series</li>
<li>Watchtower ---&gt; auto update vos containers</li>
</ul>
</li>
</ul>
<p id="bkmrk-avant-de-lancer-le-y">avant de lancer le yaml, veuiller remplacer les variables par vos chemins pour stocker vos datas ainsi que la variable ${IP_ADDRESS}: par rien dutout en laissant que les ports ! </p>
<p id="bkmrk-enregister-le-yaml-s">enregister le yaml suivant avec le nom suivant : docker-compose.yaml<textarea style="display: none;">##
</textarea></p>
<pre id="bkmrk-%23%23-%23%23---------------"><code class="language-YAML">##
## ------------------------------
## |   M E D I A  G O N E I X   |
## ------------------------------
##
version: '3.5'
services:
  # ----------------------------------------
  # DELUGEVPN
  # ----------------------------------------
  arch-delugevpn:
    image: binhex/arch-delugevpn
    container_name: delugevpn
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:8112:8112'
        - '${IP_ADDRESS}:8118:8118'
        - '${IP_ADDRESS}:58846:58846'
        - '${IP_ADDRESS}:58946:58946'
    cap_add:
        - NET_ADMIN
    environment:
        - VPN_ENABLED=yes
        - VPN_USER=${PIAUNAME}
        - VPN_PASS=${PIAPASS}
        - VPN_REMOTE=${VPN_REMOTE}
        - VPN_PORT=1198
        - VPN_PROTOCOL=udp
        - VPN_DEVICE_TYPE=tun
        - VPN_PROV=custom
        - STRONG_CERTS=no
        - ENABLE_PRIVOXY=yes
        - STRICT_PORT_FORWARD=yes
        - LAN_NETWORK=${CIDR_ADDRESS}
        - NAME_SERVERS=84.200.69.80,37.235.1.174,1.1.1.1,37.235.1.177,84.200.70.40,1.0.0.1
        - DEBUG=false
        - PUID=${PUID}
        - PGID=${PGID}
    volumes:
        - '${DLDIR}:/data'
        - './delugevpn/config:/config'
        - '/etc/localtime:/etc/localtime:ro'

  # ----------------------------------------
  # JACKETT
  # ----------------------------------------
  jackett:
    image: lscr.io/linuxserver/jackett
    container_name: jackett
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:9117:9117'
    environment:
        - PUID=${PUID}
        - PGID=${PGID}
    volumes:
        - './jackett:/config'
        - '${DLDIR}/completed:/downloads'
        - '/etc/localtime:/etc/localtime:ro'
 
  # ----------------------------------------
  # NETDATA
  # ----------------------------------------
  netdata:
    image: netdata/netdata:latest
    container_name: netdata
    hostname: '${HOSTNAME}'
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:19999:19999'
    cap_add:
        - SYS_PTRACE
    security_opt:
        - apparmor:unconfined
    environment:
        - PGID=${DOCKERGRP}
        - NETDATA_CLAIM_TOKEN=mOdmVDLrtzALoSgMioxMyvOdkaAYh4yVXkE6sq7qsy5GX15UJTWxyAHyKpbURbTJD6LYI0iBNR7J8-3HZ5TsMfZDeOYwBdb8aOgfu0vKoNcFvPRDKBwEPIhcm71LpUJyBMDtH7M
        - NETDATA_CLAIM_URL=https://app.netdata.cloud
        - NETDATA_CLAIM_ROOMS=c6bd4cfb-20cf-46ab-8707-41e344b76038
    volumes:
        - '/proc:/host/proc:ro'
        - '/sys:/host/sys:ro'
        - '/var/run/docker.sock:/var/run/docker.sock:rw'

 
  # ----------------------------------------
  # PLEX
  # ----------------------------------------
  plex:
    container_name: plex
    image: plexinc/pms-docker:${PMSTAG}
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:32400:32400/tcp'
        - '${IP_ADDRESS}:3005:3005/tcp'
        - '${IP_ADDRESS}:8324:8324/tcp'
        - '${IP_ADDRESS}:32469:32469/tcp'
        - '${IP_ADDRESS}:1900:1900/udp'
        - '${IP_ADDRESS}:32410:32410/udp'
        - '${IP_ADDRESS}:32412:32412/udp'
        - '${IP_ADDRESS}:32413:32413/udp'
        - '${IP_ADDRESS}:32414:32414/udp'
    environment:
        - PLEX_CLAIM=${PMSTOKEN}
        - ADVERTISE_IP=http://${IP_ADDRESS}:32400/
        - ALLOWED_NETWORKS=${CIDR_ADDRESS}
        - PLEX_UID=${PUID}
        - PLEX_GID=${PGID}
    hostname: ${HOSTNAME}
    volumes:
        - './plex:/config'
        - './plex/transcode:/transcode'
        - '${MISCDIR}:/data/misc'
        - '${MOVIEDIR}:/data/movies'
        - '${MUSICDIR}:/data/music'
        - '${TVDIR}:/data/tvshows'
        - '/etc/localtime:/etc/localtime:ro'

  # ----------------------------------------
  # PORTAINER
  # ----------------------------------------
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:9000:9000'
    environment:
        - PUID=${PUID}
        - PGID=${PGID}
    volumes:
        - './portainer:/data'
        - '/var/run/docker.sock:/var/run/docker.sock'
        - '/etc/localtime:/etc/localtime:ro'
    command: -H unix:///var/run/docker.sock

  # ----------------------------------------
  # RADARR
  # ----------------------------------------
  radarr:
    image: lscr.io/linuxserver/radarr
    container_name: radarr
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:7878:7878'
    environment:
        - PUID=${PUID}
        - PGID=${PGID}
    volumes:
        - './radarr:/config'
        - '${DLDIR}/completed:/data/completed'
        - '${MOVIEDIR}:/movies'
        - '/etc/localtime:/etc/localtime:ro'

  # ----------------------------------------
  # SONARR
  # ----------------------------------------
  sonarr:
    image: lscr.io/linuxserver/sonarr
    container_name: sonarr
    restart: unless-stopped
    network_mode: "bridge"
    ports:
        - '${IP_ADDRESS}:8989:8989'
    environment:
        - PUID=${PUID}
        - PGID=${PGID}
        - TZ=${TZ}
    volumes:
        - './sonarr:/config'
        - '${DLDIR}/completed:/data/completed'
        - '${TVDIR}:/tv'
        - '/etc/localtime:/etc/localtime:ro'
  # ----------------------------------------
  # WATCHTOWER
  # ----------------------------------------
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    network_mode: "bridge"
    hostname: '${HOSTNAME}'
    environment:
        - WATCHTOWER_CLEANUP=true
        - WATCHTOWER_SCHEDULE=0 0 */4 * * *
        - WATCHTOWER_INCLUDE_STOPPED=true
        - TZ=${TZ}
    volumes:
        - '/var/run/docker.sock:/var/run/docker.sock'

</code></pre>
<p id="bkmrk-lancer-votre-stack-%3A">lancer votre stack :</p>
<pre id="bkmrk-docker-compose-up--d"><code class="language-shell">docker-compose up -d</code></pre>
