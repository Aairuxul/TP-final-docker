# Projet Final - Stack Spring Boot / Frontend JS / PostgreSQL

Projet réalisé par Valentin Russeil et Mattéo Pereira.

## Objectif du projet final

Assembler et exécuter une **application web complète** composée de trois services :

* **Backend :** API REST Spring Boot
* **Frontend :** application React ou Vue
* **Base de données :** PostgreSQL

L’objectif est de conteneuriser chaque service, les orchestrer avec **Docker Compose**, et garantir la persistance des données ainsi que la bonne communication entre les services.

---

## Architecture Globale

La stack se compose de trois services principaux orchestrés par `docker-compose` :

- **API (Backend)**: `spring-api` — application Spring Boot qui fournit une API REST pour gérer les ressources (`Item`). Elle est construite avec un `Dockerfile` multi-stage et expose le port `8080`.
- **Frontend (Web)**: `webapp` — application JavaScript (Vite + React) qui est buildée puis servie par une image Nginx. Le frontend s'expose sur `localhost:8081` (mapping habituel `8081:80`).
- **Base de données (PostgreSQL)**: service `db` — stocke les données persistantes. Les données sont conservées via le volume Docker nommé `pgdata`.

Commande pour démarrer la stack :
```
docker compose up -d --build
```

Pour tester :
- Frontend : `http://localhost:8081`
- Backend : `http://localhost:8080`

Autres informations :
- Fichier `.env` pour les secrets (mot de passe DB, utilisateurs).
- Ne pas exposer PostgreSQL en production ; laisser la base accessible uniquement via le réseau Docker.

## Commandes pour builder et lancer

- Construire et démarrer la stack :
```bash
docker compose up -d --build
```
- Rebuilder les images :
```bash
docker compose build
```
- Lancer les services :
```bash
docker compose up -d
```
- Couper les services :
```bash
docker compose down
```


## Endpoints API et URLs

- Frontend : `http://localhost:8081`
- Backend (base URL) : `http://localhost:8080`

Endpoints implémentés dans l'API :
- `GET /api/health` — vérifie l'état de l'API (retourne `{ "status": "ok" }`).
- `GET /api/items` — récupère la liste de tous les items.
- `POST /api/items` — crée un nouvel item (corps JSON avec les champs de `Item`).

Note: les contrôleurs autorisent les requêtes cross-origin (`@CrossOrigin(origins = "*")`) pour faciliter le développement local.

## Problèmes rencontrés et solutions

Voici les problèmes que nous avons pu rencontrer et les solutions que nous avons touvées :

- Docker Compose : orchestration simple pour développement et tests locaux ; facilite la montée en charge d'une stack multi-service.
- Multi-stage Dockerfile (Backend) : permet de produire une image finale légère sans inclure les outils de build.
- `.env` pour secrets : séparer la configuration de l'image, plus modulable pour les changements de nom. Présence d'un .`env.example` pour créer le template à modifier

## Tâches à réaliser

1. Écrire les `Dockerfile` pour le backend (multi-stage) et le frontend (build + Nginx).
   - Chaque dossier contiendra son propre `Dockerfile`.
2. Créer le fichier `.env` pour les secrets.
3. Écrire le `docker-compose.yml` complet (API, Web, DB).
4. Tester le bon fonctionnement de la stack :
   * API accessible sur `localhost:8080`
   * Frontend sur `localhost:8081`
   * Persistance PostgreSQL via volume.
5. Ecrire une documentation claire et précise.

---

## Tests et validation

<p></p>

1️⃣ Lancer la stack :

```bash
docker compose up -d --build
```

2️⃣ Vérifier que tout fonctionne :

* Backend disponible sur [http://localhost:8080](http://localhost:8080)
* Frontend disponible sur [http://localhost:8081](http://localhost:8081)
* PostgreSQL persistant via le volume `pgdata`

3️⃣ Consulter les logs si besoin :

```bash
docker compose logs -f
```

---

## Bonus (optionnel)

<p></p>

💡 Pour aller plus loin :

* Ajouter un **service pgAdmin** pour visualiser la base.
* Ajouter un **reverse proxy Nginx** entre le frontend et le backend.
* Configurer une **intégration CI/CD** pour tester et builder la stack automatiquement.

> Notifier les bonus effectués dans la documentation.


