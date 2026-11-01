# Clés, modes et erreurs

Tout ce qui est commun à chaque appel : l'adresse, les en-têtes, le mode test, les erreurs et les limites.

## L'adresse de l'API

```texte
https://tchin.tech/api/v1
```

Les corps de requête et les réponses sont en **JSON** (`Content-Type: application/json`). Les montants sont en **francs CFA (XOF)**, des entiers ; les quantités de crypto, des nombres décimaux.

## Les en-têtes

| En-tête | Valeur |
|---|---|
| `TCHIN-PUBLIC-KEY` | la clé publique de votre application (`tchin_pk_…`) |
| `TCHIN-PRIVATE-KEY` | sa clé secrète (`tchin_sk_…`) |
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |

```curl
curl https://tchin.tech/api/v1/balance \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." \
  -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Accept: application/json"
```

## Mode test et mode réel

Chaque création accepte `"env"` : `"test"` ou `"live"`. Sans `env`, un paiement (`POST /payments`) est en **test** ; un abonnement, un versement ou une vérification d'identité est en **live**. Précisez-le toujours. Les mêmes clés servent pour les deux.

|  | Mode test | Mode réel |
|---|---|---|
| `argent` | rien n’est débité ni versé | de vrais paiements, de vrais versements |
| `Mobile Money` | les comptes test de Tchin | le vrai numéro du client |
| `crypto` | un bouton « Simuler le paiement » | un vrai virement sur le réseau |
| `compte Tchin` | le PIN est vérifié, rien ne bouge | le solde du client est débité |
| `vérification d’identité` | gratuite | 75 F sur votre solde |
| `webhooks` | envoyés, avec `"mode": "test"` | envoyés, avec `"mode": "live"` |

La feuille affiche Mode test quand l'opération est en mode test.

## Les réponses

Une réponse réussie porte `"success": true`. Une erreur porte `"success": false` et un `message` en français, à montrer tel quel si vous voulez :

```json
{ "success": false, "message": "Votre solde ne couvre pas ce montant." }
```

| Code HTTP | Sens |
|---|---|
| `200, 201` | réussi (201 : créé) |
| `400` | requête refusée (moyen ou pays non pris en charge…) |
| `401` | clés absentes ou fausses |
| `402` | solde insuffisant (vérification d’identité) |
| `404` | introuvable (token, référence…) |
| `410` | méthode arrêtée (voir le calendrier) |
| `422` | champ manquant ou invalide (`errors` détaille chaque champ) |
| `429` | trop d’appels : attendez un peu |
| `5xx` | erreur de Tchin : réessayez plus tard |

## Les limites

- **120 appels par minute** par application (et 300 par adresse IP).

- Un paiement : de 100 F à 100 000 000 F.

- Un versement Mobile Money : de 500 F à 1 500 000 F.

- Les webhooks partent de nos serveurs vers une adresse **HTTPS publique** seulement.

---
Source : https://doc.wintg.group/tchin/authentification/
