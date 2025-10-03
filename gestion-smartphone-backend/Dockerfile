# Utiliser Node.js officiel
FROM node:18

# Créer un dossier de travail
WORKDIR /app

# Copier package.json et installer
COPY package*.json ./

# Nettoyer le cache et forcer le registre npm
RUN npm cache clean --force \
    && npm config set registry https://registry.npmjs.org/ \
    && npm install

# Copier tout le code backend
COPY . .

# Exposer le port API
EXPOSE 5000

# Lancer l'app
CMD ["npm", "run", "start"]
