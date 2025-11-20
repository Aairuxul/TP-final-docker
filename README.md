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

```mermaid
graph TB
    subgraph " "
        User["👤 UTILISATEUR"]
    end

    User -->|"HTTP :80"| ReverseProxy

    subgraph Docker["🐳 Docker Compose"]
        ReverseProxy["🔀 REVERSE PROXY<br/>nginx:stable-alpine<br/>✓ Healthcheck: /api/health"]

        ReverseProxy -->|"/ (root)"| Frontend
        ReverseProxy -->|"/api/*"| Backend

        Frontend["⚛️ FRONTEND<br/>Vite + React + Nginx<br/>Port: 80 (interne)<br/>✓ Healthcheck: /"]

        Backend["☕ BACKEND<br/>Spring Boot (Java 21)<br/>Port: 8080 (interne)<br/>✓ Healthcheck: /api/health"]

        Backend -->|"JDBC :5432"| Database

        Database["🗄️ DATABASE<br/>PostgreSQL 16 Alpine<br/>Port: 5432 (interne)<br/>💾 Volume: pgdata<br/>✓ Healthcheck: pg_isready"]
    end

    style User fill:#1a1d29,stroke:#58a6ff,stroke-width:2px,color:#c9d1d9
    style ReverseProxy fill:#2d1b0e,stroke:#ff9800,stroke-width:3px,color:#c9d1d9
    style Frontend fill:#0d2818,stroke:#3fb950,stroke-width:2px,color:#c9d1d9
    style Backend fill:#2b1a24,stroke:#f85149,stroke-width:2px,color:#c9d1d9
    style Database fill:#221a2d,stroke:#a371f7,stroke-width:2px,color:#c9d1d9
    style Docker fill:#0d1117,stroke:#1f6feb,stroke-width:3px,stroke-dasharray: 5 5,color:#c9d1d9
```

**Légende** :

-   🔀 **Reverse Proxy** : Point d'entrée unique (port 80)
-   ⚛️ **Frontend** : Interface utilisateur React
-   ☕ **Backend** : API REST Spring Boot
-   🗄️ **Database** : Base de données PostgreSQL avec persistance
-   ✓ Tous les services ont des healthchecks
-   🐳 Tous les services communiquent via le réseau Docker Bridge

### Description des services

-   **API (Backend)**: `spring-api` — application Spring Boot (Java 21) qui fournit une API REST pour gérer les
    ressources (`Item`). Elle est construite avec un `Dockerfile` multi-stage et écoute sur le port `8080` (accessible
    uniquement via le réseau Docker interne). Dispose d'un healthcheck sur `/api/health`. Restart policy:
    `unless-stopped`.
-   **Frontend (Web)**: `webapp` — application JavaScript (Vite + React) qui est buildée puis servie par Nginx.
    Accessible uniquement via le reverse-proxy. Dispose d'un healthcheck. Restart policy: `unless-stopped`.
-   **Reverse Proxy**: `reverse-proxy` — Nginx qui expose le port `80` sur l'hôte et route `/` vers le frontend et
    `/api/` vers le backend. C'est le seul point d'entrée public. Dispose d'un healthcheck. Restart policy: `always`.
-   **Base de données (PostgreSQL)**: service `db` (PostgreSQL 16 Alpine) — stocke les données persistantes. Les données
    sont conservées via le volume Docker nommé `pgdata`. Dispose d'un healthcheck pour vérifier la disponibilité.
    Restart policy: `always`.

Commande pour démarrer le projet :

```
docker compose up -d
```

Pour tester :

-   Frontend : `http://localhost/`
-   Backend (via proxy) : `http://localhost/api/`

Autres informations :

-   Fichier `.env` pour les secrets (mot de passe DB, utilisateurs) à créer en se basant sur le `.env.example`.
-   Utilisation d'un réseau Bridge Docker par défaut pour la communication entre services.
-   Tous les services disposent de healthchecks pour garantir leur bon démarrage.
-   Les dépendances entre services sont gérées via `depends_on` avec conditions `service_healthy`.
-   Le reverse proxy gère les en-têtes CORS et les requêtes preflight OPTIONS.
-   **Restart policies** : `always` pour la DB et le reverse-proxy, `unless-stopped` pour l'API et le frontend.
-   Un fichier `docker-compose.override.yml` est disponible pour le développement local (voir section dédiée).

