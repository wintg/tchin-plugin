# Moyens de paiement

La feuille propose les moyens que votre application accepte (réglages de l'application sur la page API de Tchin). Le client choisit en haut de la feuille.

## Mobile Money

1. Le client choisit son **pays** (Togo, Bénin, Burkina Faso).

1. Il tape son **numéro** : Tchin reconnaît le réseau (Yas, Moov, MTN, Celtiis, Orange).

1. Il touche **Payer** : une demande arrive sur son téléphone, il la valide avec son **code secret**. Orange Money Burkina demande un code reçu par SMS.

L'argent arrive sur **votre solde du pays** où le client a payé. Les frais : 4 %.

## Crypto

1. Le client choisit sa crypto : **USDT** (TRC-20), **TRX**, **BNB**, **SOL** ou **GRAM**. Le montant est converti au cours du jour.

1. La feuille donne le montant exact et l'adresse (à toucher pour copier, ou en code QR) ; sur téléphone, ses portefeuilles s'ouvrent d'un toucher (TronLink, Trust Wallet, MetaMask, Phantom…).

1. Dès le premier bloc confirmé, le paiement passe et la feuille l'affiche.

L'argent arrive sur **votre solde de cette crypto** (USDT sur votre solde USDT, etc.). Les frais : 2 %.

## Compte Tchin

1. Le client choisit le **solde** qui paie : Mobile Money d'un pays, ou une crypto (dans une liste déroulante).

1. Il tape son **Tchin ID** (`pseudo@tchin`, ni e-mail ni numéro).

1. Il tape son **Tchin PIN**, ou utilise sa biométrie si elle est enregistrée sur cet appareil. Le paiement part tout de suite.

Entre deux comptes Tchin, **aucun frais** : le client paie le prix, vous recevez le prix, sur le **même solde** (Mobile Money Togo vers votre solde Togo, TRON vers votre solde TRON). Le reçu part à l'e-mail du compte du client. Si son solde ne suffit pas, il en choisit un autre sans retaper son PIN.

## Quels moyens proposer ?

| Moyen | Le client a besoin de | Idéal pour |
|---|---|---|
| `Mobile Money` | un compte Mobile Money au Togo, au Bénin ou au Burkina Faso | la plupart des clients |
| `Crypto` | un portefeuille crypto | les clients à l’étranger, les montants élevés |
| `Compte Tchin` | un compte Tchin vérifié | payer en deux secondes, sans frais |

La feuille se souvient du dernier moyen utilisé sur l'appareil du client et le reprend. Letou, l'assistant de Tchin, guide le client à chaque étape (un bouton **Suivant** fait l'étape à sa place).

---
Source : https://doc.wintg.group/tchin/moyens/
