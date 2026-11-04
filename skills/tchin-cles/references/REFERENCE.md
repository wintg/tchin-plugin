# Clés, modes et erreurs

Tout ce qui est commun à chaque appel : l'adresse, les formats, les en-têtes, le mode test, les erreurs, les limites, et la bonne façon de réessayer.

## L'adresse et les formats

```texte
https://tchin.tech/api/v1
```

| Quoi | Format |
|---|---|
| `Requêtes` | **JSON**, avec `Content-Type: application/json` (un formulaire classique est aussi accepté). |
| `Réponses` | **JSON**, en UTF-8. Demandez-les avec `Accept: application/json` : les erreurs de validation arrivent alors en JSON, jamais en page HTML. |
| `Montants en F CFA` | **entiers**, en francs CFA (XOF), sans centimes : `25000`. |
| `Quantités de crypto` | nombres décimaux, jusqu’à 6 décimales : `12.5`, `0.0042`. |
| `Dates` | ISO 8601 en UTC : `2026-11-03T14:05:12+00:00`. |
| `Pays` | codes ISO à deux lettres : `TG`, `BJ`, `BF`. |
| `Identifiants` | un `token` de paiement (14 lettres et chiffres), `sub_…` (abonnement), `kyc_…` (vérification), `ecr_…` (écran), `pvc_…` (paiement en caisse), un UUID (retrait crypto). |

Appelez toujours en **HTTPS** : une adresse en `http://` est redirigée, et une requête `POST` redirigée perd son contenu.

## Les en-têtes

| En-tête | Valeur |
|---|---|
| `TCHIN-PUBLIC-KEY` | la clé publique de votre application (`tchin_pk_…`) |
| `TCHIN-PRIVATE-KEY` | sa clé secrète (`tchin_sk_…`) |
| `Content-Type` | `application/json` (pour un `POST`) |
| `Accept` | `application/json` |

```curl
curl https://tchin.tech/api/v1/balance \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" \
  -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Accept: application/json"
```

```php
$ch = curl_init('https://tchin.tech/api/v1/balance');
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT => 30,
    CURLOPT_HTTPHEADER => [
        'TCHIN-PUBLIC-KEY: '.getenv('TCHIN_PUBLIC_KEY'),
        'TCHIN-PRIVATE-KEY: '.getenv('TCHIN_PRIVATE_KEY'),
        'Accept: application/json',
    ],
]);
$solde = json_decode(curl_exec($ch), true);
```

```node
const r = await fetch('https://tchin.tech/api/v1/balance', {
  headers: {
    'TCHIN-PUBLIC-KEY': process.env.TCHIN_PUBLIC_KEY,
    'TCHIN-PRIVATE-KEY': process.env.TCHIN_PRIVATE_KEY,
    'Accept': 'application/json',
  },
});
const solde = await r.json();
```

```python
import os, requests

r = requests.get('https://tchin.tech/api/v1/balance', timeout=30, headers={
    'TCHIN-PUBLIC-KEY': os.environ['TCHIN_PUBLIC_KEY'],
    'TCHIN-PRIVATE-KEY': os.environ['TCHIN_PRIVATE_KEY'],
    'Accept': 'application/json',
})
solde = r.json()
```

## Vos clés

- Elles se lisent sur la page API de votre compte (**Voir les clés**), après votre Tchin PIN ou votre biométrie, à chaque fois.

