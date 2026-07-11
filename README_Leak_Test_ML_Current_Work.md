# Projet Machine Learning — Classification des données Leak Test

## 1. Objectif du projet

L’objectif est de développer un modèle de Machine Learning capable de classifier automatiquement les données issues d’une machine **Leak Test**.

Le dataset d’entraînement contient une colonne cible appelée `quality_class`, avec les classes suivantes :

- `Excellent`
- `Good`
- `Acceptable`
- `Rework`
- `NOK`

Le défi principal est que les nouvelles données réelles provenant de la machine ne contiennent pas la colonne `quality_class`. Le modèle doit donc prédire cette classe automatiquement et créer une nouvelle colonne `quality_class` dans le fichier final.

---

## 2. Workflow du projet

```text
Analyse des données
        ↓
Nettoyage des données
        ↓
Préparation pour le Machine Learning
        ↓
Entraînement des modèles
        ↓
Évaluation des performances
        ↓
Analyse de l’importance des variables
        ↓
Choix du meilleur modèle
        ↓
Prédiction sur nouvelles données machine
```

---

## 3. Colonnes du dataset

Après renommage, les colonnes utilisées sont :

```text
reference
channel
result
leak_rate
pressure
alarm
date
time
flow_leak_pa_s
temperature
atm_pressure
quality_class
```

La colonne cible est :

```text
quality_class
```

Les autres colonnes représentent les paramètres machine.

---

## 4. Analyse des données réalisée

Les étapes d’analyse réalisées sont :

- Vérification du nombre de lignes et de colonnes.
- Affichage des noms des colonnes.
- Vérification des types de données.
- Analyse des valeurs manquantes.
- Vérification des doublons.
- Analyse de la distribution de `quality_class`.
- Analyse graphique de `pressure` et `leak_rate`.
- Détection des valeurs négatives, nulles et aberrantes.
- Heatmap de corrélation.
- Première analyse de l’importance des variables.

Les paramètres les plus liés à la classification qualité semblent être :

```text
leak_rate
pressure
result
alarm
flow_leak_pa_s
```

---

## 5. Nettoyage des données réalisé

### 5.1 Renommage des colonnes

Les noms des colonnes ont été simplifiés :

```text
leak_rate (cm3/mn)  → leak_rate
pressure (bar)      → pressure
Flow_leak (Pa/s)    → flow_leak_pa_s
temperature  (°C)   → temperature
atm_pressure  (HPa) → atm_pressure
```

### 5.2 Suppression des doublons

Les lignes dupliquées ont été vérifiées puis supprimées si nécessaire.

### 5.3 Traitement des valeurs manquantes dans `leak_rate`

Les valeurs manquantes dans `leak_rate` ont été traitées selon la logique industrielle suivante :

```text
Si alarm = LARGE LEAK TEST et leak_rate est vide
→ leak_rate = 5.0
```

Justification : `5.0` est supérieur à la limite industrielle de fuite, donc cela représente une fuite importante.

```text
Si alarm = PRESSURE LOW et leak_rate est vide
→ leak_rate = 0.0
```

Justification : dans ce cas, le problème vient d’une pression insuffisante. Le test de fuite n’est pas valide et la classe associée est généralement `Rework`.

Une vérification a été faite pour confirmer qu’il n’existe pas de valeurs manquantes de `leak_rate` non liées à `LARGE LEAK TEST` ou `PRESSURE LOW`.

### 5.4 Traitement de `flow_leak_pa_s`

La colonne `flow_leak_pa_s` contenait 115 valeurs manquantes :

```text
98 cas liés à LARGE LEAK TEST
17 cas liés à PRESSURE LOW
```

Ces valeurs ont été traitées à partir de la relation observée entre `leak_rate` et `flow_leak_pa_s` :

```text
conversion_factor = flow_leak_pa_s / leak_rate
flow_leak_pa_s = leak_rate × conversion_factor
```

Le facteur de conversion est calculé à partir des lignes valides du dataset. Il s’agit d’une estimation basée sur les données, pas d’une conversion physique universelle.

### 5.5 Nettoyage de la colonne `alarm`

Les valeurs vides dans la colonne `alarm` ont été remplacées par :

```text
NO_ALARM
```

Cela signifie qu’aucune alarme n’a été détectée pendant le test.

### 5.6 Valeurs négatives

Les valeurs négatives dans les colonnes numériques, notamment `leak_rate` et `flow_leak_pa_s`, ont été vérifiées. Une fuite négative n’ayant pas de signification physique, ces valeurs sont corrigées à `0`.

