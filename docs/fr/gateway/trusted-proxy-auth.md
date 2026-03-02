---
summary: "Déléguer l'authentification du gateway à un proxy inverse de confiance (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Exécuter OpenClaw derrière un proxy sensible à l'identité
  - Configurer Pomerium, Caddy ou nginx avec OAuth devant OpenClaw
  - Corriger les erreurs WebSocket 1008 non autorisées avec des configurations de proxy inverse
  - Décider où configurer HSTS et d'autres en-têtes de renforcement HTTP
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/trusted-proxy-auth.md
  workflow: manual
---

# Authentification par proxy de confiance

> ⚠️ **Fonctionnalité sensible du point de vue de la sécurité.** Ce mode délègue entièrement l'authentification à votre proxy inverse. Une mauvaise configuration peut exposer votre Gateway à un accès non autorise. Lisez attentivement cette page avant de l'activer.

## Quand l'utiliser

Utilisez le mode d'authentification `trusted-proxy` lorsque :

- Vous exécutez OpenClaw derrière un **proxy sensible à l'identité** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Votre proxy gère toute l'authentification et transmet l'identité de l'utilisateur via des en-têtes
- Vous êtes dans un environnement Kubernêtes ou de conteneurs où le proxy est le seul chemin vers le Gateway
- Vous rencontrez des erreurs WebSocket `1008 unauthorized` parce que les navigateurs ne peuvent pas transmettre de jetons dans les charges utiles WS

## Quand NE PAS l'utiliser

- Si votre proxy n'authentifié pas les utilisateurs (simple terminaison TLS ou répartiteur de charge)
- S'il existe un chemin vers le Gateway qui contourne le proxy (trous dans le pare-feu, accès réseau interne)
- Si vous n'êtes pas sûr que votre proxy supprimé/écrase correctement les en-têtes transmis
- Si vous n'avez besoin que d'un accès personnel mono-utilisateur (envisagez Tailscale Serve + loopback pour une configuration plus simple)

## Fonctionnement

1. Votre proxy inverse authentifié les utilisateurs (OAuth, OIDC, SAML, etc.)
2. Le proxy ajouté un en-tête avec l'identité de l'utilisateur authentifié (ex. `x-forwarded-user: nick@example.com`)
3. OpenClaw vérifie que la requête provient d'une **IP de proxy de confiance** (configurée dans `gateway.trustedProxies`)
4. OpenClaw extrait l'identité de l'utilisateur depuis l'en-tête configuré
5. Si tout est correct, la requête est autorisée

## Comportement d'appairage de l'interface de contrôle

Lorsque `gateway.auth.mode = "trusted-proxy"` est actif et que la requête passe
les vérifications de proxy de confiance, les sessions WebSocket de l'interface de contrôle peuvent se connecter sans
identité d'appairage d'appareil.

Implications :

- L'appairage n'est plus le contrôle d'accès principal pour l'interface de contrôle dans ce mode.
- La politique d'authentification de votre proxy inverse et `allowUsers` deviennent le contrôle d'accès effectif.
- Gardez l'ingress du gateway limité aux IP des proxys de confiance uniquement (`gateway.trustedProxies` + pare-feu).

## Configuration

```json5
{
  gateway: {
    // Utilisez loopback pour les configurations de proxy sur le même hôte ; utilisez lan/custom pour les hôtes de proxy distants
    bind: "loopback",

    // CRITIQUE : N'ajoutez ici que les IP de votre proxy
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // En-tête contenant l'identité de l'utilisateur authentifié (requis)
        userHeader: "x-forwarded-user",

        // Optionnel : en-têtes qui DOIVENT être présents (vérification du proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optionnel : restreindre à des utilisateurs spécifiques (vide = autoriser tous)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

Si `gateway.bind` est `loopback`, incluez une adresse proxy loopback dans
`gateway.trustedProxies` (`127.0.0.1`, `::1`, ou un CIDR loopback équivalent).

### Référence de configuration

| Champ                                       | Requis | Description                                                                             |
| ------------------------------------------- | ------ | --------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Oui    | Tableau d'adresses IP de proxy à considérer de confiance. Les requêtes d'autres IP sont rejetées. |
| `gateway.auth.mode`                         | Oui    | Doit être `"trusted-proxy"`                                                             |
| `gateway.auth.trustedProxy.userHeader`      | Oui    | Nom de l'en-tête contenant l'identité de l'utilisateur authentifié                      |
| `gateway.auth.trustedProxy.requiredHeaders` | Non    | En-têtes supplémentaires devant être présents pour que la requête soit considérée de confiance |
| `gateway.auth.trustedProxy.allowUsers`      | Non    | Liste d'autorisation des identités utilisateur. Vide signifie autoriser tous les utilisateurs authentifiés. |

## Terminaison TLS et HSTS

Utilisez un seul point de terminaison TLS et appliquez HSTS à cet endroit.

### Modèle recommandé : terminaison TLS au proxy

Lorsque votre proxy inverse gère le HTTPS pour `https://control.example.com`, définissez
`Strict-Transport-Security` au niveau du proxy pour ce domaine.

