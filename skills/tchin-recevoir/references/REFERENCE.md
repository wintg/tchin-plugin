# Recevoir

Un remboursement, des gains, un retrait de solde : votre client tape le montant sur votre site, et choisit où le recevoir dans la feuille Tchin.

## Créer un versement

| Champ | Type | Description |
|---|---|---|
| `amount` | entier | **obligatoire**. Le montant en F CFA (500 à 1 500 000). |
| `title` | texte | un titre (« Remboursement WG-1042 »). |
| `description` | texte | une précision. |
| `message` | texte | un mot pour le client. |
| `customer_email` | e-mail | facultatif : le lien part aussi par e-mail (le client peut le retirer plus tard). |
| `external_ref` | texte | votre référence. |
| `callback_url` | URL | votre webhook pour ce versement. |
| `env` | texte | `test` ou `live`. |

```curl
curl -X POST https://tchin.tech/api/v1/payouts/links \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Content-Type: application/json" \
  -d '{"amount": 15000, "title": "Remboursement WG-1042", "external_ref": "WG-1042",
       "callback_url": "https://wintg.group/webhook-tchin"}'
```

```php
$v = $tchin->creerVersement(15000, ['title' => 'Remboursement WG-1042', 'external_ref' => 'WG-1042'], 'https://wintg.group/webhook-tchin');
// $v['token'] : ouvrez la feuille
```

```node
const v = await tchin.creerVersement(15000, { title: 'Remboursement WG-1042', external_ref: 'WG-1042' }, 'https://wintg.group/webhook-tchin');
```

```python
v = tchin.creer_versement(15000, 'https://wintg.group/webhook-tchin', title='Remboursement WG-1042', external_ref='WG-1042')
```

```java
String json = tchin.creerVersement(15000, "https://wintg.group/webhook-tchin", Map.of("title", "Remboursement WG-1042"));
```

```go
v, err := c.CreerVersement(15000, "https://wintg.group/webhook-tchin", map[string]any{"title": "Remboursement WG-1042"})
```

Puis, dans votre page :

```html
<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  TchinPay.receive({
    token: v.token,
    onSuccess: function () { /* l'argent est parti */ },
    onPending: function () { /* envoi en cours chez le réseau */ },
    onFailed: function () { /* échec ou versement annulé */ },
    onClose: function () {}
  });
</script>
```

## Ce qui se passe

### Mobile Money

1. Le client choisit le pays, le réseau et son numéro. La feuille lui montre les frais et ce qu'il reçoit.

1. Tchin prend sur **votre solde du pays de réception**. S'il ne suffit pas, vos **autres pays le complètent** : un transfert entre vos soldes, et le même mouvement chez notre opérateur.

1. Le versement part vers le numéro. Pas assez sur tous vos soldes : refusé, rien n'est pris.

### Crypto

1. Le client choisit sa crypto (USDT, TRX, BNB, SOL, GRAM) et colle son adresse (un bouton **Coller**).

1. Le montant est converti au cours du jour et pris sur **votre solde de cette crypto**. Pas assez : refusé.

1. L'envoi part sur le réseau ; la confirmation arrive en quelques secondes à quelques minutes.

## Les frais

Selon le réglage de votre application (frais de versement) :

- **À votre charge** : le client reçoit exactement `amount` ; votre solde paie le montant et les frais.

- **À la charge du client** : les frais sont déduits de ce qu'il reçoit.

Mobile Money : 3,4 %, plus le transfert entre pays s'il y en a un. Crypto : 1 % plus le coût réel du réseau (et 1,155 TRX la première fois qu'une adresse TRON reçoit des TRX).

## Suivre et annuler

```json
{
  "success": true,
  "payout": {
    "token": "wnbjhfffsnhiu8",
    "status": "completed",
    "method": "mobile_money",
    "currency": "XOF",
    "payout_currency": "XOF",
    "amount": 15000,
    "fee": 528,
    "net": 15000,
    "recipient": { "moyen": "momo", "pays": "TG", "methode": "t-money-togo", "telephone": "90112233" },
    "mode": "live",
    "external_ref": "WG-1042",
    "paid_at": "2026-10-19T18:02:11+00:00"
  }
}
```

| Statut | Sens |
|---|---|
| `open` | le client n’a pas encore choisi |
| `processing` | l’envoi est en cours |
| `completed` | l’argent est arrivé |
| `failed` | échec : rien n’a été perdu, l’argent est revenu sur vos soldes |
| `cancelled` | annulé par vous |

`fee` et `net` sont dans `payout_currency` : `XOF` en Mobile Money, la crypto choisie sinon. Un lien ne sert qu'une fois.

---
Source : https://doc.wintg.group/tchin/recevoir/
