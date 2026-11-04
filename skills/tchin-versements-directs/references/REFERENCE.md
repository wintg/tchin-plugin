# Versements directs

Vous connaissez déjà le numéro ou l'adresse (vos vendeurs, vos livreurs, vos partenaires) : versez directement depuis votre solde Tchin, sans feuille et sans rien demander au destinataire.

## Vers un numéro Mobile Money

Deux appels, pour que rien ne parte par erreur : **préparer** (Tchin vérifie le pays, le réseau, calcule les frais et réserve de quoi payer), puis **envoyer**.

### 1. Préparer

| Champ | Type | Description |
|---|---|---|
| `account_alias` | texte | **obligatoire**. Le numéro du destinataire, sans indicatif (« 90112233 »). |
| `amount` | entier | **obligatoire**. Le montant en F CFA, 200 minimum. Frais à votre charge : ce que le destinataire reçoit. Frais au destinataire : ce qui quitte votre solde. |
| `withdraw_mode` | texte | **obligatoire**. Le réseau du destinataire : `t-money-togo`, `moov-togo`, `mtn-benin`, `moov-benin`, `celtiis-benin`, `orange-money-burkina` ou `moov-burkina`. Le pays en découle. |
| `callback_url` | URL | votre webhook pour ce versement. Sans elle : celle de l’application. |

```curl
curl -X POST https://tchin.tech/api/v1/disburse/initiate \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"account_alias": "90112233", "amount": 5000, "withdraw_mode": "t-money-togo",
       "callback_url": "https://wintg.group/webhook-tchin"}'
```

```php
function tchin(string $chemin, array $corps): array
{
    $ch = curl_init('https://tchin.tech/api/v1'.$chemin);
    curl_setopt_array($ch, [
        CURLOPT_POST => true, CURLOPT_RETURNTRANSFER => true, CURLOPT_TIMEOUT => 30,
        CURLOPT_POSTFIELDS => json_encode($corps),
        CURLOPT_HTTPHEADER => ['TCHIN-PUBLIC-KEY: '.getenv('TCHIN_PUBLIC_KEY'), 'TCHIN-PRIVATE-KEY: '.getenv('TCHIN_PRIVATE_KEY'),
            'Content-Type: application/json', 'Accept: application/json'],
    ]);

    return json_decode(curl_exec($ch), true) ?? [];
}

$p = tchin('/disburse/initiate', ['account_alias' => '90112233', 'amount' => 5000, 'withdraw_mode' => 't-money-togo',
    'callback_url' => 'https://wintg.group/webhook-tchin']);
if (! empty($p['success'])) {
    $e = tchin('/disburse/submit', ['disburse_token' => $p['disburse_token']]); // gardez le jeton avec le versement
}
```

```node
const tchin = (chemin, corps) => fetch('https://tchin.tech/api/v1' + chemin, {
  method: 'POST',
  headers: { 'TCHIN-PUBLIC-KEY': process.env.TCHIN_PUBLIC_KEY, 'TCHIN-PRIVATE-KEY': process.env.TCHIN_PRIVATE_KEY,
             'Content-Type': 'application/json', 'Accept': 'application/json' },
  body: JSON.stringify(corps),
}).then((r) => r.json());

const p = await tchin('/disburse/initiate', { account_alias: '90112233', amount: 5000, withdraw_mode: 't-money-togo',
  callback_url: 'https://wintg.group/webhook-tchin' });
if (p.success) {
  const e = await tchin('/disburse/submit', { disburse_token: p.disburse_token }); // gardez le jeton avec le versement
}
```

```json
{
  "success": true,
  "disburse_token": "xRu8mZkq4Lw0p2Hn",
  "country": "TG",
  "amount": 5000,
  "fee": 170,
  "debited": 5170
}
```

| Champ | Description |
|---|---|
| `disburse_token` | l’identifiant du versement : pour l’envoyer et le suivre. |
| `country` | le pays du réseau choisi. |
| `amount` | ce que le destinataire va recevoir. |
| `fee` | les frais de Tchin (3,4 %). |
| `debited` | ce qui quittera votre solde. |
| `rebalanced, rebalance_fee` | présents seulement si votre solde du pays ne suffisait pas : les transferts faits depuis vos autres soldes de pays, et leur coût. |

### 2. Envoyer

```json
{ "success": true, "status": "success", "message": "Déboursement réussi.", "transaction_id": "PD-8812734" }
```

| status | Sens | Que faire |
|---|---|---|
| `success` | l’argent est parti : votre solde est débité de `debited` | rien |
| `pending` | le réseau n’a pas encore répondu : le versement a pu partir | **ne renvoyez pas** ; lisez l’état, ou attendez le webhook |
| `failed` | refusé par le réseau (numéro inconnu, compte du destinataire plein…) : rien n’est débité | corriger, puis préparer un nouveau versement |

Rappeler `submit` avec un jeton déjà envoyé ne renvoie pas l'argent : la réponse dit « Déjà traité ».

### 3. Suivre

La réponse donne `status` (`success`, `pending` ou `failed`) et les détails du réseau. Un versement passé à `success` entre-temps est alors finalisé sur votre solde.

### Les frais et ce qui quitte votre solde

| Réglage de l’application | Vous envoyez amount = 5 000 | Le destinataire reçoit | Votre solde paie |
|---|---|---|---|
| `Frais à votre charge (par défaut)` | 5 000 F | 5 000 F | 5 170 F |
| `Frais au destinataire` | 5 000 F | 4 830 F | 5 000 F |