## Mode Développement (docker-compose.override.yml)

Le fichier `docker-compose.override.yml` permet de modifier le comportement de la stack pour le développement local. Il
est automatiquement fusionné avec `docker-compose.yml` lors de l'exécution de `docker compose up`.

### Modifications apportées en mode dev :

-   **Backend (spring-api)** :

    -   Port `8080` exposé directement sur l'hôte (accessible via `http://localhost:8080`)
    -   Variable d'environnement `SPRING_PROFILES_ACTIVE=dev` activée
    -   Permet le debugging et le hot-reload

-   **Frontend (webapp)** :

    -   Utilise le stage `dev` du Dockerfile multi-stage
    -   Commande `npm run dev -- --host` pour lancer Vite en mode développement
    -   Port `5173` exposé sur l'hôte (accessible via `http://localhost:5173`)
    -   Hot Module Replacement (HMR) activé pour le développement React

-   **Reverse Proxy** :
    -   Désactivé par défaut via le profile `with-proxy`
    -   Pour l'activer : `docker compose --profile with-proxy up -d`
    -   En mode dev, l'accès direct aux services est privilégié

### Commandes en mode développement :

```bash
# Lancer la stack en mode dev (sans reverse-proxy)
docker compose up -d

# Lancer la stack en mode dev avec le reverse-proxy
docker compose --profile with-proxy up -d

# Accès direct aux services en mode dev
# Frontend: http://localhost:5173
# Backend: http://localhost:8080
# Reverse Proxy (si activé): http://localhost:80
```

> **Note** : En mode développement, les services sont accessibles directement, ce qui facilite le debugging. En
> production, utilisez uniquement le `docker-compose.yml` sans override.

---

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

-   S'assurer que le serveur est bien lancé avec :

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
requêtes passent par le reverse-proxy. Les contrôleurs Spring n'exposent plus `@CrossOrigin` ; le reverse-proxy gère les
en-têtes CORS et les requêtes preflight OPTIONS.

## Problèmes rencontrés et solutions

Voici les problèmes que nous avons pu rencontrer et les solutions que nous avons trouvées :

-   Nous avons découvert le reverse proxy et avons mis un peu de temps à comprendre comme ça marchait réellement
-   Nous avons vu que la connexion entre le front et back n'était pas présente. La solution se trouvait dans le fait
    d'avoir le reverse proxy qui fonctionne mieux et notre docker compose qui ne gere pas les ports vu que seul le
    reverse proxy agit dessus.
-   Nous avons eu quelques difficultés avec le docker-compose.override.

## Tâches réalisées

✅ 1. Écriture des `Dockerfile` pour le backend (multi-stage) et le frontend (build + Nginx). - Chaque dossier contient
son propre `Dockerfile`.

✅ 2. Création du fichier `.env` pour les secrets (à créer à partir du `.env.example`).

✅ 3. Écriture du `docker-compose.yml` complet (API, Web, DB, Reverse Proxy).

✅ 4. Tests de bon fonctionnement de la stack : - API accessible via le reverse-proxy : `http://localhost/api/` -
Frontend sur `http://localhost/` (reverse-proxy) - Persistance PostgreSQL via volume.

✅ 5. Documentation claire et précise rédigée.

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

## Bonus réalisés

✅ **Reverse proxy Nginx** : Implémenté avec succès pour gérer le routage entre le frontend (`/`) et le backend
(`/api/`). Le reverse proxy gère également les en-têtes CORS et les requêtes OPTIONS.

✅ **Healthchecks** : Tous les services disposent de healthchecks pour garantir leur disponibilité avant que les
services dépendants ne démarrent.

✅ **Configuration optimisée** : Utilisation de `depends_on` avec conditions `service_healthy` pour orchestrer le
démarrage des services dans le bon ordre.
