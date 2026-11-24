# TP4 : Docker et Docker Compose (application MERN)

Ce TP conteneurise une application composée d'un client React (Vite) et d'une API Express, puis l'orchestre avec MongoDB grâce à Docker Compose.

## Prérequis

- Docker Desktop installé et démarré
- Git

## Structure du projet

```
MyAppReact/
├── Client/          # Application React (Vite)
│   ├── Dockerfile
│   └── .dockerignore
├── Server/          # API Express (port 9000)
│   ├── Dockerfile
│   └── .dockerignore
└── docker-compose.yml
```

## Étapes réalisées

### 1. Cloner le projet

```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

### 2. Créer les Dockerfiles

**Server/Dockerfile**

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
ENV MONGO_URI=mongodb://mongodb:27017/mern
CMD ["npm", "start"]
```

**Client/Dockerfile**

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```

Un fichier `.dockerignore` contenant `node_modules` (et `dist` pour le client) évite de copier les dépendances locales dans l'image.

Construction des images :

```bash
docker build -t mern-server ./Server
docker build -t mern-client ./Client
```

### 3. Créer un réseau Docker

```bash
docker network create mern-network
```

### 4. Lancer MongoDB

```bash
docker run -d --name mongodb --network mern-network mongo
```

### 5. Lancer le serveur et le client

```bash
docker run -d --name server --network mern-network -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

Vérification : `docker ps` affiche les trois conteneurs, http://localhost:9000/api/health répond `{"message":"API is running"}` et http://localhost:3000 affiche l'application.

### 6. Docker Compose

Avant de lancer Compose, les conteneurs et le réseau créés manuellement ont été supprimés :

```bash
docker rm -f client server mongodb
docker network rm mern-network
```

Le fichier `docker-compose.yml` à la racine définit trois services (`mongodb`, `server`, `client`), le réseau `mern-network` (driver bridge), les dépendances (`server` dépend de `mongodb`, `client` dépend de `server`) et la variable `MONGO_URI` du serveur.

```yaml
services:
  mongodb:
    image: mongo
    container_name: mongodb
    networks:
      - mern-network

  server:
    build: ./Server
    container_name: server
    ports:
      - "9000:9000"
    environment:
      - MONGO_URI=mongodb://mongodb:27017/mern
    depends_on:
      - mongodb
    networks:
      - mern-network

  client:
    build: ./Client
    container_name: client
    ports:
      - "3000:3000"
    depends_on:
      - server
    networks:
      - mern-network

networks:
  mern-network:
    driver: bridge
```

Lancer l'application :

```bash
docker compose up --build
```

Arrêter et supprimer les conteneurs :

```bash
docker compose down
```

## Remarques

- Les dossiers du dépôt s'appellent `Client` et `Server` (avec une majuscule).
- Dans les conteneurs, les services communiquent par leur nom : le serveur joint MongoDB via l'hôte `mongodb`.
- Le serveur fourni n'utilise pas encore MongoDB (il expose seulement `/api/health`). MongoDB est donc démarré mais pas interrogé.
- Le client appelle l'API sur `http://localhost:9000` depuis le navigateur, ce qui fonctionne car le port 9000 est publié.
