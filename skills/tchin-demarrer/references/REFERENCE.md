# Démarrer en 5 minutes

Créez votre application, récupérez vos clés, puis encaissez un premier paiement en mode test, de votre serveur jusqu'au webhook : rien n'est débité.

## Avant de commencer

- Un **compte Tchin** avec son **identité vérifiée** (une photo de la pièce et un selfie, depuis le téléphone, dans Profil). C'est elle qui ouvre la création d'applications et les comptes test ; la vérification prend en général quelques minutes.

- Un **serveur** qui peut faire un appel HTTPS (PHP, Node.js, Python, Java, Go… ou un simple `curl`). Les clés y restent.

- Pour recevoir les webhooks, une **adresse HTTPS publique** sur ce serveur (`https://votre-site/webhook-tchin`). En développement sur votre ordinateur, un tunnel (ngrok, Cloudflare Tunnel…) suffit.

## 1. Créer votre application

Une application, c'est votre site ou votre service vu par Tchin : son nom et son image (vos clients les voient en payant), les moyens acceptés, qui paie les frais, et une paire de clés. Créez-en une par site ou par service : chacune a ses clés, ses réglages et ses statistiques.

1. Sur tchin.tech, ouvrez la page API de votre compte et touchez **Créer**.

1. **Identité** : une image carrée (votre logo), un nom (celui que verront vos clients, par exemple « Wintg Cloud ») et une description courte.

1. **Moyens** : cochez ce que vos clients pourront utiliser : Mobile Money (Togo, Bénin, Burkina Faso), Crypto (USDT, TRX, BNB, SOL, GRAM), compte Tchin. Au moins un.

1. **Frais** : à l'encaissement, le client ou vous ; au versement, le destinataire ou vous. La page vous montre l'effet sur un exemple chiffré.

1. **Webhook** (facultatif) : l'adresse où Tchin préviendra votre serveur par défaut. Vous pourrez aussi la donner à chaque appel (`callback_url`). Touchez **Créer l'application**.

## 2. Récupérer vos clés

Touchez votre application, puis **Voir les clés**. Tchin vous demande votre Tchin PIN (ou votre biométrie), à chaque fois. Copiez les deux clés dans la configuration de votre serveur, de préférence dans des variables d'environnement :

```texte
# .env de votre serveur (jamais dans le dépôt Git, jamais dans une page)
TCHIN_PUBLIC_KEY=tchin_pk_...
TCHIN_PRIVATE_KEY=tchin_sk_...
```

| Clé | Où la mettre | Rôle |
|---|---|---|
| `tchin_pk_…` | votre serveur | la clé publique, envoyée dans l’en-tête TCHIN-PUBLIC-KEY |
| `tchin_sk_…` | votre serveur seulement | la clé secrète : en-tête TCHIN-PRIVATE-KEY, et la vérification des webhooks |

Vous changerez plus tard l'image, les moyens et les frais avec **Modifier** : vos clés ne changent pas, votre code non plus.

## 3. Créer un paiement (serveur)

Quand votre client clique sur « Payer », votre page demande à **votre serveur** de créer le paiement. Votre serveur appelle Tchin avec le montant, une description et l'adresse de votre webhook. Avec `"env": "test"`, rien n'est débité.

```curl
curl -X POST https://tchin.tech/api/v1/payments \
  -H "TCHIN-PUBLIC-KEY: $TCHIN_PUBLIC_KEY" \
  -H "TCHIN-PRIVATE-KEY: $TCHIN_PRIVATE_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "amount": 25000,
    "description": "Wintg Cloud - pack Pro",
    "callback_url": "https://wintg.group/webhook-tchin",
    "env": "test"
  }'
```

```php
require 'tchin-php/src/Tchin.php';

$tchin = new Tchin\Client(getenv('TCHIN_PUBLIC_KEY'), getenv('TCHIN_PRIVATE_KEY'));
$p = $tchin->creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', ['env' => 'test']);

// Gardez le token avec votre commande : c'est lui qui reviendra dans le webhook (champ « reference »).
enregistrerCommande($commandeId, $p['token']);

header('Content-Type: application/json');
echo json_encode(['token' => $p['token']]); // pour votre page
```

```node
const { Tchin } = require('./tchin-node');

const tchin = new Tchin(process.env.TCHIN_PUBLIC_KEY, process.env.TCHIN_PRIVATE_KEY);

app.post('/creer-paiement', async (req, res) => {
  const p = await tchin.creerPaiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', { env: 'test' });
  await enregistrerCommande(req.body.commande, p.token); // le token reviendra dans le webhook
  res.json({ token: p.token });                           // pour votre page
});
```

```python
import os
from tchin import Tchin

tchin = Tchin(os.environ['TCHIN_PUBLIC_KEY'], os.environ['TCHIN_PRIVATE_KEY'])

@app.post('/creer-paiement')
def creer_paiement():
    p = tchin.creer_paiement(25000, 'Wintg Cloud - pack Pro', 'https://wintg.group/webhook-tchin', env='test')
    enregistrer_commande(request.json['commande'], p['token'])  # le token reviendra dans le webhook
    return {'token': p['token']}                                 # pour votre page
```

```java
Tchin tchin = new Tchin(System.getenv("TCHIN_PUBLIC_KEY"), System.getenv("TCHIN_PRIVATE_KEY"));
String json = tchin.creerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", Map.of("env", "test"));
// lisez « token » avec votre bibliothèque JSON (Jackson, Gson), gardez-le avec la commande et rendez-le à votre page
```

