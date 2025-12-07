# Mise en place de NGINX sur gateway01-infomaniak

**Phase 1 – socle physique et réseau**

![[logo-loutik.png]]

---

## Informations générales

- **Date de création :** 07/12/2025
- **Dernière modification :** 07/12/2025
- **Auteur :** MEDO Louis
- **Version :** 1

---

## Objectif

Mise en place du reverse-proxy **NGINX** pour répartir et gérer le trafic web vers les différents backend de l’infrastructure Loutik.

---

## Sommaire

* A. Installation de NGINX
* B. Gestion des erreurs NGINX
* C. Gestion automatique des certificats
* D. Ajout du certificat dans les configurations NGINX
* E. Configuration d’un nouveau service dans NGINX

---
## A. Installation de NGINX

**Notions**
- `/etc/nginx/nginx.conf` → configuration globale de NGINX.  
    Par défaut, ce fichier **inclut automatiquement** les configurations présentes dans `/etc/nginx/sites-enabled/`.
- `/etc/nginx/sites-available/` → répertoire contenant les fichiers de configuration **désactivés**. Il sert uniquement de stockage. NGINX **ne lit pas** ces fichiers directement.
- `/etc/nginx/sites-enabled/` → répertoire contenant les **liens symboliques** activant réellement les configurations.  C’est ici que NGINX lit va lire la configuration finale.

> ⚠️ Bonne pratique : toujours créer/éditer dans _sites-available_, puis créer un lien symbolique vers _sites-enabled_.  
> Cela évite les erreurs et permet de désactiver un site proprement.

 1. Installation de NGINX
```bash
sudo apt install nginx -y
```

 2. Ouverture des ports 80 et 443 sur Infomaniak

Accéder au pare-feu du VPS :  
	 [https://manager.infomaniak.com/v3/887489/ng/admin3/cloud/40818/firewall](https://manager.infomaniak.com/v3/887489/ng/admin3/cloud/40818/firewall)

> Vérifier que les règles autorisent bien **entrée TCP 80** et **entrée TCP 443**.

 3. Suppression de la configuration par défaut
```bash
sudo rm /etc/nginx/sites-enabled/default
```

> Cela évite que la page par défaut apparaisse avant vos propres services.

 4. Création d’un fichier de configuration pour un service de test
```bash
sudo nano /etc/nginx/sites-available/test.conf
```

 5. Exemple de configuration vers un backend distant
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name loutik.fr *.loutik.fr;
}
}

server {
    listen 80;
    server_name rapide.loutik.fr;

    location / {
           proxy_pass http://192.168.1.155:1025;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
    }
}
```

> Notes :
> - `proxy_set_header` est indispensable pour transmettre l’IP du client au backend.
> - `server_name` doit correspondre exactement au domaine configuré dans DNS.
> - Vérifier que le backend répond sur l’IP indiquée.

 6. Activer la configuration (création du lien symbolique)
```bash
sudo ln -s /etc/nginx/sites-available/test.conf /etc/nginx/sites-enabled/
```

 7. Vérifier la syntaxe NGINX
```bash
sudo nginx -t
```

Vous devez obtenir :
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

 8. Redémarrer NGINX
```bash
sudo systemctl restart nginx
```

---

## B. Gestion des erreurs NGINX

Cette section explique comment personnaliser les pages d’erreur pour offrir une meilleure expérience utilisateur.

 1. Création du fichier `error_pages.conf`
```bash
sudo nano /etc/nginx/snippets/error_pages.conf
```

 2. Contenu du fichier
```nginx
error_page 502 503 504 /bad-gateway.html;

