---
theme: default
title: REX Audits RGAA — Ce que 4 audits m'ont appris
---

# REX Audits RGAA

Préparation — présentation développeurs

---

# Sommaire

- Intro rapide
- Focus clavier
- Formulaires
- Composants JS & ARIA
- Exemples HTML / CSS / JS
- Contenus dynamiques & lecteurs d'écran
- Conseils pratiques (dev)
- Conclusion : 4 choses à retenir

---

# Introduction

- Contexte : retours de 4 audits RGAA.
- Rappel : RGAA = WCAG-fr, 106 critères testables en binaire.
- Objectif : points concrets pour développeurs.

---

# Théorique RGAA

- **RGAA** = Référentiel Général d'Amélioration de l'Accessibilité (déclinaison française des **WCAG 2.1**).
- **Obligation légale** : sites publics et, selon seuils, certaines entreprises privées (déclaration de conformité requise).
- **Structure** : 106 critères répartis en 13 thématiques (images, couleurs, formulaires, navigation, etc.).
- **Chaque critère** = test binaire (conforme / non conforme) — audit actionnable.
- **Résultats** : taux de conformité, obligation de publication de la déclaration, plan pluriannuel + action annuelle.
- **Pour le dev** : le RGAA demande majoritairement d'utiliser correctement le HTML/ARIA — c'est une check-list.

---

# Thèmes

- Sémantique HTML (éviter le `div`/`span` roi)
- Gestion du focus clavier (ordre, visibilité, pièges)
- Formulaires : labels, erreurs, aides à la saisie
- Composants JS "faits maison" vs ARIA APG
- Contrastes & couleurs dans les design systems
- Contenus dynamiques (live regions, focus management)
- Responsive / zoom 200% et reflow
- Alternatives textuelles (images, icônes, boutons icon-only)

---

## Problèmes récurrents & causes

| Problème récurrent                                    | Cause typique                                                                                    |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Boutons/liens non accessibles au clavier              | `<div onclick>` au lieu de `<button>`/`<a>`                                                      |
| Focus invisible                                       | `outline: none` global dans le CSS reset, jamais remplacé                                        |
| Erreurs de formulaire non annoncées                   | Message inséré en DOM sans lien avec le champ (`aria-describedby` absent), pas de `role="alert"` |
| Contenu non lu par un lecteur d'écran après action JS | Pas de `aria-live`, focus non déplacé après ouverture modale/mise à jour                         |
| Images décoratives lues par le lecteur d'écran        | `alt` manquant plutôt que `alt=""`                                                               |
| Composants custom inutilisables au clavier            | Réinvention d'un `<select>` ou menu sans suivre le pattern ARIA APG                              |
| Hiérarchie de titres cassée                           | Titres choisis pour leur taille visuelle (CSS) plutôt que leur niveau sémantique                 |
| Zoom 200% qui casse la mise en page                   | Dimensions en `px` fixes, conteneurs sans `overflow`/`wrap`                                      |

---

# Le focus clavier

### Problèmes

- `outline: none` global
- Composants non focusables (`div onclick`)
- Ordre de tabulation cassé

<br />

### Conséquences

- Navigation clavier impossible
- Lecteur d'écran moins efficace

---

# Focus — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<!-- ❌ Non focusable, pas activable au clavier, pas annoncé comme bouton -->
<div class="btn" onclick="submitForm()">Valider</div>
```

<br />

### Bonne pratique

```html
<!-- ✅ Élément natif, focusable et activable au clavier automatiquement -->
<button type="submit">Valider</button>
```

---

# Formulaires — problèmes récurrents

- Labels non associés
- Erreurs non liées au champ
- Couleur seule pour indiquer l'erreur

---

# Formulaires — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<input id="name" placeholder="Nom" />
<!-- message d'erreur ajouté ailleurs sans relation -->
```

<br />

### Bonne pratique

```html
<!-- ✅ Label associé + message d'erreur lié via aria-describedby et role=alert pour annonce -->
<label for="name">Nom</label>
<input id="name" required aria-describedby="name-help" aria-invalid="true" />
<div id="name-help" role="alert">Le nom ne doit pas contenir de chiffres</div>
```

---

# Composants JS "faits maison"

- Réinventer `<select>`, `button` ou menu sans patterns ARIA
- Exemple : accordéon, modale, dropdown qui ne gèrent pas le focus

---

