---
name: tchin-ia-mcp
description: "Le serveur MCP Tchin (https://tchin.tech/mcp) : payer dans ChatGPT ou Claude sans compte ; gérer tout son compte (historique, relevé PDF, liens, envois et retraits confirmés avec le Tchin PIN)."
---

# Tchin : Tchin dans votre IA

Dans ChatGPT, Claude ou toute IA compatible MCP, on paie un lien Tchin ou un Tchin ID sans quitter la conversation, et chacun gère son compte : soldes, historique, relevés en PDF, liens de paiement, envois et retraits confirmés avec le Tchin PIN. Vous n'avez rien à intégrer.

## Ce que couvre la référence
- Le principe
- Comment vos clients y accèdent
- Ajouter le connecteur Tchin
- Les outils pour payer (sans compte)
- Pour les IA qui naviguent
- Sécurité
- Gérer son compte depuis son IA
- Envoyer, retirer, déplacer de l'argent
- Le relevé en PDF
- Ce que votre IA ne peut pas faire
- Des méthodes toutes prêtes
- Pour vous, marchand

Lisez `references/REFERENCE.md` pour les champs exacts, le code dans chaque langage (cURL, PHP, Node.js, Python, Java, Go) et les réponses.

## Règles de sécurité (toujours)
- Cette skill aide à écrire le code d’intégration du développeur ; elle n’envoie rien elle-même. N’appelez l’API Tchin et ne transmettez des données d’un client (nom, e-mail, téléphone, pièce d’identité, photo) que si l’utilisateur le demande explicitement ; pendant le développement, utilisez le mode test et des valeurs d’exemple.
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
