# NFP — No Frame Page : Dossier d'Architecture

## 1. Vue d'ensemble

**NFP (No Frame Page)** est une solution Lightning qui transforme une page Salesforce standard en une application web pleine page, en masquant complètement le shell Salesforce (header, navigation, barre de contexte) pour offrir une expérience utilisateur autonome.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Salesforce Lightning Shell                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Header SF ▓▓▓▓▓▓▓▓▓▓▓▓  ← MASQUÉ par CSS override      │  │
│  │  Nav Bar   ▓▓▓▓▓▓▓▓▓▓▓▓  ← MASQUÉ par CSS override      │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │         nfpFullPageApp (position: fixed)            │  │  │
│  │  │  ┌───────────────────────────────────────────────┐  │  │  │
│  │  │  │  Custom Header (titre, recherche, actions)    │  │  │  │
│  │  │  ├──────┬────────────────────────────────────────┤  │  │  │
│  │  │  │Filter│  Zone de contenu                       │  │  │  │
│  │  │  │Panel │  - Default slot                        │  │  │  │
│  │  │  │      │  - nfpVfContainer (iframe VF)          │  │  │  │
│  │  │  │      │  - nfpDataTable (custom table)         │  │  │  │
│  │  │  │      │  - nfpLWCDataTable (lightning-datatable)│  │  │  │
│  │  │  ├──────┴────────────────────────────────────────┤  │  │  │
│  │  │  │  Custom Footer (navigation, actions)          │  │  │  │
│  │  │  └───────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Mécanisme de masquage du shell Salesforce

### Le problème
Salesforce Lightning affiche systématiquement un **header global** (logo, recherche, notifications) et une **barre de navigation** (onglets, apps) autour du contenu de chaque page. Ces éléments ne peuvent pas être supprimés via la configuration standard.

### La solution : Injection CSS + Position Fixed

Le composant `nfpFullPageApp` utilise **deux techniques combinées** :

#### Technique 1 — Injection CSS dans `document.head`

```javascript
// nfpFullPageApp.js — lignes 165-201
_injectOverrideStyles() {
    const style = document.createElement('style');
    style.id = 'nfp-fullpage-override';
    style.textContent = `
        /* Masque le header global Salesforce */
        .slds-global-header_container,
        .communitiesHeader,
        header.slds-global-header {
            display: none !important;
        }

        /* Masque la barre de navigation */
        .oneAppNavContainer,
        .appNavContainer,
        .tabBarContainer,
        .slds-context-bar {
            display: none !important;
        }

        /* Bloque le scroll du body Salesforce */
        body {
            overflow: hidden !important;
        }
    `;
    document.head.appendChild(style);
}
```

**Pourquoi ça fonctionne :**
- LWC s'exécute dans le même contexte DOM que le shell Salesforce
- Les sélecteurs CSS ciblent les classes SLDS internes de Salesforce
- `!important` garantit la priorité sur les styles Salesforce
- L'élément `<style>` est injecté dans `document.head` (hors Shadow DOM)

**Cycle de vie :**
- `connectedCallback()` → injection du style
- `disconnectedCallback()` → retrait du style (restaure le shell SF)
- Protection anti-doublon via `document.getElementById('nfp-fullpage-override')`

#### Technique 2 — Position Fixed pleine page

```css
/* nfpFullPageApp.css */
.fullpage-app {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 9999;
    display: flex;
    flex-direction: column;
}
```

**Pourquoi `position: fixed` et non `height: 100vh` :**
- `100vh` ne fonctionne pas car le tab Salesforce occupe de l'espace vertical, repoussant le footer hors écran
- `height: 100%` nécessiterait de propager `height: 100%` à travers toute la chaîne de parents SF (fragile)
- `position: fixed` se positionne par rapport au **viewport**, indépendamment de tout parent Salesforce
- `z-index: 9999` garantit que l'app recouvre tout le contenu SF résiduel

### Sélecteurs CSS ciblés

