# Inception

*Ce projet a été réalisé dans le cadre du cursus de l'école 42.*

## Description

**Inception** est un projet d'administration système qui consiste à mettre en place une petite infrastructure web composée de plusieurs services, chacun isolé dans son propre **conteneur Docker**. L'ensemble est orchestré avec **Docker Compose** et tourne à l'intérieur d'une machine virtuelle.

L'objectif est de comprendre la conteneurisation, la création d'images Docker personnalisées, la communication entre conteneurs via un réseau dédié, la persistance des données avec les volumes et la sécurisation des connexions avec TLS.

## Prototype

```bash
make
```

Le projet se lance avec le `Makefile` qui construit les images et démarre les conteneurs avec `docker compose`.

## Services

| Service | Description |
|---------|-------------|
| NGINX | Serveur web, unique point d'entrée de l'infrastructure (port 443, TLSv1.2 / TLSv1.3) |
| WordPress + php-fpm | Site WordPress, sans NGINX intégré |
| MariaDB | Base de données utilisée par WordPress, sans NGINX intégré |

## Architecture

| Élément | Description |
|---------|-------------|
| Conteneurs | Un conteneur par service, construit depuis un `Dockerfile` personnalisé |
| Réseau | Un réseau Docker dédié relie les conteneurs entre eux |
| Volume WordPress | Stocke les fichiers du site WordPress |
| Volume MariaDB | Stocke les données de la base de données |
| Point d'entrée | Seul le port **443** est exposé, via NGINX |

Schéma de l'infrastructure :

```
        Internet
           |
        port 443
           |
      [ NGINX ] ---- TLSv1.2 / TLSv1.3
           |
      [ WordPress + php-fpm ]
           |
      [ MariaDB ]
```

## Règles du projet

| Règle | Description |
|-------|-------------|
| Images | Chaque image porte le nom du service correspondant |
| Dockerfile | Un `Dockerfile` écrit à la main par service |
| Base | Debian ou Alpine (avant-dernière version stable) |
| Images prêtes | Interdit d'utiliser des images toutes faites (hors Alpine/Debian) |
| Redémarrage | Les conteneurs redémarrent automatiquement en cas de crash |
| Boucles infinies | Interdit d'utiliser `tail -f`, `sleep infinity`, `while true` |
| Mots de passe | Aucun mot de passe dans les `Dockerfile` |
| Variables | Utilisation de variables d'environnement et d'un fichier `.env` |
| Domaine | Le domaine `login.42.fr` pointe vers l'adresse IP locale |
| Utilisateurs | Deux utilisateurs WordPress, dont un administrateur (sans `admin` dans le nom) |

## Structure du projet

| Dossier / Fichier | Description |
|-------------------|-------------|
| `Makefile` | Construit et gère toute l'infrastructure |
| `srcs/docker-compose.yml` | Définit les services, réseaux et volumes |
| `srcs/.env` | Variables d'environnement (non versionné) |
| `srcs/requirements/nginx` | Dockerfile et configuration de NGINX |
| `srcs/requirements/wordpress` | Dockerfile et configuration de WordPress |
| `srcs/requirements/mariadb` | Dockerfile et configuration de MariaDB |
| `secrets/` | Mots de passe et identifiants sensibles |

## Volumes

| Volume | Emplacement sur l'hôte |
|--------|------------------------|
| WordPress | `/home/login/data/wordpress` |
| MariaDB | `/home/login/data/mariadb` |

## Compilation

```bash
make        # construit les images et démarre les conteneurs
make down   # arrête les conteneurs
make clean  # arrête et supprime les conteneurs et le réseau
make fclean # supprime aussi les images et les volumes
make re     # reconstruit entièrement l'infrastructure
```

## Utilisation

Ajouter le domaine dans le fichier `/etc/hosts` :

```bash
127.0.0.1   login.42.fr
```

Lancer l'infrastructure :

```bash
make
```

Puis accéder au site depuis un navigateur :

```
https://login.42.fr
```

Commandes utiles pour vérifier l'état des conteneurs :

```bash
docker ps
docker compose -f srcs/docker-compose.yml logs
```

## Normes

- Respect du sujet et de l'architecture imposée
- Aucun mot de passe en clair dans le dépôt
- Fichier `.env` et dossier `secrets/` exclus du dépôt avec `.gitignore`
- Utilisation uniquement de TLSv1.2 ou TLSv1.3

## Auteur

@prsadamm
