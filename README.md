## Créer le projet 
npx create-react-app <nom de votre projet>

## Github actions générer le main.yml de base
aller dans votre git puis dans l'onglet actions et suivez les instrutions

## Modifier le .yml pour votre CI CD
### Nommer votre ci cd 
name: <NOM>
### Nommer votre ci cd 
on:
  push:
    branches: [ <Nom de votre branche par défaut: master> ]

### Build votre projet react sur un ubuntu et installer les dépendances
jobs:
  build-react:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v1
        with:
          node-version: 16.x
      - name: Install dependencies
        run: npm install
      - name: Build Artefact 
        run: npm run build

### Executer les tests de base de votre projet (déjà inclut à la création de l'app)
  test-react:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v1
        with:
          node-version: 16.x
      - name: Install dependencies
        run: npm install
      - name: Run test 
        run: npm run test

### variables secret
Dans les settings de votre projet dans git, il vous faut rentrer les variables d'environnement du votre serveur
ainsi que la clé privée de votre poste

### Ajout de la variable ssh privée
      - name: ajout Known Hosts
        run: ssh-keyscan -H ${{ secrets.HOST }} >> ~/.ssh/known_hosts

### On copie dans dossier build avec rsync en ssh avec notre login du serveur distant et l'ip de ce serveur dans le dossier /web/build
      - name: run deploy
        run: rsync -avz ./build/ ${{ secrets.USERNAME }}@${{ secrets.HOST }}:~/web/build

### Versionning
Si vous souhaitez ajouter un système de version très basique vous pouvez renommer votre dossier build après le deploiement
      - name: renom dossier build
        run: ssh ${{ secrets.USERNAME }}@${{ secrets.HOST }} 'mv ~/web/build ~/web/build-$(date +%Y%m%d_%H%M%S)'


## Fichier Dockerfile
Tout d'abord il faut créer le fichier Dockerfile avec un D majuscule et sans extension !

### Correspond à la techno que vous allez utiliser pour 
FROM node:14-alpine

### Cette instruction permet de spécifier le répertoire de travail dans lequel seront donc copiés des fichiers/dossiers dans les instructions suivantes.
WORKDIR /app

### L'instruction COPY permet de copier des fichiers/dossiers depuis le contexte de construction (depuis la machine hôte) vers le conteneur:
COPY package.json .
COPY package-lock.json .

### L'instruction RUN permet l'exécution de commandes depuis le conteneur.
RUN docker-php-ext-install pdo pdo_mysql
RUN npm install

### Copie ce qui se trouve à la base de notre dossier et le colle à la base de notre conteneur.
COPY . .

### Execute les 2 commandes dans le conteneur. Le build en production et l'installation de serve dans le conteneur
RUN npm run build --production
RUN npm install -g serve

### L'instruction CMD peut également indiquer la commande du conteneur. C'est à dire elle définit la commande que votre conteneur va utiliser
CMD [ "serve", "-s", "build" ]

## Informations complémentaires:
Une fois que votre conteneur est lancé, il vous fournira le lien http://localhost:3000 de base
Mais pour accéder à votre app il faudra changer le port en 3001 car celui ci c'est celui du conteneur. 