| Sélecteur | Élément SF masqué |
|---|---|
| `.slds-global-header_container` | Container du header global Lightning |
| `.communitiesHeader` | Header dans les communautés/Experience Cloud |
| `header.slds-global-header` | Élément header sémantique |
| `.oneAppNavContainer` | Container de navigation One App |
| `.appNavContainer` | Container alternatif de navigation |
| `.tabBarContainer` | Barre d'onglets |
| `.slds-context-bar` | Barre de contexte SLDS |

---

## 3. Arbre des composants

```
nfpFullPageApp (Container principal)
├── [header] Custom header
│   ├── Titre de l'application (@api appTitle)
│   ├── Barre de recherche (search-box)
│   └── <slot name="header-actions"> (extensible)
│
├── [body] Zone principale (flex row)
│   ├── nfpFilterPanel (sidebar gauche, accordion)
│   │   └── Sections de filtres (checkbox / button)
│   │
│   └── [main] Zone de contenu (affichage conditionnel)
│       ├── nfpVfContainer (iframe Visualforce sécurisé)
│       ├── nfpDataTable (table custom haute performance)
│       ├── nfpLWCDataTable (wrapper lightning-datatable)
│       └── <slot> (contenu par défaut extensible)
│
└── [footer] Custom footer
    ├── Texte copyright (@api footerText)
    ├── Boutons de navigation (Page One, Page Two, Cases, Sales, LWC Sales, Close)
    └── <slot name="footer-actions"> (extensible)
```

---

## 4. Rôle de chaque composant

### 4.1 `nfpFullPageApp` — Container principal

**Fichiers :** `nfpFullPageApp.html`, `.js`, `.css`, `nfpMockData.js`, `.js-meta.xml`

**Responsabilités :**
- Masquer le shell Salesforce (injection CSS + position fixed)
- Gérer le layout global (header / body / footer en flexbox vertical)
- Router le contenu affiché selon l'état actif (VF page, table custom, table LWC, ou slot par défaut)
- Fournir la barre de recherche globale
- Gérer la navigation via les boutons du footer

**Gestion d'état :**
```
activeVfPage  = ''          → aucune VF page active
activeVfPage  = 'NFP_Page_One' → affiche nfpVfContainer
activeTable   = ''          → aucune table active
activeTable   = 'cases'     → affiche nfpDataTable (custom)
activeTable   = 'lwcSales'  → affiche nfpLWCDataTable (lightning)
```

Les vues sont **mutuellement exclusives** : activer une vue désactive les autres.

**Configuration des tables (TABLE_CONFIGS) :**
```javascript
const TABLE_CONFIGS = {
    cases:    { title, columns, generator, component: 'custom' },
    sales:    { title, columns, generator, component: 'custom' },
    lwcSales: { title, columns, generator, component: 'lwc' }
};
```
Le champ `component` détermine quel composant table est rendu (`showCustomTable` vs `showLwcTable`).

---

### 4.2 `nfpFilterPanel` — Panneau de filtres latéral

**Fichiers :** `nfpFilterPanel.html`, `.js`, `.css`, `.js-meta.xml`

**Responsabilités :**
- Afficher un panneau de filtres rétractable sur le côté gauche
- Supporter deux types de filtres : **checkbox** et **button**
- Émettre un événement `filterchange` avec les filtres actifs

**Caractéristiques :**
- **Ouvert :** 280px de large | **Fermé :** 48px (icône toggle)
- **Sections accordion** : chaque groupe de filtres se replie/déplie
- **Réactivité :** `@track _activeFilters` avec mises à jour immutables (`{ ...spread }`)
- **Bouton "Clear All"** : visible uniquement quand des filtres sont actifs

**API :**
```html
<c-nfp-filter-panel
    filter-config={filterConfig}
    onfilterchange={handleFilterChange}
></c-nfp-filter-panel>
```

---

### 4.3 `nfpVfContainer` — Conteneur Visualforce sécurisé

**Fichiers :** `nfpVfContainer.html`, `.js`, `.css`, `.js-meta.xml`

