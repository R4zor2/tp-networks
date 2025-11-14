CORRE Mathéo B3-SYSOPS

L’objectif du TP était de mettre en place une infrastructure composée de 3 services :

Une base de données MariaDB (db)

Une application simple HTTP (app)

Un serveur Nginx jouant le rôle de reverse proxy (proxy)

L’infrastructure doit être réalisée avec deux réseaux distincts :
app et db dans backend_net, et proxy dans frontend_net et backend_net.

L’utilisation de deux réseaux distincts garantit une isolation stricte des services.
Le réseau backend_net regroupe app et db, qui ne sont jamais exposés directement, ainsi que proxy pour qu’il puisse faire le lien entre le réseau privé et le réseau public.
Le réseau frontend_net ne contient que le proxy, le seul point d’entrée depuis l’extérieur.

Fichier Dockerfile
FROM python:3.11-slim  # Image de base. C'est une version Python allégée.
WORKDIR /app            # Définit le répertoire de travail dans le conteneur.
COPY app.py .           # Copie le fichier app.py dans le répertoire /app défini précédemment.
RUN pip install --no-cache-dir flask pymysql  # Installe les dépendances nécessaires à l'application.
EXPOSE 5000             # Indique que le conteneur écoute sur le port 5000.
CMD ["python", "app.py"]  # Commande pour lancer l'application.
 

Fichier compose.yml

On déclare 3 services que l’on va utiliser : db, app et proxy.

Pour db (MariaDB) :

On utilise l’image officielle mariadb:11. J’aime bien utiliser la politique Docker restart, que je mets sur always : cela permet de redémarrer le conteneur en cas de crash par exemple.
environment permet de définir les variables d’environnement utilisées par MariaDB au premier démarrage.
volumes permet de conserver les données même si l’on supprime le conteneur.
networks permet de connecter le service au réseau backend_net.

Pour app :

L’image de l’application est construite à partir du Dockerfile présent dans le dossier ./app.
depends_on indique que app dépend de db, donc Docker Compose démarre d’abord db.
environment récupère les informations de connexion pour se connecter à la base.
networks permet de connecter l’application au réseau backend_net.

Pour proxy :

On utilise l’image nginx:alpine.
depends_on indique que le proxy dépend de l’application et se lance après.
volumes monte le fichier de configuration dans le conteneur en lecture seule (:ro).
ports publie le port 80 de Nginx sur le port 8080 de ma machine.
networks place le proxy dans les deux réseaux.

À la fin du fichier, les blocs volumes et networks déclarent le volume nommé db_data ainsi que les réseaux backend_net et frontend_net. 


Tests de l’infrastructure

Commande de lancement :

docker compose up -d --build


Cette commande permet de lancer les conteneurs et de reconstruire l’image Flask avant le démarrage.

(voir image.png)

Ensuite, je lance :

docker compose ps


pour vérifier que les conteneurs sont bien démarrés.

(voir image-1.png)

Je teste que l’application est accessible via http://localhost:8080
 dans mon navigateur.

(voir image-2.png)

Ensuite, je vérifie l’isolation du réseau backend_net.
En tentant d'accéder à http://localhost:5000
, j’obtiens un site inaccessible, ce qui prouve que le conteneur app n’est pas exposé à l’extérieur.

(voir image-3.png)

Je suis ensuite entré dans le conteneur app :

docker exec -it tp-networks-app-1 sh


pour tester que le réseau backend_net et la connexion à MariaDB fonctionnent correctement depuis l’application.

(voir images 4 et 5)

J’ai ensuite testé si proxy pouvait contacter app en entrant dans le conteneur du proxy.

(voir image-6.png)

Push sur Docker Hub

Après les tests, j’ai poussé mon image sur Docker Hub :

docker login
docker tag tp-networks-app r4zor21/tp-networks-app:latest
docker push r4zor21/tp-networks-app:latest


L’image est maintenant disponible sur Docker Hub : r4zor21/tp-networks-app.

(voir image-7.png)


