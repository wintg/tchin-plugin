# Versements directs

Vous connaissez déjà le numéro ou l'adresse (vos vendeurs, vos livreurs) : versez directement, sans feuille.

## Vers un numéro Mobile Money

Deux appels : préparer, puis envoyer.

| Champ | Type | Description |
|---|---|---|
| `account_alias` | texte | **obligatoire**. Le numéro, sans indicatif (« 90112233 »). |
| `amount` | entier | **obligatoire**. Ce que le destinataire reçoit, en F CFA (200 minimum). |
| `withdraw_mode` | texte | **obligatoire**. Le réseau : `t-money-togo`, `moov-togo`, `mtn-benin`, `moov-benin`, `orange-money-burkina`, `moov-burkina`… |
| `callback_url` | URL | votre webhook pour ce versement. |

```curl
curl -X POST https://tchin.tech/api/v1/disburse/initiate \
  -H "TCHIN-PUBLIC-KEY: tchin_pk_..." -H "TCHIN-PRIVATE-KEY: tchin_sk_..." \
  -H "Content-Type: application/json" \
  -d '{"account_alias": "90112233", "amount": 5000, "withdraw_mode": "t-money-togo",
       "callback_url": "https://wintg.group/webhook-tchin"}'
```

La réponse donne un `disburse_token`. Envoyez-le :

Les frais (3,4 %) suivent le réglage de votre application : par défaut, ils s'ajoutent et votre solde paie le montant plus les frais.

## Vers une adresse crypto

| Champ | Type | Description |
|---|---|---|
| `currency` | texte | **obligatoire**. `USDT`, `TRX`, `BNB` ou `SOL`. |
| `address` | texte | **obligatoire**. L’adresse du destinataire, sur le bon réseau. |
| `amount` | nombre | **obligatoire**. La quantité de crypto. |
| `reference` | texte | **obligatoire**. Votre référence unique : la même référence n’envoie qu’une fois. |
| `callback_url` | URL | votre webhook. |

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
  "crypto": { "USDT": 120.5, "TRX": 2400, "BNB": 0.12, "SOL": 1.4 }
}
```

---
Source : https://doc.wintg.group/tchin/versements-directs/
