# Recevoir

Un remboursement, des gains, un retrait de solde chez vous : vous fixez le montant, et votre client choisit où le recevoir dans la feuille Tchin, un numéro Mobile Money ou une adresse crypto. Vous n'avez rien à lui demander.

## Recevoir ou versement direct ?

|  | Recevoir (cette page) | Versement direct |
|---|---|---|
| `Qui choisit où l’argent arrive` | le client, dans la feuille | vous, dans l’appel |
| `Vous devez connaître` | seulement le montant | le numéro et le réseau, ou l’adresse crypto |
| `Erreur de numéro` | impossible : le client tape le sien et le voit | à votre charge |
| `Idéal pour` | remboursements, gains de vos clients, retraits de portefeuille | vos vendeurs, livreurs, partenaires réguliers |

## Créer un versement

| Champ | Type | Description |
|---|---|---|
| `amount` | nombre | **obligatoire**. Le montant en F CFA. En Mobile Money, le client pourra recevoir de 500 à 1 500 000 F ; en crypto, il est converti au cours du moment. |
| `title` | texte | un titre vu par le client (« Remboursement WG-1042 »), 80 caractères au plus. |
| `description` | texte | une précision (255 caractères). |
| `message` | texte | un mot pour le client (255 caractères). |
| `customer_email` | e-mail | facultatif : le lien part aussi par e-mail, et le client peut le retirer plus tard, depuis n’importe quel appareil. |
| `external_ref` | texte | votre référence (numéro de commande ou de remboursement), rendue partout. |
| `callback_url` | URL | votre webhook pour ce versement. Sans elle : celle de l’application. |
| `env` | texte | `test` ou `live`. Sans lui : **live**. |

```curl
curl -X POST https://tchin.tech/api/v1/payouts/links \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"amount": 15000, "title": "Remboursement WG-1042", "external_ref": "WG-1042",
       "callback_url": "https://wintg.group/webhook-tchin", "env": "live"}'
```

```php
$v = $tchin->creerVersement(15000, ['title' => 'Remboursement WG-1042', 'external_ref' => 'WG-1042', 'env' => 'live'], 'https://wintg.group/webhook-tchin');
// $v['token'] : ouvrez la feuille Recevoir
```

```node
const v = await tchin.creerVersement(15000, { title: 'Remboursement WG-1042', external_ref: 'WG-1042', env: 'live' }, 'https://wintg.group/webhook-tchin');
```

```python
v = tchin.creer_versement(15000, 'https://wintg.group/webhook-tchin', title='Remboursement WG-1042', external_ref='WG-1042', env='live')
```

```java
String json = tchin.creerVersement(15000, "https://wintg.group/webhook-tchin", Map.of("title", "Remboursement WG-1042", "env", "live"));
```

```go
v, err := c.CreerVersement(15000, "https://wintg.group/webhook-tchin", map[string]any{"title": "Remboursement WG-1042", "env": "live"})
```

```json
{
  "success": true,
  "message": "Ouvrez la feuille avec ce token : TchinPay.receive({ token }).",
  "token": "wnbjhfffsnhiu8",
  "url": "https://tchin.tech/retrait/wnbjhfffsnhiu8",
  "payout": {
    "token": "wnbjhfffsnhiu8",
    "status": "open",
    "method": "choice",
    "amount": 15000,
    "currency": "XOF",
    "payout_currency": null,
    "fee": null,
    "net": null,
    "recipient": null,
    "mode": "live",
    "external_ref": "WG-1042",
    "url": "https://tchin.tech/retrait/wnbjhfffsnhiu8",
    "created_at": "2026-11-03T14:20:05+00:00",
    "paid_at": null
  }
}
```

Puis, dans votre page, ouvrez la feuille **Recevoir** :

```html
<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  TchinPay.receive({
    token: v.token,                                   // rendu par votre serveur
    onSuccess: function () { /* l'argent est parti vers le client */ },
    onPending: function () { /* envoi en cours chez le réseau */ },
    onFailed: function () { /* échec, ou versement annulé */ },
    onClose: function () {}
  });
</script>
```

Sans JavaScript, donnez le lien `url` au client (e-mail, message) : la même feuille s'ouvre en page entière. Dans une application mobile, chaque kit a sa fonction `receive`.

## Ce qui se passe

### Mobile Money

1. Le client choisit le pays, le réseau et tape son numéro. La feuille lui montre les frais et ce qu'il reçoit, avant qu'il confirme.

