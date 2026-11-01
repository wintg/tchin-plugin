# Webhooks et signature

À chaque changement d'état, Tchin appelle votre callback_url. Vérifiez la signature avant de faire confiance au contenu.

## Le format

Un `POST` en `application/x-www-form-urlencoded`, avec un champ `data` (les sous-champs arrivent en `data[reference]`, `data[status]`…) et deux en-têtes :

| En-tête | Valeur |
|---|---|
| `Tchin-Signature` | `v1=` suivi de la signature (aussi dans `data[signature]`) |
| `Tchin-Timestamp` | l’heure de l’envoi, en secondes Unix (aussi dans `data[timestamp]`) |

```json
{
  "response_code": "00",
  "response_text": "Transaction completed",
  "reference": "jtkkbphr7azsn3",
  "token": "tch_9w2kq…",
  "status": "completed",
  "amount": 26042,
  "fee": 1042,
  "net": 25000,
  "currency": "XOF",
  "country": "TG",
  "method": "t-money-togo",
  "method_name": "Yas",
  "mode": "live",
  "customer": { "name": "Koffi", "email": "koffi@example.com", "phone": "+22890112233" },
  "timestamp": 1792432544,
  "signature": "3f1a…"
}
```

- **Paiement** : `reference` est le token du paiement ; payé en crypto, un bloc `crypto` (devise, quantité envoyée, quantité créditée, transaction).

- **Abonnement** : le champ `event` (`subscription.charged`…) et le bloc `subscription`.

- **Recevoir** : `kind` vaut `payout_link` ; `recipient`, `payout_currency`.

- **Vérification d'identité** : `kind` vaut `kyc` et `event` `kyc.approved`…

## Vérifier la signature

La signature est un **HMAC-SHA256**, avec votre **clé secrète**, de cette chaîne :

```texte
timestamp.reference.token.status.amount.net.mode
```

Refusez aussi un webhook trop ancien (plus de 10 minutes) : un message rejoué ne passera pas. Les kits le font pour vous :

```php
$d = $tchin->verifierWebhook($_POST['data'] ?? []);   // null si la signature est fausse ou ancienne
```

```node
const d = tchin.verifierWebhook(req.body.data);       // null si la signature est fausse ou ancienne
```

```python
data = {k[5:-1]: v for k, v in request.form.items() if k.startswith('data[')}
d = tchin.verifier_webhook(data)                      # None si la signature est fausse ou ancienne
```

```java
boolean ok = tchin.verifierWebhook(data);             // data : les champs data[xxx] → xxx
```

```go
ok := c.VerifierWebhook(r.PostForm)                   // après r.ParseForm()
```

Sans kit, dans n'importe quel langage :

```php
$d = $_POST['data'];
$chaine = implode('.', [$d['timestamp'], $d['reference'], $d['token'], $d['status'], $d['amount'], $d['net'], $d['mode']]);
$ok = hash_equals(hash_hmac('sha256', $chaine, $cleSecrete), $d['signature'])
   && abs(time() - (int) $d['timestamp']) <= 600;
```

```node
const crypto = require('crypto');
const d = req.body.data;
const chaine = [d.timestamp, d.reference, d.token, d.status, d.amount, d.net, d.mode].join('.');
const attendue = crypto.createHmac('sha256', cleSecrete).update(chaine).digest('hex');
const ok = attendue === d.signature && Math.abs(Date.now() / 1000 - d.timestamp) <= 600;
```

```python
import hmac, hashlib, time
chaine = '.'.join(str(d.get(k, '')) for k in ['timestamp', 'reference', 'token', 'status', 'amount', 'net', 'mode'])
attendue = hmac.new(cle_secrete.encode(), chaine.encode(), hashlib.sha256).hexdigest()
ok = hmac.compare_digest(attendue, d['signature']) and abs(time.time() - int(d['timestamp'])) <= 600
```

## Répondre

- Répondez `200` vite (avant 10 secondes) ; faites le travail long ensuite.

- Sans réponse `2xx`, Tchin réessaie plusieurs fois.

- Un même événement peut arriver deux fois : rendez votre traitement **idempotent** (ne livrez pas deux fois la même `reference`).

- Votre adresse doit être en **HTTPS** et publique.

## Tester

En mode test, chaque paiement simulé envoie un vrai webhook, avec `"mode": "test"`. La boutique de démonstration de Tchin montre les webhooks reçus et leur signature vérifiée.

---
Source : https://doc.wintg.group/tchin/webhooks/
