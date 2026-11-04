# Abonnements

Un montant fixe chaque mois, trimestre, semestre ou année. Le client donne son accord et choisit son moyen la première fois, dans la feuille ; Tchin s'occupe ensuite de chaque échéance, des rappels et des relances.

## Le principe

1. Votre serveur **crée l'abonnement** : l'offre, le montant, la fréquence. Tchin rend la référence de l'abonnement (`sub_…`) et le `token` de la première échéance.

1. Votre page **ouvre la feuille** avec ce token, comme pour un paiement. Le client lit l'offre, donne son accord et paie la première échéance avec le moyen de son choix : Mobile Money, compte Tchin ou crypto.

1. L'abonnement passe **actif** (`subscription.activated`). Le moyen choisi devient celui des échéances suivantes.

1. À chaque échéance, Tchin **demande le paiement** au client, relance s'il le faut, et vous prévient (`subscription.charged`, ou `subscription.expired` si rien n'est payé).

## Créer un abonnement

| Champ | Type | Description |
|---|---|---|
| `plan_name` | texte | **obligatoire**. Le nom de l’offre, vu par le client (« Wintg Meet Business »), 120 caractères au plus. |
| `amount` | entier | **obligatoire**. Le montant de chaque échéance, en F CFA, de 200 à 100 000 000. |
| `interval` | texte | **obligatoire**. `monthly` (chaque mois), `quarterly` (chaque trimestre), `biannual` (chaque semestre) ou `yearly` (chaque année). |
| `description` | texte | une précision sur l’offre (500 caractères au plus). |
| `customer_name` | texte | le nom du client. |
| `customer_email` | e-mail | son e-mail : les demandes de paiement et les rappels y partent s’il n’a pas de compte Tchin. |
| `external_ref` | texte | votre référence (le numéro du client ou de l’abonnement chez vous), rendue partout. |
| `fees_on_customer` | booléen | qui paie les frais de chaque échéance. Sans lui : le réglage de l’application. |
| `remind_email` | booléen | envoyer les rappels par e-mail (oui par défaut). |
| `callback_url` | URL | votre webhook pour cet abonnement. Sans elle : celle de l’application. |
| `return_url` | URL | où ramener le client après un paiement sur la page de paiement. |
| `env` | texte | `test` ou `live`. Sans lui : **live**. |

```curl
curl -X POST https://tchin.tech/api/v1/subscriptions \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"plan_name": "Wintg Meet Business", "amount": 10000, "interval": "monthly",
       "customer_email": "koffi@example.com", "external_ref": "client-42",
       "callback_url": "https://wintg.group/webhook-tchin", "env": "live"}'
```

```php
$a = $tchin->creerAbonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
    ['customer_email' => 'koffi@example.com', 'external_ref' => 'client-42', 'env' => 'live']);
// $a['token'] : ouvrez la feuille ; $a['subscription']['reference'] : gardez-la avec le client
```

```node
const a = await tchin.creerAbonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
  { customer_email: 'koffi@example.com', external_ref: 'client-42', env: 'live' });
```

```python
a = tchin.creer_abonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
                           customer_email='koffi@example.com', external_ref='client-42', env='live')
```

```java
String json = tchin.creerAbonnement(10000, "Wintg Meet Business", "monthly", "https://wintg.group/webhook-tchin",
        Map.of("customer_email", "koffi@example.com", "external_ref", "client-42", "env", "live"));
```

```go
a, err := c.CreerAbonnement(10000, "Wintg Meet Business", "monthly", "https://wintg.group/webhook-tchin",
	map[string]any{"customer_email": "koffi@example.com", "external_ref": "client-42", "env": "live"})
```

