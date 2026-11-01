# Calendrier et migration

Une seule façon d'encaisser par l'API : la feuille Tchin, ouverte sur votre site. Les anciennes façons s'arrêtent le 30 novembre 2026.

## Les dates

| Date | Ce qui se passe |
|---|---|
| `19 octobre 2026` | la feuille (TchinPay), les abonnements par feuille, Recevoir et les kits sont ouverts |
| `jusqu’au 30 novembre` | les anciens appels marchent encore, avec les en-têtes `Deprecation: true` et `Sunset` |
| `1er décembre 2026` | les anciens appels répondent `410` |

## Ce qui s'arrête

- **Payer sur une page à vous** : `POST /payments/{token}/charge` (Mobile Money) et `POST /payments/{token}/crypto`.

- **L'abonnement prélevé sur un numéro donné par vous** : `POST /subscriptions` avec `customer_phone`.

La page de paiement Tchin (`payment_url`) reste ouverte : les e-mails de rappel l'utilisent.

## Migrer en 3 étapes

1. Gardez votre appel `POST /payments` : il rend toujours `token`.

1. Remplacez votre formulaire de paiement (numéro, réseau, code) par `TchinPay.open({ token })` : la feuille demande tout au client.

1. Pour les abonnements, créez-les sans `customer_phone` et ouvrez la feuille avec le token rendu.

---
Source : https://doc.wintg.group/tchin/calendrier/
