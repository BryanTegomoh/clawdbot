---
summary: "Pipeline de formatage Markdown pour les canaux sortants"
read_when:
  - Vous modifiez le formatage Markdown ou le découpage pour les canaux sortants
  - Vous ajoutez un nouveau formateur de canal ou un mappage de styles
  - Vous déboguez des régressions de formatage entre les canaux
title: "Formatage Markdown"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/markdown-formatting.md
  workflow: manual
---

# Formatage Markdown

OpenClaw formate le Markdown sortant en le convertissant en une représentation intermédiaire (IR) partagée avant de produire la sortie spécifique à chaque canal. L'IR conserve le texte source intact tout en portant des spans de style/lien afin que le découpage et le rendu restent cohérents entre les canaux.

## Objectifs

- **Cohérence :** une seule étape d'analyse, plusieurs moteurs de rendu.
- **Découpage sûr :** découper le texte avant le rendu pour que le formatage en ligne ne soit jamais coupé entre les morceaux.
- **Adaptation au canal :** mapper la même IR vers Slack mrkdwn, Telegram HTML et les plages de style Signal sans ré-analyser le Markdown.

## Pipeline

1. **Analyse Markdown -> IR**
   - L'IR est du texte brut plus des spans de style (gras/italique/barré/code/spoiler) et des spans de lien.
   - Les offsets sont en unités de code UTF-16 afin que les plages de style Signal s'alignent avec son API.
   - Les tableaux sont analysés uniquement quand un canal opte pour la conversion de tableaux.
2. **Découpage de l'IR (formatage d'abord)**
   - Le découpage se fait sur le texte IR avant le rendu.
   - Le formatage en ligne n'est pas coupé entre les morceaux ; les spans sont découpés par morceau.
3. **Rendu par canal**
   - **Slack :** jetons mrkdwn (gras/italique/barré/code), liens en `<url|label>`.
   - **Telegram :** balises HTML (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   - **Signal :** texte brut + plages `text-style` ; les liens deviennent `label (url)` quand le label diffère.

## Exemple d'IR

Markdown en entrée :

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (schématique) :

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## Utilisation

- Les adaptateurs sortants Slack, Telegram et Signal effectuent le rendu depuis l'IR.
- Les autres canaux (WhatsApp, iMessage, MS Teams, Discord) utilisent encore le texte brut ou leurs propres règles de formatage, avec la conversion de tableaux Markdown appliquée avant le découpage quand elle est activée.

## Gestion des tableaux

Les tableaux Markdown ne sont pas pris en charge de manière cohérente par les clients de chat. Utilisez `markdown.tables` pour contrôler la conversion par canal (et par compte).

- `code` : afficher les tableaux comme blocs de code (par défaut pour la plupart des canaux).
- `bullets` : convertir chaque ligne en puces (par défaut pour Signal + WhatsApp).
- `off` : désactiver l'analyse et la conversion des tableaux ; le texte brut du tableau passe tel quel.

Clés de configuration :

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## Règles de découpage

- Les limités de découpage proviennent des adaptateurs/configuration du canal et sont appliquées au texte IR.
- Les blocs de code délimités sont préservés comme un seul bloc avec un saut de ligne final pour que les canaux les rendent correctement.
- Les préfixes de liste et les préfixes de citation font partie du texte IR, donc le découpage ne coupe pas au milieu d'un préfixe.
- Les styles en ligne (gras/italique/barré/code en ligne/spoiler) ne sont jamais coupés entre les morceaux ; le moteur de rendu réouvre les styles dans chaque morceau.

Si vous avez besoin de plus de détails sur le comportement de découpage entre les canaux, voir [Streaming + découpage](/concepts/streaming).

## Politique de liens

- **Slack :** `[label](url)` -> `<url|label>` ; les URLs nues restent nues. L'autolien est désactivé pendant l'analyse pour éviter le double-lien.
- **Telegram :** `[label](url)` -> `<a href="url">label</a>` (mode d'analyse HTML).
- **Signal :** `[label](url)` -> `label (url)` sauf si le label correspond à l'URL.

## Spoilers

Les marqueurs spoiler (`||spoiler||`) sont analysés uniquement pour Signal, où ils correspondent aux plages de style SPOILER. Les autres canaux les traitent comme du texte brut.

## Comment ajouter ou mettre à jour un formateur de canal

1. **Analyser une fois :** utiliser le helper partagé `markdownToIR(...)` avec les options appropriées au canal (autolien, style de titre, préfixe de citation).
2. **Rendre :** implémenter un moteur de rendu avec `renderMarkdownWithMarkers(...)` et une carte de marqueurs de style (ou les plages de style Signal).
3. **Découper :** appeler `chunkMarkdownIR(...)` avant le rendu ; rendre chaque morceau.
4. **Connecter l'adaptateur :** mettre à jour l'adaptateur sortant du canal pour utiliser le nouveau découpeur et moteur de rendu.
5. **Tester :** ajouter ou mettre à jour les tests de formatage et un test de livraison sortante si le canal utilise le découpage.

## Pièges courants

- Les jetons entre chevrons Slack (`<@U123>`, `<#C123>`, `<https://...>`) doivent être préservés ; échapper le HTML brut de manière sûre.
- Le HTML Telegram nécessite d'échapper le texte en dehors des balises pour éviter un balisage cassé.
- Les plages de style Signal dépendent des offsets UTF-16 ; ne pas utiliser les offsets de points de code.
- Préserver les sauts de ligne finaux pour les blocs de code délimités afin que les marqueurs de fermeture atterrissent sur leur propre ligne.
