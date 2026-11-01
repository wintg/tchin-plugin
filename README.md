# Tchin pour Claude

[Tchin](https://tchin.tech) est une plateforme de paiement d'Afrique de l'Ouest (Togo, Bénin, Burkina Faso) : Mobile Money, compte Tchin et crypto, en francs CFA. Ce plugin relie Claude à Tchin : payer un lien Tchin sur tchin.tech, lire son compte, faire un relevé PDF, créer des liens de paiement, relancer les impayés et vendre sur WhatsApp. Tchin est une solution de [Wintg](https://wintg.group), à Lomé.

*English summary: Tchin is a West African payment platform (Togo, Benin, Burkina Faso). This plugin connects Claude to Tchin's MCP server and adds four French-language skills: using Tchin well, bookkeeping and PDF statements, payment reminders, and selling on WhatsApp with payment links. Payments always happen on tchin.tech; the plugin never sends, withdraws or transfers money.*

## Ce que contient le plugin

| Élément | Rôle |
|---|---|
| Serveur MCP `tchin` | `https://tchin.tech/mcp/app`, l'édition des annuaires du serveur MCP de Tchin |
| Skill `tchin-bien-utiliser` | quel outil Tchin pour quelle demande, la connexion, les règles de sécurité |
| Skill `tchin-comptable` | le bilan d'une période, le relevé PDF, les frais, le rapprochement avec vos chiffres |
| Skill `tchin-relances` | les abonnements en retard et les paiements échoués, des relances polies avec le lien Tchin |
| Skill `tchin-vendre-whatsapp` | le lien de paiement d'un produit, le message et le statut WhatsApp, qui a payé |

## Les outils du serveur

- **Sans compte** : `afficher_lien_tchin`, `preparer_demande_tchin` (affichent le lien officiel avec un bouton qui ouvre la page de paiement sur tchin.tech) et `etat_paiement`.
- **Avec le compte de l'utilisateur** (connexion OAuth sur tchin.tech, demandée au premier usage) : soldes, historique complet, bilan, relevé PDF, liens de paiement et formulaires, ventes, abonnements, prélèvements, cashback, parrainage, coffres, applications API, et `ouvrir_tchin` pour les pages de Tchin.
- **Jamais** : envoyer, retirer, déplacer ou transférer de l'argent ni de la crypto. L'utilisateur le fait lui-même sur tchin.tech.

## Données et sécurité

- Le plugin n'exécute aucun programme sur votre machine : il contient des fiches (skills) et l'adresse du serveur MCP de Tchin.
- Les seules données échangées le sont entre Claude et `https://tchin.tech`, et seulement quand un outil Tchin est appelé.
- Le serveur ne demande jamais de Tchin PIN, de code secret Mobile Money, de code de connexion ni de clé secrète dans la conversation. Les numéros des clients sont masqués et leurs e-mails ne sont pas transmis.
- L'accès se coupe à tout moment dans Tchin, page API, rubrique Serveur MCP. Les relevés PDF ne s'ouvrent qu'au titulaire du compte, connecté à Tchin, et sont effacés au bout de 24 heures.
- Politique de confidentialité : https://tchin.tech/legal#confidentialite (section 3.14, assistants d'intelligence artificielle connectés).

## Installer

- **Claude** (claude.ai, application) : depuis l'annuaire des plugins, ou en ajoutant le connecteur `https://tchin.tech/mcp/app` dans Paramètres, Connecteurs.
- **Claude Code** :

```text
/plugin marketplace add wintg/tchin-plugin
/plugin install tchin@wintg
```

## Liens

- Documentation : https://doc.wintg.group/tchin/ia/
- Conditions : https://tchin.tech/legal#cgu
- Assistance : https://tchin.tech/contact ou contact@wintg.group

Licence MIT, voir [LICENSE](LICENSE).
