??? Infrastructure LaRefonte
Infrastructure compl�te et professionnelle pour l'�cosyst�me LaRefonte avec nginx, Docker et monitoring.
?? Services d�ploy�s
? Services op�rationnels

?? LaRefonte Main : https://larefonte.store (Express + Backend de scraping)
?? N8N Workflows : https://n8n.larefonte.store (Automatisation)
??? VNC Access : https://vnc.larefonte.store (Acc�s distant s�curis�)
?? Cercle des Voyages : https://cercledesvoyages.larefonte.store (Dashboard analytique)

??? Architecture
larefonte-infrastructure/
+-- nginx/                          # Configuration nginx modulaire
�   +-- sites-available/            # Configurations par service
�   �   +-- 01-larefonte-main.conf  # Express + VNC
�   �   +-- 02-n8n.conf            # N8N workflows
�   �   +-- 03-cercledesvoyages.conf # Dashboard SPA
�   +-- ssl/                        # SSL centralis�
�   �   +-- options-ssl-nginx.conf  # Configuration SSL commune
�   +-- conf.d/                     # Configuration g�n�rale
�       +-- general.conf            # Headers s�curit� + rate limiting
+-- scripts/                        # Scripts d'automatisation
�   +-- deploy-nginx.sh            # D�ploiement automatis�
�   +-- backup-nginx.sh            # Sauvegarde
�   +-- reload-nginx.sh            # Rechargement s�curis�
+-- docker-compose.yml             # Orchestration services
+-- .env                           # Variables d'environnement
+-- README.md                      # Cette documentation
?? Gestion des services
D�marrage/Arr�t
bash# D�marrer tous les services
docker compose up -d

# Arr�ter tous les services
docker compose down

# Red�marrer un service sp�cifique
docker compose restart n8n
docker compose restart scraping-backend
Logs et monitoring
bash# Logs globaux
docker compose logs

# Logs par service
docker compose logs n8n
docker compose logs scraping-backend
docker compose logs novnc

# Logs nginx par service
tail -f /var/log/nginx/larefonte-main.access.log
tail -f /var/log/nginx/n8n.access.log
tail -f /var/log/nginx/vnc.access.log
tail -f /var/log/nginx/cercledesvoyages.access.log
?? Gestion nginx
D�ploiement de configuration
bash# D�ploiement complet avec sauvegarde automatique
./scripts/deploy-nginx.sh

# Test de configuration uniquement
./scripts/deploy-nginx.sh test

# Rollback vers une sauvegarde
./scripts/deploy-nginx.sh rollback /root/nginx-backups/20250108_143022
Modification de configuration
bash# Modifier une configuration
nano nginx/sites-available/01-larefonte-main.conf

# D�ployer les changements
./scripts/deploy-nginx.sh

# En cas de probl�me, rollback automatique
?? S�curit�
SSL/HTTPS

Certificats Let's Encrypt automatiques pour tous les domaines
Configuration SSL moderne (TLS 1.2/1.3 uniquement)
HSTS activ� sur tous les sites
OCSP Stapling pour validation rapide

Headers de s�curit�

X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
X-Frame-Options adapt� par service
Content-Security-Policy pour les SPA

Rate limiting

API : 30 req/min
General : 60 req/min
Connexions simultan�es limit�es par IP

Acc�s VNC

Authentification basique requise
Certificat SSL obligatoire
Acc�s localhost uniquement (via nginx)

?? Services Docker
Ports internes (localhost uniquement)

3000 : Scraping Backend (Express)
5678 : N8N Interface
6080 : noVNC Web Interface
5900 : VNC Server (scraping-backend)

Volumes persistants

project-n8n_n8n_data : Donn�es N8N (workflows, configurations)
project-n8n_scraping-backend_data : Donn�es de scraping persistantes

?? Projets externes
L'infrastructure r�f�rence les projets dans /root/projects/ :
/root/projects/
+-- shared/                    # Services partag�s
�   +-- scraping-backend/      # Backend de scraping r�utilisable
�   +-- novnc/                # Interface VNC
+-- internal/                  # Projets internes LaRefonte
+-- external/                  # Projets clients
?? D�ploiement initial
Pr�requis
bash# Docker et Docker Compose
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Nginx
sudo apt update && sudo apt install nginx

# Certbot pour SSL
sudo apt install certbot python3-certbot-nginx
Installation
bash# Cloner l'infrastructure
git clone https://github.com/La-Refonte/la-Refonte-infrastructure.git
cd la-Refonte-infrastructure

# Copier et configurer les variables d'environnement
cp .env.example .env
nano .env

# G�n�rer les certificats SSL
sudo certbot --nginx -d larefonte.store -d www.larefonte.store -d n8n.larefonte.store -d vnc.larefonte.store -d cercledesvoyages.larefonte.store

# Cr�er l'authentification VNC
sudo htpasswd -c /etc/nginx/.htpasswd votre_utilisateur

# D�ployer nginx
./scripts/deploy-nginx.sh

# D�marrer les services Docker
docker compose up -d
?? Workflow de d�veloppement

Modifier les configurations dans leurs dossiers respectifs
Tester localement si n�cessaire
Commiter les changements :
bashgit add .
git commit -m "feat: am�lioration config nginx"
git push origin main

D�ployer sur le serveur :
bash./scripts/deploy-nginx.sh


?? D�pannage
Services Docker
bash# V�rifier l'�tat des services
docker compose ps

# Rebuilder un service
docker compose up -d --build scraping-backend

# Logs d�taill�s
docker compose logs -f --tail=100 n8n
Nginx
bash# Test de configuration
sudo nginx -t

# Statut du service
sudo systemctl status nginx

# Red�marrage complet
sudo systemctl restart nginx
Certificats SSL
bash# V�rifier l'expiration
sudo certbot certificates

# Renouvellement manuel
sudo certbot renew

# Test de renouvellement
sudo certbot renew --dry-run
?? Monitoring
M�triques importantes

Uptime des services Docker
Certificats SSL (expiration dans 30 jours)
Espace disque volumes Docker
Logs d'erreur nginx et applications

Commandes utiles
bash# Espace disque volumes
docker system df

# Nettoyage Docker
docker system prune

# Taille des logs
du -sh /var/log/nginx/

# Rotation des logs
sudo logrotate -f /etc/logrotate.d/nginx
?? Liens utiles

GitHub : https://github.com/La-Refonte/la-Refonte-infrastructure
N8N Documentation : https://docs.n8n.io/
Nginx Documentation : https://nginx.org/en/docs/
Let's Encrypt : https://letsencrypt.org/