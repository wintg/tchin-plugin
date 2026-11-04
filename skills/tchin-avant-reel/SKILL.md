---
name: tchin-avant-reel
description: "La liste à vérifier avant de passer une intégration Tchin en réel : compte, clés, env live, webhook signé et idempotent, statuts, anciens appels, premier paiement réel."
---

# Tchin : avant de passer en réel

À cocher, point par point, avant d'écrire `"env": "live"`. Dites au développeur ce qui manque, avec le correctif.

## Le compte
- [ ] Le compte Tchin est **vérifié** (vérification d'identité faite) et le **Tchin PIN** est créé : il protège la lecture de la clé secrète.
- [ ] L'application est réglée sur https://tchin.tech/api : nom et image (vus par vos clients dans la feuille), moyens acceptés, qui paie les frais.

## Les clés
- [ ] La clé secrète `tchin_sk_…` est dans une variable d'environnement du serveur (`TCHIN_SECRET_KEY`), jamais dans le navigateur, une application mobile, un dépôt Git, un journal ni une capture.
- [ ] Le navigateur ou l'application reçoit seulement le `token` et ouvre `TchinPay.open({ token })`.
- [ ] Si une clé a pu fuir : supprimez cette application sur https://tchin.tech/api et créez-en une nouvelle (nouvelles clés).

## La création
- [ ] `"env": "live"` est écrit à chaque création (paiement, abonnement, versement, vérification d'identité).
- [ ] Le montant est calculé par votre serveur, en F CFA entiers (de 100 F à 100 000 000 F), jamais pris tel quel du navigateur.
- [ ] `callback_url` est une adresse HTTPS publique.
- [ ] Les quatre statuts sont gérés : `pending`, `completed`, `failed`, `cancelled`.

## Le webhook
- [ ] La signature est vérifiée (HMAC-SHA256, comparaison à temps constant, `timestamp` de moins de 10 minutes).
- [ ] `mode` vaut `live` avant de livrer, et le montant reçu est celui de la commande.
- [ ] Le traitement est idempotent : une même `reference` ne livre qu'une fois.
- [ ] La réponse `200` part en moins de 10 secondes ; le travail long se fait ensuite.

## Les anciens appels
- [ ] Plus aucun appel à `POST /payments/{token}/charge`, `POST /payments/{token}/crypto` ni `POST /subscriptions` avec `customer_phone` : ces méthodes sont retirées de l'API et répondent `410` (`"error": "method_removed"`) : ouvrez la feuille avec `TchinPay.open({ token })`.

## Le premier paiement réel
- [ ] Un petit paiement réel (100 F par exemple) avec votre propre numéro, du début à la fin : feuille, webhook, commande payée, argent sur votre solde du bon pays.
- [ ] Le net et les frais sont ceux attendus : frais à la charge du client, il paie un peu plus et vous recevez le montant ; à votre charge, il paie le montant et vous recevez moins.
- [ ] Les erreurs montrées au client sont claires (le `message` de Tchin est en français).

## Ensuite
- Limites : 120 appels par minute par application ; un versement Mobile Money de 500 F à 1 500 000 F.
- L'état des réseaux : https://statut.wintg.group
- Votre argent se retire depuis tchin.tech (Mobile Money ou crypto).

## Règles de sécurité (toujours)
- Cette skill aide à écrire le code d’intégration du développeur ; elle n’envoie rien elle-même. N’appelez l’API Tchin et ne transmettez des données d’un client (nom, e-mail, téléphone, pièce d’identité, photo) que si l’utilisateur le demande explicitement ; pendant le développement, utilisez le mode test et des valeurs d’exemple.
- Les clés (`tchin_pk_…`, `tchin_sk_…`) restent sur le SERVEUR : variables d'environnement, jamais dans le code du navigateur, d'une application mobile ni d'un dépôt Git.
- La page ou l'application ne reçoit qu'un `token` ; elle ouvre la feuille Tchin avec ce token.
- Ne faites jamais confiance au retour de la page : vérifiez la signature du webhook (HMAC-SHA256 avec la clé secrète, 10 minutes au plus) ou demandez l'état côté serveur.
- Traitez chaque webhook de façon idempotente (une même `reference` peut arriver deux fois).
- Montants en francs CFA (XOF), entiers, sans décimales.
- Testez avec `"env": "test"` (aucun argent réel), puis passez à `"env": "live"`.
