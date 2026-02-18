# 📘 DOCUMENT D’ARCHITECTURE — Fund Explorer Postgres + ClickHouse + Materialized Views + DataWeave

## 1. Objectif du système

Mettre en place un **Fund Explorer local** capable d’explorer des données volumineuses (100M+ lignes) avec :

- tableau riche et configurable,
- filtres avancés (range, date, multiselect, search),
- agrégations (min/max, distinct_count, histogrammes),
- profils de données,
- performance stable,
- architecture **Postgres (opérationnel) + ClickHouse (analytique)**,
- orchestration via **Spring Boot + DataWeave**.

---

## 2. Architecture globale

```text
Angular Fund Explorer (Frontend)
        │
        ▼
Spring Boot API (Fund Explorer)
        │
        ▼
DataWeave (Transformation {data, meta})
        │
        ▼
ClickHouse (Moteur Analytique)
        ▲
        │
ETL Postgres → ClickHouse
        ▲
        │
Postgres (Opérationnel + Materialized Views)
```

---

## 3. Description des composants

### 3.1 Angular – Fund Explorer (Frontend)

**Rôle :**

- Interface utilisateur principale.
- Tableau dynamique + filtres metadata‑driven.
- Gestion des préférences utilisateur (colonnes, tri, vues).
- Envoi des filtres/tri/pagination au backend.

**Fonctionnalités :**

- Affichage des colonnes configurables.
- Filtres adaptés au type de données (range, date, multiselect, search).
- Pagination, tri, vues sauvegardées.
- Consommation d’un contrat unique `{data, meta}`.

---

### 3.2 Spring Boot — Fund Explorer API

**Rôle :**

- Orchestrateur central.
- Expose l’API d’exploration.
- Génère les requêtes analytiques ClickHouse.
- Utilise DataWeave pour assembler `{data, meta}`.

**Endpoints :**

- `POST /funds/query`
- `GET /funds/metadata` (optionnel)

**Fonctionnalités :**

- Lecture de la **metadata des colonnes** (JSON / table config).
- Construction dynamique de SQL ClickHouse :
  - `WHERE` selon filtres,
  - `ORDER BY` selon tri,
  - `LIMIT/OFFSET` pour pagination.
- Exécution de requêtes analytiques :
  - données paginées,
  - agrégations (min/max, distinct_count, etc.).
- Gestion des erreurs, timeouts, logs.

---

### 3.3 DataWeave (dans Spring Boot)

**Rôle :**

- Transformation JSON avancée.
- Fusion des résultats ClickHouse (data + agrégations).
- Application de la metadata (types, labels, formats, modes de filtrage).
- Construction de la réponse finale pour Angular.

**Sortie type :**

```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "pageSize": 50,
    "totalPages": 120,
    "totalCount": 6000,
    "filters": { ... }
  }
}
```

---

### 3.4 Postgres (Opérationnel)

**Rôle :**

- Base de données métier principale.
- Stockage des entités normalisées (fonds, comptes, transactions, etc.).
- Source de vérité fonctionnelle.

**Fonctionnalités :**

- CRUD applicatif.
- Intégrité référentielle.
- Support des transactions.

---

### 3.5 Materialized Views Postgres

**Rôle :**

- Pré‑traitement analytique avant ClickHouse.
- Préparation de vues “flattened” pour l’ETL.
- Réduction de la complexité des jointures côté ETL.
- Protection des tables opérationnelles des requêtes lourdes.

**Exemple :**

```sql
CREATE MATERIALIZED VIEW mv_funds_flat AS
SELECT
    f.id              AS fund_id,
    f.name            AS fund_name,
    f.aum,
    f.status,
    c.country         AS region,
    d.inception_date
FROM funds f
LEFT JOIN countries c   ON f.country_id = c.id
LEFT JOIN fund_details d ON f.id = d.fund_id;
```

