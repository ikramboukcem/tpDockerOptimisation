TP Optimisation Docker


                        #Étape 0 — Baseline#

But : Image de départ sans optimisation pour avoir un point de comparaison.

Dockerfile utilisé :

FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node","server.js"]


Commandes :

docker build -t optapp:baseline .
docker run -d -p 3000:3000 --name opt-baseline optapp:baseline
docker images optapp


Taille obtenue : ~1.6GB

                     #Étape 1 — Image plus légère#

But : Utiliser une image de base plus légère et n’installer que les dépendances nécessaires en production.

Dockerfile utilisé :

FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node","server.js"]


Commandes :

git checkout -b opt1
docker build -t optapp:opt1 .
docker run -d -p 3000:3000 --name opt1 optapp:opt1
docker images optapp
git add Dockerfile
git commit -m "opt1: node:20-alpine + npm install --production"
git push -u origin opt1


Taille obtenue : ~214MB

                             #Étape 2 — Multi-stage build#

But : Séparer la phase de build et la phase d’exécution pour ne garder que ce qui est nécessaire dans l’image finale.

Dockerfile utilisé :

FROM node:20-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app .
RUN npm prune --production
EXPOSE 3000
CMD ["node","server.js"]


Commandes :

git checkout -b opt2
docker build -t optapp:opt2 .
docker run -d -p 3000:3000 --name opt2 optapp:opt2
docker images optapp
git add Dockerfile
git commit -m "opt2: multi-stage build"
git push -u origin opt2


Taille obtenue : ~210MB

                              #Étape 3 — Ignorer les fichiers inutiles#

But : Réduire le contexte de build pour accélérer la construction et alléger l’image.

.dockerignore ajouté :

node_modules
npm-debug.log
.git
.gitignore
Dockerfile
.dockerignore


Commandes :

git checkout -b opt3
docker build -t optapp:opt3 .
docker run -d -p 3000:3000 --name opt3 optapp:opt3
docker images optapp
git add .dockerignore
git commit -m "opt3: ajout .dockerignore"
git push -u origin opt3


Taille obtenue : ~210MB

                    #Étape 4 — Mode production et utilisateur non-root#

But : Sécuriser et améliorer les performances en exécutant en mode production avec un utilisateur non-root.

Dockerfile utilisé :

FROM node:20-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app .
RUN npm prune --production
USER node
EXPOSE 3000
CMD ["node","server.js"]


Commandes :

git checkout -b opt4
docker build -t optapp:opt4 .
docker run -d -p 3000:3000 --name opt4 optapp:opt4
docker images optapp
git add Dockerfile
git commit -m "opt4: production + user non-root"
git push -u origin opt4


Taille obtenue : ~210MB