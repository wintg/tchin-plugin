---
name: tchin-cles
description: "Clés API Tchin, en-têtes, mode test ou live, réponses d’erreur et limites de requêtes."
---

# Tchin : Clés, modes et erreurs

Tout ce qui est commun à chaque appel : l'adresse, les formats, les en-têtes, le mode test, les erreurs, les limites, et la bonne façon de réessayer.

## Ce que couvre la référence
- L'adresse et les formats
- Les en-têtes
- Vos clés
- Mode test et mode réel
- Les réponses
- Les limites
- Bonnes pratiques

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Cette skill aide à écrire le code d’intégration du développeur ; elle n’envoie rien elle-même. N’appelez l’API Tchin et ne transmettez des données d’un client (nom, e-mail, téléphone, pièce d’identité, photo) que si l’utilisateur le demande explicitement ; pendant le développement, utilisez le mode test et des valeurs d’exemple.
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
