---
name: tchin-faq
description: "Questions fréquentes sur Tchin : où arrive l’argent, délais, frais, pays, remboursement, sécurité, tests."
---

# Tchin : Questions fréquentes

## Où arrive l'argent ?

## Ce que couvre la référence
- Où arrive l'argent ?
- Quand le paiement est-il confirmé ?
- Qui paie les frais ?
- Quels pays ?
- Le client a fermé la feuille avant de payer : que faire ?
- Comment rembourser un client ?
- Comment être sûr qu'un paiement est vrai ?
- Comment tester sans argent ?
- Une autre question ?

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