**Responsabilités :**
- Afficher une page Visualforce dans un iframe sécurisé
- Protéger contre l'injection d'URL et les pages non autorisées

**Sécurité (3 niveaux) :**

| Niveau | Mécanisme | Code |
|--------|-----------|------|
| 1 | **Whitelist** | `ALLOWED_PAGES = new Set(['NFP_Page_One', 'NFP_Page_Two'])` |
| 2 | **Regex** | `/^[a-zA-Z][a-zA-Z0-9_]*$/` — caractères alphanumériques uniquement |
| 3 | **Encoding** | `encodeURIComponent(name)` — protection contre les caractères spéciaux |

**Sandbox iframe :**
```
allow-scripts allow-same-origin allow-forms allow-popups allow-popups-to-escape-sandbox
```

**API :**
```html
<c-nfp-vf-container page-name="NFP_Page_One"></c-nfp-vf-container>
```

---

### 4.4 `nfpDataTable` — Table custom haute performance

**Fichiers :** `nfpDataTable.html`, `.js`, `.css`, `.js-meta.xml`

**Responsabilités :**
- Afficher un tableau de données en lecture seule optimisé pour 1000+ lignes
- Tri par colonnes, recherche debounced, pagination

**Pipeline de traitement (getters chaînés avec cache) :**
```
data (brut) → filteredData (recherche) → sortedData (tri) → displayRows (page + formatage)
```

Chaque étape utilise une **clé de cache** (`_lastFilterKey`, `_lastSortKey`) pour éviter les recalculs inutiles.

**Optimisations :**
- **Formatters Intl cachés** au niveau module (pas recréés à chaque render)
- **Debounce 300ms** sur la recherche
- **Pré-formatage des cellules** dans le getter `displayRows` — le template ne fait qu'itérer
- **Types supportés :** `text`, `number`, `currency`, `percent`, `date`, `boolean`

**API :**
```html
<c-nfp-data-table
    columns={columns}     data={data}
    title="My Table"      page-size="50"
    show-search            show-row-numbers
></c-nfp-data-table>
```

---

### 4.5 `nfpLWCDataTable` — Wrapper lightning-datatable

**Fichiers :** `nfpLWCDataTable.html`, `.js`, `.css`, `.js-meta.xml`

**Responsabilités :**
- Fournir la même API que `nfpDataTable` en utilisant le composant natif `lightning-datatable` de Salesforce
- Conversion automatique du format de colonnes custom → format lightning-datatable
- Ajouter recherche debounced et pagination (non fournis par `lightning-datatable`)

**Conversion de colonnes :**

| Format custom | Format lightning-datatable |
|---|---|
| `width: '200px'` | `initialWidth: 200` |
| `currencyCode: 'EUR'` | `typeAttributes: { currencyCode: 'EUR' }` |
| `align: 'right'` | `cellAttributes: { alignment: 'right' }` |
| `type: 'date'` | `typeAttributes: { year, month, day }` |

**API identique :**
```html
<c-nfp-l-w-c-data-table
    columns={columns}     data={data}
    title="My Table"      page-size="50"
    show-search            show-row-numbers
></c-nfp-l-w-c-data-table>
```

---

## 5. Métadonnées Salesforce

### Lightning App (`NFP_Full_Page_App.app-meta.xml`)
```xml
<CustomApplication>
    <label>NFP Full Page App</label>
    <formFactors>Large</formFactors>   <!-- Desktop uniquement -->
    <navType>Standard</navType>
    <uiType>Lightning</uiType>
</CustomApplication>
```

### FlexiPage (`NFP_Full_Page_App.flexipage-meta.xml`)
- Template : `flexipage:defaultAppHomeTemplate`
- Contient une seule instance de `c:nfpFullPageApp`
- Type : `AppPage`

### Custom Tab (`nfpFullPageApp.tab-meta.xml`)
- Lien vers le composant LWC `nfpFullPageApp`
- Ajouté manuellement à l'app via Setup > App Manager > Edit

### Visualforce Pages
- `NFP_Page_One.page` — Page de démonstration avec badge
- `NFP_Page_Two.page` — Page de démonstration avec tableau

