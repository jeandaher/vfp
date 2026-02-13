# NFP — No Frame Page: Architecture Document

## 1. Overview

**NFP (No Frame Page)** is a Lightning solution that transforms a standard Salesforce page into a full-page web application by completely hiding the Salesforce shell (header, navigation, context bar) to deliver a standalone user experience.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Salesforce Lightning Shell                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  SF Header ▓▓▓▓▓▓▓▓▓▓▓▓  ← HIDDEN by CSS override       │  │
│  │  Nav Bar   ▓▓▓▓▓▓▓▓▓▓▓▓  ← HIDDEN by CSS override       │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │         nfpFullPageApp (position: fixed)            │  │  │
│  │  │  ┌───────────────────────────────────────────────┐  │  │  │
│  │  │  │  Custom Header (title, search, actions)       │  │  │  │
│  │  │  ├──────┬────────────────────────────────────────┤  │  │  │
│  │  │  │Filter│  Content area                          │  │  │  │
│  │  │  │Panel │  - Default slot                        │  │  │  │
│  │  │  │      │  - nfpVfContainer (VF iframe)          │  │  │  │
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

## 2. Salesforce Shell Hiding Mechanism

### The Problem
Salesforce Lightning always displays a **global header** (logo, search, notifications) and a **navigation bar** (tabs, apps) around every page's content. These elements cannot be removed through standard configuration.

### The Solution: CSS Injection + Fixed Positioning

The `nfpFullPageApp` component uses **two combined techniques**:

#### Technique 1 — CSS Injection into `document.head`

```javascript
// nfpFullPageApp.js — lines 165-201
_injectOverrideStyles() {
    const style = document.createElement('style');
    style.id = 'nfp-fullpage-override';
    style.textContent = `
        /* Hide the Salesforce global header */
        .slds-global-header_container,
        .communitiesHeader,
        header.slds-global-header {
            display: none !important;
        }

        /* Hide the navigation bar */
        .oneAppNavContainer,
        .appNavContainer,
        .tabBarContainer,
        .slds-context-bar {
            display: none !important;
        }

        /* Prevent Salesforce body from scrolling */
        body {
            overflow: hidden !important;
        }
    `;
    document.head.appendChild(style);
}
```

**Why it works:**
- LWC runs in the same DOM context as the Salesforce shell
- CSS selectors target Salesforce's internal SLDS classes
- `!important` ensures priority over Salesforce styles
- The `<style>` element is injected into `document.head` (outside Shadow DOM)

**Lifecycle:**
- `connectedCallback()` — injects the style
- `disconnectedCallback()` — removes the style (restores the SF shell)
- Duplicate protection via `document.getElementById('nfp-fullpage-override')`

#### Technique 2 — Full-Page Fixed Positioning

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

**Why `position: fixed` instead of `height: 100vh`:**
- `100vh` doesn't work because the Salesforce tab header takes up vertical space, pushing the footer off-screen
- `height: 100%` would require propagating `height: 100%` through the entire SF parent chain (fragile)
- `position: fixed` positions relative to the **viewport**, independently of any Salesforce parent
- `z-index: 9999` ensures the app covers any remaining SF content

### Targeted CSS Selectors

| Selector | Hidden SF Element |
|---|---|
| `.slds-global-header_container` | Lightning global header container |
| `.communitiesHeader` | Communities / Experience Cloud header |
| `header.slds-global-header` | Semantic header element |
| `.oneAppNavContainer` | One App navigation container |
| `.appNavContainer` | Alternate navigation container |
| `.tabBarContainer` | Tab bar |
| `.slds-context-bar` | SLDS context bar |

---

## 3. Component Tree

```
nfpFullPageApp (Main container)
├── [header] Custom header
│   ├── Application title (@api appTitle)
│   ├── Search bar (search-box)
│   └── <slot name="header-actions"> (extensible)
│
├── [body] Main area (flex row)
│   ├── nfpFilterPanel (left sidebar, accordion)
│   │   └── Filter sections (checkbox / button)
│   │
│   └── [main] Content area (conditional rendering)
│       ├── nfpVfContainer (secure Visualforce iframe)
│       ├── nfpDataTable (high-performance custom table)
│       ├── nfpLWCDataTable (lightning-datatable wrapper)
│       └── <slot> (extensible default content)
│
└── [footer] Custom footer
    ├── Copyright text (@api footerText)
    ├── Navigation buttons (Page One, Page Two, Cases, Sales, LWC Sales, Close)
    └── <slot name="footer-actions"> (extensible)
```

