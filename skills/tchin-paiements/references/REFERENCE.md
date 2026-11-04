# Paiements

Un paiement unique : votre serveur le crée, votre page ouvre la feuille Tchin, le client paie sans quitter votre site, et votre serveur est prévenu par un webhook signé.

## Le cycle d'un paiement

1. **Création** (`POST /api/v1/payments`) : le paiement existe, à l'état `pending`. Il a un `token` et une `payment_url`.

1. **Ouverture** : votre page ouvre la feuille avec le token. Le client choisit son moyen. Il peut fermer et rouvrir : le même token reprend le même paiement.

1. **Paiement** : en Mobile Money, une demande arrive sur son téléphone et il la valide avec son code secret ; en crypto, il envoie le montant exact à l'adresse donnée ; avec son compte Tchin, il tape son Tchin PIN.

1. **Issue** : `completed` (payé) ou `failed` (refusé : il peut réessayer dans la feuille, avec le même token). Chaque changement envoie un webhook à votre `callback_url`.

Une fois payé, un token a fini son travail : pour un nouvel achat, créez un nouveau paiement.

## Créer un paiement

| Champ | Type | Description |
|---|---|---|
| `amount` | entier | **obligatoire**. Votre prix, en F CFA, de 100 à 100 000 000. Avec les frais à la charge du client, il paiera un peu plus (voir les frais). |
| `description` | texte | ce que paie le client, affiché dans la feuille et sur son reçu (255 caractères au plus). Sans elle : « Paiement ». |
| `callback_url` | URL | votre webhook pour CE paiement. Sans elle : l’adresse de webhook de l’application. Sans les deux : aucun webhook (lisez alors l’état). |
| `return_url` | URL | où ramener le client après le paiement, quand il passe par la page de paiement (`payment_url`) plutôt que par la feuille. |
| `cancel_url` | URL | où le ramener s’il abandonne sur la page de paiement. |
| `fees_on_customer` | booléen | qui paie les frais pour CE paiement : `true` le client, `false` vous. Sans lui : le réglage de l’application. |
| `env` | texte | `test` ou `live`. Sans lui : **test**. Mettez `"env": "live"` pour encaisser pour de vrai. |

```curl
curl -X POST https://tchin.tech/api/v1/payments \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"amount": 25000, "description": "Wintg Cloud - pack Pro",
       "callback_url": "https://wintg.group/webhook-tchin", "env": "live"}'
```

```php
$p = $tchin->creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', ['env' => 'live']);
$token = $p['token'];          // à garder avec la commande
```

```node
const p = await tchin.creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', { env: 'live' });
const token = p.token;         // à garder avec la commande
```

```python
p = tchin.creer_paiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', env='live')
token = p['token']             # à garder avec la commande
```

```java
String json = tchin.creerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", Map.of("env", "live"));
```

```go
p, err := c.CreerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", map[string]any{"env": "live"})
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

| Champ | Description |
|---|---|
| `token` | l’identifiant du paiement : il ouvre la feuille, sert à lire l’état, et revient dans le webhook (champ `reference`). |
| `payment_url` | la page de paiement complète, à donner en lien (e-mail, message, bouton) si vous n’ouvrez pas la feuille. |
| `env` | le mode du paiement : `test` ou `live`. |

## Ouvrir la feuille

### Sur un site web

```html
<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  TchinPay.open({
    token: 'jtkkbphr7azsn3',                         // rendu par votre serveur (la payment_url marche aussi)
    onSuccess: function (r) { /* r.token, r.status = 'success' : affichez « merci » */ },
    onPending: function (r) { /* la demande est partie sur le téléphone du client */ },
    onFailed: function (r) { /* refusé : le client peut réessayer dans la feuille */ },
    onClose: function (r) { /* r.reason : closed | success | replaced */ }
  });