```json
{
  "success": true,
  "message": "Ouvrez la feuille de paiement avec ce token : TchinPay.open({ token }).",
  "token": "1ca1f3b8epbxcj",
  "payment_url": "https://tchin.tech/pay/1ca1f3b8epbxcj",
  "subscription": {
    "reference": "sub_hajccouospgj0sbbnhnj",
    "external_ref": "client-42",
    "plan": "Wintg Meet Business",
    "amount": 10000,
    "interval": "monthly",
    "interval_label": "Tous les mois",
    "status": "pending",
    "currency": "XOF",
    "customer": { "name": null, "email": "koffi@example.com", "phone": null },
    "payment_method": null,
    "crypto": null,
    "charges_count": 0,
    "activated_at": null,
    "next_charge_at": null,
    "cancelled_at": null,
    "mode": "live"
  }
}
```

Ouvrez la feuille avec `token`, exactement comme un paiement : `TchinPay.open({ token })`. Vous pouvez aussi envoyer la `payment_url` au client par e-mail ou message.

## L'objet abonnement

| Champ | Description |
|---|---|
| `reference` | l’identifiant de l’abonnement chez Tchin (`sub_…`) : pour le lire, le lister, le résilier. |
| `external_ref` | votre référence, telle que vous l’avez donnée. |
| `plan, amount, interval, interval_label` | l’offre, le montant de chaque échéance, la fréquence (et sa forme lisible). |
| `status` | l’état (voir plus bas). |
| `customer` | nom, e-mail et téléphone du client, tels que connus. |
| `payment_method` | le moyen choisi à la première échéance : `momo`, `tchin` ou `crypto` (`null` tant qu’il n’a pas payé). |
| `tchin_source, crypto` | le solde Tchin ou la crypto utilisés, selon le moyen. |
| `consent_at` | quand le client a donné son accord. |
| `charges_count` | le nombre d’échéances payées. |
| `activated_at, next_charge_at` | la date d’activation, la prochaine échéance. |
| `grace_until` | pendant un retard : la date limite pour payer avant l’arrêt. |
| `cancelled_at` | la date de résiliation. |
| `mode` | `test` ou `live`. |

## Chaque échéance

1. **3 jours avant**, une demande de paiement arrive au client. S'il a un compte Tchin (il a payé avec Tchin, ou il a le même e-mail ou le même numéro qu'un compte), elle arrive sur son compte, dans Prélèvements ; sinon, par e-mail, avec le lien de paiement. Il peut payer tout de suite, avec le moyen choisi la première fois.

1. **Le jour J, en Mobile Money**, Tchin envoie lui-même la demande sur le téléphone du client : il valide avec son code secret. Pas validée : jusqu'à 3 essais ce jour-là (3 heures d'écart), puis un par jour pendant 3 jours. Seuls les réseaux à validation sur le téléphone sont proposés pour un abonnement.

1. **Compte Tchin et crypto** : rien n'est pris sans le client. Il paie la demande (son solde et son Tchin PIN, ou la quantité exacte en crypto).

1. **3 jours après l'échéance** sans paiement : le lien ne marche plus et l'abonnement s'arrête (`subscription.expired`).

