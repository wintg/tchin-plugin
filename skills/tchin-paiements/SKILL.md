---
name: tchin-paiements
description: "Encaisser un paiement unique avec Tchin : POST /payments, ouvrir la feuille TchinPay, suivre l’état, frais."
---

# Tchin : Paiements

Un paiement unique : votre serveur le crée, votre page ouvre la feuille Tchin, le client paie sans quitter votre site.

## Ce que couvre la référence
- Créer un paiement
- Ouvrir la feuille
- Suivre l'état
- Les frais et le montant payé

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
