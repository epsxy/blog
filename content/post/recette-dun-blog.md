---
title: "Recette d'un blog : Comment débuter ?"
date: 2018-05-16T22:34:45+02:00
slug: ""
tags: []
categories: ["dev"]
---
## Introduction

Pourquoi faire un blog ? Ben, parce qu'on peut !

Cet article présentera pas à pas les choix que j'ai fait et la procédure que j'ai suivi pour mettre en ligne ce blog.

## Choisir son moteur de blog

La première étape est de savoir "Comment ?". Il s'agit de choisir son moteur de blog (un CMS : Système de gestion de Contenu), ce qui nous permettra de gérer le blog et son contenu. Il s'agit par exemple de :

- Wordpress
- Jekyll
- Drupal
- Hugo

J'ai choisi [Hugo](https://gohugo.io/), un moteur léger et performant, permettant la rédaction de contenu en Markdown.

### Installation d'Hugo

Pour installer Hugo sous OSX, nous avons besoin d'un gestionnaire de paquet (si ce n'est pas déjà fait), comme [Homebrew](https://brew.sh/index_fr).

Puis on installe hugo :

```
brew install hugo
```

On peut vérifier que hugo s'est correctement installé :

```
> hugo help
hugo is the main command, used to build your Hugo site.
[...]
```


### Utilisation d'Hugo

Une fois Hugo installé correctement, on peut commencer :

```
// crée un nouveau site dans le répertoire $_{SITE_NAME}
hugo new site $_{SITE_NAME}
```

Il ne reste plus qu'à choisir un thème. Hugo les référence [ici](https://themes.gohugo.io/).

J'ai apprécié la simplicité et le design de [Coder](https://themes.gohugo.io/hugo-coder/), tout comme son support de LaTeX à l'aide de [MathJax](https://www.mathjax.org/).

```
git init
git submodule add https://github.com/luizdepra/hugo-coder themes/hugo-coder
```

Il faut maintenant déclarer le thème à utiliser dans `config.toml`, le fichier de configuration d'Hugo.

```
/--- config.toml ---/
theme = "hugo-coder"
```

C'est également dans ce fichier que seront déclarés le  nom du blog et d'autres paramètres dépendants entre autre du thème choisi. Comme les liens réseaux sociaux, les entrées de menu. Coder fournit un exemple de config.toml, ce qui permet une configuration rapide et facile après téléchargement du thème.

#### Rédaction

Après avoir initialisé notre blog et avoir fait le choix du thème, on peut désormais s'atteler à la rédactions des articles. À la racine du project hugo, on créé un article de la manière suivante :

```
hugo new posts/my-first-article.md
```

Cette commande créera un fichier `my-first-article.md` situé dans `content/posts/`, avec divers paramètres dans l'entête :

- title
- date
- draft

Le titre est initialisé à partir du nom donné au fichier, mais peut être changé. Le paramètre `draft` empêche la publication d'un article non terminé. Il faudra le retirer pour l'article soit visible au propre, en production.

Une fois le template généré, on peut enfin commencer la rédaction de notre article.

#### Lancer hugo en développement

En cours de développement, Hugo fournit un serveur permettant de visualiser le contenu du blog, que les posts soient notés comme drafts ou non.

```
hugo server -D
[...]
Web Server is available at http://localhost:$_{PORT_NUMBER}/
```

Une fois cette commande exécuté le blog est disponible à l'adresse donnée dans la console.

#### Compilation du contenu pour la production

Pour la production, le blog est compilé avec la commande `hugo` exécutée à la racine du projet.

Cette commande créé un répertoire `public`, qui peut être servi par n'importe quel serveur web classique (Apache, nginx, etc). C'est ce répertoire que nous utiliserons pour la mise en production de notre blog.

## Déployer son blog en production

On en arrive à la partie cruciale, comment faire pour mettre en ligne notre contenu ?

### Choisir son nom de domaine

Dans un premier temps, pour avoir notre contenu disponible sur une super URL, il faut choisir son nom de domaine. Le but d'un nom de domaine est de permettre l'accès à un serveur sans utiliser son adresse IP. Par exemple, `google.com` est beaucoup plus facile à retenir que `216.58.211.164`.

Il faut donc choisir le nom mais aussi l'extension (`.com`, `.fr`, `.xyz`, etc) que nous souhaitons pour notre blog. Quelques bonne pratiques consistent à éviter les tirets, à avoir un nom court et choisir une extension ne dérivant pas d'une langue, si l'on souhaite toucher le public le plus large possible. Ainsi, un nom de domaine en `.fr` n'est pas vraiment adapté à un public anglophone. L'extension `.xyz`, qui dispose d'une certaine popularité après que Alphabet, la maison mère de Google l'ait choisi pour son [site web](https://abc.xyz), est un choix possible parmi beaucoup d'autres.

### Choisir son hébergement

C'était l'aspect de cette démarche que je connaissais le moins. J'ai décidé un peu aveuglément de prendre un VPS (Virtual Private Server) chez OVH, sans vraiment comparer les prix et le marché.

Le principal problème dans le choix d'Hugo, est que les offres web dîtes classiques ne semblent pas suffire. Il faut en effet installer Go et Hugo et pouvoir paramétrer soit-même son serveur. J'ai cru comprendre que les offres OVH estampillées web ne permettaient pas d'avoir un total accès à un serveur, mais consistaient plutôt en des CMS préinstallés et fournis par leurs soins, comme Wordpress.

Choisir un serveur "privé", c'est pouvoir administrer et installer les outils que l'on souhaite en totale autonomie. Et ça c'est cool (mais un peu plus cher).

### Déployer un blog Hugo

Pour déployer le blog sur mon serveur de production, j'ai d'accord décidé de créer un repository Github pour héberger la codebase. Il me permettra de gérer facilement la transition développement - mise en production.

Il suffit de créer un repository en local, puis de pousser le code sur Github :

```
git remote add origin https://github.com/username/your-blog-repository.git
git push -u origin master
```

Notre code est désormais hébergé sur Github, yay !

#### Installation de hugo sur le serveur de production

Pour installer Hugo sur notre serveur de production, on peut utiliser les paquets Linux :

```
sudo apt-get install hugo
```

Néanmoins, j'ai eu un soucis dans mon cas, la version de hugo présente dans les paquets Debian datait de 2016 et avait un bug de rendu html. Les balises code n'étaient pas bien parsées en html. J'ai donc du installer manuellement depuis les releases fournies sur [Github](https://github.com/gohugoio/hugo/releases).

```
wget https://github.com/gohugoio/hugo/releases/download/v0.40.3/hugo_0.40.3_Linux-64bit.deb
sudo dpkg -i hugo_0.40.3_Linux-64bit.deb
source ~/.bashrc
```



#### Installation & configuration du serveur web

Pour la production, j'ai décidé d'utiliser [Nginx](https://nginx.org/en/) pour servir les fichiers statiques de mon blog.

```
sudo apt-get install nginx
// démarrer nginx
sudo nginx
```

Le répertoire qui nous intéresse maintenant est `/etc/nginx/`. Il contient les fichiers de configuration de Nginx. Après modification de chacun d'entre eux, il suffit généralement de redémarrer le serveur pour que les modifications soient bien prises en compte avec `sudo nginx -s reload`.

Pour configurer correctement nginx avec notre blog, nous devons modifier le fichier `default` au sein du répertoire `/etc/nginx/sites-available`. Vous aurez probablement remarqué qu'un dossier `sites-enabled` est également présent. La différence entre ces deux répertoires est la suivante :

- `sites-available` contient les fichiers vhost de tous les sites disponibles
- `sites-enabled` contient des liens symboliques vers les vhost de `sites-available` que l'on souhaite rendre disponible. En effet, quand on regarde dans `nginx.conf`, on remarque que les fichiers vhost importés sont ceux de `site-enabled`.

Cette fonctionnalité permet de maintenir de multiple configurations vhost sur sa machine, tout en n'en activant qu'une seule partie.

Dans notre cas, la configuration est très simple : nous n'avons qu'un seul vhost à définir. On peut donc modifier le fichier `default` de `sites-available`. Deux choses importantes ici :

- Vérifier que l'on écoute bien sur le port `80`, c'est la ligne `listen 80`.
- Modifier le dossier racine. Il s'agit de la ligne `root`. Normalement elle vaut par défaut `root /var/www/html`. Il faut modifier cette ligne pour pointer vers le répertoire `public` que va nous générer hugo. Par exemple : `root /var/www/html/hugo/public`

Ne pas oublier, après ces configurations, de redémarrer nginx :

```
sudo nginx -s reload
```

Il vous suffit ensuite d'aller cloner le repository du blog à cet endroit et le tour est joué

```
cd /var/www/html
git clone $_{REPO_URL}
```

Admettons que le repository a été cloné sous le nom `hugo`, il faut maintenant changer l'appartenance du répertoire, pour le donner à l'utilisateur courant (et non pas à root, qui possède normalement `/var/www/html`) :

```
sudo chown -R $_{USER} hugo/
```

Ceci permettra de faire des `git pull` ainsi que `hugo` sans les droits root.

Pour finir, il suffit de générer le blog avec hugo :

```
cd blog/
hugo
```

Normalement, si tout s'est bien déroulé, le dossier `public` a été généré par Hugo et nginx devrait à présent servir notre site. Celui-ci doit être accessible à l'aide d'un navigateur, en saisissant l'adresse IP du serveur.

#### Configuration du DNS

Pour pouvoir accéder à notre blog sans passer par l'adresse IP, il faut que notre nom de domaine pointe sur l'adresse IP du serveur, c'est-à-dire changer le DNS. Le DNS est un service qui permet d'établir une sorte de table de correspondance entre un nom de domaine et une adresse IP. Il suffit donc de changer le DNS associé au nom de domaine, pour qu'il pointe sur l'adresse IP de notre serveur. Pour un site web, il faut changer le Type A de son DNS et y mettre notre adresse IP.

## Pour aller plus loin

### Déployer automatiquement grâce aux webhooks

Jusqu'à maintenant, pour déployer une modification il faut :

- Pousser le code sur le repository Github
- Se connecter en ssh sur le serveur de production, hébergeant le blog
- `git pull` les modifications
- `hugo`, pour reconstruire les nouveaux fichiers statiques à servir

C'est assez pénible de devoir systématiquement faire les mêmes actions lors de la moindre modification.

Grâces aux webhooks de Github, on peut automatiser tout ça. L'idée est d'écouter certains événements - par exemple le git push - et à chaque événement, Github va appeler un service situé sur notre serveur pour le notifier du changement (via une requête `HTTP POST`). La configuration des webhooks se fait dans l'onglet 'Settings > Webhooks' des repository Github. Il suffit de choisir l'URL sur laquelle Github enverra la requête, de choisir de ne recevoir que des requêtes de *push*, de choisir du contenu en JSON et éventuellement un *secret*, qui évite le spam de requêtes par d'autres parties.

J'ai utilisé un project open source réalisé en Go : [Githook](https://github.com/arashpayan/githook), qui est un server Go et qui va parser le payload fournit par Github et appeler un script sous certaines conditions.

Que va-t-il se passer après une action de push sur Github ?

- Github requête l'URL fournit dans la configuration
- Le serveur Go (qui tourne sur un autre port que Nginx) reçoit la requête
- Le serveur Go déclenche l'exécution du script de déploiement
- Le script pull les modifications du blog et reconstruit les fichiers statiques.

```
/--- deployment_script.sh ---/
cd /path/to/your/blog/source && git pull && hugo
```

Et le tour est joué !

## Et le prix ?

Pour rendre ce blog accessible en ligne, je paye pour 2 choses : l'hébergement et le nom de domaine.

- Nom de domaine : 11.99€/an
- Hébergement : 4.79€/mois, soit 57.48€/an

## Conclusion

J'espère que cet article aura prouvé qu'il est facile de créer et de maintenir un blog en partant de zéro. Au final, le plus difficile reste de trouver le temps et les idées pour écrire du contenu tout en ne se décourageant pas en chemin.
