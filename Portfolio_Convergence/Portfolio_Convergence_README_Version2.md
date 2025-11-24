# Portfolio Convergence – Mode d’emploi

Ce pack fournit un exemple industrialisable pour piloter la convergence technologique d’un portefeuille de produits.

## Contenu

- `Products.csv` : Liste des produits et leur pondération de criticité.
- `Criteria.csv` : Référentiel des critères d’architecture/maturité.
- `Scores.csv` : Historique des scores (snapshots trimestriels).
- `DebtRegister.csv` : Backlog de résorption de dettes techniques.
- `build_excel.py` : Script Python qui génère un fichier Excel multi-feuilles enrichi.
- Workflow GitHub Actions : Génère et attache l’Excel aux artefacts à chaque push sur `master` et (optionnel) le commit dans le repo.

## 1. Import manuel dans Excel (option hors automation)
1. Ouvrez Excel > Données > À partir du texte/CSV et importez chaque fichier.
2. Convertissez chaque feuille en Tableau (Ctrl+T).
3. Créez une feuille `Calculations` pour y placer les formules de KPI.

### Formules KPI

TCI (Technology Convergence Index) – Moyenne pondérée normalisée sur 5 :
```
=LET(
  pid, A2,
  snap, B2,
  num, SUMIFS(Scores[score_weighted], Scores[product_id], pid, Scores[snapshot_date], snap),
  den, SUMIFS(Scores[criterion_weight], Scores[product_id], pid, Scores[snapshot_date], snap),
  tci, IF(den=0,0,num/den),
  tci_pct, tci/5*100
)
```

SAI (Standard Adoption Index) – % de critères avec score >= 4 :
```
=LET(
  pid, A2,
  snap, B2,
  ok, COUNTIFS(Scores[product_id], pid, Scores[snapshot_date], snap, Scores[score], ">=4"),
  tot, COUNTIFS(Scores[product_id], pid, Scores[snapshot_date], snap),
  IF(tot=0,0,ok/tot*100)
)
```

ACI (Automation Coverage Index) – % de critères automatisés :
```
=LET(
  tot, COUNTA(Criteria[criterion_id]),
  auto, COUNTIFS(Criteria[automation_type],"automated"),
  IF(tot=0,0,auto/tot*100)
)
```

REI (Risk Exposure Index – simplifié) – Nombre de dettes high ouvertes :
```
=COUNTIFS(DebtRegister[product_id],A2,DebtRegister[risk_level],"high",DebtRegister[status],"<>done")
```

DBV (Debt Burn Velocity) – % effort dette clôturé sur le quarter :
```
=LET(
  pid, A2,
  qtr, B2,
  closed, SUMIFS(DebtRegister[estimate_points],DebtRegister[product_id],pid,DebtRegister[quarter_target],qtr,DebtRegister[status],"done"),
  planned, SUMIFS(DebtRegister[estimate_points],DebtRegister[product_id],pid,DebtRegister[quarter_target],qtr),
  IF(planned=0,0,closed/planned*100)
)
```

## 2. Génération automatique de l’Excel

Le script `build_excel.py` :
- Lit les CSV.
- Calcule TCI par produit/snapshot.
- Ajoute une feuille KPI regroupée.
- Applique un formatage simple (styles de colonnes).
- Sauvegarde `Portfolio_Convergence.xlsx`.

Le workflow GitHub Actions :
- S’exécute sur chaque push sur `master`.
- Installe Python + dépendances.
- Lance le script.
- Publie l’artefact Excel.
- (Optionnel) Commit auto du fichier généré (activable en décommentant la section correspondante).

## 3. Personnalisation

- Ajouter des colonnes: `Products.csv` (ex. `owner`, `business_unit`).
- Étendre les critères: maintenir un volume maîtrisé (≤ 80).
- Ajouter une feuille « Roadmap » générée via une agrégation de `DebtRegister`.
- Intégrer une colonne `external_ref` (Jira / Azure DevOps) dans `DebtRegister.csv`.

## 4. Bonnes pratiques

- Append-only pour `Scores.csv` : chaque nouveau quarter = nouvelles lignes (nouveau `snapshot_date`).
- Ne jamais réécrire l’historique pour préserver les tendances.
- Automatiser autant que possible la production des colonnes `score` et `evidence_url`.
- Utiliser des contrôles de validation (ex: listes déroulantes pour `risk_level` et `status` si édition manuelle).

## 5. Activation du commit automatique

Dans le workflow YAML, décommente le bloc « Commit generated Excel back to repo » et ajoute un secret `BOT_PAT` si tu veux pousser automatiquement le fichier généré sans intervention.

## 6. Évolution possible

- Ajout d’un script pour produire des graphiques (matplotlib → images intégrées).
- Génération d’un tableau croisé « Heatmap Domaines » directement dans Excel via `openpyxl`.
- Export complémentaire JSON pour ingestion dans un datalake.

## 7. Structure cible

```
Portfolio_Convergence/
  README.md
  Products.csv
  Criteria.csv
  Scores.csv
  DebtRegister.csv
  build_excel.py
.github/
  workflows/
    build-portfolio-convergence.yml
```

## 8. Licence / Usage

Adapter selon la politique interne (ajouter un fichier LICENSE si nécessaire). Considérer marquer les données comme fictives si le dépôt est public.

---

Pour toute extension (ex: ajout d’un modèle prédictif pour estimer la convergence future), je peux te fournir un script supplémentaire. Dis-moi si tu veux aussi un export JSON ou une version allégée.

Bon pilotage !