---
summary: "Installation automatisée et durcie d'OpenClaw avec Ansible, VPN Tailscale et isolation par pare-feu"
read_when:
  - Vous souhaitez un déploiement serveur automatisé avec durcissement de sécurité
  - Vous avez besoin d'une configuration isolée par pare-feu avec accès VPN
  - Vous déployez sur des serveurs Debian/Ubuntu distants
title: "Ansible"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/ansible.md
  workflow: manual
---

# Installation Ansible

La méthode recommandée pour déployer OpenClaw sur des serveurs de production est via **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**, un installateur automatisé avec une architecture orientée sécurité.

## Démarrage rapide

Installation en une commande :

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **Guide complet : [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Le dépôt openclaw-ansible est la source de vérité pour le déploiement Ansible. Cette page est un aperçu rapide.

## Ce que vous obtenez

- **Sécurité pare-feu en premier** : UFW + isolation Docker (seuls SSH + Tailscale sont accessibles)
- **VPN Tailscale** : accès distant sécurisé sans exposer les services publiquement
- **Docker** : conteneurs de bac à sable isolés, liaisons localhost uniquement
- **Défense en profondeur** : architecture de sécurité à 4 couches
- **Configuration en une commande** : déploiement complet en quelques minutes
- **Intégration systemd** : démarrage automatique au boot avec durcissement

## Prérequis

- **OS** : Debian 11+ ou Ubuntu 20.04+
- **Accès** : privilèges root ou sudo
- **Réseau** : connexion Internet pour l'installation des paquets
- **Ansible** : 2.14+ (installé automatiquement par le script de démarrage rapide)

## Ce qui est installé

Le playbook Ansible installe et configure :

1. **Tailscale** (VPN mesh pour l'accès distant sécurisé)
2. **Pare-feu UFW** (ports SSH + Tailscale uniquement)
3. **Docker CE + Compose V2** (pour les bacs à sable des agents)
4. **Node.js 22.x + pnpm** (dépendances d'exécution)
5. **OpenClaw** (basé sur l'hôte, non conteneurisé)
6. **Service systemd** (démarrage automatique avec durcissement de sécurité)

Note : le gateway s'exécute **directement sur l'hôte** (pas dans Docker), mais les bacs à sable des agents utilisent Docker pour l'isolation. Voir [Bac à sable](/gateway/sandboxing) pour les détails.

## Configuration post-installation

Après la fin de l'installation, passez à l'utilisateur openclaw :

```bash
sudo -i -u openclaw
```

Le script post-installation vous guidera à travers :

1. **Assistant de configuration initiale** : configurer les paramètres OpenClaw
2. **Connexion fournisseur** : connecter WhatsApp/Telegram/Discord/Signal
3. **Test du gateway** : vérifier l'installation
4. **Configuration Tailscale** : se connecter à votre mesh VPN

### Commandes rapides

```bash
# Vérifier le statut du service
sudo systemctl status openclaw

# Voir les journaux en direct
sudo journalctl -u openclaw -f

# Redémarrer le gateway
sudo systemctl restart openclaw

# Connexion fournisseur (exécuter en tant qu'utilisateur openclaw)
sudo -i -u openclaw
openclaw channels login
```

## Architecture de sécurité

### Défense à 4 couches

1. **Pare-feu (UFW)** : seuls SSH (22) + Tailscale (41641/udp) sont exposés publiquement
2. **VPN (Tailscale)** : gateway accessible uniquement via le mesh VPN
3. **Isolation Docker** : la chaîne iptables DOCKER-USER empêche l'exposition de ports externes
4. **Durcissement systemd** : NoNewPrivileges, PrivateTmp, utilisateur non privilégié

### Vérification

Testez la surface d'attaque externe :

```bash
nmap -p- YOUR_SERVER_IP
```

Devrait montrer **uniquement le port 22** (SSH) ouvert. Tous les autres services (gateway, Docker) sont verrouillés.

### Disponibilité Docker

Docker est installé pour les **bacs à sable des agents** (exécution isolée des outils), pas pour exécuter le gateway lui-même. Le gateway se lie à localhost uniquement et est accessible via VPN Tailscale.

Voir [Bac à sable et outils multi-agents](/tools/multi-agent-sandbox-tools) pour la configuration du bac à sable.

## Installation manuelle

Si vous préférez un contrôle manuel sur l'automatisation :

```bash
# 1. Installer les prérequis
sudo apt update && sudo apt install -y ansible git

# 2. Cloner le dépôt
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Installer les collections Ansible
ansible-galaxy collection install -r requirements.yml

# 4. Exécuter le playbook
./run-playbook.sh

# Ou exécuter directement (puis exécuter manuellement /tmp/openclaw-setup.sh après)
# ansible-playbook playbook.yml --ask-become-pass
```

## Mise à jour d'OpenClaw

L'installateur Ansible configuré OpenClaw pour les mises à jour manuelles. Voir [Mise à jour](/install/updating) pour le flux de mise à jour standard.

Pour relancer le playbook Ansible (par ex., pour des changements de configuration) :

```bash
cd openclaw-ansible
./run-playbook.sh
```

Note : c'est idempotent et peut être exécuté plusieurs fois en toute sécurité.

## Dépannage

### Le pare-feu bloque ma connexion

Si vous êtes bloqué :

- Assurez-vous de pouvoir accéder via VPN Tailscale d'abord
- L'accès SSH (port 22) est toujours autorisé
- Le gateway est **uniquement** accessible via Tailscale par conception

### Le service ne démarre pas

```bash
# Vérifier les journaux
sudo journalctl -u openclaw -n 100

# Vérifier les permissions
sudo ls -la /opt/openclaw

# Tester le démarrage manuel
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Problèmes de bac à sable Docker

```bash
# Vérifier que Docker fonctionne
sudo systemctl status docker

# Vérifier l'image du bac à sable
sudo docker images | grep openclaw-sandbox

# Compiler l'image du bac à sable si manquante
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Échec de connexion au fournisseur

Assurez-vous d'exécuter en tant qu'utilisateur `openclaw` :

```bash
sudo -i -u openclaw
openclaw channels login
```

## Configuration avancée

Pour l'architecture de sécurité détaillée et le dépannage :

- [Architecture de sécurité](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Détails techniques](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Guide de dépannage](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Voir aussi

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guide de déploiement complet
- [Docker](/install/docker) — configuration du gateway conteneurisé
- [Bac à sable](/gateway/sandboxing) — configuration du bac à sable des agents
- [Bac à sable et outils multi-agents](/tools/multi-agent-sandbox-tools) — isolation par agent
