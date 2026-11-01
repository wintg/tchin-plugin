# Vérification d'identité

Vos clients prouvent leur identité en une minute : photo de la pièce, selfie avec un contrôle de vivacité. Tchin vérifie et vous donne le verdict. 75 F par vérification, pris sur votre solde Tchin.

## Créer une vérification

| Champ | Type | Description |
|---|---|---|
| `reference` | texte | votre référence (le numéro du client chez vous). |
| `callback_url` | URL | votre webhook : le verdict y arrive. |
| `env` | texte | `test` (gratuit) ou `live`. |

```curl
curl -X POST https://tchin.tech/api/v1/kyc/verifications \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Content-Type: application/json" \
  -d '{"reference": "client-wintg-42", "callback_url": "https://wintg.group/webhook-tchin"}'
```

```php
$v = $tchin->creerVerification('client-wintg-42', 'https://wintg.group/webhook-tchin');
$id = $v['data']['id'];
```

```node
const v = await tchin.creerVerification('client-wintg-42', 'https://wintg.group/webhook-tchin');
const id = v.data.id;
```

```python
v = tchin.creer_verification('client-wintg-42', 'https://wintg.group/webhook-tchin')
id = v['data']['id']
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
    "fee": 150,
    "currency": "XOF",
    "expires_at": "2026-10-20T17:56:50+00:00"
  }
}
```

## Ouvrir la fenêtre

```html
<script src="https://tchin.tech/js/tchin-kyc.js"></script>
<script>
  TchinKYC.open({
    id: 'kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp',
    onComplete: function (r) { /* r.status : approved | rejected | review | expired */ },
    onClose: function () {}
  });
</script>
```

Le client photographie sa pièce (au moins 3 mois avant son expiration), puis fait un selfie en suivant les consignes à l'écran. Tchin compare les visages, lit la pièce et contrôle la vivacité.

## Le résultat

```json
{
  "success": true,
  "data": {
    "id": "kyc_WSXKr1oNN8YFkhMuUPysBk56GTRp",
    "status": "approved",
    "score": 92,
    "person": { "last_name": "AGBEKO", "first_name": "Koffi", "birth_date": "1994-03-12" },
    "document": { "type": "cni", "number": "TG0123456", "expiry_date": "2031-05-02" },
    "liveness": true,
    "checks": [ { "check": "visage", "label": "Le selfie correspond à la pièce", "result": "ok" } ],
    "completed_at": "2026-10-19T18:10:42+00:00"
  }
}
```

| Statut | Sens |
|---|---|
| `created` | créée, le client n’a pas encore ouvert la fenêtre |
| `opened` | le client a ouvert la fenêtre |
| `processing` | photos envoyées, vérification en cours |
| `approved` | identité vérifiée |
| `rejected` | refusée (`reason` dit pourquoi) |
| `review` | à revoir par l’équipe de Tchin |
| `expired` | pas terminée à temps |

Le webhook arrive avec `event` : `kyc.approved`, `kyc.rejected`, `kyc.review`…

---
Source : https://doc.wintg.group/tchin/kyc/
