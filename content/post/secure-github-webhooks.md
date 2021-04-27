---
title: "Configurer des Webhooks sécurisés avec Github et NodeJS"
date: 2018-07-16T22:36:52+02:00
slug: "" 
tags: []
categories: ["dev"]
---

## Introduction

Dans [notre premier article](post/recette-dun-blog/), nous avons parlé de la mise en place de webhooks github pour déployer automatiquement le blog sur notre serveur de production. Pour cela, nous utilisions une librairie écrite en Go : [Githook](https://github.com/arashpayan/githook).

Néanmoins, à l'époque, nous n'utilisions pas cette fonctionnalité de manière sécurisée. N'importe quel attaquant pouvait, sous réserve de trouver la bonne route et le bon port, requêter en permanence notre serveur de webhook déclenchant le script de mise en production. En effet, Githook ne permettait pas d'utiliser cette fonctionnalité-là.

J'ai donc décidé d'implémenter rapidement un serveur léger pouvant remplir le même rôle que Githook, mais de manière sécurisée, en ne laissant pas n'importe quel appelant pouvoir déclencher le script de migration.

## Développement

### Principe des webhooks sécurisés

Le principe est simple : il s'agit tout d'abord de choisir un secret et de le rendre disponible chez Github (dans la zone de configuration webhooks) ainsi que sur notre serveur de production. Les deux parties qui vont discuter ensemble possèderont ce secret, mais aucune autre ne l'aura. Ce qui rendra impossible toute requête externe. 

Github va utiliser le secret pour "Sha1-hasher" de son côté le contenu du payload qu'il va renvoyer dans sa requête à notre serveur. Il renseignera ce hash dans l'attribut `X-Hub-Signature`. Il s'agit en fait de signer la requête pour que notre serveur sache que celle-ci est légitime et provient d'une entité connue (Github).

Du côté de notre serveur, nous recevrons la requête émise par Github, et nous calculeront nous aussi le Sha1 du payload, avec notre secret disponible côté serveur. Si le Hash fournit par l'appelant est le même que celui calculé côté serveur, alors, nous pourrons déclencher le script de déploiement. Si les deux sont différents, nous renvoyons une erreur et le script ne sera pas exécuté.

### Développement

3 étapes principales : 

- Fournir la configuration du serveur :
	- Port d'écoute
	- Route
	- Secret 
	- Script à exécuter
- Calculer le hash et le comparer à celui fourni par Github
- Exécuter le script sous certaines conditions
- Fournir un reporting précis et détaillé

Le secret devant être disponible côté serveur de manière persistante, deux possibilités peuvent être envisagées : 

- Variable d'environnement
- Fichier Json

Pour sa praticité, j'ai choisi d'implémenter une solution à base de fichier json pour le stockage du secret. 

### Quelques remarques techniques

- Utilisation de la librairie `commander` pour la mise en place des paramètres invoqués en ligne de commande.
- Utilisation de la librairie `crypto` pour les calculs cryptographiques : hash et comparaison en temps constant (pour éviter les attaques temporelles).
- Utilisation de `shelljs` pour déclencher l'exécution du script. 
- Notre serveur renvoie des réponses précises en cas d'erreurs spécifiques : `500` si le script déclenche une erreur, avec les messages d'exécutions et d'erreur du shell, `403` si la requête n'est pas légitime (les signatures ne correspondent pas) ou bien `200` si tout va bien. Ces réponses sont consultables depuis l'interface Github et fournissent des informations intéressantes sur le bon fonctionnement du script.

La librairie developpée pour cet article est disponible sur [ce repository git](https://github.com/epsxy/node-webhooks) !

## Mise en place du serveur qui traitera le webhook sous debian

Pour mettre en place notre serveur fraîchement développé, nous allons configurer un service systemd pour le faire tourner en tâche de fond. Je ne suis pas vraiment familier avec ce genre de choses, mais voici une brève introduction à `systemd` et à ses commandes de base

### Les commandes systemd

```
systemctl start $_{SERVICE_NAME} : Démarrer un service
systemctl stop $_{SERVICE_NAME} : Arrêter un service
systemctl status $_{SERVICE_NAME} : Obtenir le status d'un service
systemctl restart $_{SERVICE_NAME} : Redémarrer un service
systemctl daemon-reload : Recharger les services après une modification de conf
journalctl -u $_{SERVICE_NAME} :  Afficher les logs d'un service
```

### Le fichier de configuration d'un service

Pour configurer un nouveau service systemd : 

- Créer un fichier pour notre nouveau service `touch /etc/systemd/system/$_{SERVICE_NAME}.service`
- Il faut remplir ce fichier avec le template suivant : 

```
[Unit]
Description=Webhook server

[Service]
ExecStart=/path/to/node/webhooks/index.js --port $_{PORT} --route $_{ROUTE} --conf $_{PATH_TO_CONF_FILE} --script $_{PATH_TO_SHELL_SCRIPT}
Restart=always
User=$_{USER}
Group=$_{GROUP_OF_USER}
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/path/to/node/webhooks

[Install]
WantedBy=multi-user.target
```

Où : 

- `/path/to/node/webhooks/` est le chemin vers la librarie développée pour cet article.
- `$_{USER}` et `$_{GROUP_OF_USER}` désigne l'utilisateur principal ainsi que son groupe, depuis lesquels le service sera lancé. 

Après avoir créé ce fichier, il suffit de : 

- Recharger les fichiers de conf avec `systemctl daemon-reload`
- Lancer notre nouveau service `systemctl start $_{SERVICE_NAME}`

Normalement, si tout va bien, notre service est lancé et nous pouvons voir les logs s'afficher avec `journalctl -u $_{SERVICE_NAME}`

## Conclusion

Grâce à la simplicité d'utilisation de Node.js et de son écosystème, nous avons pû facilement développer, tester et mettre en production un serveur de réponse aux webhook Github. Ce serveur, et c'est la raison même de son existence, propose un système de vérification de signatures permettant de rejeter toute requête illégitime, faite par une partie ne connaissant pas le secret propre à nos deux parties : Github et notre serveur privé.