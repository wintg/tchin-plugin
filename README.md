# Tchin pour Claude

[Tchin](https://tchin.tech) est une plateforme de paiement d'Afrique de l'Ouest (Togo, Bénin, Burkina Faso) : Mobile Money, compte Tchin et crypto, en francs CFA. Ce plugin relie Claude à Tchin : payer un lien Tchin sur tchin.tech, lire son compte, faire un relevé PDF, créer des liens de paiement, relancer les impayés et vendre sur WhatsApp. Tchin est une solution de [Wintg](https://wintg.group), à Lomé.

*English summary: Tchin is a West African payment platform (Togo, Benin, Burkina Faso). This plugin connects Claude to Tchin's MCP server and adds 22 French-language skills: for every Tchin user (using Tchin well, bookkeeping and PDF statements, payment reminders, selling on WhatsApp with payment links) and for developers integrating the Tchin API (getting started, keys, payments, subscriptions, payouts, identity checks, webhooks, testing without real money, go-live checklist). Payments always happen on tchin.tech; the plugin never sends, withdraws or transfers money.*

## Ce que contient le plugin

Le serveur MCP `tchin` (`https://tchin.tech/mcp/app`, l'édition des annuaires du serveur MCP de Tchin) et 22 skills.

**Pour tous les utilisateurs de Tchin** (marchands et particuliers) :

| Skill | Rôle |
|---|---|
| `tchin-bien-utiliser` | quel outil Tchin pour quelle demande, la connexion, les règles de sécurité |
| `tchin-comptable` | le bilan d'une période, le relevé PDF, les frais, le rapprochement avec vos chiffres |
| `tchin-relances` | les abonnements en retard et les paiements échoués, des relances polies avec le lien Tchin |
| `tchin-vendre-whatsapp` | le lien de paiement d'un produit, le message et le statut WhatsApp, qui a payé |

**Pour les développeurs** (intégrer l'API Tchin dans un site ou une application) :

| Skill | Rôle |
|---|---|
| `tchin` | la fiche d'entrée : quelle skill lire selon la demande, et les règles de sécurité |
| `tchin-demarrer` | intégrer Tchin de zéro en 5 minutes |
| `tchin-cles` | les clés, le mode test ou réel, les erreurs et les limites |
| `tchin-paiements`, `tchin-moyens` | encaisser un paiement et ouvrir la feuille ; les réseaux, les pays et les frais |
| `tchin-abonnements` | les paiements récurrents |
| `tchin-recevoir`, `tchin-versements-directs` | envoyer de l'argent à un client ; verser à un numéro ou une adresse connue |
| `tchin-kyc`, `tchin-webhooks` | la vérification d'identité ; les webhooks et leur signature |
| `tchin-ia-mcp`, `tchin-outils-ia` | le serveur MCP ; la CLI, l'Agent Toolkit et les skills |
| `tchin-kits`, `tchin-reference-api`, `tchin-migration`, `tchin-faq` | les kits, tous les points d'entrée, le calendrier, les questions fréquentes |
| `tchin-tester` | tester sans argent réel : mode test, comptes test, webhooks en local |
| `tchin-avant-reel` | la liste à vérifier avant de passer en réel |

## Les outils du serveur

- **Sans compte** : `afficher_lien_tchin`, `preparer_demande_tchin` (affichent le lien officiel avec un bouton qui ouvre la page de paiement sur tchin.tech), `etat_paiement`, et `aide_tchin` (recherche dans la documentation officielle).
- **Avec le compte de l'utilisateur** (connexion OAuth sur tchin.tech, demandée au premier usage) : soldes, historique complet, bilan, relevé PDF, liens de paiement et formulaires, ventes, abonnements, prélèvements, cashback, parrainage, coffres, applications API, et `ouvrir_tchin` pour les pages de Tchin.
- **Jamais** : envoyer, retirer, déplacer ou transférer de l'argent ni de la crypto. L'utilisateur le fait lui-même sur tchin.tech.

## Données et sécurité

- Le plugin n'exécute aucun programme sur votre machine : il contient des fiches (skills) et l'adresse du serveur MCP de Tchin.
- Les seules données échangées le sont entre Claude et `https://tchin.tech`, et seulement quand un outil Tchin est appelé.
- Le serveur ne demande jamais de Tchin PIN, de code secret Mobile Money, de code de connexion ni de clé secrète dans la conversation. Les numéros de téléphone, les e-mails et les noms des clients sont masqués dans tout ce qu'il renvoie.
- Les textes écrits par des tiers (titres de liens, noms) qui contiennent des consignes adressées à une IA sont masqués, et refusés à la création.
- L'accès se coupe à tout moment dans Tchin, page API, rubrique Serveur MCP. Les relevés PDF ne s'ouvrent qu'au titulaire du compte, connecté à Tchin, et sont effacés au bout de 24 heures.
- Politique de confidentialité : https://tchin.tech/legal#confidentialite (section 3.14, assistants d'intelligence artificielle connectés).

## Installer

- **Claude** (claude.ai, application) : depuis l'annuaire des plugins, ou en ajoutant le connecteur `https://tchin.tech/mcp/app` dans Paramètres, Connecteurs.
- **Claude Code** :

```text
/plugin marketplace add wintg/tchin-plugin
/plugin install tchin@wintg
```

- **Cursor** : [![Ajouter à Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://tchin.tech/mcp/cursor) ou, à la main, dans `~/.cursor/mcp.json` :

```json
{ "mcpServers": { "tchin": { "url": "https://tchin.tech/mcp" } } }
```

## Liens

- Documentation : https://doc.wintg.group/tchin/ia/
- Conditions : https://tchin.tech/legal#cgu
- Assistance : https://tchin.tech/contact ou contact@wintg.group

Licence MIT, voir [LICENSE](LICENSE).