Les frais suivent le réglage de l'abonnement (ou de l'application), à chaque échéance. Le client peut **sauter une échéance** (elle est annulée, l'abonnement continue : `subscription.payment_skipped`) ou **résilier** lui-même depuis Tchin.

## Les états

| Statut | Sens | Que faire |
|---|---|---|
| `pending` | créé, la 1re échéance n’est pas encore payée | attendre que le client paie dans la feuille |
| `active` | à jour | donner l’accès au service |
| `past_due` | l’échéance est passée et pas encore payée (dans les 3 jours de grâce) | garder l’accès, ou le limiter, selon votre choix |
| `cancelled` | résilié (par vous ou par le client) | couper l’accès à la fin de la période payée |
| `expired` | arrêté : échéance non payée 3 jours après | couper l’accès ; un nouvel abonnement est possible |

## Résilier depuis votre site

Quand votre client se désabonne chez vous, résiliez : plus aucune demande ne lui sera envoyée. La période déjà payée lui reste acquise ; rien n'est remboursé automatiquement.

```curl
curl -X POST https://tchin.tech/api/v1/subscriptions/sub_hajccouospgj0sbbnhnj/cancel \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -d '{"reason": "Résilié par le client"}'
```

```php
$tchin->resilierAbonnement('sub_hajccouospgj0sbbnhnj');
```

```node
await tchin.resilierAbonnement('sub_hajccouospgj0sbbnhnj');
```

```python
tchin.resilier_abonnement('sub_hajccouospgj0sbbnhnj')
```

```java
tchin.resilierAbonnement("sub_hajccouospgj0sbbnhnj");
```

```go
c.ResilierAbonnement("sub_hajccouospgj0sbbnhnj")
```

```json
{
  "success": true,
  "message": "Abonnement résilié. La période déjà payée reste acquise au client : aucun remboursement.",
  "subscription": { "reference": "sub_hajccouospgj0sbbnhnj", "status": "cancelled", "cancelled_at": "2026-11-03T15:10:00+00:00" }
}
```

## Lister et suivre

| Paramètre | Description |
|---|---|
| `status` | filtrer : `pending`, `active`, `past_due`, `cancelled` ou `expired`. |
| `limit` | combien en rendre : 50 par défaut, 200 au plus. |

```json
{
  "success": true,
  "data": [
    {
      "period": 2,
      "attempt": 1,
      "status": "completed",
      "amount": 10000,
      "setup_fee": 0,
      "token": "tch_7f2kq…",
      "error": null,
      "scheduled_for": "2026-12-03T08:00:00+00:00",
      "charged_at": "2026-12-03T08:02:41+00:00"
    }
  ]
}
```

| Champ | Description |
|---|---|
| `period` | le numéro de l’échéance (1 : la première). |
| `attempt` | le numéro de l’essai pour cette échéance. |
| `status` | l’issue de cet essai : `pending` (en cours), `completed` (payé), `failed` (échoué), `skipped` (sautée par le client). |
| `amount` | le montant demandé. |
| `token` | la transaction de paiement correspondante. |
| `error` | la raison d’un échec, en clair. |
| `scheduled_for, charged_at` | la date prévue, la date du paiement. |

## Les webhooks

Votre `callback_url` reçoit un webhook signé (voir Webhooks et signature) avec le champ `event` et l'objet `subscription` entier :

| event | Quand |
|---|---|
| `subscription.activated` | la 1re échéance est payée : l’abonnement est actif |
| `subscription.activation_failed` | le 1er paiement a échoué |
| `subscription.charged` | une échéance suivante est payée |
| `subscription.payment_failed` | un essai de paiement d’une échéance a échoué (Tchin réessaie) |
| `subscription.payment_skipped` | le client a sauté une échéance |
| `subscription.cancelled` | résilié, par vous ou par le client |
| `subscription.expired` | arrêté faute de paiement |

```json
{
  "event": "subscription.charged",
  "response_code": "00",
  "response_text": "subscription.charged",
  "reference": "sub_hajccouospgj0sbbnhnj",
  "token": "tch_7f2kq…",
  "status": "active",
  "fail_reason": "",
  "amount": 10000,
  "fee": 0,
  "net": 10000,
  "currency": "XOF",
  "mode": "live",
  "subscription": { "reference": "sub_hajccouospgj0sbbnhnj", "status": "active", "charges_count": 2, "next_charge_at": "2027-01-03T08:00:00+00:00" },
  "timestamp": 1793712161,
  "signature": "9c41…"
}
```

Ici, `reference` est la référence de l'abonnement (`sub_…`) et `status` l'état de l'abonnement après l'événement. Donnez ou coupez l'accès selon `subscription.status`.

## Tester

Créez l'abonnement avec `"env": "test"`, ouvrez la feuille et payez la première échéance avec un compte test : le webhook `subscription.activated` arrive avec `"mode": "test"`. Les échéances suivantes d'un abonnement de test ne prennent pas d'argent.

---
Source : https://doc.wintg.group/tchin/abonnements/
