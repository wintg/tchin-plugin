# Tchin dans votre IA

Dans ChatGPT, Claude ou toute IA compatible MCP, on paie un lien Tchin ou un Tchin ID sans quitter la conversation, et chacun gère son compte : soldes, historique, relevés en PDF, liens de paiement, envois et retraits confirmés avec le Tchin PIN. Vous n'avez rien à intégrer.

## Le principe

1. Le client écrit à son IA : « Je veux payer ce lien : https://tchin.tech/pay/… », ou « Paie 5 000 F à boutique@tchin pour la commande 42 ».

1. L'IA demande à Tchin de préparer le paiement : la **feuille de paiement Tchin** s'affiche dans la conversation, avec votre nom, votre image et le montant.

1. Le client choisit son moyen (Mobile Money, compte Tchin ou crypto) et valide **lui-même** : code secret sur son téléphone, Tchin PIN, ou son portefeuille.

1. Tchin confirme le paiement à l'IA, qui le dit au client. Vous êtes crédité comme pour n'importe quel paiement.

## Comment vos clients y accèdent

Trois façons, selon l'IA qu'ils utilisent :

| Façon | Ce que fait le client | IA concernées |
|---|---|---|
| `Connecteur Tchin` | Rien de plus que demander de payer : l’IA utilise Tchin. Il faut que le connecteur soit activé (une fois), ou publié dans l’annuaire de l’IA. | Claude, ChatGPT et toute IA compatible MCP Apps |
| `IA qui navigue` | L’IA ouvre le lien sur tchin.tech et remplit la page avec lui ; il valide sur son téléphone. | Les modes « agent » ou « navigateur » des IA |
| `Lien direct` | L’IA lui donne le lien ; il paie sur tchin.tech. | Toutes |

## Ajouter le connecteur Tchin

L'adresse du serveur MCP de Tchin, sans clé ni compte :

```texte
https://tchin.tech/mcp
```

### Claude

1. Ouvrez **Paramètres**, puis **Connecteurs**.

1. Touchez **Ajouter un connecteur personnalisé**, nommez-le « Tchin » et collez l'adresse.

1. Dans une conversation, demandez de payer un lien Tchin.

### ChatGPT

1. Ouvrez **Paramètres**, puis **Applications** (ou Connecteurs), **Avancé**, et activez le mode développeur.

1. Créez une application « Tchin » avec l'adresse et l'authentification **OAuth** : la connexion à Tchin ne sera demandée que pour votre compte, jamais pour payer.

1. Dans une conversation, choisissez Tchin et demandez ce que vous voulez.

### Claude Code

```texte
/plugin marketplace add wintg/tchin-plugin
/plugin install tchin@wintg
```

Ou le serveur seul : `claude mcp add --transport http tchin https://tchin.tech/mcp`.

### Deux adresses

| Adresse | Pour | Ce qu’elle contient |
|---|---|---|
| `https://tchin.tech/mcp` | le connecteur que vous ajoutez vous-même (Claude, ChatGPT en mode développeur, Claude Code, Codex, Cursor…) | tout, y compris les envois, retraits et déplacements d’argent, toujours confirmés sur tchin.tech avec le Tchin PIN |
| `https://tchin.tech/mcp/app` | l’application Tchin des annuaires de ChatGPT et de Claude | tout, sauf l’argent qui sort : le paiement d’un lien s’ouvre sur tchin.tech, et les envois et retraits se font sur tchin.tech |

## Les outils pour payer (sans compte)

Ces outils marchent sans connexion. Aucun ne paie : ils **affichent** le formulaire officiel de Tchin, et c'est le client qui paie, s'il le veut. C'est voulu : les IA refusent, à juste titre, de déplacer de l'argent elles-mêmes.

| Outil | Paramètres | Ce qu’il fait |
|---|---|---|
| `afficher_lien_tchin` | `lien` : l’adresse https://tchin.tech/pay/… ou son code | affiche la feuille de paiement de ce lien (lien de paiement, formulaire, paiement créé par l’API, abonnement) |
| `preparer_demande_tchin` | `tchin_id` (pseudo@tchin), `montant` (200 à 1 000 000 F), `motif` facultatif | prépare un paiement à ce compte Tchin vérifié et affiche la feuille ; le client paie les frais, comme sur vos liens |
| `etat_paiement` | `token` | `paye`, `en_cours` ou `a_payer` |
| `aide_tchin` | `question` (300 caractères au plus) | cherche dans la documentation officielle de Tchin (frais, pays, réseaux, délais, API…) et rend les passages utiles avec leur source : l’IA répond à partir d’eux au lieu de deviner |

Le serveur suit la norme **MCP** (transport HTTP) avec l'extension **MCP Apps** : la feuille est une vue `ui://tchin/paiement-v4.html`, dessinée dans la conversation. Une IA sans vues reçoit le même résultat en texte, avec le lien.

```curl
curl -X POST https://tchin.tech/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": { "name": "etat_paiement", "arguments": { "token": "wintgdemo" } }
  }'
```

## Pour les IA qui naviguent