# Accordéon — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<!-- ❌ Pas de rôle/état ARIA, pas focusable correctement, clavier non géré -->
<div class="acc-item" onclick="toggle()">Titre</div>
<div class="acc-panel">Contenu</div>
```

<br />

### Bonne pratique (pattern ARIA)

```html
<!-- ✅ Utilise ARIA APG : bouton contrôlant le panneau, état explicite -->
<button aria-expanded="false" aria-controls="panel-1">Titre</button>
<div id="panel-1" hidden>Contenu</div>
```

---

# Tableau de données — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<table>
  <tr>
    <td>Nom</td>
    <td>Âge</td>
  </tr>
  <tr>
    <td>Alice</td>
    <td>30</td>
  </tr>
</table>
<!-- ❌ Pas de <th scope="col"> → lecteur d'écran ne peut pas associer en-têtes/colonnes -->
```

---

# Tableau de données — mauvaise vs bonne pratique

### Bonne pratique

```html
<table>
  <thead>
    <tr>
      <th scope="col">Nom</th>
      <th scope="col">Âge</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
```

---

# Modale — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<!-- modale ouverte sans focus déplacé ni piège de focus -->
<div id="modal" style="display:block">Modal content<button>Close</button></div>
```

<br />

### Bonne pratique (focus management)

```html
<!-- ouverture : focus sur le container ou premier élément focusable, piège de focus, Echap pour fermer -->
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <h2 id="modal-title">Titre</h2>
  <button>Fermer</button>
</div>
```

---

# Menu déroulant — mauvaise vs bonne pratique

### Mauvaise pratique

```html
<ul class="menu">
  <li onmouseover="open()">Item</li>
</ul>
<!-- non accessible au clavier (mouseover only) -->
```

<br />

### Bonne pratique

```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" role="menu">
  ...
  <!-- géré par clavier + ARIA APG -->
</ul>
```

---

# Skip link & boutons icon-only

### Skip link (bonne pratique)

```html
<a class="skip-link" href="#main">Aller au contenu</a>
```

<br />

### Icône seule — mauvaise vs bonne pratique

```html
<!-- ❌ Mauvaise : bouton icon-only avec icône décorative annoncée -->
<button class="icon">
  <svg viewBox="0 0 24 24">
    <path d="M10 4a6 6 0 1 0 0 12 6 6 0 0 0 0-12Z" />
  </svg>
</button>

<!-- ✅ Bonne : icône décorative masquée, libellé accessible fourni -->
<button class="icon" aria-label="Rechercher">
  <svg viewBox="0 0 24 24" aria-hidden="true">
    <path d="M10 4a6 6 0 1 0 0 12 6 6 0 0 0 0-12Z" />
  </svg>
</button>
```

---

# Carrousel — problème courant

### Mauvaise pratique

- Carrousel auto-play sans pause → focus saute, utilisateur perd le contrôle

<br />

### À éviter / vérifier

- Fournir contrôle pause/play, navigation clavier, et permettre d'isoler le focus

---

# Exemples CSS courants

- `outline: none` sans alternative → focus invisible
- Dimensions fixes en `px` → casse au zoom 200%

---

# CSS — mauvaise vs meilleure pratique

### Mauvaise pratique

```css
/* ❌ Supprime le focus visuel et utilise des dimensions fixes qui cassent au zoom */
* {
  outline: none;
}
.card {
  width: 400px;
}
```

<br />

### Meilleure pratique

```css
:focus-visible {
  outline: 2px solid #0050d8; /* couleur fixe, contraste vérifié */
  outline-offset: 2px;
}
.card {
  max-width: 100%;
} /* ✅ Utiliser des unités fluides pour supporter zoom/reflow */
```

---

# Contenus dynamiques & lecteurs d'écran

- Mettre à jour le DOM sans `aria-live` ou déplacement de focus empêche la lecture
- Images décoratives : `alt=""`, images informatives : `alt` descriptif

---

# Live region — exemple

### Mauvaise pratique

```html
<div id="cart-count">3</div>
<!-- JS met à jour innerText sans annonce -->
```

<br />

### Bonne pratique

```html
<!-- ✅ aria-live permet au lecteur d'écran d'annoncer la mise à jour dynamique -->
<div id="cart-count" aria-live="polite">3</div>
```

---

# Erreurs fréquentes faites sans s'en rendre compte

- Copier-coller depuis Figma/Storybook sans tester clavier
- Tester uniquement à la souris
- Penser l'accessibilité en fin de projet
- Sur-utiliser ARIA quand le HTML natif suffit

---

# Conseils pratiques pour les devs

- Réflexe n°1 : privilégier l'élément HTML natif
- Tester chaque écran **uniquement au clavier**
- Ne pas supprimer `outline` sans remplacer par un focus visible
- Suivre ARIA APG pour composants custom
- Gérer le focus programmatique (ouverture/fermeture modale)
- Intégrer outils d'audit (axe, Lighthouse) dans le flux de dev

---

# Conclusion — 4 choses à retenir

- HTML natif avant JS/ARIA
- Toujours garder un focus visible
- Tester au clavier avant chaque recette
- Les erreurs sont récurrentes et évitables — 80% évitables sans expertise RGAA
