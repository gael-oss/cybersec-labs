# Lab 1 — Réseau d'entreprise simulé (pfSense + DMZ + Suricata)

## Objectif

Construire une mini-infrastructure réseau reproduisant l'architecture d'une vraie entreprise : un pare-feu central (pfSense), une zone démilitarisée (DMZ) hébergeant un serveur web, un réseau interne (LAN) avec un serveur de fichiers, et un système de détection d'intrusion (Suricata) actif sur les interfaces réseau.

---

## Architecture

```
                        +-------------------+
                        |    Kali Linux      |
                        |  192.168.10.100    |
                        +--------+----------+
                                 |
                           intnet-LAN
                                 |
+------------+           +-------+--------+           +------------------+
|  WAN/Sim   +-- intnet-WAN --+  pfSense  +-- intnet-DMZ --+ Srv-Web-DMZ  |
| 192.168.100|           | 192.168.10.1  |           |  192.168.20.10   |
+------------+           | 192.168.20.1  |           |  Apache2         |
                         | 192.168.100.1 |           +------------------+
                         +-------+-------+
                                 |
                           intnet-LAN
                                 |
                        +--------+----------+
                        |   Srv-Files-LAN   |
                        |   192.168.10.10   |
                        |   Samba           |
                        +-------------------+
```

---

## Plan d'adressage IP

| Réseau | Plage | Passerelle (pfSense) |
|--------|-------|----------------------|
| WAN (simulé) | 192.168.100.0/24 | 192.168.100.1 |
| LAN (interne) | 192.168.10.0/24 | 192.168.10.1 |
| DMZ (exposée) | 192.168.20.0/24 | 192.168.20.1 |

| VM | IP | Rôle |
|----|----|------|
| pfSense | 192.168.10.1 / 192.168.20.1 / 192.168.100.1 | Pare-feu / routeur central |
| Srv-Web-DMZ | 192.168.20.10 | Serveur web Apache en DMZ |
| Srv-Files-LAN | 192.168.10.10 | Serveur de fichiers Samba en LAN |
| Kali Linux | 192.168.10.100 | Poste d'administration / tests |

---

## VMs utilisées

| VM | OS | RAM | Réseau(x) |
|----|----|-----|-----------|
| pfSense | pfSense CE 2.7 | 1 Go | intnet-WAN + intnet-LAN + intnet-DMZ |
| Srv-Web-DMZ | Ubuntu Server 22.04 | 1 Go | intnet-DMZ |
| Srv-Files-LAN | Ubuntu Server 22.04 | 1 Go | intnet-LAN |
| Kali Linux | Kali Linux 2024 | 2 Go | intnet-LAN |

---

## Ce qui a été mis en place

### 1. Réseaux internes VirtualBox

Trois réseaux internes ont été créés implicitement dans VirtualBox en nommant les adaptateurs réseau de chaque VM :

- `intnet-WAN` — simule la zone Internet
- `intnet-LAN` — réseau interne de l'entreprise
- `intnet-DMZ` — zone démilitarisée

Ces réseaux sont purement virtuels et n'ont aucune connexion avec le réseau physique de la machine hôte.

### 2. pfSense — Pare-feu et routeur central

pfSense a été installé avec trois interfaces réseau :

```
WAN  → em0 → intnet-WAN  → 192.168.100.1
LAN  → em1 → intnet-LAN  → 192.168.10.1
OPT1 → em2 → intnet-DMZ  → 192.168.20.1
```

Un serveur DHCP a été activé sur le LAN (plage : 192.168.10.100 – 192.168.10.200).

### 3. Serveur web en DMZ (Apache)

Ubuntu Server 22.04 avec Apache2 installé et configuré avec une IP statique `192.168.20.10`.

Une page HTML personnalisée identifie le serveur :

```html
<h1>Serveur Web — Zone DMZ</h1>
<p>Adresse IP : 192.168.20.10</p>
```

### 4. Serveur de fichiers en LAN (Samba)

