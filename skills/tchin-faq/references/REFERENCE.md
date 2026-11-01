# Questions fréquentes

## Où arrive l'argent ?

Sur votre compte Tchin : le solde du pays où le client a payé en Mobile Money, ou le solde de la crypto payée. Payé avec un compte Tchin, sur le même solde que celui du client. Vous le retirez depuis votre tableau de bord Tchin, vers un numéro Mobile Money ou une adresse crypto.

## Quand le paiement est-il confirmé ?

Mobile Money : dès que le client valide sur son téléphone, en général en moins d'une minute. Crypto : dès le premier bloc confirmé (quelques secondes sur TRON). Compte Tchin : tout de suite.

## Qui paie les frais ?

Vous choisissez dans les réglages de votre application, séparément pour l'encaissement et pour le versement : le client (ils s'ajoutent au prix) ou vous (ils sont retirés de ce que vous recevez). Entre deux comptes Tchin, il n'y a aucun frais.

## Quels pays ?

Togo, Bénin et Burkina Faso, en francs CFA. En crypto, partout dans le monde.

## Le client a fermé la feuille avant de payer : que faire ?

Rien : le paiement reste en attente. Rouvrez la feuille avec le même token, ou créez-en un nouveau. Si le client a laissé son e-mail, Tchin le relance.

## Comment rembourser un client ?

Avec Recevoir : vous créez un versement du montant, le client choisit où le recevoir.

## Comment être sûr qu'un paiement est vrai ?

Vérifiez la signature du webhook avec votre clé secrète, ou demandez l'état à `GET /api/v1/payments/{token}/status` depuis votre serveur. Le retour de la page (`onSuccess`) n'est qu'une indication.

## Comment tester sans argent ?

Ajoutez `"env": "test"` : utilisez les comptes test de Tchin (Mobile Money), le bouton « Simuler le paiement » (crypto), ou votre compte Tchin (le PIN est vérifié, rien ne bouge).

## Une autre question ?

Écrivez-nous sur WhatsApp au +228 73 38 49 90, ou depuis tchin.tech.

---
Source : https://doc.wintg.group/tchin/faq/
