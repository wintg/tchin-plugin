# Référence de l'API

Tous les points d'entrée, sur https://tchin.tech/api/v1, avec les en-têtes TCHIN-PUBLIC-KEY et TCHIN-PRIVATE-KEY. Chaque ligne renvoie à sa page, où tout est détaillé.

## Paiements

| Champ de POST /payments | Obligatoire | Défaut |
|---|---|---|
| `amount` | oui | (entier, 100 à 100 000 000) |
| `description` | non | « Paiement » |
| `callback_url` | non | le webhook de l’application |
| `return_url, cancel_url` | non | la page d’où vient le client |
| `fees_on_customer` | non | le réglage de l’application |
| `env` | non | **test** |

Détails : Paiements, GET /methods.

## Abonnements

Champs obligatoires : `plan_name`, `amount` (200 minimum), `interval` (`monthly`, `quarterly`, `biannual`, `yearly`). `env` vaut **live** par défaut. Détails : Abonnements.

## Versements

Détails : Recevoir, Versements directs. Les versements directs n'ont pas de mode test.

## Vérification d'identité

Détails : Vérification d'identité.

## Paiement en caisse

Détails : Paiement en caisse. Réservé aux comptes Business vérifiés.

## Compte

## Kits web

| Script | Fonctions |
|---|---|
| `https://tchin.tech/js/tchin-pay.js` | `TchinPay.open({ token, onSuccess, onPending, onFailed, onClose })`, `TchinPay.receive({ … })`, `TchinPay.close()` |
| `https://tchin.tech/js/tchin-kyc.js` | `TchinKYC.open({ id, onComplete, onClose })` |

## Webhooks

`POST` de formulaire vers votre `callback_url`, champ `data`, en-têtes `Tchin-Signature: v1=…` et `Tchin-Timestamp`. Signature : HMAC-SHA256, avec votre clé secrète, de `timestamp.reference.token.status.amount.net.mode`. 5 essais en tout. Détails : Webhooks et signature.

## Serveur MCP (IA)

Sur `https://tchin.tech/mcp`. Sans connexion : `afficher_lien_tchin`, `preparer_demande_tchin`, `etat_paiement`. Avec le compte de l'utilisateur (OAuth, demandé au premier usage) : ses transactions, son bilan, son relevé PDF, ses liens et ses applications, et les demandes d'argent confirmées sur tchin.tech avec le Tchin PIN. Voir Tchin dans votre IA.

## Méthodes retirées

Une seule façon d'encaisser par l'API : la **feuille Tchin**, ouverte sur votre site. Les anciennes façons, où vous construisiez vous-même le formulaire de paiement, sont retirées et répondent `410` :

```json
{
  "success": false,
  "error": "method_removed",
  "message": "Cette méthode n’existe plus. Ouvrez la feuille de paiement Tchin sur votre site : TchinPay.open({ token }). Guide : https://doc.wintg.group/tchin/paiements"
}
```

### Passer à la feuille, en trois étapes

1. Gardez votre appel `POST /payments` : il rend toujours le même `token`.

1. Remplacez votre formulaire de paiement (pays, réseau, numéro, code) par une ligne : `TchinPay.open({ token })`. La feuille demande tout au client, et propose aussi la crypto et le compte Tchin.

1. Pour les abonnements, créez-les sans `customer_phone`, et ouvrez la feuille avec le token rendu : le client donne son accord et choisit son moyen.

---
Source : https://doc.wintg.group/tchin/reference/
