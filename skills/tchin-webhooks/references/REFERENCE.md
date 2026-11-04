# Webhooks et signature

À chaque changement d'état, Tchin appelle votre callback_url. Vérifiez la signature avant de croire le contenu, répondez vite, et traitez chaque événement une seule fois.

## Quand un webhook part

| Opération | kind | Quand | reference vaut |
|---|---|---|---|
| `Paiement` | (absent) | à chaque changement d’état : `completed`, `failed`, `cancelled` | le token du paiement |
| `Abonnement` | (absent), avec `event` | activation, échéance payée ou échouée, échéance sautée, résiliation, arrêt | la référence `sub_…` |
| `Versement direct (Mobile Money ou crypto)` | `payout` | quand il est confirmé ou échoue | le `disburse_token` (ou l’identifiant du retrait crypto) |
| `Recevoir` | `payout_link` | en cours, payé, échoué, annulé (une fois par état) | le token du versement |
| `Vérification d’identité` | `kyc`, avec `event` | à chaque étape, jusqu’au verdict | (champ `id` : `kyc_…`) |
| `Paiement en caisse` | `terminal`, avec `event` | payé, refusé, expiré, annulé | (champ `id` : `pvc_…`) |

L'adresse est celle donnée à la création (`callback_url`), sinon l'adresse de webhook de votre application. Sans les deux, rien ne part : lisez alors l'état vous-même.

## Le format

Un `POST` en `application/x-www-form-urlencoded` (un formulaire), avec un champ `data` : ses sous-champs arrivent en `data[reference]`, `data[status]`, `data[customer][name]`… La plupart des frameworks les rangent tout seuls dans un tableau `data`. Deux en-têtes accompagnent chaque envoi :

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
  "fail_reason": "",
  "amount": 26042,
  "fee": 1042,
  "net": 25000,
  "currency": "XOF",
  "country": "TG",
  "method": "t-money-togo",
  "method_name": "Yas",
  "mode": "live",
  "customer": { "name": "Koffi", "email": "koffi@example.com", "phone": "+22890112233" },
  "custom_data": [],
  "hash": "…",
  "timestamp": 1792432544,
  "signature": "3f1a…"
}
```

| Champ | Description |
|---|---|
| `response_code` | `00` si l’opération a réussi, `1001` sinon. |
| `reference` | votre clé pour retrouver l’opération : le token du paiement (voir le tableau plus haut). |
| `token` | la transaction chez Tchin (ou chez le réseau). |
| `status` | `completed`, `failed`, `cancelled`, `pending` (et les états propres à chaque produit). |
| `fail_reason` | la raison d’un échec, vide sinon. |
| `amount, fee, net` | paiement : ce que le client a payé, les frais, ce que vous recevez. Versement : ce qui quitte votre solde, les frais, ce que le destinataire reçoit. |
| `country, method, method_name` | le pays et le moyen utilisés (code et nom affiché). |
| `mode` | `live` ou `test` : ne livrez en production que sur `live`. |
| `customer` | nom, e-mail et téléphone du client, tels qu’il les a donnés. |
| `crypto` | payé en crypto : `currency`, `amount` (envoyé par le client), `credited` (arrivé sur votre solde), `txid`. |
| `timestamp, signature` | l’heure d’envoi et la signature (voir ci-dessous). |
| `hash` | champ ancien, gardé pour les intégrations d’avant : il ne prouve rien. **Utilisez `signature`.** |

Les webhooks des autres produits ajoutent leurs champs : `event` et `subscription` (abonnements), `recipient` et `payout_currency` (Recevoir), le résultat complet de la vérification (identité) ou du paiement en caisse (caisse).

## Vérifier la signature

La signature est un **HMAC-SHA256**, avec votre **clé secrète**, de cette chaîne (les valeurs reçues, dans cet ordre, séparées par des points ; un champ absent compte comme vide) :

```texte
timestamp.reference.token.status.amount.net.mode
```

Elle couvre tout ce qui engage de l'argent : qui, combien, quel état, quel mode. Refusez aussi un webhook **trop ancien** (plus de 10 minutes entre `timestamp` et votre horloge) : un message intercepté et rejoué plus tard ne passera pas. Les kits le font pour vous :

```php
$d = $tchin->verifierWebhook($_POST['data'] ?? []);   // null si la signature est fausse ou trop ancienne
```

```node
const d = tchin.verifierWebhook(req.body.data);       // null si la signature est fausse ou trop ancienne
```

```python
data = {k[5:-1]: v for k, v in request.form.items() if k.startswith('data[')}
d = tchin.verifier_webhook(data)                      # None si la signature est fausse ou trop ancienne
```

```java
boolean ok = tchin.verifierWebhook(data);             // data : les champs data[xxx] → xxx
```

```go
ok := c.VerifierWebhook(r.PostForm)                   // après r.ParseForm()
```

Sans kit, dans n'importe quel langage :

```php
$d = $_POST['data'] ?? [];
$chaine = implode('.', [$d['timestamp'] ?? '', $d['reference'] ?? '', $d['token'] ?? '', $d['status'] ?? '',
                        $d['amount'] ?? '', $d['net'] ?? '', $d['mode'] ?? '']);
