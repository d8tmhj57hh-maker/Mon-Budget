# Cashorizon

## Vue d'ensemble

Cashorizon est une application web de gestion de budget personnel **privacy-first**.
Toute la logique vit dans **un seul fichier `index.html`** hébergé sur GitHub Pages.
Aucune donnée financière ne transite par un serveur tiers : le backend est
Google Sheets/Drive (par utilisateur, via OAuth). C'est le différenciateur
marketing central du produit — ne jamais introduire de dépendance qui romprait
cette promesse (pas d'appel serveur externe, pas d'analytics tiers, pas d'appel
IA sur des données utilisateur).

Utilisateurs actuels : Aurélie (développeuse) et Florence (sa mère, saisie
manuelle de relevés papier).

## Architecture

- **Toute la logique applicative vit dans `index.html`** (HTML + CSS + JS
  inline). Pas de build, pas de pipeline npm/webpack.
- **Structure réelle du dépôt** (ne pas supposer qu'il n'y a qu'`index.html`) :
  - `index.html` — l'application (logique, UI, tout le JS)
  - `manifest.json` — manifeste PWA
  - `confidentialite.html` — page de politique de confidentialité
  - `icon-192.png`, `icon-512.png` — icônes PWA
  - `pdf.min.js`, `pdf.worker.min.js` — PDF.js embarqué localement (pas de
    CDN pour ces fichiers, ils sont dans le dépôt)
  - `README.md`
- **Backend** : Google Sheets API + Google Drive, auth via OAuth (app en Test
  Mode — vérification Google nécessite un domaine possédé, différé pour
  raisons de coût).
- **Déploiement** : GitHub Pages.

## Commandes

- Valider le JS avant toute livraison : `node --check <fichier extrait>`
- Vérifier les compteurs de balises HTML ouvrantes/fermantes avant livraison
- Pas de `npm install`, pas de serveur de dev — tout se teste en ouvrant
  `index.html` dans un navigateur ou en extrayant le JS pour `node --check`

## Conventions et pièges connus

- **Écritures Sheets** : toujours utiliser `RAW`, jamais `USER_ENTERED`
  (tronque les virgules décimales françaises, ex. 183,23 → 183)
- **Init asynchrone** : `peuplerCat()` doit être appelé après que
  `chargerTout()` soit terminé, pas avant (sinon catégories manquantes par
  race condition)
- **Prévisionnel** : le calcul du solde du mois doit utiliser les totaux
  mixtes réel/programmé (`totRevMois`, `totGlobalDep`) directement — ne pas
  repartir de sommes réelles partielles
- **PDF parsing (Crédit Agricole)** : les PDF ont une vraie couche texte, pas
  besoin d'IA/serveur. La position x distingue débit/crédit de façon fiable.
  Regrouper les lignes par coordonnée y avec tolérance `Math.abs(k-y) <= 2`.
  Le symbole `¨` casse les regex sur les montants, à nettoyer en amont.
  `pdfjs-dist@3` (legacy) reproduit le mieux le comportement navigateur.
- **CSP** : ne jamais réintroduire de Content-Security-Policy — bloque
  silencieusement l'init de `gapi.client` (boucle de login infinie déjà
  rencontrée)
- **Token Google** : ne jamais stocker en localStorage, s'appuyer sur la
  reconnexion silencieuse via cookies de session

## Style et sécurité

- Toute donnée utilisateur affichée passe par la fonction de sanitization
  `esc()` (protection XSS)
- Modals de confirmation custom, jamais de `confirm()`/`alert()` natifs
- UI fintech moderne : accent violet `#a78bfa`, teal/coral pour positif/
  négatif, contraste WCAG AA en thème clair et sombre, radius 12px

## Approche de travail

- **Mockup avant implémentation** : pour toute évolution visuelle
  significative, produire un aperçu HTML autonome avant de toucher au fichier
  de prod
- **Root cause, pas symptôme** : en cas de bug signalé, corriger la cause
  réelle (souvent dans la logique de calcul ou l'ordre d'init), pas un
  correctif de surface
- **Positionnement produit** : toujours comme un produit fini et prêt, jamais
  "en développement" ou "en test"
- Préférence forte contre les coûts récurrents à ce stade