Si votre solde du pays du destinataire ne suffit pas, Tchin rassemble ce qui manque depuis vos autres soldes de pays (même monnaie), au tarif habituel du transfert entre soldes, avant de conclure. Si votre compte a été parrainé, vos premiers versements par l'API peuvent être offerts : `fee` vaut alors 0.

### Les erreurs

| Réponse | Sens |
|---|---|
| `400 « withdraw_mode non pris en charge »` | le réseau n’existe pas ou ne sait pas verser |
| `400 « Pays … non pris en charge »` | Tchin verse au Togo, au Bénin et au Burkina Faso |
| `400 « Solde insuffisant sur le pays … »` | même en rassemblant vos autres soldes : la réponse donne `balance`, `required`, `fee` et `available_zone` |
| `422 « Après les frais… »` | frais au destinataire : il recevrait moins de 100 F |
| `423` | un contrôle de sécurité suspend les versements du compte : notre équipe vérifie, contactez le support |
| `404 « Déboursement introuvable »` | `disburse_token` inconnu, ou d’un autre compte |

## Vers une adresse crypto

| Champ | Type | Description |
|---|---|---|
| `currency` | texte | **obligatoire**. `USDT` (TRON, TRC-20), `TRX`, `BNB` (BNB Smart Chain), `SOL` ou `GRAM` (TON). |
| `address` | texte | **obligatoire**. L’adresse du destinataire, sur le bon réseau. Elle est vérifiée avant l’envoi. |
| `amount` | nombre | **obligatoire**. La quantité de crypto (au moins le minimum de la crypto, voir les moyens). Frais à votre charge : ce que l’adresse reçoit. Frais au destinataire : ce qui quitte votre solde. |
| `reference` | texte | **obligatoire**. Votre référence unique (100 caractères au plus) : **la même référence n’envoie qu’une fois**. Réessayer avec elle rend le versement déjà créé. |
| `callback_url` | URL | votre webhook. |

```curl
curl -X POST https://tchin.tech/api/v1/payouts/crypto \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"currency": "USDT", "address": "TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE", "amount": 50,
       "reference": "gain-0042", "callback_url": "https://wintg.group/webhook-tchin"}'
```

```json
{
  "success": true,
  "payout_id": "9b2f6c1e-3d4a-4f7b-a1c2-5e6f7a8b9c0d",
  "reference": "gain-0042",
  "status": "sent",
  "currency": "USDT",
  "address": "TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE",
  "amount": 50,
  "debited": 51.92,
  "fee": 1.92,
  "txid": "a3f1…",
  "explorer": "https://tronscan.org/#/transaction/a3f1…",
  "created_at": "2026-11-03T14:30:00+00:00"
}
```

| Champ | Description |
|---|---|
| `payout_id` | l’identifiant du versement : pour le suivre. |
| `status` | `pending` (en préparation), `sent` (parti sur le réseau), `completed` (confirmé), `failed` (échoué : votre solde est recrédité). |
| `amount` | ce que l’adresse reçoit. |
| `debited` | ce qui quitte votre solde crypto. |
| `fee` | les frais de Tchin et du réseau (et l’activation d’une adresse TRON neuve, s’il y en a une). |
| `txid, explorer` | la transaction sur le réseau et son lien public, dès qu’elle est partie. |
| `message` | en cas d’échec : pourquoi. |

Les erreurs répondent `422` avec un `message` clair : adresse invalide pour ce réseau, quantité sous le minimum, solde crypto insuffisant, réseau momentanément injoignable (réessayez avec la **même** `reference`).

## Le webhook

Quand le versement se résout, votre `callback_url` reçoit un webhook signé avec `kind = "payout"`. Pour un versement, `amount` est ce qui a quitté votre solde (frais compris) et `net` ce que le destinataire reçoit. Un versement crypto ajoute un bloc `crypto` (`currency`, `sent`, `debited`, `address`, `txid`).

```json
{
  "response_code": "00",
  "response_text": "Décaissement completed",
  "reference": "xRu8mZkq4Lw0p2Hn",
  "token": "xRu8mZkq4Lw0p2Hn",
  "kind": "payout",
  "status": "completed",
  "fail_reason": "",
  "amount": 5170,
  "fee": 170,
  "net": 5000,
  "currency": "XOF",
  "country": "TG",
  "method": "t-money-togo",
  "mode": "live",
  "customer": { "name": null, "email": null, "phone": "90112233" },
  "timestamp": 1793716212,
  "signature": "c07d…"
}
```

## Votre solde

```json
{
  "success": true,
  "currency": "XOF",
  "total": 418500,
  "balances": [
    { "country": "TG", "name": "Togo", "balance": 250000 },
    { "country": "BJ", "name": "Bénin", "balance": 168500 },
    { "country": "BF", "name": "Burkina Faso", "balance": 0 }
  ],
  "crypto": { "USDT": 120.5, "TRX": 2400, "BNB": 0.12, "SOL": 1.4, "GRAM": 35 }
}
```

| Champ | Description |
|---|---|
| `total` | la somme de vos soldes en F CFA. |
| `balances` | un solde par pays : le Togo, le Bénin et le Burkina Faso (et un ancien pays seulement s’il vous y reste de l’argent). |
| `crypto` | vos soldes en crypto, en quantité de chaque crypto. |

Lisez-le avant une série de versements : si un solde manque, rechargez-le depuis votre tableau de bord Tchin (Mobile Money ou crypto).

---
Source : https://doc.wintg.group/tchin/versements-directs/
