# Moyens de paiement

La feuille propose les moyens que votre application accepte (ses réglages, sur la page API de Tchin). Le client choisit en haut de la feuille ; elle se souvient de son dernier choix sur son appareil.

## Mobile Money

1. Le client choisit son **pays** : Togo, Bénin ou Burkina Faso.

1. Il tape son **numéro** : Tchin reconnaît le réseau à ses premiers chiffres (il peut le changer s'il a gardé son numéro en changeant de réseau).

1. Il touche **Payer** : une demande arrive sur son téléphone, et il la valide avec son **code secret** Mobile Money. Avec Orange Money au Burkina Faso, il compose d'abord un code sur son téléphone, puis tape le code reçu par SMS dans la feuille.

| Pays | Réseau | Code | Validation | Encaisser | Verser |
|---|---|---|---|---|---|
| `TG` | Yas (ex-T-Money) | `t-money-togo` | sur le téléphone | 4 % | 3,4 % |
| `TG` | Moov | `moov-togo` | sur le téléphone | 4 % | 3,4 % |
| `BJ` | MTN | `mtn-benin` | sur le téléphone | 4 % | 3,4 % |
| `BJ` | Moov | `moov-benin` | sur le téléphone | 4 % | 3,4 % |
| `BJ` | Celtiis Cash | `celtiis-benin` | sur le téléphone | 4 % | 3,4 % |
| `BF` | Orange Money | `orange-money-burkina` | code reçu par SMS | 4 % | 3,4 % |
| `BF` | Moov | `moov-burkina` | sur le téléphone | 4 % | 3,4 % |

L'argent arrive sur **votre solde du pays** où le client a payé. Le Togo, le Bénin et le Burkina Faso partageant la même monnaie, vos soldes de pays se complètent quand vous versez (voir Versements directs).

## Crypto

1. Le client choisit sa crypto. Le prix est converti au cours du moment, et la feuille montre la quantité de chaque crypto.

1. La feuille donne la **quantité exacte** à envoyer et l'**adresse** (à toucher pour copier, ou en code QR). Sur téléphone, ses portefeuilles s'ouvrent d'un toucher : TronLink, Trust Wallet, MetaMask, Phantom, Tonkeeper…

1. Dès que le virement est confirmé sur le réseau, le paiement passe et la feuille l'affiche. Le client a 60 minutes pour envoyer ; un virement arrivé en retard (jusqu'à 24 heures) est encore reconnu.

| Crypto | Réseau | Minimum | Encaisser | Verser |
|---|---|---|---|---|
| `USDT` | TRON (TRC-20) | 1 USDT | 2 % | 1 % + réseau |
| `TRX` | TRON | 1 TRX | 2 % | 1 % + réseau |
| `BNB` | BNB Smart Chain (BEP-20) | 0,0001 BNB | 2 % | 1 % + réseau |
| `SOL` | Solana | 0,001 SOL | 2 % | 1 % + réseau |
| `GRAM` | TON (Gram, l’ancien Toncoin) | 0,1 GRAM | 2 % | 1 % + réseau |

L'argent arrive sur **votre solde de cette crypto** (USDT sur votre solde USDT, etc.), sans conversion en F CFA. Chaque paiement a sa quantité exacte, unique au millionième près : c'est elle qui permet de reconnaître le bon virement sur votre adresse.

## Compte Tchin

1. Le client choisit le **solde** qui paie : Mobile Money d'un pays, ou une crypto (dans une liste).

1. Il tape son **Tchin ID** (`pseudo@tchin`, ni e-mail ni numéro).

1. Il tape son **Tchin PIN**, ou utilise sa biométrie si elle est enregistrée sur cet appareil. Le paiement part tout de suite.

Entre deux comptes Tchin, **aucun frais** : le client paie le prix, vous recevez le prix, sur le **même solde** (Mobile Money Togo vers votre solde Togo, USDT vers votre solde USDT). Le reçu part à l'e-mail du compte du client. Si son solde ne suffit pas, il en choisit un autre sans retaper son PIN.

## Lire les moyens ouverts : GET /methods