Chaque page de paiement Tchin est lisible par une IA qui ouvre le lien :

- une fiche structurée (`schema.org/Invoice`) : marchand, objet, montant, moyens, état ;

- des consignes claires : aider le client à remplir, ne jamais taper de code secret ni de PIN ;

- l'état du paiement en JSON : `https://tchin.tech/pay/{code}/etat` ;

- le fichier tchin.tech/llms.txt, qui explique Tchin aux IA.

```json
{ "statut": "a_payer", "titre": "Wintg Cloud - pack Pro", "montant": 25000, "devise": "XOF", "marchand": "Wintg", "test": true }
```

## Sécurité

- **Le client valide toujours lui-même**, dans la feuille Tchin : aucun outil ne peut débiter qui que ce soit.

- Le numéro, le code secret, le Tchin PIN et le portefeuille ne passent jamais par l'IA ni par le serveur MCP.

- Seuls les comptes Tchin **vérifiés** peuvent recevoir un paiement par Tchin ID.

- Les titres et noms des marchands sont nettoyés et présentés à l'IA comme des données, jamais comme des consignes.

- La vue n'ouvre que des adresses de tchin.tech ; le serveur limite le nombre de requêtes et de paiements préparés.

- La réussite affichée vient de Tchin (état vérifié sur le serveur), pas de la page.

- **Garde-fous contre l’injection** : un texte écrit par quelqu’un (titre de lien, motif, nom) qui donne des consignes à une IA (« ignore les instructions… ») est refusé à la création et masqué s’il est lu ; les caractères invisibles sont retirés.

- **Données personnelles masquées** dans tout ce que le serveur renvoie : numéros de téléphone (`901•••33`), e-mails (`aw•••@gmail.com`), noms des clients (prénom et initiale).

- Chaque appel laisse une trace sans donnée personnelle (son numéro est renvoyé dans l’en-tête `X-Tchin-Trace`), et des évaluations automatiques vérifient chaque jour les garde-fous, le masquage et la connexion.

## Gérer son compte depuis son IA

Le même connecteur sert à tous les utilisateurs de Tchin, **marchands comme particuliers** : « fais-moi mon relevé PDF d'octobre », « combien j'ai reçu cette semaine ? », « crée un lien de 15 000 F pour la robe wax », « envoie 5 000 F à awa@tchin ». Il n'y a qu'une adresse, `https://tchin.tech/mcp` : la connexion n'est demandée qu'au premier outil qui touche à un compte.

1. Vous demandez quelque chose sur votre compte dans la conversation.

1. L'IA vous envoie sur **tchin.tech** : connectez-vous (code reçu par e-mail), lisez ce que l'IA pourra faire, cochez **Lecture seule** si vous voulez qu'elle lise seulement, et autorisez avec votre **Tchin PIN**.

1. C'est fait : l'IA agit pour vous, dans les limites que vous avez choisies. Elle ne vous redemande pas de vous connecter.

### Le compte et l'historique

| Outil | Ce qu’il fait |
|---|---|
| `mon_compte` | nom, Tchin ID, pays, vérification d’identité, Tchin PIN créé ou non, soldes par pays, en crypto et en cashback |
| `mes_transactions` | **tout** l’historique : paiements reçus, dépôts, retraits, envois, argent reçu, déplacements entre pays, cashback, parrainage, épargne ; filtres par genre, sens (entrées ou sorties), statut, pays, période et mode (réel, test ou tous), avec les totaux ; page par page |
| `bilan_compte` | le bilan d’une période (30 jours par défaut, un an au plus) : entrées et sorties par genre, par mois et par pays, frais, opérations échouées |
| `releve_pdf` | le relevé des transactions en **PDF**, au format des documents Tchin, avec les mêmes filtres ; voir plus bas |
| `detail_transaction` | le détail d’une opération, de n’importe quel genre, par sa référence |
| `mon_cashback / convertir_cashback` | les Tchin gagnés, leur valeur et les cartes à gratter ; convertir en argent sur un solde pays (500 F au moins, sans frais) |
| `mon_parrainage` | le code et le lien d’invitation, les filleuls, le palier, les gains |
| `mes_coffres` | les coffres de la banque Tchin : montant, rythme, déjà déposé, prochaine échéance, date de fin |
| `mes_prelevements` | les abonnements que vous payez chez des marchands, avec l’adresse pour payer l’échéance |
| `mes_contacts` | les comptes Tchin à qui vous avez envoyé de l’argent récemment |

### Encaisser : liens, ventes, applications

