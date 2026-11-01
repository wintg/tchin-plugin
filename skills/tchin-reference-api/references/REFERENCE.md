# Référence de l'API

Tous les points d'entrée, sur https://tchin.tech/api/v1, avec les en-têtes TCHIN-PUBLIC-KEY et TCHIN-PRIVATE-KEY.

## Paiements

## Abonnements

## Versements

## Vérification d'identité

## Compte

## Serveur MCP (IA)

Sur `https://tchin.tech/mcp`. Sans connexion : `afficher_lien_tchin`, `preparer_demande_tchin`, `etat_paiement`. Avec le compte de l'utilisateur (OAuth, demandé au premier usage) : 34 outils, dont `mes_transactions`, `bilan_compte`, `releve_pdf`, `creer_lien_paiement`, et les demandes d'argent confirmées sur tchin.tech avec le Tchin PIN (`envoyer_argent`, `retirer_mobile_money`, `deplacer_argent`, `etat_demande`). Voir Tchin dans votre IA.

## Arrêtés le 30 novembre 2026

Et la création d'un abonnement avec `customer_phone` (prélèvement sur un numéro donné par vous). Jusqu'au 30 novembre, ces appels répondent avec les en-têtes `Deprecation` et `Sunset` ; à partir du 1er décembre, avec `410`. Voir le calendrier.

---
Source : https://doc.wintg.group/tchin/reference/
