Fonctionnalités ajoutées
1. Drag & drop des colonnes
Fonctionnement :

Ouvrir le column picker (icône utility:table dans la toolbar)
Glisser-déposer les lignes pour réordonner les colonnes
L'ordre est immédiatement reflété dans la lightning-datatable
Détails techniques :

Chaque item du picker a draggable="true" + les handlers ondragstart, ondragover, ondragleave, ondrop, ondragend
Le drop réordonne _allColumns → visibleTableColumns (getter) reflète le nouvel ordre automatiquement
Highlight visuel (drag-over CSS class) sur la cible pendant le glisser
2. Sauvegarde de la config par utilisateur
Fonctionnement :

Après chaque réordonnancement ou changement de visibilité de colonne, la config est auto-sauvegardée (debounce 800 ms)
Un badge "Layout saved" apparaît brièvement dans la toolbar après chaque sauvegarde
Au prochain chargement du composant, l'ordre et la visibilité des colonnes sont restaurés
Architecture :

Fichier	Rôle
IMLSheetUserConfig__c.object-meta.xml	Custom Setting Hierarchy (un enregistrement par user)
ConfigJSON__c.field-meta.xml	LongTextArea 32k — JSON map { sheetId → { columnOrder, visibleColumns } }
IMLSheetService.cls	getUserConfig(sheetId) + saveUserConfig(sheetId, configJson)
Format JSON stocké :


{
  "mock-dataset-001": {
    "columnOrder": ["code", "name", "credit_limit", ...],
    "visibleColumns": ["code", "name", ...]
  }
}
A déployer : le Custom Setting IMLSheetUserConfig__c doit être déployé dans l'org avant de pousser le code (sf project deploy start).
