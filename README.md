# Configuration Docker Compose pour Traefik et WordPress

## Aperçu

Ce dépôt contient deux configurations Docker Compose pour mettre en place un reverse proxy avec Traefik et un site WordPress avec une base de données MySQL.

## Fichiers

- `compose.yml` : Configuration pour le reverse proxy Traefik.
- `compose (1).yml` : Configuration pour le site WordPress et la base de données MySQL.

## Prérequis

- Docker
- Docker Compose

## Détails de la Configuration

### Reverse Proxy Traefik

**Fichier:** `compose.yml`

Ce fichier configure Traefik en tant que reverse proxy.

- **Service:** `reverse-proxy`
  - **Image:** `traefik:v2.10`
  - **Nom du Conteneur:** `traefik`
  - **Politique de Redémarrage:** `unless-stopped`
  - **Variables d'Environnement:** 
    - `COMPOSE_PROJECT_NAME=stellarcat`
  - **Options de Sécurité:** 
    - `no-new-privileges:true`
  - **Commandes:**
    - Configure les points d'entrée et les fournisseurs pour Traefik
  - **Ports:** 
    - `80:80` (HTTP)
  - **Volumes:**
    - `/var/run/docker.sock:/var/run/docker.sock:ro`
  - **Hôtes Supplémentaires:** 
    - `host.docker.internal:host-gateway`
  - **Réseaux:**
    - `proxy-main` (externe)
    - `default` (bridge)

### WordPress et MySQL

**Fichier:** `compose (1).yml`

Ce fichier configure un site WordPress avec une base de données MySQL.

- **Service:** `stellarcat-wordpress-db`
  - **Image:** `mysql:5.7.40`
  - **Nom du Conteneur:** `stellarcat-wordpress-db`
  - **Volumes:** 
    - `db_data:/var/lib/mysql`
  - **Politique de Redémarrage:** `always`
  - **Variables d'Environnement:**
    - `MYSQL_ROOT_PASSWORD=wordpress`
    - `MYSQL_DATABASE=wordpress`
    - `MYSQL_USER=wordpress`
    - `MYSQL_PASSWORD=wordpress`
  - **Réseaux:**
    - `wordpress-net`

- **Service:** `stellarcat-wordpress`
  - **Image:** `wordpress:6.4.3-php8.2-apache`
  - **Nom du Conteneur:** `stellarcat-wordpress`
  - **Dépend de:** 
    - `stellarcat-wordpress-db`
  - **Politique de Redémarrage:** `always`
  - **Variables d'Environnement:**
    - `WORDPRESS_DB_HOST=stellarcat-wordpress-db`
    - `WORDPRESS_DB_USER=wordpress`
    - `WORDPRESS_DB_PASSWORD=wordpress`
    - `WORDPRESS_DB_NAME=wordpress`
  - **Labels:**
    - Configure le service pour Traefik
  - **Réseaux:**
    - `proxy-main` (externe)
    - `wordpress-net`

### Réseaux

- `proxy-main` : Réseau externe utilisé par Traefik et WordPress.
- `default` : Réseau bridge pour le service Traefik.
- `wordpress-net` : Réseau local pour WordPress et MySQL.

### Volumes

- `db_data` : Stockage persistant pour les données de la base de données MySQL.

## Utilisation

1. **Démarrer Traefik:**

   ```sh
   docker-compose -f compose.yml up -d
