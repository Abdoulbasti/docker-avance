# Rendu du projet Docker Avancé

## Environnements déployés

| Environnement | URL |
|---|---|
| Développement | http://localhost:8080/ |
| Production | http://143.198.15.45:8080/ |

Les deux environnements sont fonctionnels.

---

## Documents produits

### `docker-compose.yml` — Environnement de développement

Orchestre tous les services en local. Les images sont **construites directement** depuis les Dockerfiles locaux (`build: context`). Tous les ports sont exposés sur la machine hôte pour faciliter le débogage (MongoDB sur 27017, auth sur 3001, order sur 3002, product sur 3000, frontend sur 8080). Les credentials sont en dur. Inclut un service `init-products` qui initialise la base de données au démarrage. Le frontend utilise `Dockerfile.dev` (serveur Vite en mode hot-reload).

---

### `docker-compose.prod.yml` — Environnement de production

Orchestre les services en production. Les images sont **tirées depuis un registry CI** (`${CI_REGISTRY_IMAGE}/service:${IMAGE_TAG}`). Les secrets sont injectés via variables d'environnement (`.env`). Seuls les ports nécessaires sont exposés publiquement (3000 et 8080). Tous les services ont une politique `restart: unless-stopped`. Les services communiquent via un réseau interne `backend` (bridge), sans exposer MongoDB à l'extérieur.

---

### `frontend/Dockerfile.dev` — Image frontend développement

Image **mono-stage** basée sur `node:25-alpine3.22`. Installe toutes les dépendances (y compris dev), copie les sources et lance `npm run dev` (serveur de développement Vite avec hot-reload). L'utilisateur `node` est utilisé pour limiter les droits.

---

### `frontend/Dockerfile` — Image frontend production

Image **multi-stage** basée sur `node:25-alpine3.22` :
- **Stage 1 (`node-builded`)** : installe toutes les dépendances et compile l'application (`npm run build`) pour générer les fichiers statiques dans `dist/`.
- **Stage 2 (`runtime`)** : repart d'une image propre, n'installe que les dépendances de production (`--omit=dev`), copie uniquement le `dist/` et le `server.cjs` depuis le stage 1. Lance le serveur Express (`node server.cjs`). Image finale allégée, sans les sources ni les outils de build.

---

### `services/auth-service/Dockerfile` — Image auth-service

Image `node:25-alpine3.22`, installe uniquement les dépendances de production (`--omit=dev`), copie les sources, s'exécute en tant qu'utilisateur `node`. Expose le port `3001`. Démarre avec `npm start`.

---

### `services/order-service/Dockerfile` — Image order-service

Identique au Dockerfile de l'auth-service. Expose le port `3002`. Gère les commandes et communique avec le product-service via variable d'environnement `VITE_PRODUCT_SERVICE_URL`.

---

### `services/product-service/Dockerfile` — Image product-service

Identique au Dockerfile de l'auth-service. Expose le port `3000`. Gère le catalogue produits et expose une API REST.
