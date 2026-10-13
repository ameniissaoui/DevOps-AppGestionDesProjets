# ---------- Étape 1 : build Angular ----------
FROM node:24-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Les services appellent http://localhost:8080 (mode dev). Dans le conteneur, on utilise
# des URLs relatives : nginx relaie /entreprise, /equipe, ... vers le backend.
RUN sed -i 's#http://localhost:8080##g' src/app/services/*.ts \
 && npm run build

# ---------- Étape 2 : serveur nginx ----------
FROM nginx:alpine
# Adresse du backend (surchargée dans Kubernetes) + DNS local injecté dans le modèle nginx
ENV BACKEND_HOST=backend \
    BACKEND_PORT=8080 \
    NGINX_ENTRYPOINT_LOCAL_RESOLVERS=1
COPY nginx.conf.template /etc/nginx/templates/default.conf.template
COPY --from=build /app/dist/frontend/browser /usr/share/nginx/html
EXPOSE 80
