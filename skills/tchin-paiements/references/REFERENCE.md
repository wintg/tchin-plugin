# Paiements

Un paiement unique : votre serveur le crée, votre page ouvre la feuille Tchin, le client paie sans quitter votre site.

## Créer un paiement

| Champ | Type | Description |
|---|---|---|
| `amount` | entier | **obligatoire**. Le prix, en F CFA (100 à 100 000 000). |
| `description` | texte | ce que paie le client (affiché dans la feuille), 255 caractères. |
| `callback_url` | URL | votre webhook pour ce paiement. Sans elle : le webhook de l’application. |
| `return_url` | URL | où revenir après le paiement (page de paiement ouverte en dehors de la feuille). |
| `cancel_url` | URL | où revenir si le client abandonne. |
| `fees_on_customer` | booléen | qui paie les frais pour CE paiement (par défaut : le réglage de l’application). |
| `env` | texte | `test` ou `live`. Sans lui : **test**. Mettez `"env": "live"` pour encaisser pour de vrai. |

```curl
curl -X POST https://tchin.tech/api/v1/payments \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Content-Type: application/json" \
  -d '{"amount": 25000, "description": "Wintg Cloud - pack Pro", "callback_url": "https://wintg.group/webhook-tchin"}'
```

```php
$p = $tchin->creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin');
$token = $p['token'];
```

```node
const p = await tchin.creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin');
const token = p.token;
```

```python
p = tchin.creer_paiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin')
token = p['token']
```

```java
String json = tchin.creerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", null);
```

```go
p, err := c.CreerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", nil)
token := p["token"]
```

```json
{
  "success": true,
  "token": "jtkkbphr7azsn3",
  "payment_url": "https://tchin.tech/pay/jtkkbphr7azsn3",
  "env": "live"
}
```

## Ouvrir la feuille

### Sur un site web

```html
<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  TchinPay.open({
    token: 'jtkkbphr7azsn3',                         // rendu par votre serveur
    onSuccess: function (r) { /* r.token, r.status = 'success' */ },
    onPending: function (r) { /* demande envoyée sur le téléphone du client */ },
    onFailed: function (r) { /* échec */ },
    onClose: function (r) { /* r.reason : closed | success | replaced */ }
  });
</script>
```

- Sur **téléphone**, la feuille monte du bas et prend exactement la hauteur de son contenu : elle grandit quand le client choisit.

- Sur **ordinateur**, elle montre la page de paiement complète : votre application et le montant à gauche, le paiement à droite.

- Elle est servie par Tchin : son contenu ne se modifie pas. Le client voit le nom et l'image de **votre application**.

### Dans une application mobile

```kotlin
TchinPay.open(this, token, object : TchinPay.Ecouteur {
    override fun onSuccess(token: String) { /* confirmez côté serveur */ }
    override fun onClose(token: String, raison: String) {}
})
```

```swift
TchinPay.open(depuis: self, token: token,
              onSuccess: { token in /* confirmez côté serveur */ },
              onClose: { token, raison in })
```

```dart
final r = await TchinPay.open(context, token); // 'success' | 'closed'
```

```js
// React Native
<TchinPay token={token} onSuccess={() => {}} onClose={() => setToken(null)} />

// Cordova, Capacitor, Ionic
TchinCordova.payer({ token: token, onSuccess: function () {}, onClose: function () {} });
```

Les kits sont sur la page Kits à télécharger.

## Suivre l'état

```json
{
  "success": true,
  "token": "jtkkbphr7azsn3",
  "status": "completed",
  "amount": 25000,
  "currency": "XOF",
  "env": "live",
  "customer": { "name": "Koffi", "email": "koffi@example.com", "phone": "+22890112233" },
  "method": "t-money-togo",
  "crypto": null
}
```

| Statut | Sens |
|---|---|
| `pending` | en attente : pas encore payé, ou demande envoyée sur le téléphone |
| `completed` | payé : livrez |
| `failed` | échoué (refusé, code faux, solde du client insuffisant) : le client peut réessayer |
| `cancelled` | annulé |

## Les frais et le montant payé

Avec les frais **à la charge du client**, le prix affiché monte pour que vous receviez exactement `amount` : pour 25 000 F en Mobile Money (4 %), le client paie 26 042 F. Avec les frais **à votre charge**, le client paie 25 000 F et vous recevez 24 000 F.

L'argent arrive sur **votre solde du pays** où le client a payé (Mobile Money), ou sur **votre solde de la crypto** payée. Payé avec un compte Tchin, il arrive sur le même solde que celui du client, sans frais.

---
Source : https://doc.wintg.group/tchin/paiements/
