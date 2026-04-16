# Notes certification GCP associate Cloud Engineer  

## IAM (Identiy and acccess management)

La gestion des droits dans GCP est fait via IAM, avec des roles, des ressources et actions associées. Il faut se poser les questions suivantes : Qui, peut faire quoi, sur quelles resources.
Basic roles set project-level permissions and, unless otherwise specified, control access and management to all Google Cloud services.

| Role | Name	 | Permissions | 
| ------------- | -------------- | -------------- |
|roles/viewer	| Permissions for read-only actions that do not affect state, such as viewing (but not modifying) existing resources or data. |
|roles/editor	| All viewer permissions, plus permissions for actions that modify state, such as changing existing resources. |
|roles/owner	| All editor permissions and permissions for the following actions: manage roles and permissions for a project and all resources within the project; set up billing for a project. |



## Google Cloud Observability

Permet de monitorer les produits.

## Cloud Billing

Gère la partie facturation liée au projet GCP.
Un projet doit être lié à un compte de facturation dédié, mais une compte de facturation peut être lié à plusieurs projets.

## Régions

Les régions dans GCP correspondent aux zones géographiques où sont les serveurs physiques de Google.

# Base de données et stockage

## Google Cloud SQL

L'un des composants principaux de GCP. C'est une base de données entièrement gérée par GCP (setup, backups, mises à jour, scaling...). Cela permet de se concentrer sur l'appli en elle même. Plusieurs moteurs SQL sont supportés, comme MySQL, PostgresSQL, SQL Server.

- Cloud SQL permet également de choisir la configuration de l'infra : type de machines, vCPU, mémoire vive, HDD SSD, zones et régions pour la disponibilité.

## Cloud Spanner

Cloud Spanner est une base de données disitribuée dont les forces sont la fiabilité et la scalabilité horizontale. Elle est utilisée pour les données structurées et semi structurées, et pour gérer les lectures et écritures transactionnelles.

### Fonctionnalités

- Fiabilité, avec des indexes secondaires.
- Support SQL, avec les statements ALTER pour modifier le schema
- Instances multi régionales
- 3 replicas lecture/écriture inclus dans la configuration régionale répartis dans plusieurs zones au sein de la même région Spanner

### Modèle de données, Schema

- Spanner possède des tables, lignes, colonnes et des clés primaires et secondaires, en tant que BDD relationnelles
- Les données des tables liées sont entrelacées pour améliorer les performances
- Spanner divise les données en petits groupes, appelés splits
- Spanner ajoute et supprime automatiquement les "split boundaries" (clé de début et de fin des splits), ce qui fait varier en continu le nombre de splits

### Replication

- Spanner crée plusieurs copies des lignes et stocke ces répliques dans différentes régions
- Chaque configuration multi-régions contient deux régions dédiées en tant que régions de lecture/écriture
- Chacune de ces régions contient 2 réplicas lecture/écriture. L'un des réplicas est le leader par défaut, et l'autre est le "témoin"

### Intégrité des données

- Spanner garantit un haut niveau de concurrence pour les transactions, appelé External Consistency
- C'est meilleur que la linéarisation ou la sérialisation



# Cours skills boost google

- Il existe des produits dédiés pour chaque usage.
- On peut utiliser le marketplace pour trouver des déploiements qui sont tout faits, par exemple déployer une stack LAMP sans avoir à configurer quoi que ce soit, via le Deployment Manager


## gcloud CLI cheatsheet

### Stockage

Utilisation de l'outil `Cloud Storage`

#### buckets

Toute commande pour le stockage commence par: `gcloud storage`

- `gcloud storage buckets create gs://qwklabs-pierre-22` :  Créer le bucket `qwklabs-pierre-22`
- `gcloud storage buckets list` :  Liste les buckets existants
- `gcloud storage cp la-regalante.gpx gs://qwklabs-pierre-22`
- `gcloud storage ls gs://qwklabs-pierre-2`
- `gcloud compute regions list`

### Déploiement d'un stack LAMP

Utilisation de l'outil `Market Place`, qui permet de chercher des softs et de les déployer à la volée, pré configuré


### Réseaux dans GCP

#### VPC

Un VPC sur GCP correspond à un ensemble d'objets réseaux managés par Google.

Plusieurs niveaux de vision :
- Project : Il enapsule tous les objets, dont les réseaux
- Networks : objet qui a plusieurs modes : Default, auto mode, custom mode
- SubNetworks : Permettent de ségréger les environnements
- Regions & Zones: Représentent les data center de Google, et prodigue de la fiabilité de service, et de la protection des données
- IP adresses: VPC donne également des IP adresses internes et externes, ainsi que des ranges
- VMs : VPC permet de configurer le réseau des VMs 
- Routes et règles firewall: les règles qui permettent d'affiner les flux réseaux

#### Networks & SubNetworks

Le Projet est le composant mère qui contiendra toutes les ressources. 
Il permet notamment d'associer les ressources et services à la facturation.

Par défaut, chaque projet peut avoir jusqu'à 15 réseaux.

Les réseaux peuvent être partagés entre projets, ou peuvent être associés à d'autres réseaux.
Ces réseaux en eux-mêmes n'ont pas de range d'adresses IP, mais est constitué des adresses IP et des services dans ce réseau.

