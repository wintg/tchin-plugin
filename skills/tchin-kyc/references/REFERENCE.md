# Vérification d'identité

Vos clients prouvent leur identité en une minute : photo de la pièce, selfie avec un contrôle de vivacité. Tchin compare, lit la pièce et vous donne le verdict. 75 F par vérification en mode réel, pris sur votre solde Tchin ; gratuite en mode test.

## Le principe

1. Votre serveur **crée une vérification** pour un client. Tchin rend son identifiant (`kyc_…`) et son adresse.

1. Votre page ouvre la **fenêtre de vérification** avec le kit `tchin-kyc.js`. Sur téléphone, le client fait tout dans la fenêtre ; sur ordinateur, il scanne un code QR et continue sur son téléphone (il faut un appareil photo de téléphone).

1. Le client **photographie sa pièce** (carte d'identité, passeport ou permis, encore valable au moins 3 mois), puis fait un **selfie** en suivant les petits gestes demandés à l'écran : c'est le contrôle de vivacité, qui écarte les photos de photos.

1. Tchin **vérifie** : le visage du selfie correspond à la pièce, la pièce est lisible et valable, le client est bien présent. Le verdict arrive en général en quelques minutes, par webhook et par lecture.

## Créer une vérification

| Champ | Type | Description |
|---|---|---|
| `reference` | texte | votre référence (le numéro du client chez vous), 120 caractères au plus. Elle revient dans le résultat et le webhook. |
| `callback_url` | URL | votre webhook : chaque étape et le verdict y arrivent. |
| `env` | texte | `test` (gratuit) ou `live`. Sans lui : **live**. |

```curl
curl -X POST https://tchin.tech/api/v1/kyc/verifications \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"reference": "client-wintg-42", "callback_url": "https://wintg.group/webhook-tchin", "env": "live"}'
```

```php
$v = $tchin->creerVerification('client-wintg-42', 'https://wintg.group/webhook-tchin');
$id = $v['data']['id'];   // à garder avec le client
```

```node
const v = await tchin.creerVerification('client-wintg-42', 'https://wintg.group/webhook-tchin');
const id = v.data.id;      // à garder avec le client
```

```python
v = tchin.creer_verification('client-wintg-42', 'https://wintg.group/webhook-tchin')
id = v['data']['id']        # à garder avec le client
```

```java
String json = tchin.creerVerification("client-wintg-42", "https://wintg.group/webhook-tchin", "live");
```

```go
v, err := c.CreerVerification("client-wintg-42", "https://wintg.group/webhook-tchin", "live")
```

```json
{
  "success": true,
  "data": {
    "id": "kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp",
    "reference": "client-wintg-42",
    "status": "created",
    "mode": "live",
    "url": "https://tchin.tech/verifier/kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp",
    "score": null,
    "reason": null,
    "person": null,
    "document": null,
    "liveness": false,
    "checks": [],
    "fee": 75,
    "currency": "XOF",
    "created_at": "2026-11-03T14:40:00+00:00",
    "submitted_at": null,
    "completed_at": null,
    "expires_at": "2026-11-04T14:40:00+00:00"
  }
}
```

## Ouvrir la fenêtre

```html
<script src="https://tchin.tech/js/tchin-kyc.js"></script>
<script>
  TchinKYC.open({
    id: 'kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp',          // rendu par votre serveur
    onComplete: function (r) { /* r.status : approved | rejected | review | expired (affichage) */ },
    onClose: function () {}
  });
</script>
```

Sans JavaScript, ou dans un e-mail, donnez le lien `url` : la même vérification s'ouvre en page entière. Comme pour un paiement, le verdict qui compte est celui que lit votre serveur (webhook ou lecture), pas le retour de la fenêtre.

## Lire le résultat

```json
{
  "success": true,
  "data": {
    "id": "kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp",
    "reference": "client-wintg-42",
    "status": "approved",
    "mode": "live",
    "score": 92,
    "reason": null,
    "person": { "last_name": "AGBEKO", "first_name": "Koffi", "birth_date": "1994-03-12" },
    "document": { "type": "cni", "number": "TG0123456", "expiry_date": "2031-05-02" },
    "liveness": true,
    "checks": [
      { "check": "visage", "label": "Le selfie correspond à la pièce", "result": "ok" }
    ],
    "fee": 75,
    "currency": "XOF",
    "submitted_at": "2026-11-03T14:43:10+00:00",
    "completed_at": "2026-11-03T14:44:02+00:00",
    "expires_at": null
  }
}
```

| Champ | Description |
|---|---|
| `status` | l’état (tableau ci-dessous). |
| `score` | la confiance de la vérification, de 0 à 100. |
| `reason` | pourquoi elle est refusée ou à revoir, en clair (« photo de la pièce floue »…). |
| `person` | nom, prénom et date de naissance lus sur la pièce. |
| `document` | type, numéro et date d’expiration de la pièce. |
| `liveness` | `true` si le contrôle de vivacité est réussi. |
| `checks` | le détail de chaque contrôle : `check` (son nom), `label` (en clair), `result`. |
| `fee` | le prix de cette vérification (75 F en réel, 0 en test). |
| `expires_at` | tant qu’elle n’est pas envoyée : la date limite. Une vérification jamais terminée expire au bout de 24 heures. |

| Statut | Sens | Que faire |
|---|---|---|
| `created` | créée, le client n’a pas encore ouvert la fenêtre | ouvrir la fenêtre |
| `opened` | le client a ouvert la fenêtre | attendre |
| `processing` | photos envoyées, vérification en cours | attendre le webhook |
| `approved` | identité vérifiée | ouvrir l’accès |
| `rejected` | refusée (`reason` dit pourquoi) | expliquer au client et créer une nouvelle vérification s’il veut réessayer |
| `review` | à revoir par l’équipe de Tchin (cas douteux) | attendre : le verdict final suivra |
| `expired` | pas terminée à temps | créer une nouvelle vérification |

## Le webhook

Votre `callback_url` reçoit un webhook signé avec `kind = "kyc"` et `event` (`kyc.processing`, `kyc.approved`, `kyc.rejected`, `kyc.review`, `kyc.expired`…), et tous les champs du résultat ci-dessus. Pour la signature, `token` est l'identifiant `kyc_…`, `amount` le prix et `net` vaut 0 ; voir la signature.

## Tester

Avec `"env": "test"`, la vérification est gratuite et passe par les mêmes écrans : faites-la vous-même, avec votre pièce et votre selfie, pour voir le parcours de vos clients et recevoir les webhooks avec `"mode": "test"`.

---
Source : https://doc.wintg.group/tchin/kyc/
