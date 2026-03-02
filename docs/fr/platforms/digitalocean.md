---
summary: "OpenClaw sur DigitalOcean (option VPS payante simple)"
read_when:
  - Configuration d'OpenClaw sur DigitalOcean
  - Recherche d'un hébergément VPS economique pour OpenClaw
title: "DigitalOcean"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/digitalocean.md
  workflow: manual
---

# OpenClaw sur DigitalOcean

## Objectif

Exécuter un Gateway OpenClaw persistant sur DigitalOcean pour **6 $/mois** (ou 4 $/mois avec tarification réservée).

Si vous souhaitez une option a 0 $/mois et que la configuration ARM spécifique au fournisseur ne vous derange pas, consultez le [guide Oracle Cloud](/platforms/oracle).

## Comparaison des coûts (2026)

| Fournisseur  | Plan            | Specs                    | Prix/mois   | Notes                                         |
| ------------ | --------------- | ------------------------ | ----------- | --------------------------------------------- |
| Oracle Cloud | Always Free ARM | jusqu'a 4 OCPU, 24 Go RAM | 0 $         | ARM, capacité limitée / particularites d'inscription |
| Hetzner      | CX22            | 2 vCPU, 4 Go RAM        | 3,79 € (~4 $) | Option payante la moins chere                |
| DigitalOcean | Basic           | 1 vCPU, 1 Go RAM        | 6 $         | Interface simple, bonne documentation          |
| Vultr        | Cloud Compute   | 1 vCPU, 1 Go RAM        | 6 $         | Nombreux emplacements                          |
| Linode       | Nanode          | 1 vCPU, 1 Go RAM        | 5 $         | Fait maintenant partie d'Akamai                |

**Choisir un fournisseur :**

- DigitalOcean : UX la plus simple + configuration previsible (ce guide)
- Hetzner : bon rapport prix/performances (voir [guide Hetzner](/install/hetzner))
- Oracle Cloud : peut être a 0 $/mois, mais est plus capricieux et ARM uniquement (voir [guide Oracle](/platforms/oracle))

---

## Prerequis

- Compte DigitalOcean ([inscription avec 200 $ de credit gratuit](https://m.do.co/c/signup))
- Paire de clés SSH (ou volonte d'utiliser l'authentification par mot de passe)
- ~20 minutes

## 1) Créer un Droplet

<Warning>
Utilisez une image de base propre (Ubuntu 24.04 LTS). Evitez les images Marketplace tierces en un clic, sauf si vous avez examine leurs scripts de démarrage et paramètres de pare-feu par défaut.
</Warning>

1. Connectez-vous a [DigitalOcean](https://cloud.digitalocean.com/)
2. Cliquez sur **Create → Droplets**
3. Choisissez :
   - **Region :** la plus proche de vous (ou de vos utilisateurs)
   - **Image :** Ubuntu 24.04 LTS
   - **Taille :** Basic → Regular → **6 $/mois** (1 vCPU, 1 Go RAM, 25 Go SSD)
   - **Authentification :** clé SSH (recommandee) ou mot de passe
4. Cliquez sur **Create Droplet**
5. Notez l'adresse IP

## 2) Se connecter via SSH

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) Installer OpenClaw

```bash
# Mettre a jour le système
apt update && apt upgrade -y

# Installer Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Installer OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verifier
openclaw --version
```

## 4) Lancer l'intégration

```bash
openclaw onboard --install-daemon
```

L'assistant vous guidera à travers :

- Authentification des modèles (clés API ou OAuth)
- Configuration des canaux (Telegram, WhatsApp, Discord, etc.)
- Jeton du Gateway (génère automatiquement)
- Installation du daemon (systemd)

## 5) Vérifier le Gateway

```bash
# Verifier le statut
openclaw status

# Verifier le service
systemctl --user status openclaw-gateway.service

# Voir les logs
journalctl --user -u openclaw-gateway.service -f
```

## 6) Acceder au Tableau de bord

Le gateway se lie au loopback par défaut. Pour acceder a l'Interface de contrôle :

**Option A : Tunnel SSH (recommandé)**

```bash
# Depuis votre machine locale
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Puis ouvrez : http://localhost:18789
```

**Option B : Tailscale Serve (HTTPS, loopback uniquement)**

```bash
# Sur le droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configurer le Gateway pour utiliser Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Ouvrez : `https://<magicdns>/`

Notes :

- Serve garde le Gateway sur loopback uniquement et authentifié le trafic de l'Interface de contrôle/WebSocket via les en-tetes d'identité Tailscale (l'authentification sans jeton suppose un hôte gateway de confiance ; les API HTTP requierent toujours un jeton/mot de passe).
- Pour exiger un jeton/mot de passe a la place, définissez `gateway.auth.allowTailscale: false` ou utilisez `gateway.auth.mode: "password"`.

**Option C : Liaison tailnet (sans Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Ouvrez : `http://<tailscale-ip>:18789` (jeton requis).

## 7) Connecter vos canaux

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scannez le QR code
```

Voir [Canaux](/channels) pour les autres fournisseurs.

---

## Optimisations pour 1 Go de RAM

Le droplet a 6 $ n'a que 1 Go de RAM. Pour que tout fonctionne correctement :

### Ajouter du swap (recommandé)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Utiliser un modèle plus leger

Si vous rencontrez des OOM, envisagez :

- Utiliser des modèles bases sur des API (Claude, GPT) au lieu de modèles locaux
- Définir `agents.defaults.model.primary` sur un modèle plus petit

### Surveiller la mémoire

```bash
free -h
htop
```

---

## Persistance

Tout l'état reside dans :

- `~/.openclaw/` : configuration, identifiants, données de session
- `~/.openclaw/workspace/` : espace de travail (SOUL.md, mémoire, etc.)

Ceux-ci survivent aux redemarrages. Sauvegardez-les periodiquement :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

---

## Alternative gratuite Oracle Cloud

Oracle Cloud offre des instances ARM **Always Free** significativement plus puissantes que toute option payante ici, pour 0 $/mois.

| Ce que vous obtenez    | Specs                  |
| ---------------------- | ---------------------- |
| **4 OCPU**             | ARM Ampere A1          |
| **24 Go de RAM**       | Plus que suffisant     |
| **200 Go de stockage** | Volume bloc            |
| **Gratuit a vie**      | Pas de frais de carte  |

**Mises en garde :**

- L'inscription peut être capricieuse (réessayez si elle échoue)
- Architecture ARM : la plupart des choses fonctionnent, mais certains binaires nécessitent des builds ARM

Pour le guide de configuration complet, voir [Oracle Cloud](/platforms/oracle). Pour les astuces d'inscription et le dépannage du processus, voir ce [guide communautaire](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

---

## Dépannage

### Le Gateway ne démarre pas

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### Port déjà utilisé

```bash
lsof -i :18789
kill <PID>
```

### Mémoire insuffisante

```bash
# Verifier la mémoire
free -h

# Ajouter plus de swap
# Ou passer au droplet a 12 $/mois (2 Go de RAM)
```

---

## Voir aussi

- [Guide Hetzner](/install/hetzner) : moins cher, plus puissant
- [Installation Docker](/install/docker) : configuration conteneurisee
- [Tailscale](/gateway/tailscale) : accès distant sécurisé
- [Configuration](/gateway/configuration) : référence complète de la configuration
