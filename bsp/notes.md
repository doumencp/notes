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