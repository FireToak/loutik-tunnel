# 🛡️ Edge Gateway Sécurisée : Alternative Open Source à Cloudflare Tunnel


## Informations générales

* **Auteur :** MEDO Louis
* [cite_start]**Date de Création :** 07/12/2025 [cite: 5]
* **Version :** 1.0

---

## 🎯 Objectif du Projet

Ce projet vise à **concevoir et déployer une Passerelle de Sécurité Déportée (Edge Gateway)** dans le cloud (VPS Infomaniak) pour sécuriser et rendre accessible l'infrastructure interne **HomeLab Loutik**.

[cite_start]L'objectif principal est de fournir une alternative **Zero Trust** et **Open Source** aux solutions de tunnels propriétaires, en assurant trois fonctions clés[cite: 65, 66, 68, 69]:

1.  **Masquer totalement** l'infrastructure locale derrière l'IP du VPS.
2.  **Filtrer le trafic malveillant** en amont (Mitigation DDoS, IPS).
3.  **Assurer la continuité de service** via un réseau Overlay résilient.

---

## 🧠 Problématiques Résolues (Contexte BTS SIO)

[cite_start]L'infrastructure initiale HomeLab de Loutik est confrontée à des contraintes critiques pour un environnement de production[cite: 59, 61, 62, 63]:

| Problématique Identifiée | Solution Mise en Œuvre | Outils Clés |
| :--- | :--- | :--- |
| **Exposition LAN / DDoS** (Risque de sécurité majeur) | Déport de l'IP publique sur un **VPS Cloud** (Point d'entrée unique). | **VPS Infomaniak** |
| **Accès instable** (IP dynamique, CGNAT) | [cite_start]Réseau **Overlay VPN** résilient (WireGuard)[cite: 69]. | **Tailscale** |
| **Attaques Applicatives** (L7: Brute Force, Scans) | **IPS/WAF** en amont, avec une approche collaborative (CTI). | **CrowdSec**, **NGINX** |

---

## ⚙️ Architecture Technique (Zero Trust Overlay)

[cite_start]L'architecture est segmentée en trois zones distinctes pour isoler les risques[cite: 87, 88, 89]:

### 1. Zone Publique (`gateway01-infomaniak`)
C'est le seul point exposé à Internet. Il contient :
* [cite_start]**NGINX** : Reverse Proxy et terminaison SSL[cite: 106].
* [cite_start]**CrowdSec EDR/IPS** : Détection et blocage des attaques (L4/L7)[cite: 107].
* **Tailscale VPN** : Point d'entrée du tunnel chiffré.

### 2. Zone Transport (VPN)
Le réseau **chiffré de bout en bout** (WireGuard/Tailscale) qui connecte le VPS à l'infrastructure locale.

### 3. Zone Privée (`gateway01-loutik` + HomeLab)
Contient l'infrastructure physique et virtuelle (Cluster Proxmox, Cluster K3s). [cite_start]Seul `gateway01-loutik` est connecté au VPN pour router le trafic vers les services internes (`192.168.1.0/24`)[cite: 24, 26].

---

## 🛠️ Composants Technologiques

| Composant | Solution (Justification) |
| :--- | :--- |
| **Réseau Overlay** | [cite_start]**Tailscale** (Basé sur WireGuard, UDP performant, NAT Traversal)[cite: 113]. |
| **Reverse Proxy** | [cite_start]**NGINX** (Gestion asynchrone, support HTTP/2, terminaison SSL)[cite: 113]. |
| **Sécurité (IPS/WAF)** | [cite_start]**CrowdSec** (Approche collaborative pour bloquer les IPs malveillantes)[cite: 113]. |

---

## 🚀 Mise en Œuvre et Procédures (Annexes)

Les étapes détaillées d'installation et de configuration de chaque composant sont documentées dans le répertoire `procédures/`.

### 1. Mise en place de Tailscale (VPN Overlay)
[cite_start]Procédure pour l'installation, la configuration du Subnet Router (`--advertise-routes`) sur `gateway01-loutik`, et l'activation du routage sur `gateway01-infomaniak` (`--accept-routes`)[cite: 12, 25, 36].
* **Lien vers la procédure :** [`procédures/Phase_1_Tailscale.md`](./procédures/Phase_1_Tailscale.md)

### 2. Installation et Configuration NGINX (Reverse Proxy)
[cite_start]Procédure pour le déploiement de NGINX, la gestion modulaire des configurations (Snippets DRY [cite: 214][cite_start]), la personnalisation des pages d'erreur 502/503/504 [cite: 225][cite_start], et la gestion des certificats Wildcard via Certbot/Cloudflare[cite: 382].
* **Lien vers la procédure :** [`procédures/Phase_2_NGINX.md`](./procédures/Phase_2_NGINX.md)

### 3. Installation de CrowdSec (IPS/WAF)
[cite_start]Procédure pour l'installation du moteur d'analyse et du bouncer NGINX, permettant le blocage dynamique des attaquants au niveau réseau (`nftables`) et applicatif (logs NGINX)[cite: 127, 128].
* **Lien vers la procédure :** [`procédures/Phase_3_CrowdSec.md`](./procédures/Phase_3_CrowdSec.md)

---

## 🔒 Durcissement (OS Hardening)

Une attention particulière a été portée à la sécurité du VPS (`gateway01-infomaniak`) :
* [cite_start]**SSH Sécurisé :** Changement de port par défaut et interdiction de l'authentification par mot de passe (clés cryptographiques uniquement)[cite: 123].
* [cite_start]**Micro-segmentation :** Utilisation des **ACLs Tailscale** pour restreindre la communication du VPS uniquement aux serveurs applicatifs du LAN, et non au reste du réseau domestique (Imprimantes, IoT)[cite: 124].

---

## 📂 Organisation du Répertoire

| Dossier | Contenu |
| :--- | :--- |
| **docs/** | Dossier Technique (Analyse, Architecture), Schémas. |
| **procédures/** | Étapes détaillées pour l'installation et la configuration de chaque outil. |
| **configs/nginx/snippets/** | Fichiers de configuration NGINX réutilisables (`tls.conf`, `error_pages.conf`). |
| **assets/error-pages/** | Pages d'erreur HTML personnalisées (Ex: `bad-gateway.html`). |

---

## 🔗 Liens Utiles

* **Dossier Technique Complet :** [`docs/Dossier_Technique.pdf`](./docs/Dossier_Technique.pdf)
* **Infrastructure Loutik :** [https://contact.loutik.fr](https://contact.loutik.fr)