1. Tchin prend sur **votre solde du pays de réception**. S'il ne suffit pas, vos **autres soldes de pays le complètent** : un transfert entre vos soldes, visible dans vos transactions.

1. Le versement part vers le numéro, en quelques secondes. Pas assez sur tous vos soldes : refusé, rien n'est pris, et le client le voit.

### Crypto

1. Le client choisit sa crypto (USDT, TRX, BNB, SOL, GRAM) et colle son adresse (un bouton **Coller**). L'adresse est vérifiée pour le réseau choisi.

1. Le montant est converti au cours du moment et pris sur **votre solde de cette crypto**. Pas assez : refusé.

1. L'envoi part sur le réseau ; la confirmation arrive en quelques secondes à quelques minutes.

Un lien ne sert qu'**une fois** : dès que le client a retiré, il est fermé.

## Les frais

Selon le réglage de votre application (frais de versement) :

- **À votre charge** : le client reçoit exactement `amount` ; votre solde paie le montant et les frais.

- **À la charge du client** : les frais sont pris dans le montant, il reçoit un peu moins (la feuille le lui montre avant).

Mobile Money : 3,4 %, plus le transfert entre vos soldes de pays s'il en faut un. Crypto : 1 % plus le coût réel du réseau (et 1,1 TRX la première fois qu'une adresse TRON reçoit des TRX).

## Suivre et annuler

```json
{
  "success": true,
  "payout": {
    "token": "wnbjhfffsnhiu8",
    "status": "completed",
    "method": "mobile_money",
    "country": "TG",
    "currency": "XOF",
    "payout_currency": "XOF",
    "amount": 15000,
    "fee": 528,
    "net": 15000,
    "recipient": { "moyen": "momo", "pays": "TG", "methode": "t-money-togo", "telephone": "90112233" },
    "mode": "live",
    "external_ref": "WG-1042",
    "fail_reason": null,
    "url": "https://tchin.tech/retrait/wnbjhfffsnhiu8",
    "created_at": "2026-11-03T14:20:05+00:00",
    "paid_at": "2026-11-03T14:22:11+00:00"
  }
}
```

| Champ | Description |
|---|---|
| `status` | l’état (tableau ci-dessous). |
| `method` | `choice` tant que le client n’a pas choisi, puis `mobile_money` ou `crypto`. |
| `amount, currency` | le montant que vous avez créé, en F CFA. |
| `payout_currency` | la monnaie de ce qui part vers le client : `XOF` en Mobile Money, la crypto choisie sinon. `fee` et `net` sont dans cette monnaie. |
| `fee, net` | les frais, et ce que le client a reçu. |
| `recipient` | où l’argent est parti : pays, réseau et numéro, ou crypto et adresse. |
| `fail_reason` | la raison d’un échec, en clair. |
| `paid_at` | quand l’argent est parti. |

| Statut | Sens |
|---|---|
| `open` | le client n’a pas encore choisi : vous pouvez annuler |
| `processing` | l’envoi est en cours chez le réseau |
| `completed` | l’argent est arrivé |
| `failed` | échec : rien n’a été perdu, l’argent est revenu sur vos soldes |
| `cancelled` | annulé par vous |

## Le webhook

À chaque changement d'état (sauf `open`), une seule fois par état, votre `callback_url` reçoit un webhook signé avec `kind = "payout_link"` :

```json
{
  "response_code": "00",
  "response_text": "Décaissement completed",
  "reference": "wnbjhfffsnhiu8",
  "token": "wnbjhfffsnhiu8",
  "kind": "payout_link",
  "status": "completed",
  "fail_reason": "",
  "amount": 15000,
  "fee": 528,
  "net": 15000,
  "currency": "XOF",
  "payout_currency": "XOF",
  "country": "TG",
  "method": "t-money-togo",
  "mode": "live",
  "recipient": { "moyen": "momo", "pays": "TG", "methode": "t-money-togo", "telephone": "90112233" },
  "external_ref": "WG-1042",
  "timestamp": 1793715731,
  "signature": "51be…"
}
```

## Tester

Avec `"env": "test"`, la feuille Recevoir fonctionne comme en vrai, mais aucun argent ne part et aucun solde ne bouge : le versement passe tout de suite à `completed`, et le webhook arrive avec `"mode": "test"`.

---
Source : https://doc.wintg.group/tchin/recevoir/
