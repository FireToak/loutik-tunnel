# 🛡️ Edge Gateway Sécurisée : Alternative Open Source à Cloudflare Tunnel


## Informations générales

* **Auteur :** MEDO Louis
* **Date de Création :** 07/12/2025
* **Version :** 1

---

## 🎯 Objectif du Projet

Ce projet vise à **concevoir et déployer une Passerelle de Sécurité Déportée (Edge Gateway)** dans le cloud (VPS Infomaniak) pour sécuriser et rendre accessible l'infrastructure interne **HomeLab Loutik**.

L'objectif principal est de fournir une alternative **Zero Trust** et **Open Source** aux solutions de tunnels propriétaires, en assurant trois fonctions clés :

1.  **Masquer totalement** l'infrastructure locale derrière l'IP du VPS.
2.  **Filtrer le trafic malveillant** en amont (Mitigation DDoS, IPS).
3.  **Assurer la continuité de service** via un réseau Overlay résilient.

---

## 🧠 Problématiques Résolues (Contexte BTS SIO)

L'infrastructure initiale HomeLab de Loutik est confrontée à des contraintes critiques pour un environnement de production :

| Problématique Identifiée | Solution Mise en Œuvre | Outils Clés |
| :--- | :--- | :--- |
| **Exposition LAN / DDoS** (Risque de sécurité majeur) | Déport de l'IP publique sur un **VPS Cloud** (Point d'entrée unique). | **VPS Infomaniak** |
| **Accès instable** (IP dynamique, CGNAT) | Réseau **Overlay VPN** résilient (WireGuard). | **Tailscale** |
| **Attaques Applicatives** (L7: Brute Force, Scans) | **IPS/WAF** en amont, avec une approche collaborative (CTI). | **CrowdSec**, **NGINX** |

---

## ⚙️ Architecture Technique (Zero Trust Overlay)

L'architecture est segmentée en trois zones distinctes pour isoler les risques:

### 1. Zone Publique (`gateway01-infomaniak`)
C'est le seul point exposé à Internet. Il contient :
* **NGINX** : Reverse Proxy et terminaison SSL.
* **CrowdSec EDR/IPS** : Détection et blocage des attaques (L4/L7).
* **Tailscale VPN** : Point d'entrée du tunnel chiffré.

### 2. Zone Transport (VPN)
Le réseau **chiffré de bout en bout** (WireGuard/Tailscale) qui connecte le VPS à l'infrastructure locale.

### 3. Zone Privée (`gateway01-loutik` + HomeLab)
Contient l'infrastructure physique et virtuelle (Cluster Proxmox, Cluster K3s). Seul `gateway01-loutik` est connecté au VPN pour router le trafic vers les services internes (`192.168.1.0/24`).

---

## 🛠️ Composants Technologiques

| Composant | Solution (Justification) |
| :--- | :--- |
| **Réseau Overlay** | **Tailscale** (Basé sur WireGuard, UDP performant, NAT Traversal). |
| **Reverse Proxy** | **NGINX** (Gestion asynchrone, support HTTP/2, terminaison SSL). |
| **Sécurité (IPS/WAF)** | **CrowdSec** (Approche collaborative pour bloquer les IPs malveillantes). |

---

## 🚀 Mise en Œuvre et Procédures (Annexes)

Les étapes détaillées d'installation et de configuration de chaque composant sont documentées dans le répertoire `procédures/`.

### 1. Mise en place de Tailscale (VPN Overlay)
Procédure pour l'installation, la configuration du Subnet Router (`--advertise-routes`) sur `gateway01-loutik`, et l'activation du routage sur `gateway01-infomaniak` (`--accept-routes`).
* **Lien vers la procédure :** [`procédures/Phase_1_Tailscale.md`](./procédures/Phase_1_Mise.md)

### 2. Installation et Configuration NGINX (Reverse Proxy)
Procédure pour le déploiement de NGINX, la gestion modulaire des configurations (Snippets DRY [cite: 214][cite_start]), la personnalisation des pages d'erreur 502/503/504 [cite: 225][cite_start], et la gestion des certificats Wildcard via Certbot/Cloudflare.
* **Lien vers la procédure :** [`procédures/Phase_2_NGINX.md`](./procédures/Phase_2_NGINX.md)

### 3. Installation de CrowdSec (IPS/WAF)
Procédure pour l'installation du moteur d'analyse et du bouncer NGINX, permettant le blocage dynamique des attaquants au niveau réseau (`nftables`) et applicatif (logs NGINX).
* **Lien vers la procédure :** [`procédures/Phase_3_CrowdSec.md`](./procédures/Phase_3_CrowdSec.md)

---

## 🔒 Durcissement (OS Hardening)

Une attention particulière a été portée à la sécurité du VPS (`gateway01-infomaniak`) :
* **SSH Sécurisé :** Changement de port par défaut et interdiction de l'authentification par mot de passe (clés cryptographiques uniquement).
* **Micro-segmentation :** Utilisation des **ACLs Tailscale** pour restreindre la communication du VPS uniquement aux serveurs applicatifs du LAN, et non au reste du réseau domestique (Imprimantes, IoT).

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
* **Infrastructure Loutik :** [https://infra.loutik.fr](https://infra.loutik.fr)
* **Portfolio :** [https://louis.loutik.fr](https://louis.loutik.fr)