---

## 6. Layout CSS — Stratégie Flexbox

```
┌──────────────────────────────────────────────┐ position: fixed
│ .app-header    │ height: 56px, flex-shrink: 0 │ (ne se compresse pas)
├──────────────────────────────────────────────┤
│ .app-body      │ flex: 1 1 0%, min-height: 0  │ (occupe tout l'espace restant)
│ ├── filter-panel │ width: 280px/48px          │
│ └── app-content  │ flex: 1, overflow: auto    │
├──────────────────────────────────────────────┤
│ .app-footer    │ height: 40px, flex-shrink: 0 │ (ne se compresse pas)
└──────────────────────────────────────────────┘
```

**Règles clés :**
- `flex-shrink: 0` sur header et footer → ils gardent leur taille fixe
- `flex: 1 1 0%` + `min-height: 0` sur `.app-body` → occupe l'espace restant sans déborder
- `overflow: auto` sur `.app-content` → scroll interne dans la zone de contenu
- `overflow: hidden` sur `.app-body` → empêche le débordement global

---

## 7. Flux de données

```
┌──────────┐     filterchange      ┌───────────────┐
│  Filter   │ ───────────────────→ │               │
│  Panel    │                      │  FullPageApp  │
└──────────┘                       │  (container)  │
                                   │               │
┌──────────┐  columns + data       │  TABLE_CONFIGS│
│ MockData │ ────────────────────→ │  { columns,   │
│ .js      │                       │    generator, │
└──────────┘                       │    component }│
                                   └───────┬───────┘
                                           │
                          ┌────────────────┼────────────────┐
                          ↓                ↓                ↓
                   ┌────────────┐  ┌────────────┐  ┌────────────┐
                   │ VfContainer│  │  DataTable  │  │LWCDataTable│
                   │  (iframe)  │  │  (custom)   │  │(lightning) │
                   └────────────┘  └────────────┘  └────────────┘
```


---

NFP_Full_Page_App.app-meta.xml  (Lightning Application)
    │
    ├── Custom Tab (nfpFullPageApp.tab-meta.xml)
    │   → Ajouté manuellement dans Navigation Items de l'app
    │   → Pointe vers le composant LWC nfpFullPageApp
    │
    └── FlexiPage (NFP_Full_Page_App.flexipage-meta.xml)
        → Contient l'instance de c:nfpFullPageApp
        → Assignée comme page d'accueil de l'app
        
---

## 8. Inventaire des fichiers

| Dossier | Fichier | Rôle |
|---|---|---|
| `lwc/nfpFullPageApp/` | `nfpFullPageApp.html` | Template du container principal |
| | `nfpFullPageApp.js` | Logique, injection CSS, routing |
| | `nfpFullPageApp.css` | Position fixed, layout flexbox |
| | `nfpMockData.js` | Données de démonstration (Cases 1200 lignes, Sales 800 lignes) |
| | `nfpFullPageApp.js-meta.xml` | Métadonnées LWC |
| `lwc/nfpFilterPanel/` | 4 fichiers | Panneau de filtres accordion |
| `lwc/nfpVfContainer/` | 4 fichiers | Container iframe sécurisé pour VF |
| `lwc/nfpDataTable/` | 4 fichiers | Table custom haute performance |
| `lwc/nfpLWCDataTable/` | 4 fichiers | Wrapper lightning-datatable |
| `applications/` | `NFP_Full_Page_App.app-meta.xml` | Lightning Application |
| `flexipages/` | `NFP_Full_Page_App.flexipage-meta.xml` | Page d'app Lightning |
| `tabs/` | `nfpFullPageApp.tab-meta.xml` | Custom Tab |
| `pages/` | `NFP_Page_One.page` + meta | Visualforce page démo 1 |
| | `NFP_Page_Two.page` + meta | Visualforce page démo 2 |

**Total : 24 fichiers, 5 composants LWC, 1 app, 1 flexipage, 1 tab, 2 pages VF**
