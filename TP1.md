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

* Connexion sous Debian avoir la permission de sudo
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
| `sites-enabled/*`           | Le répertoire où sont stockés les "blocs serveur" activés par site. En général, ceux-ci sont créés en établissant un lien avec les fichiers de configuration qui se trouvent dans le répertoire `sites-available`.                                                                         |
| `conf.d/*`                  | A la même utilité que le répertoire `sites-enabled`, cependant, il peut également être utilisé pour y stocker des configurations globales du serveur Web.                                                                                                                                  |
| `snippets/*`                | Ce répertoire contient des fragments de configuration qui peuvent être inclus partout dans la configuration de NGINX. Les segments de configuration potentiellement répétables sont de bons candidats pour le remaniement en fragments.                                                    |
| `/var/log/nginx/access.log` | Chaque requête adressée à votre serveur web est enregistrée dans ce fichier journal, à moins que NGINX ne soit configuré autrement.                                                                                                                                                        |
| `/var/log/nginx/error.log`  | Chaque erreur NGINX sera enregistrée dans ce seul journal.                                                                                                                                                                                                                                 |

##### Le fichier `/etc/nginx/sites-available/default` contient la configuration du site Web par défaut. Il est possible de créer un nouveau fichier de configuration pour chaque site Web que l'on souhaite.

### Création de notre première configuration de site web :

* Créer le fichier `mon_site.conf` dans le répertoire `/etc/nginx/sites-available`

* Copier le code suivant dans ce fichier :

````nginx configuration
    server {
        root /var/www/html/mon_site;
        
        location {
            index index.html index.htm index.php;    
        }
    }
````