La liste des moyens du moment, lue en direct : les réseaux de chaque pays, s'ils sont ouverts à l'instant (un réseau peut être coupé quelques minutes chez l'opérateur), leurs tarifs pour votre compte, et les réglages de votre application. Pratique pour afficher les logos des moyens acceptés sur votre site, ou pour savoir avant un versement si le réseau du destinataire répond.

| Paramètre | Effet |
|---|---|
| `(aucun)` | tous les moyens Mobile Money des pays de Tchin, et les cryptos si votre application les accepte |
| `?for=payout` | seulement ceux qui peuvent **recevoir** un versement en ce moment |
| `?for=subscription` | seulement les réseaux prélevables pour un abonnement (validation sur le téléphone) ; pas de crypto |

```json
{
  "success": true,
  "countries": [
    {
      "country": "TG",
      "country_name": "Togo",
      "dial": "+228",
      "methods": [
        {
          "code": "t-money-togo",
          "name": "Yas",
          "type": "mobile_money",
          "logo": "https://tchin.tech/images/operators/yas.jpg",
          "flow": "push",
          "needs_otp": false,
          "ussd": null,
          "otp_hint": null,
          "payin": true,
          "payout": true,
          "available": true,
          "payout_available": true,
          "payin_fee": 4,
          "payout_fee": 3.4,
          "subscription_eligible": true
        }
      ]
    }
  ],
  "cryptos": [
    {
      "code": "USDT",
      "name": "USDT (TRC-20)",
      "type": "crypto",
      "network": "TRON (TRC-20)",
      "logo": "https://tchin.tech/images/cryptos/usdt.svg",
      "available": true,
      "payout_available": true,
      "payin_fee": 2,
      "payout_fee": 1,
      "minimum": 1
    }
  ],
  "app": {
    "accepts": { "mobile_money": true, "crypto": true, "tchin": true },
    "fees": { "payments": "customer", "payouts": "merchant" },
    "countries": ["TG", "BJ", "BF"]
  }
}
```

### Un réseau Mobile Money

| Champ | Description |
|---|---|
| `code` | le code du réseau, celui des versements (`withdraw_mode`) et du champ `method` des webhooks. |
| `name, logo` | le nom commercial et son logo, à afficher. |
| `flow` | `push` : le client valide sur son téléphone ; `otp` : il tape un code reçu par SMS (`needs_otp` vaut alors `true`, `ussd` et `otp_hint` disent comment l’obtenir). |
| `payin, payout` | le réseau sait encaisser, verser. |
| `available, payout_available` | ouvert **en ce moment** pour encaisser, pour verser. Un réseau en pause chez l’opérateur passe à `false` quelques minutes. |
| `payin_fee, payout_fee` | les taux en pour cent qui s’appliquent à votre compte. |
| `subscription_eligible` | utilisable pour les échéances d’un abonnement. |

### Une crypto

| Champ | Description |
|---|---|
| `code` | `USDT`, `TRX`, `BNB`, `SOL` ou `GRAM` : celui des versements crypto (`currency`). |
| `network` | le réseau sur lequel envoyer : TRON (TRC-20), TRON, BNB Smart Chain, Solana, TON. |
| `available, payout_available` | ouverte en ce moment pour encaisser, pour verser. |
| `payin_fee, payout_fee` | les taux en pour cent (au versement, le coût du réseau s’ajoute). |
| `minimum` | la plus petite quantité acceptée. |

### Le bloc app

| Champ | Description |
|---|---|
| `accepts` | les moyens que votre application accepte : `mobile_money`, `crypto`, `tchin` (compte Tchin). |
| `fees.payments` | `customer` : le client paie les frais d’encaissement ; `merchant` : vous. |
| `fees.payouts` | `merchant` : vous payez les frais de versement ; `recipient` : le destinataire. |
| `countries` | les pays où votre application encaisse et verse. |

## Quels moyens proposer ?

| Moyen | Le client a besoin de | Idéal pour |
|---|---|---|
| `Mobile Money` | un compte Mobile Money au Togo, au Bénin ou au Burkina Faso | la plupart des clients, au quotidien |
| `Crypto` | un portefeuille crypto | les clients à l’étranger, la diaspora, les montants élevés |
| `Compte Tchin` | un compte Tchin | payer en deux secondes, sans frais, ni pour lui ni pour vous |

Le plus simple : cochez les trois. La feuille ne montre que ce qui est ouvert, et le client prend ce qu'il a. Letou, l'assistant de Tchin, le guide à chaque étape (un bouton **Suivant** fait l'étape à sa place).

---
Source : https://doc.wintg.group/tchin/moyens/