</script>
```

| Option | Description |
|---|---|
| `token` | **obligatoire**. Le token rendu par `POST /payments`. La `payment_url` entière est aussi acceptée. |
| `onSuccess(r)` | le client a payé (`r.status = "success"`). La feuille montre « Paiement réussi » deux secondes, puis se ferme toute seule. |
| `onPending(r)` | le paiement attend une action du client hors de la feuille (valider sur son téléphone). |
| `onFailed(r)` | le paiement a été refusé. La feuille reste ouverte : le client peut changer de moyen ou réessayer. |
| `onClose(r)` | la feuille s’est fermée. `r.reason` : `closed` (le client l’a fermée, ou a touché à côté), `success` (après un paiement réussi), `replaced` (une autre feuille a été ouverte). |

| Fonction | Rôle |
|---|---|
| `TchinPay.open(options)` | ouvre la feuille de paiement (alias : `TchinPay.ouvrir`). Pour un paiement ou la première échéance d’un abonnement. |
| `TchinPay.receive(options)` | ouvre la feuille « Recevoir », pour un versement (alias : `TchinPay.recevoir`). Voir Recevoir. |
| `TchinPay.close()` | ferme la feuille ouverte (alias : `TchinPay.fermer`). |

- Sur **téléphone**, la feuille monte du bas et prend exactement la hauteur de son contenu : elle grandit quand le client choisit, sans jamais dépasser l'écran.

- Sur **ordinateur**, elle montre la page de paiement complète : votre application et le montant à gauche, le paiement à droite.

- Elle est servie par Tchin, dans un cadre isolé de votre page : son contenu ne se modifie pas, et votre page ne voit ni le numéro, ni le code du client. Le client voit le nom et l'image de **votre application**.

- Une seule feuille à la fois : en ouvrir une autre ferme la première (`replaced`).

- La feuille se souvient du dernier moyen utilisé sur l'appareil du client, et Letou, l'assistant de Tchin, l'aide à chaque étape.

### Dans une application mobile

```kotlin
TchinPay.open(this, token, object : TchinPay.Ecouteur {
    override fun onSuccess(token: String) { /* demandez la confirmation à votre serveur */ }
    override fun onClose(token: String, raison: String) {}
})
```

```swift
TchinPay.open(depuis: self, token: token,
              onSuccess: { token in /* demandez la confirmation à votre serveur */ },
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

Les kits mobiles ouvrent la même feuille dans une vue web ; les portefeuilles crypto et les applications de paiement du téléphone s'ouvrent dans le système. Ils sont sur la page Kits à télécharger.

### Sans JavaScript : la page de paiement

Pour un lien dans un e-mail, un SMS, un message WhatsApp ou un bouton simple, donnez la `payment_url` : c'est la même chose que la feuille, en page entière, sur `tchin.tech`. Après le paiement, la page affiche « Paiement réussi » puis ramène le client à votre `return_url`, avec `?status=success&token=…` ajouté à l'adresse (sans `return_url` : la page d'où il venait). S'il abandonne, il revient à `cancel_url`.

## Suivre l'état

```curl
curl https://tchin.tech/api/v1/payments/jtkkbphr7azsn3/status \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" -H "Accept: application/json"
```

```php
$e = $tchin->etatPaiement('jtkkbphr7azsn3');
if ($e['status'] === 'completed') { /* livrez */ }
```

```node
const e = await tchin.etatPaiement('jtkkbphr7azsn3');
if (e.status === 'completed') { /* livrez */ }
```

```python
e = tchin.etat_paiement('jtkkbphr7azsn3')
if e['status'] == 'completed':
    pass  # livrez
```

```java
String json = tchin.etatPaiement("jtkkbphr7azsn3");
```

```go
e, err := c.EtatPaiement("jtkkbphr7azsn3")
```

```json
{
  "success": true,
  "token": "jtkkbphr7azsn3",
  "status": "completed",
  "amount": 25000,
  "currency": "XOF",
  "env": "live",
  "customer": { "name": "Koffi", "email": "koffi@example.com", "phone": "+22890112233" },
  "method": "t-money-togo"
}
```

| Champ | Description |
|---|---|
| `status` | l’état (tableau ci-dessous). |
| `amount` | votre prix, tel que vous l’avez créé (sans les frais ajoutés au client). |
| `currency` | toujours `XOF` : le prix est en F CFA, même payé en crypto. |
| `env` | `test` ou `live`. |
| `customer` | ce que le client a donné en payant (nom, e-mail, téléphone), ou `null` s’il n’a pas encore payé. |
| `method` | présent dès qu’il y a un paiement : le moyen utilisé (`t-money-togo`, `mtn-benin`, `crypto-usdt`, `tchin`…). |
| `crypto` | présent seulement pour un paiement en crypto : la crypto, la quantité exacte attendue, l’adresse et l’état du virement. |

| Statut | Sens | Que faire |
|---|---|---|
| `pending` | en attente : pas encore payé, ou demande en cours sur le téléphone du client | attendre ; rouvrir la feuille si le client revient |
| `completed` | payé | livrer, une seule fois |
| `failed` | refusé (code faux, solde du client insuffisant, demande expirée) | proposer de réessayer : même token, ou un nouveau paiement |
| `cancelled` | annulé | créer un nouveau paiement si le client veut toujours acheter |

En crypto, lire l'état regarde aussi la blockchain tout de suite : un virement arrivé est vu sans attendre.

## Les frais et le montant reçu

Les frais dépendent du moyen choisi par le client (Mobile Money 4 %, crypto 2 %, compte Tchin 0 %) et de qui les paie :

|  | Frais à la charge du client | Frais à votre charge |
|---|---|---|
| `Prix créé (<code>amount</code>)` | 25 000 F | 25 000 F |
| `Le client paie (Mobile Money)` | 26 042 F | 25 000 F |
| `Frais Tchin` | 1 042 F | 1 000 F |
| `Vous recevez` | **25 000 F** | **24 000 F** |

Avec les frais à la charge du client, le prix affiché monte pour que vous receviez exactement votre prix : `payé = prix ÷ (1 − taux)`, arrondi au franc supérieur. Avec les frais à votre charge : `reçu = prix − prix × taux`. Le webhook donne les trois nombres : `amount` (ce que le client a payé), `fee` et `net` (ce que vous recevez).

L'argent arrive sur **votre solde du pays** où le client a payé (Mobile Money), ou sur **votre solde de la crypto** payée (USDT sur votre solde USDT, etc., sans conversion). Payé avec un compte Tchin, il arrive sur le même solde que celui du client, sans frais. Vous le retirez ensuite depuis votre tableau de bord, vers un numéro ou une adresse.

## Bonnes pratiques

- **Un paiement par commande**, créé au moment où le client veut payer. Enregistrez le `token` avec la commande avant d'ouvrir la feuille.

- **Au webhook**, vérifiez la signature, retrouvez la commande par `reference`, vérifiez `mode` et le montant (`net` ou `amount`), puis livrez sur `completed` seulement, une seule fois.

- **Pas de webhook reçu** au bout de quelques minutes alors que le client dit avoir payé : lisez l'état. C'est toujours lui qui tranche.

- **Le client change d'avis** sur le montant : créez un nouveau paiement. Un paiement garde son montant.

## Erreurs courantes

| Réponse | Cause | Solution |
|---|---|---|
| `401 « Clés API manquantes »` | un des deux en-têtes est absent (souvent une variable d’environnement vide) | vérifier `TCHIN-PUBLIC-KEY` et `TCHIN-PRIVATE-KEY` |
| `422 sur <code>amount</code>` | montant absent, décimal, en texte ou hors de 100 à 100 000 000 | envoyer un entier en F CFA |
| `404 « Paiement introuvable »` | le token vient d’une autre application, ou est mal recopié | lire l’état avec les clés de l’application qui l’a créé |
| `La feuille dit « Mode test »` | `env` absent : le défaut est `test` | ajouter `"env": "live"` |
| `Pas de webhook` | ni `callback_url`, ni adresse dans l’application ; ou adresse en HTTP, locale ou injoignable | donner une adresse HTTPS publique |

---
Source : https://doc.wintg.group/tchin/paiements/
