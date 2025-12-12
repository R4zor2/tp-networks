CORRE Mathéo B3-SYSOPS

J'ai utilisé la commande openssl après l'avoir installé sur ma machine :

& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs\tls.key -out certs\tls.crt -subj "/CN=app.localhost"

Les différentes options de la commande sont :
- `req` : utilise le module de génération de certificat / requête X.509. [web:252]
- `-x509` : génère directement un certificat auto-signé (pas d’autorité externe). [web:252]
- `-newkey rsa:2048` : crée une nouvelle clé privée RSA 2048 bits. [web:252]
- `-nodes` : clé privée non chiffrée (pas de mot de passe), pratique pour Traefik au démarrage. [web:252]
- `-days 365` : certificat valable 365 jours. [web:252]
- `-keyout certs\tls.key` : chemin de la clé privée générée. [web:252]
- `-out certs\tls.crt` : chemin du certificat généré. [web:252]
- `-subj "/CN=app.localhost"` : définit le Common Name à `app.localhost`. [web:252]

Voici un screen du terminal donnant le résultat de la commande : 
 (voir image.png)

J'ai crée un dossier certs à la racine du projet pour stocker les certificats.
Cela permet de  d'aoir tous les fichiers importants du projet au même endroit.

Le routeur TLS (traefik) fonctionne de cette manière : 

Le routeur HTTPS (`app-https`) écoute sur l’entrypoint `websecure` (port 443) et active TLS (`tls=true`). [web:301][web:245]
Quand on accède à `https://app.localhost`, Traefik établit la connexion chiffrée (handshake TLS) avec le certificat chargé via `dynamic.yml` (section `tls.certificates`). [web:245]
Une fois la connexion HTTPS établie, Traefik route la requête grâce à la règle `Host(app.localhost)` vers le service `proxy` (load balancer sur le port 80). [web:304][web:301]

La première fois que l'on se connecte on doit accepter le risque qui nous dit que la connexion n'est pas sécuriser sur le site. ( voir image-2.png)

Une fois accepter on se retrouve sur la page Hello From app! comme prévue on peut voir que le site n'est pas sécurisé en haut à gauche. ! (voir image-1.png)

Je suis resté bloqué un moment sur cette étape cela m'affichait bad gateway c'était du à un problème sur ce label : traefik.docker.network=tp-networks_frontend_net je l'avais appelé traefik.docker.network=frontend_net 

J'avais aussi un problème de virtuallisation je n'arrivais pas à démarré docker j'ai mis une petite heure à le résoudre mais cela venait des fonctionnalité window que j'avais du modifier pour un cours en début de semaine. Ducoup je n'ai pas eu le temps de continuer pour la suite du TP j'ai préférée faire correctement la partie 1. 