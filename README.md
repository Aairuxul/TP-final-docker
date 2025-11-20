# Projet Final - Stack Spring Boot / Frontend JS / PostgreSQL

Projet réalisé par Valentin Russeil et Mattéo Pereira.

## Objectif du projet final

Assembler et exécuter une **application web complète** composée de trois services :

-   **Backend :** API REST Spring Boot
-   **Frontend :** application React ou Vue
-   **Base de données :** PostgreSQL

L’objectif est de conteneuriser chaque service, les orchestrer avec **Docker Compose**, et garantir la persistance des
données ainsi que la bonne communication entre les services.

---

## Architecture Globale

La stack se compose de trois services principaux orchestrés par `docker compose` :

-   **API (Backend)**: `spring-api` — application Spring Boot qui fournit une API REST pour gérer les ressources
    (`Item`). Elle est construite avec un `Dockerfile` multi-stage et écoute sur le port `8080` (accessible via le
    réseau Docker et le reverse-proxy).
-   **Frontend (Web)**: `webapp` — application JavaScript (Vite + React) qui est buildée puis servie par une image
    Nginx. Le frontend est accessible via le reverse-proxy (port `80` sur l'hôte).
-   **Base de données (PostgreSQL)**: service `db` — stocke les données persistantes. Les données sont conservées via le
    volume Docker nommé `pgdata`.

Commande pour démarrer le projet :

```
docker compose up -d
```

Pour tester :

-   Frontend : `http://localhost/`
-   Backend (via proxy) : `http://localhost/api/`

Autres informations :

-   Fichier `.env` pour les secrets (mot de passe DB, utilisateurs) à créer en se basant sur le `.env.example`.
-   En mode dev; utilisation d'un reseau interne via le réseau Docker.

## Commandes pour builder et lancer

-   Construire et démarrer la stack :

```bash
docker compose up -d --build
```

-   Rebuilder les images :

```bash
docker compose build
```

-   Lancer les services :

```bash
docker compose up -d
```

-   Couper les services :

```bash
docker compose down
```

- S'assurer que le serveur est bien lancé avec :

```bash
docker logs -f tp-spring-api-1
```
Veillez à bien attendre que la base de données affiche son contenu avant de tester si tout fonctionne.

## Endpoints API et URLs

-   Frontend : `http://localhost/` (reverse-proxy)
-   Backend (base URL proxied) : `http://localhost/api/` (via reverse-proxy)

Endpoints implémentés dans l'API :

-   `GET /api/health` — vérifie l'état de l'API (retourne `{ "status": "ok" }`).
-   `GET /api/items` — récupère la liste de tous les items. (Ce sera vide pour notre cas)
-   `POST /api/items` — crée un nouvel item (corps JSON avec les champs de `Item`).

Note: les contrôleurs n'exposent plus `@CrossOrigin`; le reverse-proxy centralisé résout les problèmes CORS en gérant
les en-têtes et les préflight OPTIONS.

## Problèmes rencontrés et solutions

Voici les problèmes que nous avons pu rencontrer et les solutions que nous avons touvées :

-   Nous avons découvert le reverse proxy et avons mis un peu de temps à comprendre comme ça marchait réellement

## Tâches à réaliser

1. Écrire les `Dockerfile` pour le backend (multi-stage) et le frontend (build + Nginx).
    - Chaque dossier contiendra son propre `Dockerfile`.
2. Créer le fichier `.env` pour les secrets.
3. Écrire le `docker-compose.yml` complet (API, Web, DB).
4. Tester le bon fonctionnement de la stack :
    - API accessible via le reverse-proxy : `http://localhost/api/`
    - Frontend sur `http://localhost/` (reverse-proxy)
    - Persistance PostgreSQL via volume.
5. Ecrire une documentation claire et précise.

---

## Tests et validation

<p></p>

1️⃣ Lancer la stack :

```bash
docker compose up -d --build
```

2️⃣ Vérifier que tout fonctionne :

-   Frontend disponible sur [http://localhost/](http://localhost/)
-   API accessible via le proxy : [http://localhost/api/health](http://localhost/api/health)
-   PostgreSQL persistant via le volume `pgdata`

## Bonus (optionnel)

<p></p>

💡 Pour aller plus loin :

-   Ajouter un **service pgAdmin** pour visualiser la base.
-   Ajouter un **reverse proxy Nginx** entre le frontend et le backend.
-   Configurer une **intégration CI/CD** pour tester et builder la stack automatiquement.

> Notifier les bonus effectués dans la documentation.
