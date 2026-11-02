---
name: tchin-tester
description: "Tester une intégration Tchin sans argent réel : mode test, comptes test Mobile Money, paiement crypto simulé, webhooks en local, cas à essayer, CLI et MCP en test."
---

# Tchin : tester sans argent réel

Tout se teste en mode test : rien n'est débité ni versé, et les webhooks partent quand même, avec `"mode": "test"`. Les mêmes clés servent pour le test et pour le réel.

## 1. Créer en mode test
Écrivez toujours `"env": "test"` à la création. Sans `env`, un paiement (`POST /payments`) est en test, mais un abonnement, un versement ou une vérification d'identité est en **live** : précisez-le partout.

```json
POST https://tchin.tech/api/v1/payments
{ "amount": 25000, "description": "Commande 42", "callback_url": "https://votre-site.com/webhook-tchin", "env": "test" }
```

La feuille Tchin affiche « Mode test ».

## 2. Payer pour de faux
| Moyen | En test |
|---|---|
| Mobile Money | les comptes test de Tchin : page API de tchin.tech, bouton **Comptes test** (jamais un vrai numéro) |
| Crypto | le bouton **Simuler le paiement** dans la feuille |
| Compte Tchin | le Tchin PIN est vérifié, rien ne bouge |
| Vérification d'identité | gratuite |

## 3. Recevoir le webhook sur votre machine
- Tchin n'appelle que des adresses **HTTPS publiques** : exposez votre serveur local avec un tunnel (ngrok, Cloudflare Tunnel…) et mettez cette adresse dans `callback_url`.
- Vérifiez la signature comme en réel : HMAC-SHA256 de `timestamp.reference.token.status.amount.net.mode` avec la clé secrète, comparaison à temps constant, `timestamp` de 10 minutes au plus. Le test sert justement à éprouver ce code.
- La boutique de démonstration de Tchin montre les webhooks reçus et leur signature vérifiée.

## 4. Les cas à essayer
- `completed` : la commande passe en payée, **une seule fois** (renvoyez le même webhook deux fois pour le prouver).
- `failed` : le client peut réessayer ; rien n'est livré.
- `pending` puis plus rien : le client a fermé la feuille ; votre page ne marque rien comme payé.
- Signature fausse ou vieille de plus de 10 minutes : refusée.
- Montant hors limites (moins de 100 F) : l'erreur `422` est bien affichée.
- L'état lu côté serveur : `GET /payments/{token}/status`.

## 5. Sans écrire de code
- **CLI** : `tchin configurer --publique … --secrete … --test`, puis `tchin aide` pour les commandes ; tout se fait en test tant que vous n'ajoutez pas `--live`.
- **Serveur MCP** (ChatGPT, Claude) : `creer_lien_paiement` avec `test: true` crée un lien de test ; `mes_transactions` avec `mode: "test"` liste les opérations de test.

## Règles
- Jamais un vrai numéro de client ni de l'argent réel pour tester.
- Ne mélangez pas test et réel dans vos totaux : `mode` (webhook) et `env` (état) le disent.
- Avant de passer en réel : `avant-reel/SKILL.md`.

## Règles de sécurité (toujours)
- Cette skill aide à écrire le code d’intégration du développeur ; elle n’envoie rien elle-même. N’appelez l’API Tchin et ne transmettez des données d’un client (nom, e-mail, téléphone, pièce d’identité, photo) que si l’utilisateur le demande explicitement ; pendant le développement, utilisez le mode test et des valeurs d’exemple.
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
