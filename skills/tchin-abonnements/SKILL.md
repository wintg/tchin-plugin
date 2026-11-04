---
name: tchin-abonnements
description: "Abonnements Tchin (paiements récurrents) : créer, lister, résilier, échéances, webhooks subscription.*."
---

# Tchin : Abonnements

Un montant fixe chaque mois, trimestre, semestre ou année. Le client donne son accord et choisit son moyen la première fois, dans la feuille ; Tchin s'occupe ensuite de chaque échéance, des rappels et des relances.

## Ce que couvre la référence
- Le principe
- Créer un abonnement
- L'objet abonnement
- Chaque échéance
- Les états
- Résilier depuis votre site
- Lister et suivre
- Les webhooks
- Tester

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Cette skill aide à écrire le code d’intégration du développeur ; elle n’envoie rien elle-même. N’appelez l’API Tchin et ne transmettez des données d’un client (nom, e-mail, téléphone, pièce d’identité, photo) que si l’utilisateur le demande explicitement ; pendant le développement, utilisez le mode test et des valeurs d’exemple.
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