- Adapté aux déploiements exposés sur Internet.
- Garde le certificat et la politique de renforcement HTTP au même endroit.
- OpenClaw peut rester en HTTP loopback derrière le proxy.

Exemple de valeur d'en-tête :

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Terminaison TLS au Gateway

Si OpenClaw sert directement en HTTPS (sans proxy de terminaison TLS), définissez :

```json5
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

`strictTransportSecurity` accepte une valeur d'en-tête sous forme de chaîne, ou `false` pour désactiver explicitement.

### Conseils de déploiement

- Commencez avec un max-age court d'abord (par exemple `max-age=300`) pendant la validation du trafic.
- Augmentez vers des valeurs durables (par exemple `max-age=31536000`) uniquement après avoir acquis une confiance élevée.
- Ajoutez `includeSubDomains` uniquement si chaque sous-domaine est prêt pour le HTTPS.
- N'utilisez le preload que si vous respectez intentionnellement les exigences de preload pour l'ensemble de vos domaines.
- Le développement local en loopback uniquement ne bénéficie pas du HSTS.

## Exemples de configuration de proxy

### Pomerium

Pomerium transmet l'identité dans `x-pomerium-claim-email` (ou d'autres en-têtes de claims) et un JWT dans `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Extrait de configuration Pomerium :

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy avec OAuth

Caddy avec le plugin `caddy-security` peut authentifier les utilisateurs et transmettre les en-têtes d'identité.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP de Caddy (si sur le même hôte)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Extrait de Caddyfile :

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy authentifié les utilisateurs et transmet l'identité dans `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Extrait de configuration nginx :

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik avec Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP du conteneur Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Liste de vérification de sécurité

Avant d'activer l'authentification par proxy de confiance, vérifiez :

- [ ] **Le proxy est le seul chemin** : le port du Gateway est protégé par un pare-feu vis-à-vis de tout sauf votre proxy
- [ ] **trustedProxies est minimal** : uniquement les IP réelles de votre proxy, pas des sous-réseaux entiers
- [ ] **Le proxy supprimé les en-têtes** : votre proxy écrase (et non ajoute) les en-têtes `x-forwarded-*` des clients
- [ ] **Terminaison TLS** : votre proxy gère le TLS ; les utilisateurs se connectent via HTTPS
- [ ] **allowUsers est défini** (recommandé) : restreindre aux utilisateurs connus plutôt que d'autoriser quiconque est authentifié

## Audit de sécurité

`openclaw security audit` signalera l'authentification par proxy de confiance avec un niveau de gravité **critique**. C'est intentionnel : c'est un rappel que vous déléguez la sécurité à la configuration de votre proxy.

L'audit vérifie :

- Configuration `trustedProxies` manquante
- Configuration `userHeader` manquante
- `allowUsers` vide (autorisé tout utilisateur authentifié)

## Dépannage

### "trusted_proxy_untrusted_source"

La requête ne provient pas d'une IP dans `gateway.trustedProxies`. Vérifiez :

- L'IP du proxy est-elle correcte ? (Les IP des conteneurs Docker peuvent changer)
- Y a-t-il un répartiteur de charge devant votre proxy ?
- Utilisez `docker inspect` ou `kubectl get pods -o wide` pour trouver les IP réelles

### "trusted_proxy_user_missing"

L'en-tête utilisateur était vide ou manquant. Vérifiez :

- Votre proxy est-il configure pour transmettre les en-têtes d'identité ?
- Le nom de l'en-tête est-il correct ? (insensible à la casse, mais l'orthographe compte)
- L'utilisateur est-il réellement authentifié au niveau du proxy ?

### "trusted*proxy_missing_header*\*"

Un en-tête requis n'était pas présent. Vérifiez :

- La configuration de votre proxy pour ces en-têtes spécifiques
- Si les en-têtes sont supprimés quelque part dans la chaîne

### "trusted_proxy_user_not_allowed"

L'utilisateur est authentifié mais ne figure pas dans `allowUsers`. Ajoutez-le ou supprimez la liste d'autorisation.

### Le WebSocket ne fonctionne toujours pas

Assurez-vous que votre proxy :

- Prend en charge les mises à niveau WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Transmet les en-têtes d'identité lors des requêtes de mise à niveau WebSocket (pas seulement HTTP)
- N'a pas de chemin d'authentification séparé pour les connexions WebSocket

## Migration depuis l'authentification par jeton

Si vous passez de l'authentification par jeton au proxy de confiance :

1. Configurez votre proxy pour authentifier les utilisateurs et transmettre les en-têtes
2. Testez la configuration du proxy de manière indépendante (curl avec les en-têtes)
3. Mettez à jour la configuration OpenClaw avec l'authentification par proxy de confiance
4. Redémarrez le Gateway
5. Testez les connexions WebSocket depuis l'interface de contrôle
6. Exécutez `openclaw security audit` et examinez les résultats

## Connexe

- [Sécurité](/gateway/security) : guide complet de sécurité
- [Configuration](/gateway/configuration) : référence de configuration
- [Accès distant](/gateway/remote) : autres modèles d'accès distant
- [Tailscale](/gateway/tailscale) : alternative plus simple pour un accès limité au tailnet
