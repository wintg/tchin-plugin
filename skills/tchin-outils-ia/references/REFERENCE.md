# Outils pour développeurs et IA

Pour aller plus vite : la connaissance de Tchin dans votre assistant de code, des méthodes toutes prêtes pour ChatGPT et Claude, les fonctions de Tchin pour vos agents d'IA, et Tchin dans votre terminal.

| Outil | Pour qui | Ce qu’il fait |
|---|---|---|
| `Agent Skills` | Développeurs avec Claude Code, Cursor, Codex, Copilot, Gemini… | l’assistant connaît Tchin et écrit une intégration correcte |
| `Skills pour votre IA` | Marchands et particuliers dans Claude, ChatGPT… | des méthodes toutes prêtes avec le serveur MCP : comptabilité, relances, vente sur WhatsApp, bon usage |
| `Agent Toolkit` | Vous construisez votre propre agent (WhatsApp, site, support) | les fonctions de l’API Tchin, prêtes pour OpenAI, Vercel AI SDK, LangChain |
| `CLI tchin` | Développeurs, équipes techniques, scripts | créer et suivre paiements, abonnements, versements, vérifications depuis le terminal |
| `Serveur MCP` | Clients, marchands et particuliers dans ChatGPT, Claude… | payer dans la conversation ; gérer tout son compte : historique, relevé PDF, liens, envois et retraits confirmés avec le Tchin PIN (voir Tchin dans votre IA) |

## Agent Skills

Des fiches de connaissance installées dans votre projet : une fiche d'entrée dit à l'assistant laquelle lire selon la demande (paiements, abonnements, versements, identité, webhooks, MCP…), et chaque fiche a sa référence complète (champs, code dans chaque langage, réponses). L'assistant lit d'abord la fiche courte, la référence seulement si besoin.

```texte
curl -fsSL https://tchin.tech/outils/tchin-skills.js -o tchin-skills.js
node tchin-skills.js add                                   # détecte vos assistants
node tchin-skills.js add --frameworks claude-code,cursor   # ou choisissez
node tchin-skills.js remove                                # tout retirer
```

| Assistant | Fiches | Consigne |
|---|---|---|
| `claude-code` | `.claude/skills/tchin/` | `CLAUDE.md` |
| `cursor` | `.cursor/tchin-skills/` | `.cursor/rules/tchin.mdc` |
| `codex, opencode, antigravity` | `.agents/skills/tchin/` (ou équivalent) | `AGENTS.md` |
| `vscode-copilot, copilot-cli` | `.github/skills/tchin/` | `.github/copilot-instructions.md` |
| `gemini-cli` | `.gemini/skills/tchin/` | `GEMINI.md` |

Ensuite, redémarrez l'assistant et demandez par exemple : « Intègre le paiement Tchin dans ce projet ». La consigne est ajoutée entre deux marqueurs : vos propres consignes ne sont pas touchées.

### Les fiches du paquet

| Fiche | Ce qu’elle apprend à l’assistant |
|---|---|
| `tchin` | la fiche d’entrée : laquelle lire selon la demande, et les règles de sécurité |
| `tchin-demarrer` | intégrer Tchin de zéro en 5 minutes |
| `tchin-cles` | les clés, le mode test ou réel, les erreurs et les limites |
| `tchin-paiements / tchin-moyens` | encaisser un paiement, ouvrir la feuille, suivre l’état ; les réseaux, les pays et les frais |
| `tchin-abonnements` | les paiements récurrents |
| `tchin-recevoir / tchin-versements-directs` | envoyer de l’argent à un client ; verser à un numéro ou une adresse connue |
| `tchin-kyc / tchin-webhooks` | la vérification d’identité ; les webhooks et leur signature |
| `tchin-ia-mcp / tchin-outils-ia` | le serveur MCP ; ces outils |
| `tchin-kits / tchin-reference-api / tchin-migration / tchin-faq` | les kits, tous les points d’entrée, le calendrier, les questions fréquentes |
| `tchin-tester` | **tester sans argent réel** : mode test, comptes test Mobile Money, paiement crypto simulé, webhooks sur votre machine, les cas à essayer |
| `tchin-avant-reel` | **la liste à vérifier avant de passer en réel** : compte, clés, `env`, webhook signé et idempotent, statuts, anciens appels, premier paiement réel |

Demandez par exemple : « Teste mon intégration Tchin sans argent réel » ou « Vérifie que mon intégration est prête pour le réel » : l'assistant suit la fiche et vous dit ce qui manque.

## Skills pour votre IA

Des méthodes toutes prêtes pour les utilisateurs de Tchin dans **Claude**, **ChatGPT** ou toute IA reliée au serveur MCP Tchin. Chaque skill dit à l'IA quels outils Tchin appeler, dans quel ordre, et ce qu'elle ne doit jamais faire. Elles ne contiennent aucune clé et ne donnent aucun accès : l'accès au compte passe toujours par la connexion à Tchin.

| Skill | Ce qu’elle fait | Télécharger |
|---|---|---|
| `tchin-bien-utiliser` | quel outil Tchin pour quelle demande, la connexion, et la règle d’or : l’argent qui sort se confirme sur tchin.tech avec le Tchin PIN, jamais dans la conversation | zip |
| `tchin-comptable` | le bilan d’une période (entrées, sorties, frais, par pays et par moyen), le relevé PDF, le rapprochement avec vos propres chiffres | zip |
| `tchin-relances` | trouver les abonnements en retard et les paiements échoués, puis rédiger des relances polies (WhatsApp, SMS, e-mail) avec le lien Tchin | zip |
| `tchin-vendre-whatsapp` | créer le lien de paiement d’un produit, écrire le statut et le message WhatsApp, voir qui a payé, arrêter le lien quand le stock est fini | zip |

