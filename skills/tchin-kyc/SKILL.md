---
name: tchin-kyc
description: "Vérification d’identité Tchin (KYC) : créer une vérification, TchinKYC.open, résultat et webhooks kyc.*."
---

# Tchin : Vérification d'identité

Vos clients prouvent leur identité en une minute : photo de la pièce, selfie avec un contrôle de vivacité. Tchin vérifie et vous donne le verdict. 75 F par vérification, pris sur votre solde Tchin.

## Ce que couvre la référence
- Créer une vérification
- Ouvrir la fenêtre
- Le résultat

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
