---
summary: "Se connecter à GitHub Copilot depuis OpenClaw via le flux d'appareil"
read_when:
  - Vous souhaitez utiliser GitHub Copilot comme fournisseur de modèles
  - Vous avez besoin du flux `openclaw models auth login-github-copilot`
title: "GitHub Copilot"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/github-copilot.md
  workflow: manual
---

# GitHub Copilot

## Qu'est-ce que GitHub Copilot ?

GitHub Copilot est l'assistant de codage IA de GitHub. Il donne accès aux modèles Copilot pour votre compte et abonnement GitHub. OpenClaw peut utiliser Copilot comme fournisseur de modèles de deux manières différentes.

## Deux manières d'utiliser Copilot dans OpenClaw

### 1) Fournisseur GitHub Copilot intégré (`github-copilot`)

Utilisez le flux de connexion natif par appareil pour obtenir un token GitHub, puis échangez-le contre des tokens API Copilot lorsqu'OpenClaw s'exécute. C'est le chemin **par défaut** et le plus simple car il ne nécessite pas VS Code.

### 2) Plugin Copilot Proxy (`copilot-proxy`)

Utilisez l'extension VS Code **Copilot Proxy** comme pont local. OpenClaw communique avec l'endpoint `/v1` du proxy et utilise la liste de modèles que vous y configurez. Choisissez cette option si vous utilisez déjà Copilot Proxy dans VS Code ou si vous devez acheminer les requêtes via celui-ci. Vous devez activer le plugin et garder l'extension VS Code en cours d'exécution.

Utilisez GitHub Copilot comme fournisseur de modèles (`github-copilot`). La commande de connexion exécute le flux d'appareil GitHub, enregistre un profil d'authentification et met à jour votre configuration pour utiliser ce profil.

## Configuration CLI

```bash
openclaw models auth login-github-copilot
```

Vous serez invité à visiter une URL et à saisir un code à usage unique. Gardez le terminal ouvert jusqu'à la fin du processus.

### Options facultatives

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## Définir un modèle par défaut

```bash
openclaw models set github-copilot/gpt-4o
```

### Extrait de configuration

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## Notes

- Nécessite un TTY interactif ; exécutez-le directement dans un terminal.
- La disponibilité des modèles Copilot dépend de votre abonnement ; si un modèle est rejeté, essayez
  un autre ID (par exemple `github-copilot/gpt-4.1`).
- La connexion stocke un token GitHub dans le magasin de profils d'authentification et l'échange contre un
  token API Copilot lorsqu'OpenClaw s'exécute.