```go
c := tchin.Nouveau(os.Getenv("TCHIN_PUBLIC_KEY"), os.Getenv("TCHIN_PRIVATE_KEY"))

http.HandleFunc("/creer-paiement", func(w http.ResponseWriter, r *http.Request) {
	p, err := c.CreerPaiement(25000, "Wintg Cloud - pack Pro", "https://wintg.group/webhook-tchin", map[string]any{"env": "test"})
	if err != nil { http.Error(w, err.Error(), 502); return } // err.Error() : le message de Tchin
	json.NewEncoder(w).Encode(map[string]any{"token": p["token"]})
})
```

Tchin répond :

```json
{
  "success": true,
  "token": "jtkkbphr7azsn3",
  "payment_url": "https://tchin.tech/pay/jtkkbphr7azsn3",
  "env": "test"
}
```

## 4. Ouvrir la feuille (page)

Ajoutez le kit web (une ligne de script, servie par Tchin), puis ouvrez la feuille avec le token rendu par votre serveur :

```html
<button id="payer">Payer avec Tchin</button>

<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  document.getElementById('payer').onclick = async function () {
    // 1. votre serveur crée le paiement (étape 3) et rend le token
    const r = await fetch('/creer-paiement', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ commande: 'WG-1042' })
    }).then((x) => x.json());

    // 2. la feuille s'ouvre par-dessus votre page
    TchinPay.open({
      token: r.token,
      onSuccess: function (e) { location.href = '/merci?commande=WG-1042'; }, // affichage seulement
      onPending: function (e) { /* le client valide sur son téléphone */ },
      onFailed: function (e) { /* paiement refusé : il peut réessayer dans la feuille */ },
      onClose: function (e) { /* e.reason : closed | success | replaced */ }
    });
  };
</script>
```

## 5. Recevoir le résultat (webhook)

Tchin appelle votre `callback_url` à chaque changement d'état, en `POST` de formulaire, avec un champ `data`. Vérifiez la signature, retrouvez la commande par `reference` (le token du paiement), vérifiez le montant, puis livrez si `status` vaut `completed` :

```php
$d = $tchin->verifierWebhook($_POST['data'] ?? []);
if (! $d) { http_response_code(401); exit; }            // signature fausse ou trop ancienne

$commande = trouverCommandeParToken($d['reference']);
if ($commande && $d['status'] === 'completed' && ! $commande->payee) {
    $commande->marquerPayee();                          // une seule fois, même si le webhook revient
}
http_response_code(200);
```

```node
app.post('/webhook-tchin', express.urlencoded({ extended: true }), async (req, res) => {
  const d = tchin.verifierWebhook(req.body.data);
  if (!d) return res.sendStatus(401);                   // signature fausse ou trop ancienne

  const commande = await trouverCommandeParToken(d.reference);
  if (commande && d.status === 'completed' && !commande.payee) await commande.marquerPayee();
  res.sendStatus(200);
});
```

```python
@app.post('/webhook-tchin')
def webhook_tchin():
    data = {k[5:-1]: v for k, v in request.form.items() if k.startswith('data[')}
    d = tchin.verifier_webhook(data)
    if not d:
        abort(401)                                        # signature fausse ou trop ancienne
    commande = trouver_commande_par_token(d['reference'])
    if commande and d['status'] == 'completed' and not commande.payee:
        commande.marquer_payee()
    return '', 200
```

```java
// Spring : @PostMapping("/webhook-tchin") public ResponseEntity<?> w(@RequestParam Map<String,String> form)
Map<String, String> data = new HashMap<>();
form.forEach((k, v) -> { if (k.startsWith("data[")) data.put(k.substring(5, k.length() - 1), v); });
if (!tchin.verifierWebhook(data)) return ResponseEntity.status(401).build();
if ("completed".equals(data.get("status"))) livrerUneFois(data.get("reference"));
return ResponseEntity.ok().build();
```

```go
r.ParseForm()
if !c.VerifierWebhook(r.PostForm) { http.Error(w, "signature", 401); return }
if r.PostForm.Get("data[status]") == "completed" { livrerUneFois(r.PostForm.Get("data[reference]")) }
w.WriteHeader(200)
```

## 6. Tester

En mode test, la feuille affiche Mode test. Payez avec les **comptes test** donnés sur la page API de Tchin (bouton **Comptes test**, une fois votre identité vérifiée) :

| Moyen | Comment payer en test |
|---|---|
| `Mobile Money` | pays Togo, le numéro du compte test, puis son mot de passe : la feuille le demande à la place du code secret. |
| `Crypto` | choisissez une crypto, puis touchez **Simuler le paiement** : aucun virement, aucun réseau. |
| `Compte Tchin` | votre propre Tchin ID et votre Tchin PIN : le PIN est vérifié, rien ne bouge sur les soldes. |

Essayez au moins ces trois cas : un paiement réussi (le webhook arrive avec `completed` et `"mode": "test"`), un paiement refusé (`failed`), et une feuille fermée sans payer (rien n'arrive, le paiement reste `pending`).

## 7. Passer en réel

Quand tout marche, remplacez `"env": "test"` par `"env": "live"` : les mêmes clés, le même code, de vrais paiements. Avant, parcourez la liste de Passer en production.

| Étape | Fait quand |
|---|---|
| `Application` | créée, avec son image, ses moyens et ses frais |
| `Clés` | dans les variables d’environnement du serveur, nulle part ailleurs |
| `Paiement` | créé par le serveur, token gardé avec la commande |
| `Feuille` | ouverte avec `TchinPay.open({ token })` |
| `Webhook` | signature vérifiée, livraison sur `completed` seulement, une seule fois |
| `Test` | réussi, refusé et fermé essayés en mode test |

La suite : les paiements en détail, les moyens, les webhooks.

---
Source : https://doc.wintg.group/tchin/demarrer/
