# Module Réseaux - TP NGINX

## Introduction

NGINX, prononcé "engine-ex", est un service Web open-source, principalement utilisé en tant que serveur Web, reverse
proxy, cache HTTP, et load balancer.

Créé à l'origine par Igor Sysoev, avec sa première sortie publique en octobre 2004, ce logiciel répond au problème du
C10K, où le but est d'être capable de réponse à plus de 10.000 connexions simultannées.

De nos jours, c'est le service Web le plus utilisé dans le monde, notamment par des entreprises telles que Google,
Microsoft, Facebook, Twitter ou bien Apple.

## Objectifs du TP

* Installer et configurer NGINX
* Comprendre les fondamentaux de NGINX
* Servir des sites Web et des ressources statiques
* Configurer NGINX en tant que [reverse proxy](https://fr.wikipedia.org/wiki/Proxy_inverse)
* Mettre en place un [load-balancer](https://www.ovhcloud.com/fr/public-cloud/what-load-balancing/)

## Pré-requis

* Permission de sudo
* Utilisation du terminal de commande
* Disponibilité des 6 RPi

## #1. Installer et configurer NGINX

### 1.1 Installation

L'installation de NGINX sous Debian est simple, il suffit d'utiliser le gestionnaire de paquets `apt`.
> sudo apt update && sudo apt install nginx -y

### 1.2 Réglage du pare-feu

Avant de tester NGINX, il est pertinent de s'assurer que le pare-feu autorise les connexions entrantes sur les ports
HTTP (80) et HTTPS (443).

> sudo ufw allow 'Nginx Full'

### 1.3 Vérification du service

> systemctl status nginx

Si le service est en marche, il est prêt à être utilisé. Sinon, il faut le démarrer ou contacter l'enseignant en cas d'
erreur.

> systemctl start nginx

À partir du navigateur, vous pouvez accéder à la page d'accueil du serveur, à l'adresse : `http://localhost/`

### 1.4 Configuration

La configuration et le comportement de NGINX est défini par l'ensemble des fichiers et dossiers situés dans le
répertoire `/etc/nginx/`.

| Emplacement                 | Utilité                                                                                                                                                                                                                                                                                    |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `nginx.conf`                | Configuration générale du service, ici se trouve le bloc global qui régit l'ensemble des blocs de NGINX.<br/>Par exemple, il est possible de définir l'utilisateur utilisé par le service, les paramétrages globaux du protocole HTTP (logs, compression des requêtes, certifications TLS) |
| `sites-available/*`         | Le répertoire où les "blocs de serveur" par site peuvent être stockés. NGINX n'utilisera pas les fichiers de configuration trouvés dans ce répertoire à moins qu'ils ne soient liés au répertoire `sites-enabled`.                                                                         |
| `sites-enabled/*`           | Le répertoire où sont stockés les "blocs serveur" activés par site. En général, ceux-ci sont créés en établissant un lien symbolique avec les fichiers de configuration qui se trouvent dans le répertoire `sites-available`.                                                              |
| `conf.d/*`                  | A la même utilité que le répertoire `sites-enabled`, cependant, il peut également être utilisé pour y stocker des configurations globales du serveur Web.                                                                                                                                  |
| `snippets/*`                | Ce répertoire contient des fragments de configuration qui peuvent être inclus partout dans la configuration de NGINX. Les segments de configuration potentiellement répétables sont de bons candidats pour le remaniement en fragments.                                                    |
| `/var/log/nginx/access.log` | Chaque requête adressée à votre serveur web est enregistrée dans ce fichier journal, à moins que NGINX ne soit configuré autrement.                                                                                                                                                        |
| `/var/log/nginx/error.log`  | Chaque erreur NGINX sera enregistrée dans ce seul journal.                                                                                                                                                                                                                                 |

##### Le fichier `/etc/nginx/sites-available/default` contient la configuration du site Web par défaut. Il est possible de créer un nouveau fichier de configuration pour chaque site Web que l'on souhaite.

_Par ailleurs, il est
possible [d'avoir plusieurs sites sur la même machine](https://webdock.io/en/docs/how-guides/shared-hosting-multiple-websites/how-configure-nginx-to-serve-multiple-websites-single-vps)
, mais il faut utiliser l'instruction `server_name` dans le bloc server et disposer de plusieurs DNS / Sous domaines, ce
qui n'est pas notre cas au sein de notre environnement de TP._

## #2. Servir des sites Web et des ressources statiques

### Création de notre première configuration de site web :

* Créer le fichier `mon_site.conf` dans le répertoire `/etc/nginx/sites-available`

* Copier le code suivant dans ce fichier :

````nginx configuration
    server {
        root /var/www/html/mon_site;
        
        location / {
            index index.html index.htm;    
        }
    }
````

- Et placer le dossier `mon_site` dans le répertoire `/var/www/html`.

> #### Exo 1. Comment remplacer la page par défaut de nginx par la configuration `mon_site` ? (Indice : `sites-enabled`)
> #### Exo 2. Lister toutes les images présentes dans le dossier `images` du dossier `mon_site`. (Indice : `location` & `nginx autoindex`)
> #### Exo 3. Définir une page d'erreur 404, vous ferez la page manuellement. (Indice : `error_page`)

## 3. Configurer NGINX en tant que reverse proxy

#### On parle de reverse-proxy quand une application placée en front au contact des clients, tel que le serveur Web, joue le rôle d'un intermédiaire avec des applications placées en backend, en redirigeant notamment les requêtes.

<img height="220" src="https://user.oc-static.com/upload/2021/11/30/16382909620871_p3c2-1.png" width="348"/>

Nous ne disposons pas de serveur Backend mais nous pouvons rediriger les requêtes vers un autre serveur (celui de Google
par exemple).
> #### Exo 4. Modifier la location / de `mon_site.conf` afin de rediriger toutes les requêtes vers le serveur `www.google.com` (Indice : `proxy_pass`, `proxy_set_header` & `Reverse-Proxy-for-Google`)
> #### Exo 5. Ajouter une location dans `mon_site.conf` permettant de rechercher la suite de l'URL avec le moteur de recherche de Google (Indice : `https://google.com/search?q=`, `rewrite`, `$uri`, `$args`)

## 4. Mettre en place un load-balancer

Le load balancing consiste à rediriger les requêtes entre plusieurs serveurs. Il permet d'assurer la scalabilité
horizontale de l'infrastructure, et de ne pas surcharger un seul serveur.

**Les 3 principaux algorithmes de load balancing utilisés sont :**

- **Le round-robin (le plus simple)** : Les requêtes sont réparties uniformément entre les serveurs, en tenant compte de
  leur pondération. Cette méthode est utilisée par défaut (il n'y a pas de directive pour l'activer).

- **Le least-connections** : Une requête est envoyée au serveur ayant le plus petit nombre de connexions actives, en
  tenant
  compte, là encore, du poids des serveurs.

- **IP Hash** : Le serveur auquel une requête est envoyée est déterminé à partir de l'adresse IP du client. Dans ce cas,
  les
  trois premiers octets de l'adresse IPv4 ou l'adresse IPv6 entière sont utilisés pour calculer la valeur de hachage.
  Cette méthode garantit que les demandes provenant de la même adresse arrivent au même serveur, sauf si celui-ci n'est
  pas disponible. Cela permet également à utilisateur de maintenir ses données de navigation (cookies, etc.), car ces
  dernières seront stockées sur le même serveur.

- Il en existe d'autres, mais inutiles à exploiter. Toutes ces méthodes sont disponibles
  ici : https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/

Un load-balance se fait en deux étapes :

> #### Exo 6. Définir un upstream (`upstream`) en dehors du bloc `server` mais en restant dans le fichier `mon_site.conf`

```nginx configuration
    upstream mon_upstream {
        # Définir l'algorithme de load balancing avant les serveurs (si aucun algorithme défini, round-robin par défaut)
        server node1;
        server node2;

        # Il est possible de définir des poids pour chaque serveur, des serveurs en maintenance.
        # Les serveurs en maintenance ne seront pas pris en compte dans le calcul du serveur le plus proche.
        # Pour cela, il faut définir une directive `weight` pour chaque serveur.
        # Pour le moment, on ne définit pas de poids.
        
        #server node1 weight=1;
        #server node2 weight=2;
        #server node3 weight=3;
        
    }
  ```

> #### Exo 7. Redéfinir la location '/' pour qu'elle utilise le load-balancer (Indice : `proxy_pass`)

Pour tester le load-balancer, vous pouvez m'appeler pour réaliser des benchmarks, grâce à
cet [outil](https://github.com/mcollina/autocannon) (Autocannon HTTP /1 Benchmark tool).