- Elles valent pour une application : chaque application a sa paire. Les paiements créés avec une paire n'appartiennent qu'à cette application (un token d'une autre application répond `404`).

- Les mêmes clés servent au mode test et au mode réel : c'est le champ `env` de chaque création qui choisit.

- Elles ne vont jamais dans une page web, une application mobile, un dépôt Git ou un message. Si une clé a fuité, créez une nouvelle application, remplacez les clés sur votre serveur, puis supprimez l'ancienne : ses clés ne marcheront plus.

## Mode test et mode réel

Chaque création accepte `"env"` : `"test"` ou `"live"`. **Précisez-le toujours** : selon le point d'entrée, la valeur par défaut n'est pas la même.

| Point d’entrée | Sans env | Mode test possible |
|---|---|---|
| `POST /payments` | **test** | oui |
| `POST /subscriptions` | live | oui |
| `POST /payouts/links` | live | oui |
| `POST /kyc/verifications` | live | oui (gratuit) |
| `POST /terminal/payments` | live | oui (l’écran propose « Simuler ») |
| `POST /disburse/initiate` | live | non : de l’argent réel, toujours |
| `POST /payouts/crypto` | live | non : de la crypto réelle, toujours |

|  | Mode test | Mode réel |
|---|---|---|
| `argent` | rien n’est débité ni versé | de vrais paiements, de vrais versements |
| `Mobile Money` | le compte test de Tchin (numéro et mot de passe) | le vrai numéro du client, son code secret |
| `crypto` | un bouton « Simuler le paiement » | un vrai virement sur le réseau |
| `compte Tchin` | le Tchin PIN est vérifié, aucun solde ne bouge | le solde du client est débité |
| `vérification d’identité` | gratuite | 75 F sur votre solde |
| `webhooks` | envoyés et signés, avec `"mode": "test"` | envoyés et signés, avec `"mode": "live"` |

La feuille affiche Mode test quand l'opération est en mode test. Les réponses et les webhooks disent toujours dans quel mode vous êtes (`env` ou `mode`) : vérifiez-le avant de livrer en production.

## Les réponses

Une réponse réussie porte `"success": true`, avec le code HTTP `200` (lu, fait) ou `201` (créé). Une erreur porte `"success": false` et un `message` en français, clair, que vous pouvez montrer tel quel à votre client ou à votre équipe :

```json
{ "success": false, "message": "Solde insuffisant sur le pays TG.", "balance": 4200, "required": 5170, "fee": 170 }
```

Une erreur de **champ** (manquant, mauvais type, hors bornes) répond `422` avec le détail de chaque champ dans `errors` :

```json
{
  "message": "Le champ amount doit être au moins 100.",
  "errors": {
    "amount": ["Le champ amount doit être au moins 100."]
  }
}
```

| Code HTTP | Sens | Que faire |
|---|---|---|
| `200, 201` | réussi (201 : créé) | lire la réponse |
| `400` | requête refusée : réseau ou pays non pris en charge, solde insuffisant pour un versement, refus du réseau | lire `message` ; ne pas réessayer tel quel |
| `401` | clés absentes ou fausses | vérifier les deux en-têtes et les variables d’environnement |
| `402` | solde insuffisant pour une vérification d’identité | recharger votre solde Tchin |
| `403` | compte suspendu | écrire à Letou ou au support depuis votre compte |
| `404` | introuvable (token, référence, identifiant), ou appartenant à une autre application | vérifier l’identifiant et les clés utilisées |
| `409` | conflit : l’opération ne peut plus être faite (un paiement en caisse déjà payé, par exemple) | relire l’état |
| `410` | méthode retirée de l’API (`"error": "method_removed"`) | passer par la feuille : voir les méthodes retirées |
| `422` | champ manquant ou invalide (`errors`), ou demande impossible expliquée dans `message` | corriger la requête |
| `423` | versement suspendu par un contrôle de sécurité du compte | contacter le support : notre équipe vérifie |
| `429` | trop d’appels : la limite est atteinte | attendre le nombre de secondes de l’en-tête `Retry-After` |
| `5xx` | erreur chez Tchin | réessayer plus tard, avec un délai croissant |

## Les limites

| Limite | Valeur |
|---|---|
| `Appels` | **120 par minute** par clé publique, et 300 par minute par adresse IP |
| `Un paiement` | de 100 F à 100 000 000 F |
| `Un abonnement (chaque échéance)` | de 200 F à 100 000 000 F |
| `Un versement Mobile Money` | 200 F minimum, et dans les plafonds du réseau du destinataire |
| `Recevoir (au choix du client)` | Mobile Money de 500 F à 1 500 000 F ; en crypto, le minimum de la crypto choisie |
| `Un paiement en caisse` | de 200 F à 500 000 F |
| `Webhooks` | vers une adresse **HTTPS publique** seulement (pas d’adresse locale ni privée) |

## Bonnes pratiques

- **Délai** : donnez 30 secondes à chaque appel. Un versement peut prendre quelques secondes, le temps que le réseau réponde.

- **Réessayer** : seulement sur `429`, `5xx` ou une coupure réseau, avec un délai qui grandit (1 s, 5 s, 30 s…). Jamais sur `400`, `401`, `404` ou `422` : la même requête donnera la même réponse.

- **Ne rien créer deux fois** : un paiement par commande, et gardez son token. Un versement crypto porte votre `reference` : la même référence n'envoie qu'une fois, même si vous réessayez. Un versement Mobile Money se fait en deux appels : ne rappelez `submit` que si `status` vous le dit.

- **Journaux** : gardez, pour chaque appel, l'heure, le point d'entrée, le code HTTP et le `message`. Jamais les clés.

- **Faire foi** : l'état d'une opération se lit de votre serveur (webhook signé ou lecture de l'état), jamais depuis la page du client.

---
Source : https://doc.wintg.group/tchin/authentification/
