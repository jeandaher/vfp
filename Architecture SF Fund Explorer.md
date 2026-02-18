# **📘 DOCUMENT D’ARCHITECTURE — Fund Explorer Salesforce + ClickHouse**

## **1. Objectif du système**

Mettre en place un **Fund Explorer** intégré à Salesforce, capable d’explorer des données volumineuses (100M+ lignes) avec :

- filtres avancés (range, date, multiselect, search),
- agrégations (min/max, distinct_count),
- tri et pagination performants,
- profils de données dynamiques,
- interface native Salesforce (LWC),
- moteur analytique externe (ClickHouse).

L’objectif est de contourner les limites de SOQL et de Salesforce en matière d’analytics, tout en conservant une expérience utilisateur 100% Salesforce.

---

# **2. Architecture globale**

```
Salesforce (LWC Frontend)
        │
        ▼
Apex Controller (HTTP Callout)
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
ETL Salesforce → ClickHouse
```

---

# **3. Description des composants**

## **3.1 Salesforce LWC (Frontend)**

### Rôle :
- Interface utilisateur principale.
- Tableau dynamique + filtres metadata‑driven.
- Gestion des préférences utilisateur (optionnel).
- Appels Apex pour récupérer `{data, meta}`.

### Fonctionnalités :
- Affichage des colonnes configurables.
- Filtres intelligents selon type de données.
- Pagination, tri, vues sauvegardées.
- UX native Salesforce.

---

## **3.2 Apex Controller**

### Rôle :
- Passerelle entre Salesforce et Spring Boot.
- Effectue les callouts HTTP sécurisés.
- Sérialise/désérialise les payloads JSON.

### Exemple de callout :
```apex
HttpRequest req = new HttpRequest();
req.setEndpoint('callout:FundExplorer/funds/query');
req.setMethod('POST');
req.setHeader('Content-Type', 'application/json');
req.setBody(payloadJson);
```

---

## **3.3 Spring Boot — Fund Explorer API**

### Rôle :
- Orchestrateur central.
- Génère les requêtes analytiques ClickHouse.
- Applique la metadata des colonnes.
- Appelle DataWeave pour assembler `{data, meta}`.

### Endpoints :
- `POST /funds/query`
- `GET /funds/metadata` (optionnel)

### Fonctionnalités :
- Lecture de la metadata des colonnes.
- Construction dynamique de SQL ClickHouse.
- Exécution des requêtes analytiques :
  - données paginées,
  - min/max,
  - distinct_count,
  - histogrammes.
- Gestion des erreurs et timeouts.

---

## **3.4 DataWeave (dans Spring Boot)**

### Rôle :
- Transformation JSON avancée.
- Fusion des résultats ClickHouse.
- Application de la metadata.
- Formatage final pour LWC.

### Sortie :
```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "totalPages": 120,
    "filters": { ... }
  }
}
```

---

## **3.5 ClickHouse (Moteur Analytique)**

### Rôle :
- Stockage analytique haute performance.
- Exécution des filtres complexes.
- Agrégations massives.
- Tri et pagination rapides.

### Caractéristiques :
- Columnar storage.
- Partitionnement (par date, région, etc.).
- Index primaires optimisés.
- Scalabilité horizontale.

### Exemple de table :
```sql
CREATE TABLE funds_analytics (
    fund_id String,
    fund_name String,
    aum Float64,
    status String,
    region String,
    inception_date Date
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(inception_date)
ORDER BY (status, region, aum);
```

---

## **3.6 ETL Salesforce → ClickHouse**

### Rôle :
- Extraire les données Salesforce.
- Les transformer (flatten, normalisation).
- Les charger dans ClickHouse.

### Options :
- MuleSoft (recommandé si déjà disponible).
- Spring Batch (custom, Bulk API).

### Flux :
1. Extraction via Bulk API / CDC.
2. Transformation (mapping, flatten).
3. Chargement ClickHouse (JSONEachRow ou JDBC).

---

# **4. Flux de données complet**

## **4.1 Ingestion Salesforce → ClickHouse**

1. Extraction des objets Salesforce.
2. Transformation (flatten + mapping).
3. Chargement dans ClickHouse.
4. Rafraîchissement périodique (toutes les heures / 15 min / temps réel selon besoin).

---

## **4.2 Requête Fund Explorer (LWC → ClickHouse)**

### Étapes :

1. **LWC** envoie les filtres/tri/pagination à Apex.
2. **Apex** fait un callout vers Spring Boot.
3. **Spring Boot** :
   - lit la metadata,
   - génère SQL ClickHouse,
   - exécute les requêtes analytiques.
4. **DataWeave** assemble `{data, meta}`.
5. **Apex** renvoie la réponse au LWC.
6. **LWC** met à jour le tableau et les filtres.

---

# **5. Metadata des colonnes**

### Exemple :
```json
{
  "id": "aum",
  "label": "AUM",
  "dataType": "number",
  "filterType": "range",
  "backendMode": "minmax",
  "aggregation": { "min": true, "max": true },
  "includeInFilters": true,
  "sortable": true
}
```

### Types supportés :
- `search`
- `contains`
- `range`
- `range_date`
- `distinct_count`
- `none`

---

# **6. Sécurité**

### Authentification :
- OAuth 2.0 JWT Bearer Flow (Salesforce → Spring Boot).

### Autorisations :
- Contrôle des droits côté Salesforce (profils, rôles).
- Optionnel : propagation du contexte utilisateur.

### Réseau :
- Spring Boot exposé en HTTPS.
- IP allowlist Salesforce.

### Données :
- Chiffrement au repos (ClickHouse).
- Masquage éventuel de champs sensibles.

---

# **7. Non-fonctionnel**

### Performance :
- Requêtes analytiques < 1 seconde.
- ClickHouse dimensionné pour > 500M lignes.

### Scalabilité :
- ClickHouse clusterisable.
- Spring Boot scalable horizontalement.

### Observabilité :
- Logs Spring Boot.
- Monitoring ClickHouse (queries, CPU, IO).

### Maintenabilité :
- Ajout de colonnes = mise à jour metadata + schéma ClickHouse.
- LWC reste générique et metadata‑driven.

---

# **8. Résultat final**

Cette architecture permet :

- une **UI Salesforce native** (LWC),
- un **moteur analytique Big Data** (ClickHouse),
- une **orchestration propre** (Spring Boot + DataWeave),
- une **scalabilité sans limite**,
- une **expérience utilisateur fluide** même sur des centaines de millions de lignes.

---
