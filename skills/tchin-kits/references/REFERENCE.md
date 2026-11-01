# Kits à télécharger

Chaque kit contient le code, un exemple et la marche à suivre. Sans dépendance, à copier dans votre projet.

## Lequel prendre ?

Une intégration a toujours deux côtés :

- **Votre serveur** (PHP, Node.js, Python, Java, Go, ou deux appels HTTP dans n'importe quel langage) crée l'opération et vérifie les webhooks. Vos clés ne quittent jamais votre serveur.

- **Votre page ou votre application** ouvre la feuille avec le token : le kit web (une ligne de script) pour tous les sites, ou le kit de votre application mobile.

## Web

```html
<script src="https://tchin.tech/js/tchin-pay.js"></script>
<script>
  TchinPay.open({ token: tokenDePaiement });      // paiement ou abonnement
  TchinPay.receive({ token: tokenDeVersement });  // Recevoir
</script>
<script src="https://tchin.tech/js/tchin-kyc.js"></script>
<script>
  TchinKYC.open({ id: idDeVerification });        // vérification d'identité
</script>
```

## Serveur

| Kit | Fonctions |
|---|---|
| `tchin-php` | `creerPaiement`, `etatPaiement`, `creerAbonnement`, `abonnements`, `resilierAbonnement`, `creerVersement`, `versement`, `annulerVersement`, `creerVerification`, `verification`, `solde`, `verifierWebhook` |
| `tchin-node` | les mêmes, en JavaScript (promesses) |
| `tchin-python` | les mêmes, en `snake_case` (`creer_paiement`…) |
| `tchin-java` | les mêmes ; chaque appel rend le JSON de Tchin en texte |
| `tchin-go` | les mêmes, en `CamelCase` (`CreerPaiement`…), rendent `map[string]any` |

## Applications mobiles

Les kits Android, iOS, Flutter, React Native et Cordova ouvrent la même feuille dans une vue web, avec les mêmes rappels (`onSuccess`, `onPending`, `onClose`) et une fonction `receive` pour Recevoir. Les portefeuilles crypto et les applications de paiement s'ouvrent dans le système.

## Sans code

Bientôt : WordPress et WooCommerce, Shopify, Bubble.

---
Source : https://doc.wintg.group/tchin/kits/
