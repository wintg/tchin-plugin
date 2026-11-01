---
name: tchin-demarrer
description: "Intégrer Tchin de zéro en 5 minutes : créer l’application, récupérer les clés, créer un paiement de test, ouvrir la feuille, recevoir le webhook."
---

# Tchin : démarrer en 5 minutes

1. Le marchand crée une **application** sur https://tchin.tech/api (nom, image, moyens acceptés, qui paie les frais) puis **Voir les clés** (Tchin PIN demandé).
2. Serveur : `POST /api/v1/payments` avec `amount`, `description`, `callback_url`, `"env": "test"` → `token`.
3. Page : `TchinPay.open({ token, onSuccess, onPending, onClose })`.
4. Webhook : vérifier la signature, marquer la commande payée si `status = completed`.
5. Tester (comptes de test Mobile Money, bouton « Simuler » en crypto), puis `"env": "live"`.

Détails, code complet dans chaque langage : `references/REFERENCE.md`.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
