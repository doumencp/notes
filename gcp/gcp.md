# Notes certification GCP associate Cloud Engineer  

GCP consiste en une multitude de services, qui peuvent être classés en 3 grandes catégories : Compute, Storage, et Networking.
## GCE (Google Compute Engine) 

C'est le service de GCP qui permet de créer des VMs.
On peut venir utiliser des fonctionnalités telles que le load balancing ou le autoscaling, pour avoir une infrastructure qui peut s'adapter en fonction de la charge.

Il est souvent utilisé avec les services de stockage de GCP, de base de données, et de Cloud Load Balancing.

Lorsque l'on instancie une VM depuis le GCE, on a allrs la possibilité de choisir le système d'exploitation, le type de machine, le nombre de CPU, la quantité de RAM, le type de disque, et le réseau auquel la VM sera connectée. Il y a également des paramètres pour choisir d'ajouter des labels, des tags, et des clés SSH pour l'accès sécurisé, ainsi que les règles de firewall (ex : on autorise le traffic entrant HTTP si on sert une application via HTTP).

Chaque VM sera verra attribuer une adresse IP interne et une adresse IP externe (qui change si on stoppe la VM). On peut également choisir de créer une VM avec une adresse IP statique.
Il est possible de choisir de ne pas exposer une VM à l'extérieur, et de la rendre accessible uniquement via un VPN (intégré à GCP), ou via un proxy sécurisé (IAP : Identity Aware Proxy) ou même en configurant une VM bastion (qui sert de passerelle sécurisée pour accéder au réseau privé et donc pouvoir atteindre les VMs non exposées ).

**Attention : l'adresse IP statique est facturée même si la VM est arrêtée.**

--- 

On cherche à réaliser les étapes précédentes de manière automatisée, car fastidieux de tout faire via l'interface web. Ici, on cherche seulement à réduire le nombres de clics dans l'interface, mais pas de s'en détacher complètement. Pour ce faire, il existe plusieurs manières de faire. 

### Startup script

**Important pour l'exam**

Il s'agit de scripts qui sont exécutés au démarrage de la VM. Ils peuvent être utilisés pour mettre à jour des paquets, installer des logiciels, configurer des services, ou tout autre tâche de configuration. Les scripts doivent être rédigés dans le langage de script de l'OS de la VM. 

### Instance Template

Un template d'instance est une configuration préconfigurée qui peut être utilisée pour créer des instances de VM. Il contient des informations telles que le type de machine, l'image de disque, les paramètres de réseau, et les scripts de démarrage. Les templates d'instances permettent de standardiser la configuration des VMs et de réduire le temps nécessaire pour créer de nouvelles instances. 

Les templates ne peuvent pas être modifiés après leur création. Si vous devez apporter des modifications, vous devez créer un nouveau template.

### Custom Image

Dans le cas d'un besoin d'avoir des paquets ou soft spécifiques sur la VM, il devient lour dde devoir exécuter un script à chaque démarrage. Dans ce cas, il est possible de créer une image personnalisée, qui embarque directement ces derniers.

Pour ce faire, il faut une image et l'enrichir. Il y a plusieurs possibilités. Par exemple, on peut créer une image à partir d'un disque d'une VM dans GCE, qui nous permettra d'avoir une image qui réplique exactement la VM et sans installation au démarrage de la machine en question.
