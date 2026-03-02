---
summary: "OpenClaw sur Raspberry Pi (configuration auto-hébergée economique)"
read_when:
  - Configuration d'OpenClaw sur un Raspberry Pi
  - Exécution d'OpenClaw sur des appareils ARM
  - Construction d'une IA personnelle toujours allumee et economique
title: "Raspberry Pi"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/raspberry-pi.md
  workflow: manual
---

# OpenClaw sur Raspberry Pi

## Objectif

Exécuter un Gateway OpenClaw persistant et toujours allume sur un Raspberry Pi pour un coût unique de **~35-80 $** (pas de frais mensuels).

Ideal pour :

- Assistant IA personnel 24/7
- Hub de domotique
- Bot Telegram/WhatsApp basse consommation et toujours disponible

## Configuration materielle requise

| Modèle Pi       | RAM     | Fonctionne ? | Notes                                 |
| --------------- | ------- | ------------ | ------------------------------------- |
| **Pi 5**        | 4 Go/8 Go | Optimal    | Le plus rapide, recommandé            |
| **Pi 4**        | 4 Go   | Bon          | Le meilleur compromis pour la plupart |
| **Pi 4**        | 2 Go   | OK           | Fonctionne, ajoutez du swap           |
| **Pi 4**        | 1 Go   | Juste        | Possible avec swap, config minimale   |
| **Pi 3B+**      | 1 Go   | Lent         | Fonctionne mais lentement             |
| **Pi Zero 2 W** | 512 Mo | Non recommandé | Non recommandé                     |

**Specs minimales :** 1 Go de RAM, 1 coeur, 500 Mo de disque
**Recommandé :** 2 Go+ de RAM, OS 64 bits, carte SD 16 Go+ (ou SSD USB)

## Ce dont vous aurez besoin

- Raspberry Pi 4 ou 5 (2 Go+ recommandé)
- Carte MicroSD (16 Go+) ou SSD USB (meilleures performances)
- Alimentation (alimentation officielle Pi recommandee)
- Connexion réseau (Ethernet ou WiFi)
- ~30 minutes

## 1) Flasher l'OS

Utilisez **Raspberry Pi OS Lite (64 bits)** : pas de bureau nécessaire pour un serveur headless.

