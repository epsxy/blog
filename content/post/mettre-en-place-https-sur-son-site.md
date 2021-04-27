---
date: 2018-07-06T21:48:25+02:00
title: "Mettre en place HTTPS sur son site avec Let's Encrypt"
slug: ""
tags: []
categories: ["dev"]
---

## Introduction 

Nous allons parler de la mise en place du protocole https sur un serveur web nginx sous debian stretch. HTTPS, pour *Hyper Text Transfer Protocol Secure* est un protocole de sécurité qui vise à chiffrer les informations échangées lors de la navigation sur différents sites internet afin de sécuriser toutes les transactions (au sens transfert d'informations) qui peuvent s'y dérouler. L'intérêt est double : 

- Chiffrer les informations afin d'empêcher leur interception et leur lecture par un attaquant. 
- Vérifier l'identité de l'entité du site internet avec lequel le client discute.

Migrer en HTTPS, c'est assurer la sécurité de ses utilisateurs et améliorer son référencement par les moteurs de recherche. Au début, il fallait passer par une autorité de certification, qui fournissait le certificat SSL nécessaire au passage en HTTPS moyennant une transaction financière. 

Depuis, d'autres acteurs ont émergé et proposent une certification gratuite, comme l'initiative de la Linux Foundation : *Let's encrypt*. Nous proposerons ainsi de voir en détail la mise en place d'un certificat fourni par [Let's Encrypt](https://letsencrypt.org/), sur un serveur Nginx installé sur une Debian Stretch. 


## Pré-requis

Les pré-requis sont les suivants : 

- Un serveur sous Debian 9 - Stretch
- Nginx installé sur la Debian
- Un nom de domaine : www.mydomain.com - par exemple


## Installation de l'utilitaire de configuration de Lets Encrypt 

Nous allons tout d'abord installer l'utilitaire [Certbot](https://certbot.eff.org/), qui va permettre de déployer facilement le certificat fourni par Let's Encrypt et de configurer notre serveur nginx. 

Commençons par ajouter le repository de backports Stretch dans `/etc/apt/sources.list`. Il suffit de dé-commenter les 2 lignes suivantes : 

```
deb http://deb.debian.org/debian stretch-backports main contrib non-free
deb-src http://deb.debian.org/debian stretch-backports main contrib non-free
```

On applique ces modifications avec `sudo apt-get update && sudo apt-get ugprade`, puis on installe l'utilitaire cerbot pour nginx : 

```
sudo apt-get install python-certbot-nginx -t stretch-backports
```

## Mise en place du HTTPS 

### Déclaration du serveur dans nginx

Avant d'utiliser Certbot, il faut s'assurer que le serveur est bien déclaré dans les fichiers de configuration de Nginx.

```
sudo vim /etc/nginx/sites-available/default
```

Ce fichier contient normalement par défaut la ligne suivante : 

```
server_name _;
```

Il suffit de la remplacer par la déclaration suivante : 

```
server_name mydomain.com www.mydomain.com;
```

Normalement, si tout va bien, `sudo nginx -t` indique que la configuration est valide, et on peut redémarrer le serveur avec `sudo systemctl restart nginx`

Cette modification est nécessaire pour que Certbot puisse trouver le bon serveur auquel appliquer les modifications et les redirections HTTPS.

### Installation via certbot

Maintenant, nous pouvons exécuter la commande suivante pour installer notre certificat : 

```
sudo certbot --nginx -d mydomain.com -d www.mydomain.com
```

Les paramètres `-d` se rapportent à ce que nous avons déclaré après `server_name` dans le fichier de configuration Nginx. Normalement, si tout se passe bien, vous allez passer par les étapes suivantes : 

- Accepter les conditions d'utilisation  
- Fournir une adresse e-mail (permet notamment d'avoir des notifications d'expiration du certificat)
- Choisir entre les deux options `1: No Redirect` et `2: Redirect`. Dans mon cas, j'ai choisi `2` pour rediriger automatiquement les requêtes HTTP effectuées sur mon domaine en HTTPS. 

### Modifications effectuées par certbot

Normalement, certbot a rajouté automatiquement tout ce qu'il faut dans le fichier de configuration de votre serveur pour : 

- Aller chercher les certificats au bon endroit sur le disque
- Rediriger les connexions HTTP à votre domaine en HTTPS

Désormais, nous pouvons tester le bon déroulement de l'opération et de la redirection en accédant à notre site `http://mydomain.com` devrait nous rediriger automatiquement vers `https://mydomain.com`. Normalement, en cliquant sur le cadenas à côté de l'URL, nous pouvons voir notre certificat tout neuf : 

![Certificat mis en place](/img/articles/https-lets-encrypt/certificat.png)

## Automatisation du renouvellement des certificats 

Les certificats Let's Encrypt sont valables pour une durée maximale de 90 jours. Il est conseillé de les mettre à jour au bout de 60 jours. En ayant installé notre certificat avec l'utilitaire `certbot`, nous n'avons rien d'autre à faire si ce n'est tester le renouvellement automatique. En effet, le renouvellement automatique sera assuré par un script placé dans `/etc/cron.d`, qui sera exécuté toutes les douze heures et vérifiera la validité du certificat, qui sera renouvelé s'il reste moins de 30 jours de validité.

La commande suivante permet de tester le bon déroulement de cette tâche, sans renouveler le certificat (dry-run) : 

```
sudo certbot renew --dry-run
```

Si tout se passe bien et si la commande s'est terminée sans erreur, il ne reste plus rien à faire, votre certificat sera automatiquement renouvelé tous les 60 jours.

## Sources

Cet article a été rédigé suite à la mise en place de HTTPS sur ce blog, qui a été rendu possible par 2 excellents articles : 

- [How To Secure Nginx with Let's Encrypt on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
- [How To Secure Nginx with Let’s Encrypt on Debian Strech](https://medium.com/@kwa29/how-to-secure-nginx-with-lets-encrypt-on-debian-strech-6ff2e1008c55)
