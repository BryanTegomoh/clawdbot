---
summary: "Support Windows (WSL2) + statut de l'application compagnon"
read_when:
  - Installation d'OpenClaw sur Windows
  - Recherche du statut de l'application compagnon Windows
title: "Windows (WSL2)"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/windows.md
  workflow: manual
---

# Windows (WSL2)

OpenClaw sur Windows est recommandé **via WSL2** (Ubuntu recommandé). Le
CLI + Gateway s'executent dans Linux, ce qui maintient le runtime coherent et rend
l'outillage bien plus compatible (Node/Bun/pnpm, binaires Linux, skills). Windows
natif peut être plus delicat. WSL2 vous offre l'experience Linux complète, une seule commande
pour installer : `wsl --install`.

Des applications compagnon natives pour Windows sont prevues.

## Installation (WSL2)

- [Premiers pas](/start/getting-started) (utiliser dans WSL)
- [Installation et mises a jour](/install/updating)
- Guide officiel WSL2 (Microsoft) : [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## Gateway

- [Runbook Gateway](/gateway)
- [Configuration](/gateway/configuration)

## Installation du service Gateway (CLI)

Dans WSL2 :

```
openclaw onboard --install-daemon
```

Ou :

```
openclaw gateway install
```

Ou :

```
openclaw configure
```

Selectionnez **Gateway service** lorsque vous y etes invite.

Réparation/migration :

```
openclaw doctor
```

## Avance : exposer les services WSL sur le LAN (portproxy)

WSL a son propre réseau virtuel. Si une autre machine doit atteindre un service
s'executant **dans WSL** (SSH, un serveur TTS local ou le Gateway), vous devez
rediriger un port Windows vers l'IP WSL actuelle. L'IP WSL change après les redemarrages,
vous devrez donc peut-être actualiser la règle de redirection.

Exemple (PowerShell **en tant qu'Administrateur**) :

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Autorisez le port dans le pare-feu Windows (une seule fois) :

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Actualisez le portproxy après les redemarrages de WSL :

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notes :

- Le SSH depuis une autre machine cible l'**IP de l'hôte Windows** (exemple : `ssh user@windows-host -p 2222`).
- Les nœuds distants doivent pointer vers une URL du Gateway **accessible** (pas `127.0.0.1`) ; utilisez
  `openclaw status --all` pour confirmer.
- Utilisez `listenaddress=0.0.0.0` pour l'accès LAN ; `127.0.0.1` le garde local uniquement.
- Si vous voulez que cela soit automatique, enregistrez une tache planifiee pour exécuter l'étape
  d'actualisation a la connexion.

## Installation WSL2 étape par étape

### 1) Installer WSL2 + Ubuntu

Ouvrez PowerShell (Admin) :

```powershell
wsl --install
# Ou choisissez une distribution explicitement :
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Redemarrez si Windows le demande.

### 2) Activer systemd (requis pour l'installation du gateway)

Dans votre terminal WSL :

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Puis depuis PowerShell :

```powershell
wsl --shutdown
```

Reouvrez Ubuntu, puis verifiez :

```bash
systemctl --user status
```

### 3) Installer OpenClaw (dans WSL)

Suivez le flux Premiers pas Linux dans WSL :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les deps UI au premier lancement
pnpm build
openclaw onboard
```

Guide complet : [Premiers pas](/start/getting-started)

## Application compagnon Windows

Nous n'avons pas encore d'application compagnon Windows. Les contributions sont les bienvenues si vous souhaitez
contribuer a sa realisation.