1. Telechargez [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Choisissez l'OS : **Raspberry Pi OS Lite (64-bit)**
3. Cliquez sur l'icone engrenage pour pre-configurer :
   - Définir le nom d'hôte : `gateway-host`
   - Activer SSH
   - Définir nom d'utilisateur/mot de passe
   - Configurer le WiFi (si vous n'utilisez pas Ethernet)
4. Flashez sur votre carte SD / disque USB
5. Inserez et démarrez le Pi

## 2) Se connecter via SSH

```bash
ssh user@gateway-host
# ou utilisez l'adresse IP
ssh user@192.168.x.x
```

## 3) Configuration système

```bash
# Mettre a jour le système
sudo apt update && sudo apt upgrade -y

# Installer les paquets essentiels
sudo apt install -y git curl build-essential

# Definir le fuseau horaire (important pour cron/rappels)
sudo timedatectl set-timezone America/Chicago  # Changez pour votre fuseau horaire
```

## 4) Installer Node.js 22 (ARM64)

```bash
# Installer Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verifier
node --version  # Devrait afficher v22.x.x
npm --version
```

## 5) Ajouter du swap (important pour 2 Go ou moins)

Le swap prévient les crashs par manque de mémoire :

```bash
# Creer un fichier swap de 2 Go
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Rendre permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimiser pour la faible RAM (reduire le swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) Installer OpenClaw

### Option A : Installation standard (recommandee)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Option B : Installation modifiable (pour bidouiller)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

L'installation modifiable vous donne un accès direct aux logs et au code, utile pour deboguer les problèmes spécifiques a ARM.

## 7) Lancer l'intégration

```bash
openclaw onboard --install-daemon
```

Suivez l'assistant :

1. **Mode Gateway :** Local
2. **Auth :** clés API recommandees (OAuth peut être capricieux sur un Pi headless)
3. **Canaux :** Telegram est le plus facile pour commencer
4. **Daemon :** Oui (systemd)

## 8) Vérifier l'installation

```bash
# Verifier le statut
openclaw status

# Verifier le service
sudo systemctl status openclaw

# Voir les logs
journalctl -u openclaw -f
```

## 9) Acceder au Tableau de bord

Puisque le Pi est headless, utilisez un tunnel SSH :

```bash
# Depuis votre ordinateur portable/bureau
ssh -L 18789:localhost:18789 user@gateway-host

# Puis ouvrez dans le navigateur
open http://localhost:18789
```

Ou utilisez Tailscale pour un accès permanent :

```bash
# Sur le Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Mettre a jour la configuration
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

---

## Optimisations de performance

### Utiliser un SSD USB (amelioration majeure)

Les cartes SD sont lentes et s'usent. Un SSD USB ameliore considerablement les performances :

```bash
# Verifier si vous demarrez depuis USB
lsblk
```

Voir le [guide de boot USB Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) pour la configuration.

### Réduire l'utilisation de la mémoire

```bash
# Désactiver l'allocation mémoire GPU (headless)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Désactiver le Bluetooth si non necessaire
sudo systemctl disable bluetooth
```

### Surveiller les ressources

```bash
# Verifier la mémoire
free -h

# Verifier la temperature du CPU
vcgencmd measure_temp

# Surveillance en direct
htop
```

---

## Notes spécifiques a ARM

### Compatibilité des binaires

La plupart des fonctionnalités d'OpenClaw fonctionnent sur ARM64, mais certains binaires externes peuvent nécessiter des builds ARM :

| Outil              | Statut ARM64 | Notes                               |
| ------------------ | ------------ | ----------------------------------- |
| Node.js            | OK           | Fonctionne tres bien                |
| WhatsApp (Baileys) | OK           | JS pur, aucun problème              |
| Telegram           | OK           | JS pur, aucun problème              |
| gog (CLI Gmail)    | A vérifier   | Cherchez une version ARM            |
| Chromium (navigateur) | OK        | `sudo apt install chromium-browser` |

Si un skill échoue, verifiez si son binaire a un build ARM64. Beaucoup d'outils Go/Rust en ont ; certains non.

### 32 bits vs 64 bits

**Utilisez toujours un OS 64 bits.** Node.js et de nombreux outils modernes le nécessitent. Verifiez avec :

```bash
uname -m
# Devrait afficher : aarch64 (64 bits) et non armv7l (32 bits)
```

---

## Configuration de modèles recommandee

Puisque le Pi n'est que le Gateway (les modèles tournent dans le cloud), utilisez des modèles bases sur des API :

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**N'essayez pas d'exécuter des LLM locaux sur un Pi** : même les petits modèles sont trop lents. Laissez Claude/GPT faire le gros du travail.

---

## Démarrage automatique au boot

L'assistant d'intégration configure cela, mais pour vérifier :

```bash
# Verifier que le service est active
sudo systemctl is-enabled openclaw

# Activer si ce n'est pas le cas
sudo systemctl enable openclaw

# Demarrer au boot
sudo systemctl start openclaw
```

---

## Dépannage

### Mémoire insuffisante (OOM)

```bash
# Verifier la mémoire
free -h

# Ajouter plus de swap (voir etape 5)
# Ou reduire les services qui tournent sur le Pi
```

### Performances lentes

- Utilisez un SSD USB au lieu d'une carte SD
- Désactivez les services inutilises : `sudo systemctl disable cups bluetooth avahi-daemon`
- Verifiez le throttling CPU : `vcgencmd get_throttled` (devrait retourner `0x0`)

### Le service ne démarre pas

```bash
# Verifier les logs
journalctl -u openclaw --no-pager -n 100

# Correction courante : reconstruire
cd ~/openclaw  # si vous utilisez l'installation modifiable
npm run build
sudo systemctl restart openclaw
```

### Problèmes de binaires ARM

Si un skill échoue avec « exec format error » :

1. Verifiez si le binaire a un build ARM64
2. Essayez de compiler depuis les sources
3. Ou utilisez un conteneur Docker avec support ARM

### Déconnexions WiFi

Pour les Pi headless en WiFi :

```bash
# Désactiver la gestion d'alimentation WiFi
sudo iwconfig wlan0 power off

# Rendre permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

---

## Comparaison des coûts

| Configuration    | Coût unique | Coût mensuel | Notes                          |
| ---------------- | ----------- | ------------ | ------------------------------ |
| **Pi 4 (2 Go)**  | ~45 $       | 0 $          | + électricité (~5 $/an)        |
| **Pi 4 (4 Go)**  | ~55 $       | 0 $          | Recommandé                     |
| **Pi 5 (4 Go)**  | ~60 $       | 0 $          | Meilleures performances        |
| **Pi 5 (8 Go)**  | ~80 $       | 0 $          | Excessif mais perenne          |
| DigitalOcean     | 0 $         | 6 $/mois     | 72 $/an                        |
| Hetzner          | 0 $         | 3,79 €/mois  | ~50 $/an                       |

**Seuil de rentabilite :** un Pi est rentabilise en ~6-12 mois par rapport a un VPS cloud.

---

## Voir aussi

- [Guide Linux](/platforms/linux) : configuration Linux générale
- [Guide DigitalOcean](/platforms/digitalocean) : alternative cloud
- [Guide Hetzner](/install/hetzner) : configuration Docker
- [Tailscale](/gateway/tailscale) : accès distant
- [Nœuds](/nodes) : appairez votre ordinateur portable/téléphone avec le gateway Pi