Un réseau est global et peut exister dans toutes les régions du monde simultanément.
Au sein d'un réseau, on peut ségréger les ressources avec des SubNetworks régionaux.

Il y a 3 types de réseaux :

##### Réseaux type "Default"

Chaque projet est fourni avec un réseau VPC de type Default, avec des subnets et règles firewall.
De manière plus précise, un subnet est alloué pour chaque région avec des CIDR qui ne se chevauchent pas, et des règles firewall qui autorisent le trafic entrant (ingress) pour l'ICMP, RDP et SSH, depuis n'importe où. Pour le traffic interne, le firewall autorisent tout le flux entrant.

##### Réseaux type "Auto"

En mode Auto, un subnet de chaque région est créé automatiquement au sein du réseau.
Ces subnets créés automatiquement utilisent un ensemble d'adresses IP pré définies, avec un sous réseaux /20, qui peut être étendu à /16.

##### Réseaux type "Custom"

En mode Custom, aucun subnet n'est créé automatiquement, ce qui donne un contrôle total sur les sous réseaux et les ranges d'adresses IPs.


#### Networks isole systems

Il faut que les machines soient sur le même réseaux pour communiquer en interne entre elles, même si elles tournent dans la même région.

Inversement, deux machines qui tournent dans des régions différentes peuvent faire partie du même réseau, via une VPN Gateway.
Un sous réseau (subnet), au sein d'une région, peut être réparti sur plusieurs zones.

Il est possible d'étendre un sous réseau sans avoir à recréer une instance.
Dans ce cas, le nouveau réseau ne doit pas avoir de chevauchements avec d'autres subnets du même réseau VPC dans n'importe quelle région.

On peut étendre un réseau, mais pas le réduire.

Il est recommandé d'éviter les grands subnets.

Pour chaque sous réseau, 4 adresses IP sont réservées pour GCP (la gateway etc..).

Pour étendre un réseau, il suffit de modifier le masque du subnet.

### Adresses IP

Les machines virtuelles ont obligatoirement une adresse IP interne, et peuvent également avoir une adresse IP externe.

Google Cloud a deux types de DNS internes, un de zone et un global.

Chaque instance a un hostname qui peut être résolu vers une adresse IP.

Les hostname est le même que le nom de l'instance.
Le FQDN (Fully qualified domain name) est de la forme : hostname.zone.c.project-id.internal

Chaque instance a un serveur de metadata associé. Il peut jouer le rôle de serveur DNS interne (x.x.x.254). Il est fourni en tant que ressource GCE.

On peut également, via l'adresse IP externe, venir faire de la résolution DNS depuis l'extérieur de GCP, via les serveurs DNS publics. L'entrée DNS associée n'est pas ajouté automatiquement. Si besoin, il est possible d'utiliser un autre service de GCP, "Cloud DNS". 

### Cloud DNS

Cloud DNS est un service managé de DNS, qui permet de faire de la résolution d'adresses IPs publiques, et qui garantit une grande fiabilité et disponibilité.

### Alias IP Ranges

Alias IP Ranges est un service qui permet d'associer une range d'adresses IP à une machine, ce qui permet d'attribuer une adresse IP distincte pour chaque service qui tourne sur une même VM, sans avoir besoin de plusieurs interfaces réseau.

### Routes et firewall

Chaque réseau a une gateway (routeur) par défaut, à destination duquel toutes les machines du réseau envoient leurs paquets.

De plus, si le réseau a été créé de manière automatique, il y aura par défaut un firewall, qui peut bloquer du traffic entrant et sortant.

Les réseaux créés manuellement n'ont pas de firewall pré configuré, il faudra alors le configurer soi même.

Les réseaux VPC fonctionnent aussi en tant que firewall distribué. Les règles firewall sont appliqués à l'échelle du réseau.

Les connexions sont acceptées ou refusées au niveau de l'instance.

La politique firewall de base est allow all egress traffic, et deny all ingress traffic.

Une règle firewall est composée de :

- une direction (ingress, egress)
- une source ou une destination
- un protocole et un port
- une action (allow/deny)
- une priorité (La première règle qui matche est appliquée)

### Pricing

Dans GCP, les sollicitations réseaux sont facturées, dont voici quelques détails :

- Le traffic entrant vanilla n'est pas facturé, mais s'il arrive sur un loadbalancer, ou s'il y a une réponse, alors si
- Le traffic sortant vers la même zone n'est pas facturé (adresses IP internes)
- Le traffic sortant vers les produits google ne sont pas facturés
- Le traffic sortant vers un autre service google cloud n'est pas facturé (exceptions selon régions)
- Le traffic sortant entre zones de la même région : $0.01/GB
- Le traffic sortant vers la même zone (adresses IP externes) : $0.01/GB
- Le traffic sortant entre régions : $0.01/GB
- ...

- Adresse IP statique : $0.010/heure (assignée et non utilisée)
- Adresse IP statique et éphémère en utilisation sur des VM standard: $0.005/heure
- Adresse IP statique et éphémère en utilisation sur des VM preemptible and Spot : $0.0025/heure
- ...

Il est possible d'estimer les coûts avec le Google Cloud Pricing calculator.
On peut combiner l'estimation du prix des VMs sur GCE avec le Google Cloud Network.