$ok = hash_equals(hash_hmac('sha256', $chaine, getenv('TCHIN_PRIVATE_KEY')), (string) ($d['signature'] ?? ''))
   && abs(time() - (int) ($d['timestamp'] ?? 0)) <= 600;
if (! $ok) { http_response_code(401); exit; }
```

```node
const crypto = require('crypto');
const d = req.body.data || {};
const chaine = [d.timestamp, d.reference, d.token, d.status, d.amount, d.net, d.mode].map((v) => v ?? '').join('.');
const attendue = crypto.createHmac('sha256', process.env.TCHIN_PRIVATE_KEY).update(chaine).digest('hex');
const ok = d.signature && attendue.length === d.signature.length
  && crypto.timingSafeEqual(Buffer.from(attendue), Buffer.from(d.signature))
  && Math.abs(Date.now() / 1000 - Number(d.timestamp)) <= 600;
if (!ok) return res.sendStatus(401);
```

```python
import hmac, hashlib, os, time
chaine = '.'.join(str(d.get(k, '')) for k in ['timestamp', 'reference', 'token', 'status', 'amount', 'net', 'mode'])
attendue = hmac.new(os.environ['TCHIN_PRIVATE_KEY'].encode(), chaine.encode(), hashlib.sha256).hexdigest()
ok = hmac.compare_digest(attendue, d.get('signature', '')) and abs(time.time() - int(d.get('timestamp', 0))) <= 600
```

```java
String chaine = String.join(".", d.getOrDefault("timestamp", ""), d.getOrDefault("reference", ""), d.getOrDefault("token", ""),
        d.getOrDefault("status", ""), d.getOrDefault("amount", ""), d.getOrDefault("net", ""), d.getOrDefault("mode", ""));
Mac mac = Mac.getInstance("HmacSHA256");
mac.init(new SecretKeySpec(System.getenv("TCHIN_PRIVATE_KEY").getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
String attendue = HexFormat.of().formatHex(mac.doFinal(chaine.getBytes(StandardCharsets.UTF_8)));
boolean ok = MessageDigest.isEqual(attendue.getBytes(), d.getOrDefault("signature", "").getBytes())
        && Math.abs(Instant.now().getEpochSecond() - Long.parseLong(d.getOrDefault("timestamp", "0"))) <= 600;
```

```go
f := func(k string) string { return r.PostForm.Get("data[" + k + "]") }
chaine := strings.Join([]string{f("timestamp"), f("reference"), f("token"), f("status"), f("amount"), f("net"), f("mode")}, ".")
m := hmac.New(sha256.New, []byte(os.Getenv("TCHIN_PRIVATE_KEY")))
m.Write([]byte(chaine))
ts, _ := strconv.ParseInt(f("timestamp"), 10, 64)
ok := hmac.Equal([]byte(hex.EncodeToString(m.Sum(nil))), []byte(f("signature"))) && math.Abs(float64(time.Now().Unix()-ts)) <= 600
```

## Répondre

- Répondez `200` (ou tout code `2xx`) **vite**, en moins de 10 secondes. Faites le travail long ensuite (e-mail au client, facture…), par exemple dans une file.

- Tchin attend 15 secondes au plus. Sans réponse `2xx` à temps, il réessaie : **5 essais en tout**, tout de suite, puis 10 secondes, 1 minute, 5 minutes et 15 minutes plus tard.

- Une signature fausse : répondez `401` et ne faites rien. Un vrai webhook de Tchin n'aura jamais une signature fausse : c'est une erreur de clé (une autre application ?) ou une tentative de fraude.

- Votre adresse doit être en **HTTPS** et publique : les adresses locales ou privées (`localhost`, `192.168…`, `10…`) sont refusées.

## Doublons et ordre

- Un même événement peut arriver **deux fois** (un réessai après une réponse trop lente, par exemple). Rendez votre traitement **idempotent** : avant de livrer, regardez si cette `reference` est déjà traitée.

- Les webhooks peuvent arriver **dans le désordre** : fiez-vous à `status`, pas à l'ordre d'arrivée. Un paiement `completed` ne redevient jamais `pending`.

- En cas de doute, ou si votre serveur était en panne plus de 20 minutes, **relisez l'état** (`GET /api/v1/payments/{token}/status` pour un paiement) : c'est la source de vérité.

```php
// Livrer une seule fois, même si le webhook revient (une contrainte UNIQUE sur reference suffit)
$deja = $db->prepare('SELECT 1 FROM paiements_traites WHERE reference = ?');
$deja->execute([$d['reference']]);
if ($d['status'] === 'completed' && $d['mode'] === 'live' && ! $deja->fetch()) {
    $db->prepare('INSERT INTO paiements_traites (reference, traite_le) VALUES (?, NOW())')->execute([$d['reference']]);
    livrer($d['reference']);
}
http_response_code(200);
```

## Tester

- En mode test, chaque paiement simulé envoie un **vrai webhook**, signé, avec `"mode": "test"` : votre code de vérification se teste tel quel.

- En développement sur votre ordinateur, exposez votre serveur avec un tunnel HTTPS (ngrok, Cloudflare Tunnel…) et donnez cette adresse en `callback_url`.

- La boutique de démonstration de Tchin montre les webhooks qu'elle reçoit et leur signature vérifiée.

---
Source : https://doc.wintg.group/tchin/webhooks/
