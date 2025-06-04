# TP Docker : Mise en place de services dans des conteneurs

## Exécuter un serveur web (apache, nginx, …) dans un conteneur Docker

a. Récupérer l’image sur le Docker Hub  
```bash
docker pull nginx
```

b. Vérifier que cette image est présente en local  
```bash
docker images
```

c. Créer un fichier `index.html` simple  
```bash
mkdir nginx
echo "<h1>Hello World</h1>" > nginx/index.html
```

d. Démarrer un conteneur et servir la page HTML créée précédemment à l’aide d’un volume  
```bash
cd nginx
docker run -d --name nginx -p 80:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html nginx
```

e. Supprimer le conteneur précédent et arriver au même résultat en utilisant `docker cp`  
```bash
docker rm -f nginx
docker run --name nginx -d -p 80:80 nginx
docker cp ./index.html nginx:/usr/share/nginx/html/index.html
```

## Builder une image

a. À l’aide d’un Dockerfile, créer une image  
```bash
# On crée le Dockerfile dans le dossier TP1/Q5
docker build -t nginx-custom .
```

b. Exécuter cette nouvelle image pour servir la page HTML  
```bash
docker run --name nginx -d -p 80:80 nginx-custom
```

c. Quelles différences entre les procédures 5. et 6. ?  

Le `docker build` permet un déploiement entièrement automatisé du serveur web nginx en copiant dès le build de l'image le fichier `index.html`, tandis qu’avec la première méthode, il fallait indiquer dans la commande `run` que l'on voulait copier le fichier.

## Utiliser une base de données dans un conteneur Docker

a. Récupérer les images `mysql:5.7` et `phpmyadmin/phpmyadmin` depuis Docker Hub  
```bash
docker pull mysql:5.7 && docker pull phpmyadmin/phpmyadmin
```

b. Exécuter deux conteneurs pour MySQL et phpMyAdmin, puis ajouter une table via l’interface  

Création du réseau :  
```bash
docker network create tp
```

Lancement de MySQL (version 8 utilisée à la place de 5.7 à cause de bugs) :  
```bash
docker run -d --name mysql --network tp \
  -e MYSQL_ROOT_PASSWORD=monmotdepasse \
  -e MYSQL_ROOT_HOST='%' \
  --memory=1g \
  mysql:8
```

Lancement de phpMyAdmin :  
```bash
docker run -d --name phpmyadmin --network tp \
  -e PMA_HOST=mysql \
  -e PMA_PORT=3306 \
  -p 8080:80 \
  phpmyadmin/phpmyadmin
```

Connexion à `http://localhost:8080/` pour créer graphiquement les entrées dans la base.

## Utiliser un fichier `docker-compose.yml`

(voir le fichier dans le dossier `TP1/Q8`)

a. Avantages du fichier `docker-compose` par rapport aux commandes `docker run`  

Le fichier `docker-compose` permet de définir et lancer plusieurs conteneurs avec une seule commande. Il centralise la configuration (ports, volumes, réseaux, variables d’environnement) dans un fichier lisible. Cela simplifie l’orchestration, assure la reproductibilité et facilite la maintenance, contrairement aux commandes `docker run` répétitives et sujettes à erreur.

b. Moyens de configuration de MySQL au lancement  

On utilise des variables d’environnement (`MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`) dans le fichier `docker-compose.yml` pour automatiser la configuration du conteneur MySQL (utilisateur, mot de passe, base).

## Observation de l’isolation réseau entre 3 conteneurs

a. Utilisation de `docker-compose` avec l’image `praqma/network-multitool` pour créer 3 services (`web`, `app`, `db`) et 2 réseaux (`frontend`, `backend`). Les services `web` et `db` ne doivent pas pouvoir se pinguer.

(voir le `docker-compose.yml` dans `TP1/Q9`)

b. Lignes de la commande `docker inspect` qui justifient le comportement :

```bash
docker inspect q9-db-1
```

Montre que `db` est uniquement sur le réseau `backend`.

```bash
docker inspect q9-app-1
```

Montre que `app` est sur les réseaux `frontend` **et** `backend`.

```bash
docker inspect q9-web-1
```

Montre que `web` est uniquement sur le réseau `frontend`.

c. Cas d’usage réel d’une telle configuration réseau  

Dans une architecture multi-niveaux (3-tier), on sépare :  
- le **frontend** (ex. nginx, apache) pour l’interface utilisateur  
- le **middleware** (ex. Node.js, Django) pour la logique métier  
- le **backend** (ex. PostgreSQL, MySQL) pour les données  

Le frontend communique uniquement avec le middleware via HTTP/HTTPS, et seul ce dernier accède à la base. Cela permet une meilleure isolation, sécurité, et organisation modulaire du système.