Les quatre ensemble : tchin-skills-ia.zip.

### Les installer

| IA | Comment |
|---|---|
| `Claude` | Paramètres › Capacités › Skills › **Importer une skill**, puis choisissez le zip d’une skill. Ajoutez aussi le connecteur Tchin (voir Ajouter le connecteur). |
| `Claude Code` | Le plugin Tchin installe le serveur et les quatre skills : `/plugin marketplace add wintg/tchin-plugin` puis `/plugin install tchin@wintg` (github.com/wintg/tchin-plugin). Ou décompressez le zip dans `~/.claude/skills/` et ajoutez le serveur : `claude mcp add --transport http tchin https://tchin.tech/mcp`. |
| `ChatGPT` | Elles seront incluses dans l’application Tchin de l’annuaire de ChatGPT ; en attendant, demandez simplement « bilan d’octobre » ou « relance mes impayés » : les outils Tchin suffisent. |
| `Toute IA compatible MCP` | Le serveur les sert aussi comme **prompts** : `tchin-bien-utiliser`, `tchin-comptable`, `tchin-relances`, `tchin-vendre-whatsapp` (avec un argument `demande` facultatif), dans le menu des prompts de votre IA. |

```curl
curl -X POST https://tchin.tech/mcp \
  -H "Content-Type: application/json" \
  -d '{ "jsonrpc": "2.0", "id": 1, "method": "prompts/get",
        "params": { "name": "tchin-comptable", "arguments": { "demande": "le bilan d’octobre" } } }'
```

## Agent Toolkit

Les fonctions de l'API Tchin pour votre agent, par appel de fonctions : créer un paiement, un abonnement, un versement, une vérification d'identité, lire leur état et votre solde. Node 18+, sans dépendance, **côté serveur seulement**.

```js
const { TchinAgentToolkit } = require('./tchin-agent-toolkit');
const tchin = new TchinAgentToolkit(process.env.TCHIN_PUBLIC_KEY, process.env.TCHIN_PRIVATE_KEY, {
  env: 'test',          // 'live' pour de vrai
  sansVersement: true,  // pas d'argent sortant (conseillé pour un agent public)
});

// OpenAI Chat Completions
const r = await openai.chat.completions.create({ model: 'gpt-5', messages, tools: tchin.getTools() });
for (const appel of r.choices[0].message.tool_calls || []) {
  const resultat = await tchin.executer(appel.function.name, appel.function.arguments);
  messages.push({ role: 'tool', tool_call_id: appel.id, content: JSON.stringify(resultat) });
}

// OpenAI Agents SDK
const { Agent, tool } = require('@openai/agents');
const agent = new Agent({ name: 'Caisse', instructions: 'Tu aides les clients à payer.', tools: tchin.getAgentTools(tool) });

// Vercel AI SDK
const { tool: outil, jsonSchema } = require('ai');
const tools = tchin.getVercelTools(outil, jsonSchema);

// LangChain
const { tool: lcTool } = require('@langchain/core/tools');
const lcTools = tchin.getLangChainTools(lcTool);
```

| Outil | Ce qu’il fait |
|---|---|
| `tchin_creer_paiement` | un paiement : rend `token` et `payment_url` (le client paie lui-même) |
| `tchin_etat_paiement` | pending, completed, failed ou cancelled |
| `tchin_creer_abonnement` | un abonnement (monthly, quarterly, biannual, yearly) |
| `tchin_abonnement / tchin_resilier_abonnement` | lire / résilier un abonnement |
| `tchin_creer_versement` | envoyer de l’argent à un client ; il choisit où le recevoir (argent sortant : retiré avec `sansVersement`) |
| `tchin_versement / tchin_annuler_versement` | l’état d’un versement / l’annuler |
| `tchin_creer_verification / tchin_verification` | une vérification d’identité (75 F) et son résultat |
| `tchin_solde` | vos soldes par pays et en crypto |

Télécharger le toolkit · aussi sur `https://tchin.tech/outils/tchin-agent-toolkit.zip`.

## CLI tchin

```texte
curl -fsSL https://tchin.tech/outils/install.sh | sh        # macOS, Linux
irm https://tchin.tech/outils/install.ps1 | iex             # Windows (PowerShell)
```

```texte
tchin configurer --publique tchin_pk_... --secrete tchin_sk_... --test
tchin solde
tchin paiement creer --montant 5000 --description "Commande 42"
tchin paiement etat y20vcyegjk85bs
tchin abonnement creer --montant 10000 --offre "Pro" --intervalle monthly
tchin abonnement liste --statut active
tchin abonnement resilier sub_xxx --oui
tchin versement creer --montant 2000 --email client@exemple.com --oui
tchin kyc creer --reference client-42
tchin webhook verifier --fichier data.json
tchin mcp
```

- Les clés sont gardées dans `~/.tchin/config.json`, lisible par vous seul, ou lues dans `TCHIN_PUBLIC_KEY`, `TCHIN_PRIVATE_KEY` et `TCHIN_ENV`. `tchin config` les montre masquées.

- Mode par défaut : `test`. Ajoutez `--live` pour de vrai.

- Un versement (argent qui sort) et une résiliation demandent `--oui`.

- `--json` donne la réponse brute (pour vos scripts).

Les empreintes SHA-256 de tous les fichiers (CLI, toolkit, skills) : tchin.tech/outils/SHA256SUMS.txt.

---
Source : https://doc.wintg.group/tchin/outils-ia/
