# Questions fréquentes

Les questions que les développeurs et les marchands nous posent le plus, avec des réponses courtes. Une autre question ? Écrivez-nous, ou demandez à Letou dans votre compte.

## Où arrive l'argent ?

Sur votre compte Tchin : le solde du pays où le client a payé en Mobile Money, ou le solde de la crypto payée (sans conversion). Payé avec un compte Tchin, sur le même solde que celui du client. Vous le retirez depuis votre tableau de bord, vers un numéro Mobile Money ou une adresse crypto, ou vous le versez par l'API.

## Quand le paiement est-il confirmé ?

Mobile Money : dès que le client valide sur son téléphone, en général en moins d'une minute. Crypto : dès que le virement est confirmé sur le réseau (quelques secondes sur TRON, Solana et TON). Compte Tchin : tout de suite. Le webhook part dans la foulée.

## Qui paie les frais ?

Vous choisissez dans les réglages de votre application, séparément pour l'encaissement et pour le versement : le client (ils s'ajoutent au prix) ou vous (ils sont retirés de ce que vous recevez). Vous pouvez aussi choisir paiement par paiement avec `fees_on_customer`. Entre deux comptes Tchin, il n'y a aucun frais. Voir les frais.

## Quels pays ?

Togo, Bénin et Burkina Faso pour le Mobile Money, en francs CFA. En crypto (USDT, TRX, BNB, SOL, GRAM), partout dans le monde : c'est le moyen idéal pour un client à l'étranger.

## Mon paiement s'affiche en « Mode test » en production

Le champ `env` manque : pour `POST /payments`, sa valeur par défaut est `test`. Ajoutez `"env": "live"`. Les mêmes clés servent aux deux modes.

## Comment tester sans argent ?

Ajoutez `"env": "test"`, puis payez avec les comptes test de la page API de Tchin (bouton **Comptes test**) : un compte Mobile Money fictif, le bouton « Simuler le paiement » en crypto, ou votre propre compte Tchin (le PIN est vérifié, rien ne bouge). Les webhooks partent, avec `"mode": "test"`.

## Le client a fermé la feuille avant de payer : que faire ?

Rien : le paiement reste en attente (`pending`). Rouvrez la feuille avec le même token quand il revient, ou envoyez-lui la `payment_url`. Pour un autre montant, créez un nouveau paiement.

## Le paiement a échoué : pourquoi ?

Le plus souvent : le client n'a pas validé à temps sur son téléphone, s'est trompé de code secret, ou n'avait pas assez sur son compte Mobile Money. Le webhook donne `fail_reason`. Le client peut réessayer tout de suite dans la feuille, avec le même token ou un autre moyen.

## Je ne reçois pas les webhooks

- Vérifiez qu'une adresse est donnée : `callback_url` à la création, ou l'adresse de webhook de l'application.

- Elle doit être en **HTTPS**, publique (pas `localhost`, pas une adresse privée), et répondre `2xx` en moins de 15 secondes.

- Un pare-feu ou une protection anti-robots (Cloudflare…) peut bloquer nos appels : autorisez les `POST` sur cette adresse.

- En attendant, lisez l'état avec `GET /api/v1/payments/{token}/status` : il fait foi.

## La signature ne correspond pas

Trois causes à vérifier : la clé secrète n'est pas celle de l'application qui a créé l'opération ; la chaîne n'est pas recomposée dans l'ordre `timestamp.reference.token.status.amount.net.mode` ; ou une valeur a été transformée (un nombre reformaté, un champ absent remplacé par `null` au lieu d'un texte vide). Voir la signature.

## Comment être sûr qu'un paiement est vrai ?

Vérifiez la signature du webhook avec votre clé secrète, ou demandez l'état à `GET /api/v1/payments/{token}/status` depuis votre serveur, puis comparez le montant et le mode. Le retour de la page (`onSuccess`) et l'adresse de retour ne sont que des indications : n'importe qui peut les imiter.

## Comment rembourser un client ?

Avec Recevoir : vous créez un versement du montant, le client choisit où le recevoir (son numéro Mobile Money ou son adresse crypto). Si vous connaissez déjà son numéro, un versement direct suffit.

## Ma clé secrète a fuité

Créez une nouvelle application sur la page API de Tchin, mettez ses clés sur votre serveur, puis supprimez l'ancienne : ses clés ne marchent plus, immédiatement. Pensez à donner la même adresse de webhook à la nouvelle.

## Mon intégration répond « 410 »

Elle utilise une ancienne façon d'encaisser (votre propre formulaire de paiement, ou un abonnement prélevé sur un numéro donné par vous), retirée de l'API. Passez à la feuille en trois étapes : voir les méthodes retirées.

## Je n'ai pas de développeur

Sans code, créez des **liens de paiement** depuis votre compte Tchin (une adresse à partager, un bouton à coller sur votre site), ou demandez à Letou de les créer pour vous.

## Une autre question ?

Écrivez-nous sur WhatsApp au +228 73 38 49 90 avec le `token` de l'opération concernée, ou depuis tchin.tech. Letou, l'assistant de votre compte, connaît cette documentation et écrit le code avec vous. Ne partagez jamais votre clé secrète, même avec nous.

---
Source : https://doc.wintg.group/tchin/faq/
