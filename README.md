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

La stack se compose de quatre services orchestrés par `docker compose` :

-   **API (Backend)**: `spring-api` — application Spring Boot (Java 21) qui fournit une API REST pour gérer les ressources
    (`Item`). Elle est construite avec un `Dockerfile` multi-stage et écoute sur le port `8080` (accessible uniquement
    via le réseau Docker interne). Restart policy : `unless-stopped`.
-   **Frontend (Web)**: `webapp` — application JavaScript (Vite + React) qui est buildée puis servie par Nginx.
    Accessible uniquement via le reverse-proxy. Restart policy : `unless-stopped`.
-   **Reverse Proxy**: `reverse-proxy` — Nginx qui expose le port `80` sur l'hôte et route `/` vers le frontend et `/api/`
    vers le backend. C'est le seul point d'entrée public. Restart policy : `always`.
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

-   Redémarrer les services (sans rebuild) :

```bash
docker compose restart
```

-   Couper les services (conserve les volumes) :

```bash
docker compose down
```

-   Couper et supprimer les volumes (⚠️ perte des données DB) :

```bash
docker compose down -v
```

- S'assurer que le serveur est bien lancé avec :

```bash
docker logs -f tp-spring-api-1
```
Veillez à bien attendre que la base de données affiche son contenu avant de tester si tout fonctionne.

## Endpoints API et URLs

-   Frontend : `http://localhost/` (reverse-proxy sur port 80)
-   Backend (base URL proxied) : `http://localhost/api/` (via reverse-proxy)

Endpoints implémentés dans l'API :

-   `GET /api/health` — vérifie l'état de l'API (retourne `{ "status": "ok" }`).
-   `GET /api/items` — récupère la liste de tous les items.
-   `POST /api/items` — crée un nouvel item (corps JSON avec les champs de `Item`).

**Important** : Le frontend utilise des URLs relatives (`/api/...`) pour appeler l'API, ce qui garantit que toutes les
requêtes passent par le reverse-proxy. Les contrôleurs Spring n'exposent plus `@CrossOrigin` ; le reverse-proxy gère les en-têtes CORS et les requêtes preflight OPTIONS.

## Problèmes rencontrés et solutions

Voici les problèmes que nous avons pu rencontrer et les solutions que nous avons trouvées :

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
