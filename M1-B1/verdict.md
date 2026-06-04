# Verdict — Modèle de scoring Pyrenex Crédit v2

> Document destiné à Sophie Léger (Lead Data, Pyrenex Crédit).
> 1 page max.

## Contexte

Ce travail vise à ré-entraîner et moderniser le modèle historique de 2017 afin de lever son angle mort critique : l'incapacité à détecter les clients en défaut de paiement (`Charged Off`) sur les nouvelles cohortes de demandes de crédit.

## Démarche

L'évaluation a été menée sur le nouveau dataset Lending Club ($n=24000$), scindé de manière stratifiée en 80% entraînement et 20% validation interne pour tester deux configurations majeures du framework `RandomForestClassifier` (Standard vs Balanced). La validation finale et indépendante a été matérialisée sur un échantillon "Holdout" secret et isolé de 6 000 clients. Les performances ont été arbitrées au regard du score F1-Macro et de la sensibilité (Recall) sur la classe à risque.

## Verdict chiffré

|| Métrique | Baseline 2017 (Pyrenex-risk-v1) | Modèle retenu (v2 - Jeu B) | Variation |
|---|---|---|---|
| **F1 macro (holdout)** | 0.5018 | 0.4926 | 🔴 -0.0092 |
| **F1 défaut** | 0.0800 | 0.0900 | 🟢 +0.0100 |
| **ROC-AUC** | 0.7296 | 0.7177 | 🔴 -0.0119 |
| **Recall défaut** | 0.0500 | 0.0500 | ⚪ 0.0000 |

**Configuration retenue** : `RandomForestClassifier(n_estimators=100, class_weight='balanced', random_state=42, n_jobs=-1)`.

## Trade-off explicité au métier

L'introduction de la pénalisation mathématique `class_weight='balanced'` s'avère insuffisante sur ce paysage de données. Le modèle v2 ne parvient pas à capturer les clients insolvables : le Rappel défaut stagne à un niveau critique de 5%, ce qui signifie que la banque laisse passer 95% des profils à risque (1 050 non-détectés sur 1 103). Tenter de forcer la détection via cette configuration détruit la précision globale et fait régresser le F1-Macro sous la barre des 0.50 ($0.4926$), n'offrant aucun gain financier ou sécuritaire par rapport à la situation actuelle.

## Précautions avant mise en production

- Vérifier que le **schéma d'entrée** en production correspond exactement
  au schéma d'entraînement (cf. `pyrenex_risk_v2.json` → `feature_columns`)
- Re-évaluer le **seuil de décision** (0.5 par défaut) avec l'équipe
  métier — un seuil 0.3 peut être plus adapté selon l'appétence au risque
- Mettre en place un **monitoring** dès le déploiement (cf. M5/M6)
- Surveiller les **variables sensibles** identifiées (FICO, état US,
  revenu) — risque de disparate impact à auditer (M2/M7)

## Recommandation

✅ **Remplacer Pyrenex-risk-v1** par v2 *OU* ⛔ **Ne pas remplacer** —
le modèle v2 échoue techniquement à corriger l'angle mort sur le risque (Recall figé à 5%) et dégrade la performance globale sur le holdout ($F1 < 0.50$), ce qui nécessite un gel du déploiement en attendant un enrichissement du signal via de nouvelles variables d'endettement.

---

*Signé : Joelle Germanos, FastIA, le 2026-06-03*