### 5.7 Nettoyage des colonnes catégorielles

Les colonnes textuelles ont été standardisées :

```text
result
alarm
reference
quality_class
```

Exemple :

```text
(OK) → OK
(TD) → TD
(AL) → AL
```

---

## 6. Préparation pour le Machine Learning

La colonne cible est :

```text
y = quality_class
```

Les variables d’entrée sont :

```text
X = paramètres machine sans quality_class
```

Les colonnes `date` et `time` ont été retirées car elles ne représentent pas des paramètres physiques de qualité.

Le dataset a été divisé comme suit :

```text
80 % entraînement
20 % test
```

Le paramètre `stratify` a été utilisé pour garder la même distribution des classes dans les ensembles d’entraînement et de test.

---

## 7. Modèles entraînés

Les modèles entraînés sont :

- Random Forest
- SVM
- XGBoost
- Logistic Regression

Résultats observés jusqu’à maintenant :

```text
Random Forest : accuracy = 100 %
SVM           : accuracy ≈ 92 %
XGBoost       : accuracy = 100 %
```

---

## 8. Analyse de l’importance des variables

### 8.1 Observation avec Random Forest

Random Forest a parfois donné une importance à des variables comme :

```text
reference
channel
```

Cela n’est pas totalement logique industriellement, car ces colonnes sont plutôt des informations d’identification ou de ligne/process, et non des mesures directes de qualité.

### 8.2 Amélioration proposée

Une version améliorée a été proposée en supprimant :

```text
reference
channel
date
time
```

L’objectif est de forcer le modèle à apprendre à partir des paramètres réellement liés à la qualité :

```text
leak_rate
pressure
result
alarm
temperature
atm_pressure
flow_leak_pa_s
```

### 8.3 Observation avec XGBoost

L’analyse de XGBoost semble plus cohérente, car le modèle apprend principalement à partir de :

```text
leak_rate
result
pressure
```

Ces variables sont plus logiques dans le contexte industriel du Leak Test.

---

## 9. Point à valider avec l’encadrant

Un doute reste à clarifier concernant les métriques.

Random Forest et XGBoost donnent une accuracy de 100 %, même après plusieurs essais et après suppression de certains paramètres non nécessaires.

Question principale à discuter :

```text
Est-ce que ces métriques de 100 % sont normales,
étant donné que quality_class est fortement liée aux paramètres machine,
ou bien est-ce que cela peut indiquer un risque de surapprentissage
ou de fuite d’information ?
```

Deuxième point à valider :

```text
Faut-il garder les colonnes result et alarm dans le modèle final,
puisqu’elles existent dans les nouvelles données machine,
ou faut-il entraîner un modèle basé uniquement sur les mesures physiques ?
```

---

## 10. Prochaines étapes

Les prochaines étapes prévues sont :

1. Valider les variables finales à utiliser.
2. Réentraîner les modèles avec les colonnes sélectionnées.
3. Comparer les modèles dans un tableau final.
4. Choisir le meilleur modèle.
5. Sauvegarder le modèle final.
6. Tester le modèle sur un nouveau fichier machine sans `quality_class`.
7. Générer un fichier final contenant les données machine + la colonne prédite `quality_class`.

---

## 11. Résultat attendu final

Le système final doit fonctionner ainsi :

```text
Nouveau fichier machine sans quality_class
        ↓
Prétraitement automatique
        ↓
Prédiction par le modèle entraîné
        ↓
Création de la colonne quality_class
        ↓
Export du fichier final classifié
```

Exemple attendu :

```text
leak_rate | pressure | result | alarm             | quality_class
0.35      | 2.65     | OK     | NO_ALARM          | Excellent
1.45      | 2.66     | OK     | NO_ALARM          | Good
2.70      | 2.62     | OK     | NO_ALARM          | Acceptable
5.00      | 2.73     | TD     | LARGE LEAK TEST   | NOK
0.00      | 2.45     | AL     | PRESSURE LOW      | Rework
```

---

## 12. Notes importantes

- Les résultats à 100 % ne doivent pas être présentés directement comme une perfection du modèle.
- Il faut expliquer qu’ils sont probablement dus à la forte relation entre les règles industrielles et les variables machine.
- L’analyse de l’importance des variables est nécessaire pour vérifier si le modèle apprend à partir de paramètres logiques.
- XGBoost semble actuellement plus interprétable que Random Forest.
- La validation avec l’encadrant est nécessaire avant de choisir le modèle final.
