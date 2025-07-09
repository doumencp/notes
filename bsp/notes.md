# Notes techniques BSP

## Debug podman

Une piste serait de checker quel network mode le podman de la machine utilise (pasta, slirp4netns ou autre). Sur mon PC, c'est pasta
```bash
podman inspect snmp | grep -i network
```

## Debug snmp

En lancant un simulateur snmp directement sur mon host rockylinux (ma VM), avec 

```bash
pip install snmpsim
```

puis 

```bash
/home/vagrant/.local/bin/snmpsim-command-responder --agent-udpv4-endpoint=0.0.0.0:1161
```
et 

```bash
snmpwalk -v 2c -c public localhost:1161
```

J'arrive bien à contacter le serveur snmp.

La solution a été d'activer explicitement l'UDP au run du conteneur avec /udp sur les ports qui l'utilisent : 

```bash
podman run --name snmp -p 1161:1161/udp --rm -it snmpsim
```
ou 
```bash
podman run --name snmp -p 1161:161/udp --rm -it docker.io/tandrup/snmpsim:latest
```

l'image snmpsim est une image buildée par mes soins, qui démarre un serveur snmp, avec quelques utilitaires de debug dans l'image :

```Dockerfile
# Dockerfile
FROM python:3.11-slim

# Installer les dépendances système
RUN apt-get update && apt-get install -y \
    snmp iproute2 tcpdump sudo \
    && rm -rf /var/lib/apt/lists/*

# Installer snmpsim
RUN pip install snmpsim

# Exposer le port UDP utilisé par SNMP
EXPOSE 1161/udp

# Créer l'utilisateur snmpsimuser avec mot de passe et permissions sudo
# Création utilisateur
RUN useradd -m -s /bin/bash snmpsimuser && echo "snmpsimuser:password" | chpasswd

# Ajouter au groupe sudo
RUN usermod -aG sudo snmpsimuser

# Optionnel : autoriser sudo sans mot de passe (à éviter en prod)
RUN echo "snmpsimuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER snmpsimuser


# Commande par défaut
CMD ["snmpsim-command-responder", "--agent-udpv4-endpoint=0.0.0.0:1161"]
```

et ce compose permet de simuler encore plus proche du setup cible

```yaml
version: '3'
services:
  snmp:
    image: docker.io/tandrup/snmpsim:latest
    container_name: snmp_container
    hostname: snmp
    logging:
      driver: "journald"
    # security_opt:
    #   - label=type:snmp.process
    # userns_mode: keep-id:uid=1000,gid=1000
    # sysctls:
    #   - fs.mqueue.msg_max=100
    ports:
      - "1161:161/udp"
    networks:
      snmp_net:
        ipv4_address: 172.16.60.2
  # nginx:
  #   image: docker.io/library/nginx
  #   container_name: nginx_container
  #   hostname: nginx
  #   logging:
  #     driver: "journald"
  #   ports:
  #     - "8080:80/tcp"
  #   networks:
  #     snmp_net:
  #       ipv4_address: 172.16.60.3
networks:
  snmp_net:
    ipam:
      driver: default
      config:
        - subnet: 172.16.60.0/30
```

D'après nos infos, le conteneur cible écoute sur 1162 en UDP, mais il faut peut etre également être capable de recevoir des paquets sur 161 en UDP.
Il est donc nécessaire de forward les paquets entrants sur le port 161 UDP du host vers le port 1161 UDP, pour que le conteneur les recoive. Dans ce cas là, 
on peut faire (en tant que root) :
```bash
sudo socat -v UDP4-RECVFROM:161,reuseaddr,fork UDP4-SENDTO:192.168.1.198:1161
```
ou
```bash
sudo iptables -t nat -A PREROUTING -p udp --dport 161 -j REDIRECT --to-port 1161
```