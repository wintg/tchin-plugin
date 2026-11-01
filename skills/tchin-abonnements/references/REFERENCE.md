# Abonnements

Un montant fixe chaque mois, trimestre, semestre ou année. Le client accepte et choisit son moyen la première fois ; Tchin s'occupe ensuite de chaque échéance.

## Créer un abonnement

| Champ | Type | Description |
|---|---|---|
| `plan_name` | texte | **obligatoire**. Le nom de l’offre (« Wintg Meet Business »). |
| `amount` | entier | **obligatoire**. Le montant de chaque échéance, en F CFA (200 minimum). |
| `interval` | texte | **obligatoire**. `monthly`, `quarterly`, `biannual` ou `yearly`. |
| `description` | texte | une précision sur l’offre. |
| `customer_name` | texte | le nom du client. |
| `customer_email` | e-mail | son e-mail : les rappels y partent. |
| `external_ref` | texte | votre référence (le numéro de client chez vous). |
| `callback_url` | URL | votre webhook pour cet abonnement. |
| `return_url` | URL | où revenir après un paiement. |
| `env` | texte | `test` ou `live`. |

```curl
curl -X POST https://tchin.tech/api/v1/subscriptions \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Content-Type: application/json" \
  -d '{"plan_name": "Wintg Meet Business", "amount": 10000, "interval": "monthly",
       "customer_email": "koffi@example.com", "callback_url": "https://wintg.group/webhook-tchin"}'
```

```php
$a = $tchin->creerAbonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
    ['customer_email' => 'koffi@example.com']);
// $a['token'] : ouvrez la feuille ; $a['subscription']['reference'] : gardez-la
```

```node
const a = await tchin.creerAbonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
  { customer_email: 'koffi@example.com' });
```

```python
a = tchin.creer_abonnement(10000, 'Wintg Meet Business', 'monthly', 'https://wintg.group/webhook-tchin',
                           customer_email='koffi@example.com')
```

```java
String json = tchin.creerAbonnement(10000, "Wintg Meet Business", "monthly", "https://wintg.group/webhook-tchin",
        Map.of("customer_email", "koffi@example.com"));
```

```go
a, err := c.CreerAbonnement(10000, "Wintg Meet Business", "monthly", "https://wintg.group/webhook-tchin",
	map[string]any{"customer_email": "koffi@example.com"})
```

```json
{
  "success": true,
  "token": "1ca1f3b8epbxcj",
  "payment_url": "https://tchin.tech/pay/1ca1f3b8epbxcj",
  "subscription": {
    "reference": "sub_hajccouospgj0sbbnhnj",
    "plan_name": "Wintg Meet Business",
    "amount": 10000,
    "interval": "monthly",
    "status": "pending",
    "payment_method": null,
    "next_charge_at": null
  }
}
```

Ouvrez la feuille avec `token`, comme un paiement : `TchinPay.open({ token })`. Le premier paiement active l'abonnement et fixe le moyen : `payment_method` vaut `momo`, `crypto` ou `tchin`.

## Chaque échéance

1. **3 jours avant**, une demande de paiement arrive au client. S'il a un compte Tchin (il paie avec Tchin, ou même e-mail ou même numéro qu'un compte), elle arrive sur son compte, dans Demandes de paiement ; sinon, par e-mail, avec le lien de paiement. Il peut payer tout de suite, avec le moyen choisi la première fois.

1. **Le jour J, en Mobile Money**, Tchin envoie lui-même la demande sur le téléphone du client : il valide avec son code secret. Pas validée : jusqu'à 3 fois ce jour-là (3 heures d'écart), puis une fois par jour pendant 3 jours. Seuls les réseaux à code secret sont proposés pour un abonnement.

1. **Compte Tchin et crypto** : rien n'est pris tout seul. Le client paie la demande (son solde et son PIN, ou le montant exact en crypto).

1. **3 jours après l'échéance** sans paiement : le lien ne marche plus et l'abonnement s'arrête (`subscription.expired`).

Les frais suivent le réglage de votre application, à chaque échéance. Le client peut **annuler une échéance** (elle est sautée, l'abonnement continue) ou **résilier** depuis Tchin.

## Résilier depuis votre site

Quand votre client se désabonne chez vous, résiliez : plus aucune demande ne lui sera envoyée.

```curl
curl -X POST https://tchin.tech/api/v1/subscriptions/sub_hajccouospgj0sbbnhnj/cancel \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
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

## Lister et suivre

| Statut | Sens |
|---|---|
| `pending` | créé, la 1re échéance n’est pas encore payée |
| `active` | à jour |
| `past_due` | l’échéance est passée et pas encore payée (dans les 3 jours) |
| `cancelled` | résilié (par vous ou par le client) |
| `expired` | arrêté : échéance non payée 3 jours après |

## Les événements

Votre `callback_url` reçoit un webhook signé avec le champ `event` :

| event | Quand |
|---|---|
| `subscription.activated` | le 1er paiement est fait |
| `subscription.charged` | une échéance est payée |
| `subscription.payment_skipped` | le client a annulé une échéance |
| `subscription.activation_failed` | le 1er paiement a échoué |
| `subscription.cancelled` | résilié |
| `subscription.expired` | arrêté faute de paiement |

---
Source : https://doc.wintg.group/tchin/abonnements/
