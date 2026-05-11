# 🛡️ Cybersec Labs — Environnements de cybersécurité virtualisés

Projet personnel de mise en pratique de la cybersécurité défensive et offensive dans des environnements virtuels isolés.

> ⚠️ **Usage éducatif uniquement.** Ces techniques ne doivent jamais être utilisées sur des systèmes tiers sans autorisation explicite.

---

## 📋 Contenu du repo

| Lab | Thème | Outils principaux | Statut |
|-----|-------|-------------------|--------|
| [Lab 1 — pfSense + DMZ + Suricata](./lab1-pfsense-dmz/) | Architecture réseau d'entreprise simulée | pfSense, Suricata, Apache, Samba | ✅ Terminé |
| [Lab 2 — Kali + Metasploitable 2](./lab2-metasploitable/) | Pentest guidé | Metasploit, Nmap, Hydra | 🔄 En cours |
| [Lab 3 — Flare-VM](./lab3-flare-vm/) | Analyse de malware | PEStudio, FLOSS, x64dbg, Ghidra | 🔄 En cours |

---

## 🖥️ Environnement technique

- **Hyperviseur** : VirtualBox 7.x
- **Machine hôte** : Linux
- **Isolation réseau** : réseaux internes VirtualBox uniquement (aucune VM exposée sur le réseau physique)

---

## 📂 Structure du repo

```
cybersec-labs/
├── README.md
├── lab1-pfsense-dmz/
│   ├── README.md
│   └── configs/
│       ├── config-pfSense.xml
│       └── smb.conf
├── lab2-metasploitable/
│   └── README.md
└── lab3-flare-vm/
    └── README.md
```

---

## ✍️ Auteur

**gael-oss** — Projet personnel d'apprentissage en cybersécurité.
