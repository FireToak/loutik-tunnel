# Mise en place de Tailscale sur les gateways

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

Mettre en place le réseau VPN Tailscale sur les deux machines _gateway01-infomaniak_ et _gateway01-loutik_, ainsi que procéder à leur configuration.

---

## Sommaire

A. Installation de Tailscale sur les deux machines
B. Configuration de Tailscale sur gateway01-loutik
C. Configuration de Tailscale sur gateway01-infomaniak
D. Vérification du fonctionnement

---

## A. Installation de Tailscale sur les deux machines

1. Installez Tailscale via le script proposé depuis l’interface web :
```bash
    curl -fsSL https://tailscale.com/install.sh | sh
```

> Installez le paquet **curl** si ce n’est pas déjà fait.

2. Connectez-vous à votre compte Tailscale via l’URL affichée dans le terminal :
 ```bash
    To authenticate, visit:
    
            https://login.tailscale.com/a/1D32e940015cde
```

2. Vous devriez voir un message de succès une fois connecté :
 ```bash
    Success.
```

---

## B. Configuration de Tailscale sur gateway01-loutik

Pour que le VPS (_gateway01-infomaniak_) puisse accéder au réseau LAN du homelab, nous devons configurer des règles de routage sur _gateway01-loutik_.

1. Annoncer le réseau local à Tailscale :
```bash
    tailscale up --advertise-routes=192.168.1.0/24
```

2. Approuver la route dans le Dashboard Tailscale :  
    Cliquez sur les trois petits points à droite de la machine, puis ouvrez **Edit route settings** pour approuver la route.

3. Activer l’IP forwarding sur Debian afin de permettre le routage des paquets.  
    Par défaut, cette fonctionnalité est désactivée :
 ```bash
    sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
```

4. Modifier également les valeurs persistantes du kernel :
  ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo nano /etc/sysctl.d/99-ip-forward.conf
    # Ajouter la ligne suivante dans le fichier :
    # net.ipv4.ip_forward = 1
```

---

## C. Configuration de Tailscale sur gateway01-infomaniak

Sur le VPS, vous devez accepter le routage du réseau Tailscale afin de pouvoir atteindre le réseau LAN du homelab.

1. Accepter le routage :
```bash
    tailscale up --accept-routes
```
    

---

## D. Vérification du fonctionnement

1. Effectuez un ping vers le serveur web de loutik.fr :
```bash
    ping 192.168.1.209
```

> Vous devriez voir les paquets arriver avec succès.

---

## Bibliographie

- [Subnet routers - documentation tailscale](https://tailscale.com/kb/1019/subnets)
- [Install debian trixie - documentation tailscale](https://tailscale.com/kb/1626/install-debian-trixie)