location = /bad-gateway.html {
    root /var/www/html;
    proxy_connect_timeout 3s;
    proxy_read_timeout 30s;
    proxy_send_timeout 30s;
    internal;
}
```

> - `internal` empêche l’accès direct via l’URL.
> - Placer un fichier `/var/www/html/bad-gateway.html`.
> - Le `root` définit où est stockée la page d’erreur.

 3. Ajouter le fichier HTML

Créer :  
`/var/www/html/bad-gateway.html`

Répertoire github : https://github.com/FireToak/loutik-tunnel/blob/main/error-pages/bad-gateway.html

 4. Inclure les pages d’erreur dans les configurations des services

Exemple :
```nginx
server { 
    listen 80; 
    server_name rapide.loutik.fr;
    include /etc/nginx/snippets/error_pages.conf;

    location / { 
        proxy_pass http://192.168.1.55; 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
    } 
}
```

 5. Réduire le timeout TCP du kernel (optionnel)
```bash
echo 2 | sudo tee /proc/sys/net/ipv4/tcp_syn_retries
```

> À utiliser seulement si un backend met trop longtemps à répondre.

---

## C. Gestion automatique des certificats

 1. Installer Certbot + plugin Cloudflare
```bash
sudo apt install certbot python3-certbot-nginx python3-certbot-dns-cloudflare -y
```

 2. Création du Token API Cloudflare
Étapes résumées :

3. [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
4. _Create Token_
5. Modèle **Edit zone DNS**
6. Permissions :
    - _Zone / DNS / Edit_
    - Sélectionner la zone **loutik.fr**
7. Create Token

> Sauvegarder bien votre token, car vous ne pourrez plus le voir ensuite !

 8. Création du répertoire pour les secrets
```bash
sudo mkdir -p /etc/letsencrypt/secrets
```

 4. Création du fichier cloudflare.ini
```bash
sudo nano /etc/letsencrypt/secrets/cloudflare.ini
```

Ajouter :
```ini
dns_cloudflare_api_token = TON_TOKEN_SUPER_SECRET_ICI
```

 5. Sécuriser le fichier
```bash
sudo chmod 600 /etc/letsencrypt/secrets/cloudflare.ini
```

 6. Générer le certificat wildcard
```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/secrets/cloudflare.ini \
  --dns-cloudflare-propagation-seconds 20 \
  -d "loutik.fr" \
  -d "*.loutik.fr" \
  --deploy-hook "systemctl reload nginx"
```

 7. Vérifier le timer certbot
```bash
systemctl list-timers | grep certbot
```

---

## D. Ajout du certificat dans les configurations NGINX

 1. Créer un snippet TLS
```bash
sudo nano /etc/nginx/snippets/tls.conf
```

 2. Contenu du fichier
```nginx
ssl_certificate /etc/letsencrypt/live/loutik.fr/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/loutik.fr/privkey.pem;

ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
```

 3. Exemple de configuration HTTPS
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name loutik.fr *.loutik.fr;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name rapide.loutik.fr;

    include /etc/nginx/snippets/tls.conf;
    include /etc/nginx/snippets/error_pages.conf;

    location / {
           proxy_pass http://192.168.1.55;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## E. Configuration d’un nouveau service dans NGINX

1. Aller dans le répertoire
```bash
cd /etc/nginx/sites-available
```

 2. Créer un fichier pour un sous-domaine
```bash
sudo vi inserrer_sous_domaine.loutik.fr.conf
```

Respecter la règle :  
➡️ `<sous-domaine>.loutik.fr.conf`

3. Coller le template
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name inserrer_sous_domaine.loutik.fr;

    location / {
           return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name inserrer_sous_domaine.loutik.fr;
    http2 on;

    include /etc/nginx/snippets/tls.conf;
    include /etc/nginx/snippets/error_pages.conf;

    location / {
           proxy_pass http://192.168.1.209;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
    }
}
```

 4. Vérifier l’accès au service
```bash
curl -I https://sous_domaine.loutik.fr
```

---

## Bibliographie

- [Beginner's guide – documentation nginx](https://nginx.org/en/docs/beginners_guide.html)
- [Introduction – documentation nginx](https://nginx.org/en/docs/index.html)