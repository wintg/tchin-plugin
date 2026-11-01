---
name: tchin-moyens
description: "Les moyens de paiement Tchin par pays (Togo, Bénin, Burkina Faso), la crypto, le compte Tchin, et les frais."
---

# Tchin : Moyens de paiement

La feuille propose les moyens que votre application accepte (réglages de l'application sur la page API de Tchin). Le client choisit en haut de la feuille.

## Ce que couvre la référence
- Mobile Money
- Crypto
- Compte Tchin
- Quels moyens proposer ?

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
