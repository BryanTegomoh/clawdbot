---
summary: "OpenClaw sur Oracle Cloud (Always Free ARM)"
read_when:
  - Configuration d'OpenClaw sur Oracle Cloud
  - Recherche d'un hébergément VPS economique pour OpenClaw
  - Vous voulez OpenClaw 24/7 sur un petit serveur
title: "Oracle Cloud"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/oracle.md
  workflow: manual
---

# OpenClaw sur Oracle Cloud (OCI)

## Objectif

Exécuter un Gateway OpenClaw persistant sur le tier **Always Free** ARM d'Oracle Cloud.

Le tier gratuit d'Oracle peut bien convenir a OpenClaw (surtout si vous avez déjà un compte OCI), mais il comporte des compromis :

- Architecture ARM (la plupart des choses fonctionnent, mais certains binaires peuvent être x86 uniquement)
- La capacité et l'inscription peuvent être capricieuses

## Comparaison des coûts (2026)

| Fournisseur  | Plan            | Specs                    | Prix/mois | Notes                         |
| ------------ | --------------- | ------------------------ | --------- | ----------------------------- |
| Oracle Cloud | Always Free ARM | jusqu'a 4 OCPU, 24 Go RAM | 0 $       | ARM, capacité limitée         |
| Hetzner      | CX22            | 2 vCPU, 4 Go RAM        | ~ 4 $     | Option payante la moins chere |
| DigitalOcean | Basic           | 1 vCPU, 1 Go RAM        | 6 $       | Interface simple, bonne doc   |
| Vultr        | Cloud Compute   | 1 vCPU, 1 Go RAM        | 6 $       | Nombreux emplacements         |
| Linode       | Nanode          | 1 vCPU, 1 Go RAM        | 5 $       | Fait maintenant partie d'Akamai |

---

## Prerequis