---

## 4. Component Roles

### 4.1 `nfpFullPageApp` — Main Container

**Files:** `nfpFullPageApp.html`, `.js`, `.css`, `nfpMockData.js`, `.js-meta.xml`

**Responsibilities:**
- Hide the Salesforce shell (CSS injection + fixed positioning)
- Manage the global layout (header / body / footer in vertical flexbox)
- Route displayed content based on active state (VF page, custom table, LWC table, or default slot)
- Provide the global search bar
- Manage navigation through footer buttons

**State management:**
```
activeVfPage  = ''              → no VF page active
activeVfPage  = 'NFP_Page_One'  → displays nfpVfContainer
activeTable   = ''              → no table active
activeTable   = 'cases'         → displays nfpDataTable (custom)
activeTable   = 'lwcSales'      → displays nfpLWCDataTable (lightning)
```

Views are **mutually exclusive**: activating one view deactivates the others.

**Table configuration (TABLE_CONFIGS):**
```javascript
const TABLE_CONFIGS = {
    cases:    { title, columns, generator, component: 'custom' },
    sales:    { title, columns, generator, component: 'custom' },
    lwcSales: { title, columns, generator, component: 'lwc' }
};
```
The `component` field determines which table component is rendered (`showCustomTable` vs `showLwcTable`).

---

### 4.2 `nfpFilterPanel` — Side Filter Panel

**Files:** `nfpFilterPanel.html`, `.js`, `.css`, `.js-meta.xml`

**Responsibilities:**
- Display a collapsible filter panel on the left side
- Support two filter types: **checkbox** and **button**
- Emit a `filterchange` event with active filters

**Features:**
- **Open:** 280px wide | **Closed:** 48px (toggle icon)
- **Accordion sections**: each filter group collapses/expands
- **Reactivity:** `@track _activeFilters` with immutable updates (`{ ...spread }`)
- **"Clear All" button**: visible only when filters are active

**API:**
```html
<c-nfp-filter-panel
    filter-config={filterConfig}
    onfilterchange={handleFilterChange}
></c-nfp-filter-panel>
```

---

### 4.3 `nfpVfContainer` — Secure Visualforce Container

**Files:** `nfpVfContainer.html`, `.js`, `.css`, `.js-meta.xml`

**Responsibilities:**
- Display a Visualforce page inside a secure iframe
- Protect against URL injection and unauthorized pages

**Security (3 layers):**

| Layer | Mechanism | Code |
|-------|-----------|------|
| 1 | **Whitelist** | `ALLOWED_PAGES = new Set(['NFP_Page_One', 'NFP_Page_Two'])` |
| 2 | **Regex** | `/^[a-zA-Z][a-zA-Z0-9_]*$/` — alphanumeric characters only |
| 3 | **Encoding** | `encodeURIComponent(name)` — protection against special characters |

**Iframe sandbox:**
```
allow-scripts allow-same-origin allow-forms allow-popups allow-popups-to-escape-sandbox
```

**API:**
```html
<c-nfp-vf-container page-name="NFP_Page_One"></c-nfp-vf-container>
```

---

### 4.4 `nfpDataTable` — High-Performance Custom Table

**Files:** `nfpDataTable.html`, `.js`, `.css`, `.js-meta.xml`

**Responsibilities:**
- Display a read-only data table optimized for 1,000+ rows
- Column sorting, debounced search, pagination

**Processing pipeline (chained getters with cache):**
```
data (raw) → filteredData (search) → sortedData (sort) → displayRows (page + formatting)
```

Each step uses a **cache key** (`_lastFilterKey`, `_lastSortKey`) to avoid unnecessary recalculations.

**Optimizations:**
- **Cached Intl formatters** at module level (not recreated on each render)
- **300ms debounce** on search input
- **Pre-formatted cells** in the `displayRows` getter — the template simply iterates
- **Supported types:** `text`, `number`, `currency`, `percent`, `date`, `boolean`

**API:**
```html
<c-nfp-data-table
    columns={columns}     data={data}
    title="My Table"      page-size="50"
    show-search            show-row-numbers
></c-nfp-data-table>
```

---

