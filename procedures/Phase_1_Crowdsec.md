# Mise en place de CrowdSec sur gateway01-infomaniak

**Phase 1 – socle physique et réseau**

![This is an alt text.](./logo-loutik.png "This is a sample image.")

---

## Informations générales

- **Date de création :** 07/12/2025
- **Dernière modification :** 07/12/2025
- **Auteur :** MEDO Louis
- **Version :** 1

---

## Objectif

Mettre en place l’IPS (Intrusion Prevention System) ainsi que le WAF (Web Application Firewall) afin de sécuriser les services de Loutik en amont.  
CrowdSec permettra d’analyser les logs système et applicatifs, de détecter automatiquement des comportements malveillants et de les bloquer via le bouncer NGINX.

---

## Sommaire

* A. Installation de CrowdSec sur les deux machines
* B. Vérification du fonctionnement de CrowdSec

---

## A. Installation de CrowdSec sur les deux machines

1. **Lancer le script d’installation automatique de CrowdSec**  
    (documentation disponible dans la bibliographie)
```bash
    sudo curl -s https://install.crowdsec.net | sudo sh
```

> Installez le paquet **curl** au préalable s’il n’est pas disponible.

2. **Installer CrowdSec depuis apt**
```bash
    sudo apt install crowdsec
```

> Cela installe :
>	- le moteur d’analyse (security engine)
>	- les scénarios de détection par défaut
>	- le service crowdsec (`/etc/crowdsec/`)

3. **Installer le bouncer NGINX**  
    Le bouncer permet à CrowdSec de renvoyer des décisions de blocage directement au niveau du reverse-proxy.
```bash
    sudo apt install crowdsec-nginx-bouncer
```
    
4. **Connecter la machine au dashboard CrowdSec**  
    Dans la section **"Connect with the console"** du site CrowdSec, copier la commande fournie :
```bash
    sudo cscli console enroll [TA_CLE_ENROLL_AFFICHEE_SUR_LE_SITE]
```
    
> Cela permet de remonter automatiquement :
    > 
    > - les scénarios actifs
    >     
    > - les alertes
    >     
    > - les décisions (bans)
    >     
    > - l’état du moteur de détection
    >     

5. **Redémarrer les services** pour appliquer les configurations :
```bash
    sudo systemctl reload crowdsec
    sudo systemctl reload nginx
```

> ⚠️ Un _reload_ suffit, mais en cas de problème, remplacer par `restart`.

---

## B. Vérification du fonctionnement de CrowdSec

1. **Vérifier localement que des attaques ont été détectées et bloquées**
```bash
    sudo cscli alerts list
```

Exemple de sortie attendue :

```
    ╭─────┬───────────────────┬───────────────────────────────────┬─────────┬────────────────────────┬───────────┬──────────────────────╮
    │  ID │       value       │               reason              │ country │           as           │ decisions │      created_at      │
    ├─────┼───────────────────┼───────────────────────────────────┼─────────┼────────────────────────┼───────────┼──────────────────────┤
    │ 378 │ Ip:159.65.207.162 │ crowdsecurity/ssh-slow-bf         │ NL      │ 14061 DIGITALOCEAN-ASN │ ban:1     │ 2025-12-07T08:34:16Z │
    │ 377 │ Ip:178.128.253.30 │ crowdsecurity/ssh-slow-bf         │ NL      │ 14061 DIGITALOCEAN-ASN │ ban:1     │ 2025-12-07T08:33:37Z │
    │ …   │        …          │                 …                 │   …     │            …           │    …      │          …           │
    ╰─────┴───────────────────┴───────────────────────────────────┴─────────┴────────────────────────┴───────────┴──────────────────────╯
```

> Cela prouve que CrowdSec détecte les attaques et applique bien des _bans_.

2. **Vérifier l’état du service**
```bash
    sudo systemctl status crowdsec
    sudo systemctl status crowdsec-nginx-bouncer
```

> Assurez-vous que les services sont en _active (running)_.

3. **Vérifier que NGINX charge bien les règles de blocage**
```bash
    sudo tail -f /var/log/nginx/error.log
```

> Vous devriez voir des entrées indiquant le chargement des décisions de CrowdSec.

4. **Vérifier sur le dashboard CrowdSec**  
    Accéder à :  
    👉 [https://app.crowdsec.net/security-engines](https://app.crowdsec.net/security-engines)
    
    Vous devriez voir :
    
    - votre serveur VPS identifié par son hostname
    - les scénarios actifs (ex : ssh-bf, nginx-401, http-bf, etc.)
    - les alertes générées
    - les décisions envoyées au bouncer

---

## Bibliographie

- [Installation Linux – documentation CrowdSec](https://docs.crowdsec.net/u/getting_started/installation/linux)
- [Introduction – documentation CrowdSec](https://docs.crowdsec.net/u/getting_started/intro)