- Compte Oracle Cloud ([inscription](https://www.oracle.com/cloud/free/)) : voir le [guide communautaire d'inscription](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) en cas de problème
- Compte Tailscale (gratuit sur [tailscale.com](https://tailscale.com))
- ~30 minutes

## 1) Créer une instance OCI

1. Connectez-vous a la [Console Oracle Cloud](https://cloud.oracle.com/)
2. Naviguez vers **Compute → Instances → Create Instance**
3. Configurez :
   - **Nom :** `openclaw`
   - **Image :** Ubuntu 24.04 (aarch64)
   - **Shape :** `VM.Standard.A1.Flex` (ARM Ampere)
   - **OCPU :** 2 (ou jusqu'a 4)
   - **Mémoire :** 12 Go (ou jusqu'a 24 Go)
   - **Volume de boot :** 50 Go (jusqu'a 200 Go gratuit)
   - **Clé SSH :** ajoutez votre clé publique
4. Cliquez sur **Create**
5. Notez l'adresse IP publique

**Astuce :** si la création de l'instance échoue avec « Out of capacity », essayez un domaine de disponibilité different ou réessayez plus tard. La capacité du tier gratuit est limitée.

## 2) Se connecter et mettre a jour

```bash
# Se connecter via IP publique
ssh ubuntu@YOUR_PUBLIC_IP

# Mettre a jour le système
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Note :** `build-essential` est requis pour la compilation ARM de certaines dependances.

## 3) Configurer l'utilisateur et le nom d'hôte

```bash
# Definir le nom d'hote
sudo hostnamectl set-hostname openclaw

# Definir le mot de passe pour l'utilisateur ubuntu
sudo passwd ubuntu

# Activer le lingering (maintient les services utilisateur apres deconnexion)
sudo loginctl enable-linger ubuntu
```

## 4) Installer Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Cela active le SSH Tailscale, vous pouvez donc vous connecter via `ssh openclaw` depuis n'importe quel appareil sur votre tailnet, sans IP publique nécessaire.

Verifiez :

```bash
tailscale status
```

**A partir de maintenant, connectez-vous via Tailscale :** `ssh ubuntu@openclaw` (ou utilisez l'IP Tailscale).

## 5) Installer OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

Lorsqu'on vous demande « How do you want to hatch your bot? », selectionnez **« Do this later »**.

> Note : si vous rencontrez des problèmes de build ARM natif, commencez par les paquets système (par exemple `sudo apt install -y build-essential`) avant de recourir a Homebrew.

## 6) Configurer le Gateway (loopback + authentification par jeton) et activer Tailscale Serve

Utilisez l'authentification par jeton par défaut. C'est previsible et evite d'avoir besoin de drapeaux « insecure auth » pour l'Interface de contrôle.

```bash
# Garder le Gateway prive sur la VM
openclaw config set gateway.bind loopback

# Exiger l'authentification pour le Gateway + l'Interface de controle
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Exposer via Tailscale Serve (HTTPS + acces tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) Vérifier

```bash
# Verifier la version
openclaw --version

# Verifier le statut du daemon
systemctl --user status openclaw-gateway

# Verifier Tailscale Serve
tailscale serve status

# Tester la réponse locale
curl http://localhost:18789
```

## 8) Verrouiller la sécurité du VCN

Maintenant que tout fonctionne, verrouillez le VCN pour bloquer tout le trafic sauf Tailscale. Le réseau cloud virtuel (VCN) d'OCI agit comme un pare-feu au niveau du réseau : le trafic est bloque avant d'atteindre votre instance.

1. Allez dans **Networking → Virtual Cloud Networks** dans la Console OCI
2. Cliquez sur votre VCN → **Security Lists** → Default Security List
3. **Supprimez** toutes les règles d'entrée sauf :
   - `0.0.0.0/0 UDP 41641` (Tailscale)
4. Gardez les règles de sortie par défaut (autoriser tout le sortant)

Cela bloque SSH sur le port 22, HTTP, HTTPS et tout le reste au niveau du réseau. A partir de maintenant, vous ne pouvez vous connecter que via Tailscale.

---

## Acceder a l'Interface de contrôle

Depuis n'importe quel appareil sur votre réseau Tailscale :

```
https://openclaw.<tailnet-name>.ts.net/
```

Remplacez `<tailnet-name>` par le nom de votre tailnet (visible dans `tailscale status`).

Pas de tunnel SSH nécessaire. Tailscale fournit :

- Chiffrement HTTPS (certificats automatiques)
- Authentification via l'identité Tailscale
- Accès depuis n'importe quel appareil sur votre tailnet (ordinateur portable, téléphone, etc.)

---

## Sécurité : VCN + Tailscale (base recommandee)

Avec le VCN verrouille (seul UDP 41641 ouvert) et le Gateway lie au loopback, vous obtenez une defense en profondeur solide : le trafic public est bloque au niveau du réseau, et l'accès administrateur se fait via votre tailnet.

Cette configuration elimine souvent le _besoin_ de règles de pare-feu supplementaires au niveau de l'hôte purement pour stopper les attaques SSH par force brute depuis Internet, mais vous devriez toujours garder l'OS a jour, exécuter `openclaw security audit`, et vérifier que vous n'ecoutez pas accidentellement sur des interfaces publiques.

### Ce qui est déjà protege

| Étape traditionnelle       | Nécessaire ? | Pourquoi                                                                             |
| -------------------------- | ------------ | ------------------------------------------------------------------------------------ |
| Pare-feu UFW               | Non          | Le VCN bloque avant que le trafic n'atteigne l'instance                              |
| fail2ban                   | Non          | Pas de force brute si le port 22 est bloque au VCN                                   |
| Durcissement sshd          | Non          | Le SSH Tailscale n'utilisé pas sshd                                                  |
| Désactiver le login root   | Non          | Tailscale utilisé l'identité Tailscale, pas les utilisateurs système                 |
| Auth par clé SSH uniquement| Non          | Tailscale authentifié via votre tailnet                                               |
| Durcissement IPv6          | Généralement non | Depend de vos paramètres VCN/sous-réseau ; verifiez ce qui est reellement assigne/expose |

### Toujours recommandé

- **Permissions des identifiants :** `chmod 700 ~/.openclaw`
- **Audit de sécurité :** `openclaw security audit`
- **Mises a jour système :** `sudo apt update && sudo apt upgrade` régulièrement
- **Surveiller Tailscale :** Verifiez les appareils dans la [console d'administration Tailscale](https://login.tailscale.com/admin)

### Vérifier la posture de sécurité

```bash
# Confirmer qu'aucun port public n'ecoute
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Verifier que le SSH Tailscale est actif
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Optionnel : désactiver sshd entièrement
sudo systemctl disable --now ssh
```

---

## Solution de repli : tunnel SSH

Si Tailscale Serve ne fonctionne pas, utilisez un tunnel SSH :

```bash
# Depuis votre machine locale (via Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Puis ouvrez `http://localhost:18789`.

---

## Dépannage

### La création d'instance échoue (« Out of capacity »)

Les instances ARM du tier gratuit sont populaires. Essayez :

- Un domaine de disponibilité different
- Réessayez pendant les heures creuses (tot le matin)
- Utilisez le filtré « Always Free » lors de la sélection du shape

### Tailscale ne se connecte pas

```bash
# Verifier le statut
sudo tailscale status

# Re-authentifier
sudo tailscale up --ssh --hostname=openclaw --reset
```

### Le Gateway ne démarre pas

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### Impossible d'acceder a l'Interface de contrôle

```bash
# Verifier que Tailscale Serve est en cours d'execution
tailscale serve status

# Verifier que le gateway ecoute
curl http://localhost:18789

# Redémarrer si necessaire
systemctl --user restart openclaw-gateway
```

### Problèmes de binaires ARM

Certains outils peuvent ne pas avoir de builds ARM. Verifiez :

```bash
uname -m  # Devrait afficher aarch64
```

La plupart des paquets npm fonctionnent bien. Pour les binaires, cherchez les versions `linux-arm64` ou `aarch64`.

---

## Persistance

Tout l'état reside dans :

- `~/.openclaw/` : configuration, identifiants, données de session
- `~/.openclaw/workspace/` : espace de travail (SOUL.md, mémoire, artefacts)

Sauvegardez periodiquement :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

---

## Voir aussi

- [Accès distant au Gateway](/gateway/remote) : autres modèles d'accès distant
- [Intégration Tailscale](/gateway/tailscale) : documentation complète de Tailscale
- [Configuration du Gateway](/gateway/configuration) : toutes les options de configuration
- [Guide DigitalOcean](/platforms/digitalocean) : si vous voulez du payant + inscription plus facile
- [Guide Hetzner](/install/hetzner) : alternative basee sur Docker