Ubuntu Server 22.04 avec Samba configuré avec deux partages réseau :

- `documents` — partage interne LAN
- `public` — partage accessible à tous les hôtes LAN

### 5. Règles firewall pfSense

| Interface | Action | Source | Destination | Port | Objectif |
|-----------|--------|--------|-------------|------|----------|
| LAN | Pass | LAN net | 192.168.20.10 | 80 | Accès web LAN → DMZ |
| LAN | Pass | LAN net | LAN net | Any | Trafic interne LAN |
| DMZ | Block | DMZ net | LAN net | Any | Isolation DMZ → LAN |
| DMZ | Pass | DMZ net | WAN net | 80, 443 | Mises à jour DMZ |

### 6. Suricata IDS/IPS

Suricata a été déployé sur les interfaces WAN et LAN avec les catégories de règles suivantes activées :

- `emerging-scan.rules` — détection de scans de ports
- `emerging-exploit.rules` — tentatives d'exploitation de CVE
- `emerging-web-server.rules` — attaques web (SQLi, XSS…)
- `emerging-malware.rules` — communications malware (C2)
- `emerging-dos.rules` — déni de service

---

## Tests et résultats

### Test 1 — Connectivité LAN → DMZ

```bash
ping -c 3 192.168.20.10   # ✅ OK
curl http://192.168.20.10  # ✅ Page Apache visible
```

### Test 2 — Isolation DMZ → LAN

```bash
# Depuis le serveur DMZ (192.168.20.10) :
ping 192.168.10.10   # ❌ Bloqué par pfSense (attendu)
```

### Test 3 — Accès au partage Samba

```bash
smbclient -L //192.168.10.10 -N   # ✅ Liste des partages visible
```

### Test 4 — Détection Suricata

```bash
nmap -sV -sC -p 1-1000 192.168.20.10
```

Alertes générées dans Services > Suricata > Alerts :

| Date | Proto | Source | Destination | Alerte |
|------|-------|--------|-------------|--------|
| 05/08/2026 14:04 | TCP | 192.168.10.100 | 192.168.20.10:80 | SURICATA Applayer Detect protocol only one direction |
| 05/08/2026 14:04 | ICMP | 192.168.10.100 | 192.168.20.10 | SURICATA ICMPv4 unknown code |
| 05/08/2026 13:56 | TCP | 192.168.10.100 | 192.168.10.10:139 | SURICATA Applayer Detect protocol only one direction |

---

## Problème rencontré et résolution

**Symptôme** : `ping -c 3 192.168.20.10` retournait `Network is unreachable` depuis Kali, alors que pfSense et le serveur DMZ étaient correctement configurés.

**Diagnostic** : La VM Kali avait deux IPs sur `eth0` (`192.168.98.3` et `192.168.10.100`) mais aucune route vers le réseau `192.168.20.0/24` dans sa table de routage.

**Résolution** :

```bash
sudo ip route add 192.168.20.0/24 via 192.168.10.1 dev eth0
```

pfSense jouait correctement son rôle de routeur — il manquait uniquement la route côté client pour indiquer que le trafic vers la DMZ devait passer par pfSense.

**Persistance de la route** :

```bash
sudo nmcli connection modify eth0 +ipv4.routes "192.168.20.0/24 192.168.10.1"
sudo nmcli connection up eth0
```

---

## Compétences acquises

- Configuration d'un pare-feu multi-interfaces avec pfSense
- Mise en place d'une architecture DMZ / LAN / WAN isolée
- Création de règles firewall stateful pour contrôler les flux réseau
- Déploiement de Suricata en mode IDS avec règles Emerging Threats
- Diagnostic réseau (ip route, ping, traceroute) dans un environnement virtualisé
- Résolution d'un problème de routage inter-réseaux

---

## Fichiers de configuration

| Fichier | Description |
|---------|-------------|
| `configs/config-pfSense.xml` | Export complet de la configuration pfSense |
| `configs/smb.conf` | Configuration Samba du serveur de fichiers LAN |
