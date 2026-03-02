---
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/security/CONTRIBUTING-THREAT-MODEL.md
  workflow: manual
---

# Contribuer au modèle de menaces OpenClaw

Merci de contribuer a rendre OpenClaw plus sûr. Ce modèle de menaces est un document évolutif et nous accueillons les contributions de tous, sans nécessiter d'expertise en sécurité.

## Facons de contribuer

### Ajouter une menace

Vous avez repere un vecteur d'attaque ou un risque que nous n'avons pas couvert ? Ouvrez un ticket sur [openclaw/trust](https://github.com/openclaw/trust/issues) et décrivez-le avec vos propres mots. Vous n'avez pas besoin de connaître de frameworks ou de remplir chaque champ : décrivez simplement le scenario.

**Utile a inclure (mais pas obligatoire) :**

- Le scenario d'attaque et comment il pourrait être exploite
- Quelles parties d'OpenClaw sont affectees (CLI, Gateway, canaux, ClawHub, serveurs MCP, etc.)
- Votre estimation de la gravite (faible / moyen / élevé / critique)
- Tout lien vers des recherches connexes, des CVE ou des exemples concrets

Nous nous chargeons du mappage ATLAS, des identifiants de menace et de l'évaluation des risques lors de la revue. Si vous souhaitez inclure ces details, parfait, mais ce n'est pas attendu.

> **Ceci concerne l'ajout au modèle de menaces, pas le signalement de vulnérabilités activés.** Si vous avez trouve une vulnérabilité exploitable, consultez notre [page de confiance](https://trust.openclaw.ai) pour les instructions de divulgation responsable.

### Suggerer une mesure d'attenuation

Vous avez une idée pour traiter une menace existante ? Ouvrez un ticket ou une PR en référençant la menace. Les mesures d'attenuation utiles sont spécifiques et actionnables. Par exemple, "limitation de débit par expéditeur de 10 messages/minute au niveau du Gateway" est meilleur que "implémenter une limitation de débit."

### Proposer une chaine d'attaque

Les chaines d'attaque montrent comment plusieurs menaces se combinent en un scenario d'attaque realiste. Si vous voyez une combinaison dangereuse, décrivez les étapes et comment un attaquant les enchaînerait. Un court récit expliquant comment l'attaque se deroule en pratique est plus précieux qu'un modèle formel.

### Corriger ou améliorer le contenu existant

Fautes de frappe, clarifications, informations obsoletes, meilleurs exemples : les PR sont les bienvenues, pas besoin de ticket.

## Ce que nous utilisons

### MITRE ATLAS

Ce modèle de menaces est construit sur [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), un framework conçu spécifiquement pour les menaces IA/ML comme l'injection de prompt, l'utilisation abusive d'outils et l'exploitation d'agents. Vous n'avez pas besoin de connaître ATLAS pour contribuer : nous mappons les soumissions au framework lors de la revue.

### Identifiants de menace

Chaque menace reçoit un identifiant comme `T-EXEC-003`. Les categories sont :

| Code    | Categorie                                         |
| ------- | ------------------------------------------------- |
| RECON   | Reconnaissance - collecte d'informations          |
| ACCESS  | Accès initial - obtenir l'entrée                  |
| EXEC    | Exécution - exécuter des actions malveillantes    |
| PERSIST | Persistance - maintenir l'accès                   |
| EVADE   | Évasion de defense - éviter la détection          |
| DISC    | Découverte - apprendre l'environnement            |
| EXFIL   | Exfiltration - vol de données                     |
| IMPACT  | Impact - dommages ou perturbation                 |

Les identifiants sont attribues par les mainteneurs lors de la revue. Vous n'avez pas besoin d'en choisir un.

### Niveaux de risque

| Niveau       | Signification                                                        |
| ------------ | -------------------------------------------------------------------- |
| **Critique** | Compromission complète du système, ou probabilite élevée + impact critique |
| **Élevé**    | Dommages importants probables, ou probabilite moyenne + impact critique    |
| **Moyen**    | Risque modere, ou probabilite faible + impact élevé                        |
| **Faible**   | Peu probable et impact limité                                              |

Si vous n'êtes pas sur du niveau de risque, décrivez simplement l'impact et nous l'evaluerons.

## Processus de revue

1. **Tri** - Nous examinons les nouvelles soumissions sous 48 heures
2. **Évaluation** - Nous verifions la faisabilite, attribuons le mappage ATLAS et l'identifiant de menace, validons le niveau de risque
3. **Documentation** - Nous nous assurons que tout est formate et complet
4. **Fusion** - Ajout au modèle de menaces et a la visualisation

## Ressources

- [Site web ATLAS](https://atlas.mitre.org/)
- [Techniques ATLAS](https://atlas.mitre.org/techniques/)
- [Études de cas ATLAS](https://atlas.mitre.org/studies/)
- [Modèle de menaces OpenClaw](./THREAT-MODEL-ATLAS.md)

## Contact

- **Vulnérabilités de sécurité :** Consultez notre [page de confiance](https://trust.openclaw.ai) pour les instructions de signalement
- **Questions sur le modèle de menaces :** Ouvrez un ticket sur [openclaw/trust](https://github.com/openclaw/trust/issues)
- **Discussion générale :** Canal Discord #security

## Reconnaissance

Les contributeurs au modèle de menaces sont reconnus dans les remerciements du modèle de menaces, les notes de version et le temple de la renommee sécurité d'OpenClaw pour les contributions significatives.
