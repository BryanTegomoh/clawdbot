---
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/security/THREAT-MODEL-ATLAS.md
  workflow: manual
---

# Modèle de menaces OpenClaw v1.0

## Framework MITRE ATLAS

**Version :** 1.0-draft
**Dernière mise a jour :** 2026-02-04
**Méthodologie :** MITRE ATLAS + Diagrammes de flux de données
**Framework :** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Attribution du framework

Ce modèle de menaces est construit sur [MITRE ATLAS](https://atlas.mitre.org/), le framework standard de l'industrie pour documenter les menaces adverses contre les systèmes IA/ML. ATLAS est maintenu par [MITRE](https://www.mitre.org/) en collaboration avec la communauté de sécurité IA.

**Ressources clés ATLAS :**

- [Techniques ATLAS](https://atlas.mitre.org/techniques/)
- [Tactiques ATLAS](https://atlas.mitre.org/tactics/)
- [Études de cas ATLAS](https://atlas.mitre.org/studies/)
- [GitHub ATLAS](https://github.com/mitre-atlas/atlas-data)
- [Contribuer a ATLAS](https://atlas.mitre.org/resources/contribute)

### Contribuer a ce modèle de menaces

Ceci est un document évolutif maintenu par la communauté OpenClaw. Consultez [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) pour les directives de contribution :

- Signaler de nouvelles menaces
- Mettre a jour les menaces existantes
- Proposer des chaines d'attaque
- Suggerer des mesures d'attenuation

---

## 1. Introduction

### 1.1 Objectif

Ce modèle de menaces documente les menaces adverses contre la plateforme d'agent IA OpenClaw et la marketplace de Skills ClawHub, en utilisant le framework MITRE ATLAS conçu spécifiquement pour les systèmes IA/ML.

### 1.2 Périmètre

| Composant                      | Inclus  | Notes                                            |
| ------------------------------ | ------- | ------------------------------------------------ |
| Runtime d'agent OpenClaw       | Oui     | Exécution de l'agent principal, appels d'outils, sessions |
| Gateway                        | Oui     | Authentification, routage, intégration des canaux |
| Intégrations de canaux         | Oui     | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| Marketplace ClawHub            | Oui     | Publication de Skills, modération, distribution   |
| Serveurs MCP                   | Oui     | Fournisseurs d'outils externes                   |
| Appareils utilisateur          | Partiel | Applications mobiles, clients bureau              |

### 1.3 Hors périmètre

Rien n'est explicitement hors périmètre pour ce modèle de menaces.

---

## 2. Architecture du système

### 2.1 Limités de confiance

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZONE NON FIABLE                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│            LIMITE DE CONFIANCE 1 : Acces aux canaux              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Appairage d'appareils (periode de grace de 30s)        │   │
│  │  • Validation AllowFrom / AllowList                       │   │
│  │  • Auth Token/Password/Tailscale                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            LIMITE DE CONFIANCE 2 : Isolation des sessions        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESSIONS D'AGENT                        │   │
│  │  • Cle de session = agent:canal:pair                      │   │
│  │  • Politiques d'outils par agent                          │   │
│  │  • Journalisation des transcriptions                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            LIMITE DE CONFIANCE 3 : Execution des outils          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                BAC A SABLE D'EXECUTION                    │   │
│  │  • Bac a sable Docker OU Hote (approbations d'execution)  │   │
│  │  • Execution distante de noeuds                            │   │
│  │  • Protection SSRF (epinglage DNS + blocage IP)           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            LIMITE DE CONFIANCE 4 : Contenu externe               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              URL RECUPEREES / EMAILS / WEBHOOKS            │   │
│  │  • Encapsulation du contenu externe (balises XML)         │   │
│  │  • Injection d'avis de sécurité                           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            LIMITE DE CONFIANCE 5 : Chaine d'approvisionnement    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publication de Skills (semver, SKILL.md requis)        │   │
│  │  • Indicateurs de moderation par motifs                   │   │
│  │  • Analyse VirusTotal (bientot disponible)                │   │
│  │  • Verification de l'anciennete du compte GitHub          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Flux de données

| Flux | Source  | Destination | Données              | Protection              |
| ---- | ------- | ----------- | -------------------- | ----------------------- |
| F1   | Canal   | Gateway     | Messages utilisateur | TLS, AllowFrom          |
| F2   | Gateway | Agent       | Messages routes      | Isolation de session    |
| F3   | Agent   | Outils      | Invocations d'outils | Application de politique |
| F4   | Agent   | Externe     | Requêtes web_fetch   | Blocage SSRF            |
| F5   | ClawHub | Agent       | Code de Skill        | Modération, analyse     |
| F6   | Agent   | Canal       | Réponses             | Filtrage de sortie      |

---

## 3. Analyse des menaces par tactique ATLAS

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001 : Découverte des points d'accès de l'agent

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0006 - Activé Scanning                                                 |
| **Description**            | L'attaquant recherche les points d'accès du Gateway OpenClaw exposes         |
| **Vecteur d'attaque**      | Scan réseau, requêtes Shodan, énumération DNS                                |
| **Composants affectes**    | Gateway, points d'accès API exposes                                          |
| **Mesures actuelles**      | Option d'auth Tailscale, liaison au loopback par défaut                      |
| **Risque residuel**        | Moyen - Les Gateways publics sont découverts                                 |
| **Recommandations**        | Documenter le déploiement sécurisé, ajouter une limitation de débit sur les points d'accès de découverte |

#### T-RECON-002 : Sondage des intégrations de canaux

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0006 - Activé Scanning                                                 |
| **Description**            | L'attaquant sonde les canaux de messagerie pour identifier les comptes gérés par l'IA |
| **Vecteur d'attaque**      | Envoi de messages de test, observation des modèles de réponse                |
| **Composants affectes**    | Toutes les intégrations de canaux                                            |
| **Mesures actuelles**      | Aucune spécifique                                                            |
| **Risque residuel**        | Faible - Valeur limitée de la découverte seule                               |
| **Recommandations**        | Envisager la randomisation du temps de réponse                               |

---

### 3.2 Accès initial (AML.TA0004)

#### T-ACCESS-001 : Interception du code d'appairage

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0040 - AI Model Inference API Accèss                                    |
| **Description**            | L'attaquant intercepte le code d'appairage pendant la période de grace de 30s |
| **Vecteur d'attaque**      | Espionnage visuel, interception réseau, ingénierie sociale                   |
| **Composants affectes**    | Système d'appairage d'appareils                                              |
| **Mesures actuelles**      | Expiration a 30s, codes envoyés via le canal existant                        |
| **Risque residuel**        | Moyen - Periode de grace exploitable                                         |
| **Recommandations**        | Réduire la période de grace, ajouter une étape de confirmation               |

#### T-ACCESS-002 : Usurpation AllowFrom

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0040 - AI Model Inference API Accèss                                    |
| **Description**            | L'attaquant usurpe l'identité d'un expéditeur autorisé dans le canal         |
| **Vecteur d'attaque**      | Selon le canal : usurpation de numéro, usurpation de nom d'utilisateur       |
| **Composants affectes**    | Validation AllowFrom par canal                                               |
| **Mesures actuelles**      | Vérification d'identité spécifique au canal                                  |
| **Risque residuel**        | Moyen - Certains canaux sont vulnérables a l'usurpation                      |
| **Recommandations**        | Documenter les risques spécifiques aux canaux, ajouter une vérification cryptographique la ou c'est possible |

#### T-ACCESS-003 : Vol de jetons

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0040 - AI Model Inference API Accèss                                    |
| **Description**            | L'attaquant vole les jetons d'authentification des fichiers de configuration  |
| **Vecteur d'attaque**      | Malware, accès non autorise a l'appareil, exposition de sauvegarde de config  |
| **Composants affectes**    | ~/.openclaw/credentials/, stockage de configuration                           |
| **Mesures actuelles**      | Permissions de fichiers                                                       |
| **Risque residuel**        | Élevé - Jetons stockes en clair                                               |
| **Recommandations**        | Implémenter le chiffrement des jetons au repos, ajouter la rotation des jetons |

---

### 3.3 Exécution (AML.TA0005)

#### T-EXEC-001 : Injection directe de prompt

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0051.000 - LLM Prompt Injection: Direct                                |
| **Description**            | L'attaquant envoie des prompts élaborés pour manipuler le comportement de l'agent |
| **Vecteur d'attaque**      | Messages de canal contenant des instructions adverses                         |
| **Composants affectes**    | LLM de l'agent, toutes les surfaces d'entrée                                 |
| **Mesures actuelles**      | Détection de motifs, encapsulation du contenu externe                         |
| **Risque residuel**        | Critique - Détection uniquement, pas de blocage ; les attaques sophistiquees contournent |
| **Recommandations**        | Implémenter une defense multicouche, validation de sortie, confirmation utilisateur pour les actions sensibles |

#### T-EXEC-002 : Injection indirecte de prompt

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0051.001 - LLM Prompt Injection: Indirect                              |
| **Description**            | L'attaquant intégré des instructions malveillantes dans le contenu récupéré   |
| **Vecteur d'attaque**      | URL malveillantes, emails empoisonnés, webhooks compromis                    |
| **Composants affectes**    | web_fetch, ingestion d'emails, sources de données externes                   |
| **Mesures actuelles**      | Encapsulation du contenu avec balises XML et avis de sécurité                |
| **Risque residuel**        | Élevé - Le LLM peut ignorer les instructions d'encapsulation                  |
| **Recommandations**        | Implémenter l'assainissement du contenu, séparer les contextes d'exécution    |

#### T-EXEC-003 : Injection d'arguments d'outils

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0051.000 - LLM Prompt Injection: Direct                                |
| **Description**            | L'attaquant manipule les arguments d'outils via l'injection de prompt         |
| **Vecteur d'attaque**      | Prompts élaborés qui influencent les valeurs des paramètres d'outils          |
| **Composants affectes**    | Toutes les invocations d'outils                                              |
| **Mesures actuelles**      | Approbations d'exécution pour les commandes dangereuses                       |
| **Risque residuel**        | Élevé - Depend du jugement de l'utilisateur                                   |
| **Recommandations**        | Implémenter la validation des arguments, appels d'outils paramètres           |

#### T-EXEC-004 : Contournement de l'approbation d'exécution

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0043 - Craft Adversarial Data                                           |
| **Description**            | L'attaquant élaboré des commandes qui contournent la liste d'autorisation d'approbation |
| **Vecteur d'attaque**      | Obfuscation de commandes, exploitation d'alias, manipulation de chemins       |
| **Composants affectes**    | exec-approvals.ts, liste d'autorisation de commandes                          |
| **Mesures actuelles**      | Liste d'autorisation + mode demande                                           |
| **Risque residuel**        | Élevé - Pas d'assainissement de commandes                                     |
| **Recommandations**        | Implémenter la normalisation de commandes, élargir la liste de blocage        |

---

### 3.4 Persistance (AML.TA0006)

#### T-PERSIST-001 : Installation de Skill malveillant

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0010.001 - Supply Chain Compromise: AI Software                         |
| **Description**            | L'attaquant publie un Skill malveillant sur ClawHub                           |
| **Vecteur d'attaque**      | Créer un compte, publier un Skill avec du code malveillant cache              |
| **Composants affectes**    | ClawHub, chargement de Skills, exécution de l'agent                           |
| **Mesures actuelles**      | Vérification de l'anciennete du compte GitHub, indicateurs de modération par motifs |
| **Risque residuel**        | Critique - Pas de bac a sable, revue limitée                                  |
| **Recommandations**        | Intégration VirusTotal (en cours), bac a sable pour Skills, revue communautaire |

#### T-PERSIST-002 : Empoisonnement de mise a jour de Skill

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0010.001 - Supply Chain Compromise: AI Software                         |
| **Description**            | L'attaquant compromet un Skill populaire et pousse une mise a jour malveillante |
| **Vecteur d'attaque**      | Compromission de compte, ingénierie sociale du propriétaire du Skill          |
| **Composants affectes**    | Versionnage ClawHub, flux de mise a jour automatique                          |
| **Mesures actuelles**      | Empreinte de version                                                          |
| **Risque residuel**        | Élevé - Les mises a jour automatiques peuvent tirer des versions malveillantes |
| **Recommandations**        | Implémenter la signature des mises a jour, capacité de retour arriere, épinglage de version |

#### T-PERSIST-003 : Falsification de la configuration de l'agent

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0010.002 - Supply Chain Compromise: Data                                |
| **Description**            | L'attaquant modifie la configuration de l'agent pour maintenir l'accès        |
| **Vecteur d'attaque**      | Modification de fichiers de configuration, injection de paramètres            |
| **Composants affectes**    | Configuration de l'agent, politiques d'outils                                 |
| **Mesures actuelles**      | Permissions de fichiers                                                       |
| **Risque residuel**        | Moyen - Nécessite un accès local                                              |
| **Recommandations**        | Vérification d'intégrité de la configuration, journalisation d'audit pour les changements de configuration |

---

### 3.5 Évasion de defense (AML.TA0007)

#### T-EVADE-001 : Contournement des motifs de modération

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0043 - Craft Adversarial Data                                           |
| **Description**            | L'attaquant élaboré le contenu d'un Skill pour contourner les motifs de modération |
| **Vecteur d'attaque**      | Homoglyphes Unicode, astuces d'encodage, chargement dynamique                |
| **Composants affectes**    | modération.ts de ClawHub                                                     |
| **Mesures actuelles**      | FLAG_RULES base sur des motifs                                               |
| **Risque residuel**        | Élevé - Regex simple facilement contournée                                    |
| **Recommandations**        | Ajouter l'analyse comportementale (VirusTotal Code Insight), détection basee sur l'AST |

#### T-EVADE-002 : Échappement de l'encapsulation de contenu

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0043 - Craft Adversarial Data                                           |
| **Description**            | L'attaquant élaboré un contenu qui échappe au contexte d'encapsulation XML    |
| **Vecteur d'attaque**      | Manipulation de balises, confusion de contexte, remplacement d'instructions   |
| **Composants affectes**    | Encapsulation du contenu externe                                              |
| **Mesures actuelles**      | Balises XML + avis de sécurité                                                |
| **Risque residuel**        | Moyen - De nouveaux échappements sont découverts régulièrement                |
| **Recommandations**        | Couches d'encapsulation multiples, validation côté sortie                     |

---

### 3.6 Découverte (AML.TA0008)

#### T-DISC-001 : Énumération des outils

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0040 - AI Model Inference API Accèss                                    |
| **Description**            | L'attaquant énumère les outils disponibles par le biais de prompts            |
| **Vecteur d'attaque**      | Requêtes de type "Quels outils avez-vous ?"                                  |
| **Composants affectes**    | Registre d'outils de l'agent                                                 |
| **Mesures actuelles**      | Aucune spécifique                                                            |
| **Risque residuel**        | Faible - Les outils sont généralement documentes                              |
| **Recommandations**        | Envisager des contrôles de visibilité des outils                              |

#### T-DISC-002 : Extraction de données de session

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0040 - AI Model Inference API Accèss                                    |
| **Description**            | L'attaquant extrait des données sensibles du contexte de session              |
| **Vecteur d'attaque**      | Requêtes "De quoi avons-nous discute ?", sondage de contexte                  |
| **Composants affectes**    | Transcriptions de session, fenêtre de contexte                                |
| **Mesures actuelles**      | Isolation de session par expéditeur                                           |
| **Risque residuel**        | Moyen - Données intra-session accessibles                                     |
| **Recommandations**        | Implémenter la redaction des données sensibles dans le contexte               |

---

### 3.7 Collecte et exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001 : Vol de données via web_fetch

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0009 - Collection                                                       |
| **Description**            | L'attaquant exfiltre des données en instruisant l'agent d'envoyer a une URL externe |
| **Vecteur d'attaque**      | Injection de prompt amenant l'agent a POST des données vers le serveur de l'attaquant |
| **Composants affectes**    | Outil web_fetch                                                               |
| **Mesures actuelles**      | Blocage SSRF pour les réseaux internes                                        |
| **Risque residuel**        | Élevé - Les URL externes sont autorisées                                      |
| **Recommandations**        | Implémenter une liste d'autorisation d'URL, sensibilisation a la classification des données |

#### T-EXFIL-002 : Envoi de messages non autorise

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0009 - Collection                                                       |
| **Description**            | L'attaquant amene l'agent a envoyer des messages contenant des données sensibles |
| **Vecteur d'attaque**      | Injection de prompt amenant l'agent a contacter l'attaquant                   |
| **Composants affectes**    | Outil message, intégrations de canaux                                         |
| **Mesures actuelles**      | Contrôle d'accès de la messagerie sortante                                    |
| **Risque residuel**        | Moyen - Le contrôle d'accès peut être contourne                               |
| **Recommandations**        | Exiger une confirmation explicite pour les nouveaux destinataires              |

#### T-EXFIL-003 : Collecte d'identifiants

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0009 - Collection                                                       |
| **Description**            | Un Skill malveillant collecte des identifiants depuis le contexte de l'agent  |
| **Vecteur d'attaque**      | Le code du Skill lit les variables d'environnement, les fichiers de configuration |
| **Composants affectes**    | Environnement d'exécution des Skills                                          |
| **Mesures actuelles**      | Aucune spécifique aux Skills                                                  |
| **Risque residuel**        | Critique - Les Skills s'exécutent avec les privileges de l'agent               |
| **Recommandations**        | Bac a sable pour les Skills, isolation des identifiants                        |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001 : Exécution de commande non autorisée

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0031 - Erode AI Model Integrity                                        |
| **Description**            | L'attaquant exécute des commandes arbitraires sur le système de l'utilisateur |
| **Vecteur d'attaque**      | Injection de prompt combinee avec contournement de l'approbation d'exécution  |
| **Composants affectes**    | Outil Bash, exécution de commandes                                            |
| **Mesures actuelles**      | Approbations d'exécution, option de bac a sable Docker                        |
| **Risque residuel**        | Critique - Exécution sur l'hôte sans bac a sable                              |
| **Recommandations**        | Bac a sable par défaut, améliorer l'UX d'approbation                          |

#### T-IMPACT-002 : Épuisement des ressources (DoS)

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0031 - Erode AI Model Integrity                                        |
| **Description**            | L'attaquant épuise les credits API ou les ressources de calcul                |
| **Vecteur d'attaque**      | Inondation automatisee de messages, appels d'outils couteux                   |
| **Composants affectes**    | Gateway, sessions d'agent, fournisseur d'API                                  |
| **Mesures actuelles**      | Aucune                                                                        |
| **Risque residuel**        | Élevé - Pas de limitation de débit                                            |
| **Recommandations**        | Implémenter des limités de débit par expéditeur, budgets de coûts              |

#### T-IMPACT-003 : Atteinte a la reputation

| Attribut                   | Valeur                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Identifiant ATLAS**      | AML.T0031 - Erode AI Model Integrity                                        |
| **Description**            | L'attaquant amene l'agent a envoyer du contenu nuisible/offensant             |
| **Vecteur d'attaque**      | Injection de prompt provoquant des réponses inappropriees                      |
| **Composants affectes**    | Génération de sortie, messagerie par canal                                    |
| **Mesures actuelles**      | Politiques de contenu du fournisseur LLM                                      |
| **Risque residuel**        | Moyen - Filtrés du fournisseur imparfaits                                     |
| **Recommandations**        | Couche de filtrage de sortie, contrôles utilisateur                            |

---

## 4. Analyse de la chaine d'approvisionnement ClawHub

### 4.1 Contrôles de sécurité actuels

| Contrôle                    | Implementation              | Efficacite                                                 |
| --------------------------- | --------------------------- | ---------------------------------------------------------- |
| Anciennete du compte GitHub | `requireGitHubAccountAge()` | Moyenne - Élevé la barre pour les nouveaux attaquants       |
| Assainissement des chemins  | `sanitizePath()`            | Élevée - Empêche la traversee de répertoire                 |
| Validation du type de fichier | `isTextFile()`            | Moyenne - Fichiers texte uniquement, mais peuvent être malveillants |
| Limités de taille           | Bundle total de 50 Mo       | Élevée - Empêche l'épuisement des ressources                |
| SKILL.md requis             | Readme obligatoire          | Faible valeur securitaire - Informatif uniquement            |
| Modération par motifs       | FLAG_RULES dans modération.ts | Faible - Facilement contournée                             |
| Statut de modération        | Champ `moderationStatus`    | Moyenne - Revue manuelle possible                           |

### 4.2 Motifs d'indicateurs de modération

Motifs actuels dans `moderation.ts` :

```javascript
// Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+\|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**Limitations :**

- Vérifié uniquement le slug, displayName, summary, frontmatter, metadata, chemins de fichiers
- N'analyse pas le contenu reel du code du Skill
- Regex simple facilement contournée par obfuscation
- Pas d'analyse comportementale

### 4.3 Améliorations prévues

| Amélioration                | Statut                                 | Impact                                                                 |
| --------------------------- | -------------------------------------- | ---------------------------------------------------------------------- |
| Intégration VirusTotal      | En cours                               | Élevé - Analyse comportementale Code Insight                            |
| Signalement communautaire   | Partiel (table `skillReports` existe)  | Moyen                                                                  |
| Journalisation d'audit      | Partiel (table `auditLogs` existe)     | Moyen                                                                  |
| Système de badges           | Implémenté                             | Moyen - `highlighted`, `official`, `deprecated`, `redactionApproved`   |

---

## 5. Matrice de risques

### 5.1 Probabilite vs Impact

| ID de menace  | Probabilite | Impact   | Niveau de risque | Priorité |
| ------------- | ----------- | -------- | ---------------- | -------- |
| T-EXEC-001    | Élevée      | Critique | **Critique**     | P0       |
| T-PERSIST-001 | Élevée      | Critique | **Critique**     | P0       |
| T-EXFIL-003   | Moyenne     | Critique | **Critique**     | P0       |
| T-IMPACT-001  | Moyenne     | Critique | **Élevé**        | P1       |
| T-EXEC-002    | Élevée      | Élevé    | **Élevé**        | P1       |
| T-EXEC-004    | Moyenne     | Élevé    | **Élevé**        | P1       |
| T-ACCESS-003  | Moyenne     | Élevé    | **Élevé**        | P1       |
| T-EXFIL-001   | Moyenne     | Élevé    | **Élevé**        | P1       |
| T-IMPACT-002  | Élevée      | Moyen    | **Élevé**        | P1       |
| T-EVADE-001   | Élevée      | Moyen    | **Moyen**        | P2       |
| T-ACCESS-001  | Faible      | Élevé    | **Moyen**        | P2       |
| T-ACCESS-002  | Faible      | Élevé    | **Moyen**        | P2       |
| T-PERSIST-002 | Faible      | Élevé    | **Moyen**        | P2       |

### 5.2 Chaines d'attaque critiques

**Chaine d'attaque 1 : Vol de données via un Skill**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publier un Skill malveillant) → (Contourner la moderation) → (Collecter les identifiants)
```

**Chaine d'attaque 2 : De l'injection de prompt a l'exécution de code a distance**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Injecter un prompt) → (Contourner l'approbation d'execution) → (Executer des commandes)
```

**Chaine d'attaque 3 : Injection indirecte via du contenu récupéré**

```
T-EXEC-002 → T-EXFIL-001 → Exfiltration externe
(Empoisonner le contenu d'une URL) → (L'agent recupere et suit les instructions) → (Donnees envoyees a l'attaquant)
```

---

## 6. Résumé des recommandations

### 6.1 Immédiat (P0)

| ID    | Recommandation                                   | Adresse                    |
| ----- | ------------------------------------------------ | -------------------------- |
| R-001 | Compléter l'intégration VirusTotal               | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implémenter le bac a sable pour les Skills       | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Ajouter la validation de sortie pour les actions sensibles | T-EXEC-001, T-EXEC-002 |

### 6.2 Court terme (P1)

| ID    | Recommandation                                    | Adresse      |
| ----- | ------------------------------------------------- | ------------ |
| R-004 | Implémenter la limitation de débit                | T-IMPACT-002 |
| R-005 | Ajouter le chiffrement des jetons au repos        | T-ACCESS-003 |
| R-006 | Améliorer l'UX et la validation de l'approbation d'exécution | T-EXEC-004 |
| R-007 | Implémenter une liste d'autorisation d'URL pour web_fetch | T-EXFIL-001 |

### 6.3 Moyen terme (P2)

| ID    | Recommandation                                                | Adresse       |
| ----- | ------------------------------------------------------------- | ------------- |
| R-008 | Ajouter la vérification cryptographique des canaux la ou c'est possible | T-ACCESS-002 |
| R-009 | Implémenter la vérification d'intégrité de la configuration   | T-PERSIST-003 |
| R-010 | Ajouter la signature des mises a jour et l'épinglage de version | T-PERSIST-002 |

---

## 7. Annexes

### 7.1 Mappage des techniques ATLAS

| Identifiant ATLAS | Nom de la technique                  | Menaces OpenClaw                                                 |
| ----------------- | ------------------------------------ | ---------------------------------------------------------------- |
| AML.T0006         | Activé Scanning                      | T-RECON-001, T-RECON-002                                         |
| AML.T0009         | Collection                           | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001     | Supply Chain: AI Software            | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002     | Supply Chain: Data                   | T-PERSIST-003                                                    |
| AML.T0031         | Erode AI Model Integrity             | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040         | AI Model Inference API Accèss        | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043         | Craft Adversarial Data               | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000     | LLM Prompt Injection: Direct         | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001     | LLM Prompt Injection: Indirect       | T-EXEC-002                                                       |

### 7.2 Fichiers de sécurité clés

| Chemin                              | Fonction                              | Niveau de risque |
| ----------------------------------- | ------------------------------------- | ---------------- |
| `src/infra/exec-approvals.ts`       | Logique d'approbation de commandes    | **Critique**     |
| `src/gateway/auth.ts`               | Authentification du Gateway           | **Critique**     |
| `src/web/inbound/access-control.ts` | Contrôle d'accès aux canaux           | **Critique**     |
| `src/infra/net/ssrf.ts`             | Protection SSRF                       | **Critique**     |
| `src/security/external-content.ts`  | Attenuation de l'injection de prompt  | **Critique**     |
| `src/agents/sandbox/tool-policy.ts` | Application de la politique d'outils  | **Critique**     |
| `convex/lib/moderation.ts`          | Modération ClawHub                    | **Élevé**        |
| `convex/lib/skillPublish.ts`        | Flux de publication de Skills         | **Élevé**        |
| `src/routing/resolve-route.ts`      | Isolation de session                  | **Moyen**        |

### 7.3 Glossaire

| Terme                | Définition                                                       |
| -------------------- | ---------------------------------------------------------------- |
| **ATLAS**            | Adversarial Threat Landscape for AI Systems de MITRE             |
| **ClawHub**          | Marketplace de Skills d'OpenClaw                                  |
| **Gateway**          | Couche de routage de messages et d'authentification d'OpenClaw    |
| **MCP**              | Model Context Protocol - interface de fournisseur d'outils        |
| **Injection de prompt** | Attaque ou des instructions malveillantes sont intégrées en entrée |
| **Skill**            | Extension téléchargeable pour les agents OpenClaw              |
| **SSRF**             | Server-Side Request Forgery                                       |

---

_Ce modèle de menaces est un document évolutif. Signalez les problèmes de sécurité a security@openclaw.ai_
