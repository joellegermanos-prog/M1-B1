# Expériences — M1-B1 Pyrenex Crédit (Lending Club)

> Trace tes runs au fur et à mesure. Format imposé : un bloc par run, avec
> date, modèle, hyperparams, métriques **test interne uniquement**, verdict.
> Commit à chaque run final (pas à chaque essai jetable).
>
> ⚠️ **Règle d'or — comparabilité.** Le holdout **n'apparaît jamais** dans les
> blocs `exp_NNN`. Il sort **une seule fois**, pour le modèle retenu, dans
> la section finale en bas de fichier. Cf. mini-cours 04.

---

## exp_001 — RF par défaut

- **Date** : 2026-06-02 14:15
- **Modèle** : RandomForestClassifier (sklearn 1.6.1)
- **Dataset** : lending_club_train.csv, n=24000 (split 19200 train / 4800 val)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : tous par défaut, `n_estimators=100`, `class_weight=None`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : SimpleImputer + OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (validation interne)** :
  - F1 macro : 0.5135
  - F1 défaut (Charged Off) : ~0.0810
  - Recall défaut : ~0.0450
- **Temps d'entraînement** : 6 s
- **Verdict** : Modèle inutilisable en l'état. L'algorithme souffre d'un angle mort sévère : il maximise l'exactitude globale en prédisant massivement la classe majoritaire (`Fully Paid`). Le Recall sur le risque de défaut est dramatique pour la Direction des Risques.

---

## exp_002 — RF balanced (Jeu B)

- **Date** : 2026-06-02 15:30
- **Modèle** : RandomForestClassifier (sklearn 1.6.1)
- **Dataset** : lending_club_train.csv, n=24000 (split 19200 train / 4800 val)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=100`, `class_weight='balanced'`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : SimpleImputer + OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (validation interne)** :
  - F1 macro : 0.5011
  - F1 défaut (Charged Off) : 0.1000
  - Recall défaut : 0.0600
- **Temps d'entraînement** : 6 s
- **Verdict** : L'activation de la compensation `balanced` force les arbres à prêter une attention mathématique plus forte à la classe minoritaire. Le Recall progresse très légèrement (de 4.5% à 6%), mais cela dégrade le F1 macro global qui tombe à 0.5011. L'angle mort persiste malgré la pénalisation.

---

## exp_003 — (mission étoile ⭐ )

- **Date** : 2026-06-02 16:45
- **Modèle** : HistGradientBoostingClassifier (sklearn 1.6.1)
- **Dataset** : lending_club_train.csv, n=24000
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `class_weight='balanced'`, `random_state=42`
- **Pré-traitement** : SimpleImputer + OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (validation interne)** :
  - F1 macro : 0.5245
  - Recall défaut : 0.0820
- **Temps d'entraînement** : 21 s (incluant l'initialisation du package SHAP)
- **Verdict** : Le Gradient Boosting construit ses arbres en série et s'avère structurellement supérieur à la Forêt Aléatoire sur ce paysage de données (F1 macro à 0.5245). C'est le framework retenu pour l'extraction de l'explicabilité globale et locale via les plots SHAP.

---

## 🏁 Évaluation finale sur holdout (modèle retenu)

> **À remplir une seule fois**, à la tâche 5 du brief, **après** avoir choisi
> ton modèle retenu parmi les `exp_NNN` ci-dessus. Le holdout n'est consulté
> qu'ici.

- **Date** : 2026-06-02 18:00
- **Expérience retenue** : exp_002 (Pipeline `v2.0.0` basé sur le RandomForest original corrigé)
- **Modèle persisté** : `models/pyrenex_risk_v2.joblib`
- **Données holdout** : `lending_club_holdout.csv` (n=6000)
- **Métriques** :
  - F1 macro : 0.4926
  - ROC-AUC : 0.7177 
  - F1 défaut (Charged Off) : 0.0900
  - Recall défaut : 0.0500
- **Matrice de confusion** :

|  | Pred Charged Off | Pred Fully Paid |
|---|---|---|
| **Vrai Charged Off** | 53 | 1050 |
| **Vrai Fully Paid** | 54 | 4843 |

- **Comparaison baseline 2017** : (cf. `verdict.md`)