### 4.5 `nfpLWCDataTable` — lightning-datatable Wrapper

**Files:** `nfpLWCDataTable.html`, `.js`, `.css`, `.js-meta.xml`

**Responsibilities:**
- Provide the same API as `nfpDataTable` using Salesforce's native `lightning-datatable` component
- Automatic column format conversion from custom format to lightning-datatable format
- Add debounced search and pagination (not provided by `lightning-datatable`)

**Column conversion:**

| Custom format | lightning-datatable format |
|---|---|
| `width: '200px'` | `initialWidth: 200` |
| `currencyCode: 'EUR'` | `typeAttributes: { currencyCode: 'EUR' }` |
| `align: 'right'` | `cellAttributes: { alignment: 'right' }` |
| `type: 'date'` | `typeAttributes: { year, month, day }` |

**Identical API:**
```html
<c-nfp-l-w-c-data-table
    columns={columns}     data={data}
    title="My Table"      page-size="50"
    show-search            show-row-numbers
></c-nfp-l-w-c-data-table>
```

---

## 5. Salesforce Metadata

### Lightning App (`NFP_Full_Page_App.app-meta.xml`)
```xml
<CustomApplication>
    <label>NFP Full Page App</label>
    <formFactors>Large</formFactors>   <!-- Desktop only -->
    <navType>Standard</navType>
    <uiType>Lightning</uiType>
</CustomApplication>
```

### FlexiPage (`NFP_Full_Page_App.flexipage-meta.xml`)
- Template: `flexipage:defaultAppHomeTemplate`
- Contains a single instance of `c:nfpFullPageApp`
- Type: `AppPage`

### Custom Tab (`nfpFullPageApp.tab-meta.xml`)
- Links to the `nfpFullPageApp` LWC component
- Manually added to the app via Setup > App Manager > Edit

### Visualforce Pages
- `NFP_Page_One.page` — Demo page with badge
- `NFP_Page_Two.page` — Demo page with sample data table

---

## 6. CSS Layout — Flexbox Strategy

```
┌──────────────────────────────────────────────┐ position: fixed
│ .app-header    │ height: 56px, flex-shrink: 0 │ (does not compress)
├──────────────────────────────────────────────┤
│ .app-body      │ flex: 1 1 0%, min-height: 0  │ (fills all remaining space)
│ ├── filter-panel │ width: 280px/48px          │
│ └── app-content  │ flex: 1, overflow: auto    │
├──────────────────────────────────────────────┤
│ .app-footer    │ height: 40px, flex-shrink: 0 │ (does not compress)
└──────────────────────────────────────────────┘
```

**Key rules:**
- `flex-shrink: 0` on header and footer — they maintain their fixed size
- `flex: 1 1 0%` + `min-height: 0` on `.app-body` — fills remaining space without overflow
- `overflow: auto` on `.app-content` — internal scroll within the content area
- `overflow: hidden` on `.app-body` — prevents global overflow

---

## 7. Data Flow

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

## 8. File Inventory

| Directory | File | Role |
|---|---|---|
| `lwc/nfpFullPageApp/` | `nfpFullPageApp.html` | Main container template |
| | `nfpFullPageApp.js` | Logic, CSS injection, routing |
| | `nfpFullPageApp.css` | Fixed positioning, flexbox layout |
| | `nfpMockData.js` | Demo data (Cases 1,200 rows, Sales 800 rows) |
| | `nfpFullPageApp.js-meta.xml` | LWC metadata |
| `lwc/nfpFilterPanel/` | 4 files | Accordion filter panel |
| `lwc/nfpVfContainer/` | 4 files | Secure VF iframe container |
| `lwc/nfpDataTable/` | 4 files | High-performance custom table |
| `lwc/nfpLWCDataTable/` | 4 files | lightning-datatable wrapper |
| `applications/` | `NFP_Full_Page_App.app-meta.xml` | Lightning Application |
| `flexipages/` | `NFP_Full_Page_App.flexipage-meta.xml` | Lightning App Page |
| `tabs/` | `nfpFullPageApp.tab-meta.xml` | Custom Tab |
| `pages/` | `NFP_Page_One.page` + meta | Visualforce demo page 1 |
| | `NFP_Page_Two.page` + meta | Visualforce demo page 2 |

**Total: 24 files, 5 LWC components, 1 app, 1 flexipage, 1 tab, 2 VF pages**