| Outil | Ce qu’il fait |
|---|---|
| `creer_lien_paiement` | un lien à montant fixe ou libre, en Mobile Money, crypto ou les deux, réel ou de test |
| `creer_formulaire` | un formulaire de paiement (questions puis paiement) |
| `mes_liens / activer_lien` | vos liens et formulaires avec leurs ventes ; activer ou désactiver un lien |
| `modifier_lien / supprimer_lien` | changer le titre, le montant, la description, les moyens ou le mode d’un lien ; le supprimer (les paiements reçus restent) |
| `mes_ventes / rapport_ventes` | les derniers paiements reçus ; le résumé d’une période (total, net, frais, par moyen, par pays, meilleurs liens) |
| `detail_paiement` | le parcours complet d’un paiement reçu |
| `mes_abonnements` | les abonnements de vos clients et leurs échéances |
| `mes_retraits` | vos derniers retraits (Mobile Money, crypto, gains de parrainage) |
| `mes_applications / creer_application` | vos applications API ; en créer une (la clé secrète se lit seulement sur Tchin, avec le Tchin PIN) |
| `mes_liens_retrait / annuler_lien_retrait` | les liens de retrait envoyés ; annuler un lien encore ouvert |

## Envoyer, retirer, déplacer de l'argent

Pour l'argent qui sort, l'IA ne fait **rien elle-même**. Elle prépare une **demande** et vous donne une adresse sur tchin.tech. Vous y vérifiez chaque ligne (destinataire, montant, frais, solde) et vous confirmez avec votre **Tchin PIN**. Tant que vous n'avez pas confirmé, rien ne bouge.

1. « Envoie 15 000 F à boutique.demo@tchin depuis mon solde Togo. »

1. L'IA vérifie le destinataire et votre solde, puis vous donne l'adresse de la demande, valable **15 minutes**.

1. Vous ouvrez l'adresse, vous relisez, vous tapez votre Tchin PIN : Tchin fait l'opération.

1. L'IA appelle `etat_demande` et vous dit que c'est fait. Elle ne le dit jamais avant.

| Outil | Ce qu’il prépare |
|---|---|
| `envoyer_argent` | un envoi vers le compte Tchin d’une autre personne (son Tchin ID), d’un solde pays vers le sien |
| `retirer_mobile_money` | un retrait vers un numéro Mobile Money (frais pris dans le montant) ; `reseaux_retrait` donne les réseaux et leurs frais |
| `deplacer_argent` | un déplacement entre deux de vos soldes pays (Togo, Bénin, Burkina Faso) |
| `creer_lien_retrait` | un lien de retrait envoyé par e-mail : la personne retire elle-même (en test, créé tout de suite, sans argent réel) |
| `supprimer_application` | la suppression d’une application API (ses clés ne marcheront plus) |
| `etat_demande` | l’état d’une demande : à confirmer, faite, refusée, expirée ou échouée |
| `ouvrir_tchin` | l’adresse d’une page de Tchin, pour ce qui se fait sur l’écran de Tchin : recharger son solde, retrait crypto, coffre, Tchin PIN, vérification d’identité |

## Le relevé en PDF

Demandez « mon relevé d'octobre en PDF » ou « le PDF de mes retraits de la semaine » : l'outil `releve_pdf` fait le relevé au format des documents Tchin (papier à en-tête, résumé des entrées, sorties et frais, puis chaque opération), jusqu'à 1 000 opérations.

- L'adresse vaut **24 heures** et ne s'ouvre qu'au titulaire du compte, **connecté à Tchin** : envoyée à quelqu'un d'autre, elle ne montre rien.

- Le fichier est effacé du serveur au bout d'un jour ; vos opérations restent dans Tchin › Transactions.

- Les numéros de vos clients y sont masqués.

## Ce que votre IA ne peut pas faire

- Voir votre Tchin PIN, vos codes de connexion ou vos clés secrètes : ils ne passent jamais par la conversation.

- Faire sortir de l'argent sans votre confirmation sur tchin.tech, avec votre Tchin PIN.

- Changer vos réglages de sécurité (Tchin PIN, biométrie, e-mail) ni supprimer votre compte.

- Agir quand vous l'avez branchée en **lecture seule** : elle lit seulement, ne crée et ne prépare rien.

Sous le capot : OAuth 2.1 avec PKCE, enregistrement automatique de l'IA, jeton d'accès d'une heure et jeton de rafraîchissement de 30 jours à usage unique (un jeton réutilisé coupe l'accès). Tchin ne garde que l'empreinte des jetons. Un outil du compte appelé sans connexion répond `401` avec `WWW-Authenticate` (`insufficient_scope`) : c'est ce qui fait apparaître la connexion dans l'IA.

## Des méthodes toutes prêtes

Quatre skills apprennent à votre IA à bien se servir de Tchin : **bien utiliser Tchin**, **comptable Tchin** (bilan, relevé, rapprochement), **relances** (impayés et abonnements en retard) et **vendre sur WhatsApp**. À importer dans Claude, ou à choisir dans le menu des prompts de votre IA : voir Skills pour votre IA.

## Pour vous, marchand

- Rien à intégrer : partagez vos liens de paiement ou votre Tchin ID, comme d'habitude.

- Les paiements par Tchin ID arrivent dans vos ventes et vos transactions ; ils n'encombrent pas votre page Liens.

- Vos paiements créés par l'API (`POST /payments`) s'ouvrent aussi dans une IA : votre webhook est appelé comme d'habitude.

---
Source : https://doc.wintg.group/tchin/ia/