**Rafraîchissement :**

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_funds_flat;
```

---

### 3.6 ETL Postgres → ClickHouse

**Rôle :**

- Extraire les données préparées (via MV).
- Les transformer si nécessaire.
- Les charger dans ClickHouse.

**Implémentation possible :**

- **Spring Batch** (job dédié).
- ou **DataWeave** dans un module Spring Boot.

**Flux :**

1. Lecture de `mv_funds_flat`.
2. Transformation (mapping, normalisation).
3. Insertion dans ClickHouse (JDBC ou HTTP `INSERT ... FORMAT JSONEachRow`).

---

### 3.7 ClickHouse (Moteur Analytique)

**Rôle :**

- Stockage analytique haute performance.
- Exécution des filtres complexes.
- Agrégations massives.
- Tri et pagination rapides.

**Exemple de table :**

```sql
CREATE TABLE funds_analytics (
    fund_id        String,
    fund_name      String,
    aum            Float64,
    status         String,
    region         String,
    inception_date Date
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(inception_date)
ORDER BY (status, region, aum);
```

---

## 4. Flux de données

### 4.1 Préparation des données (Postgres → ClickHouse)

1. Rafraîchissement des Materialized Views dans Postgres.
2. ETL lit `mv_funds_flat`.
3. Transformation (si besoin).
4. Chargement dans `funds_analytics` (ClickHouse).

---

### 4.2 Requête Fund Explorer (Angular → ClickHouse)

1. **Angular** envoie à `/funds/query` :

   ```json
   {
     "filters": {
       "aum": { "min": 1000000, "max": 50000000 },
       "status": ["Active", "Closed"],
       "region": ["EU", "US"],
       "inceptionDate": { "from": "2015-01-01", "to": "2020-12-31" }
     },
     "sort": { "column": "aum", "direction": "desc" },
     "page": 1,
     "pageSize": 50,
     "viewId": "default"
   }
   ```

2. **Spring Boot** :
   - lit la metadata des colonnes,
   - génère SQL ClickHouse (WHERE, ORDER BY, LIMIT/OFFSET),
   - exécute :
     - une requête pour les données paginées,
     - des requêtes pour les agrégations (min/max, distinct_count…).

3. **DataWeave** assemble `{data, meta}`.

4. **Angular** met à jour :
   - tableau,
   - filtres (sliders, listes, datepickers),
   - pagination.

---

## 5. Metadata des colonnes

**Exemple :**

```json
{
  "columns": [
    {
      "id": "fund_name",
      "label": "Fund Name",
      "dataType": "text",
      "filterType": "search",
      "backendMode": "none",
      "includeInFilters": true,
      "sortable": true
    },
    {
      "id": "aum",
      "label": "AUM",
      "dataType": "number",
      "filterType": "range",
      "backendMode": "minmax",
      "aggregation": { "min": true, "max": true },
      "includeInFilters": true,
      "sortable": true
    },
    {
      "id": "status",
      "label": "Status",
      "dataType": "category",
      "filterType": "multiselect",
      "backendMode": "distinct_count",
      "aggregation": { "distinct": true, "count": true },
      "includeInFilters": true,
      "sortable": true
    }
  ]
}
```

---

## 6. Sécurité

- **Accès base locale** :
  - Postgres et ClickHouse accessibles uniquement sur le réseau interne.
- **API Spring Boot** :
  - sécurisée (JWT / OAuth2 / API Key selon contexte).
- **Données** :
  - chiffrement disque (optionnel),
  - logs contrôlés (pas de données sensibles en clair).

---

## 7. Non-fonctionnel

- **Performance** :
  - ClickHouse dimensionné pour > 500M lignes.
  - Requêtes analytiques < 1–2 secondes.
- **Scalabilité** :
  - ClickHouse peut être déployé en cluster.
  - Spring Boot scalable horizontalement.
- **Évolutivité** :
  - Ajout de colonnes = mise à jour :
    - Materialized Views Postgres,
    - schéma ClickHouse,
    - metadata.
  - Angular reste générique et metadata‑driven.
- **Observabilité** :
  - logs Spring Boot,
  - métriques ClickHouse (requêtes, CPU, IO),
  - supervision ETL (jobs, erreurs).

---

## 8. Résultat

Cette architecture te donne :

- Postgres comme **source de vérité métier**,
- Materialized Views comme **zone de pré‑traitement analytique**,
- ClickHouse comme **moteur Big Data**,
- Spring Boot + DataWeave comme **moteur de requêtes et de transformation**,
- Angular comme **UI metadata‑driven**.

Tu obtiens un Fund Explorer local, performant, extensible, et prêt à monter en volume sans changer de paradigme.
