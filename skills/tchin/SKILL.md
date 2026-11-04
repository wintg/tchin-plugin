---
name: tchin
description: "Point d’entrée Tchin : encaisser (Mobile Money, crypto, compte Tchin), abonnements, versements, vérification d’identité, webhooks, MCP. À lire en premier pour toute question Tchin."
---

# Tchin : démarrer

Tchin (tchin.tech, une solution Wintg) encaisse par Mobile Money au Togo, au Bénin et au Burkina Faso, en crypto (USDT, TRX, BNB, SOL, GRAM) et par compte Tchin.

## Le principe
1. Votre **serveur** crée l'opération avec vos clés : `POST https://tchin.tech/api/v1/payments` (en-têtes `TCHIN-PUBLIC-KEY`, `TCHIN-PRIVATE-KEY`). Tchin rend un `token`.
2. Votre **page** ouvre la feuille : `<script src="https://tchin.tech/js/tchin-pay.js"></script>` puis `TchinPay.open({ token })`.
3. Le client paie dans la feuille. Votre serveur reçoit un **webhook signé** sur `callback_url`.

## Quelle compétence lire
| Besoin | Skill à lire |
|---|---|
| Première intégration pas à pas | `tchin-demarrer` |
| Clés, mode test/live, erreurs, limites | `tchin-cles` |
| Encaisser un paiement unique | `tchin-paiements` |
| Réseaux et pays acceptés, frais | `tchin-moyens` |
| Paiement récurrent | `tchin-abonnements` |
| Envoyer de l'argent à un client (il choisit où recevoir) | `tchin-recevoir` |
| Verser à un numéro ou une adresse crypto connue | `tchin-versements-directs` |
| Vérifier l'identité d'un client (KYC) | `tchin-kyc` |
| Webhooks et signature | `tchin-webhooks` |
| Serveur MCP : payer dans ChatGPT / Claude, gérer son compte (historique, relevé PDF, envois confirmés) | `tchin-ia-mcp` |
| Kits (PHP, Node, Python, Java, Go, Android, iOS, Flutter, React Native, Cordova) | `tchin-kits` |
| Tous les points d'entrée | `tchin-reference-api` |
| Méthodes retirées (réponse 410) et passer à la feuille | `tchin-reference-api` |
| Questions fréquentes | `tchin-faq` |
| Agent Toolkit, CLI tchin, Agent Skills | `tchin-outils-ia` |
| Tester sans argent réel (mode test, comptes test, webhooks en local) | `tchin-tester` |
| La liste à vérifier avant de passer en réel | `tchin-avant-reel` |

La plupart de ces skills ont un `references/REFERENCE.md` complet (champs, code dans chaque langage, réponses). Lisez-le seulement si le SKILL.md ne suffit pas.

## Règles de sécurité (toujours)
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
