# BottleNeck – Contrôle qualité automatisé des données

Projet de formation OpenClassrooms (Business Data Analyst) : amélioration, avec l'appui de l'IA, du projet 6 consacré au rapprochement et à l'analyse des données de BottleNeck, caviste en ligne.

## Le besoin

BottleNeck pilote son activité à partir de trois exports mensuels : l'ERP (prix, stock, prix d'achat), l'export WordPress du site (ventes, descriptions) et une table de liaison maintenue à la main. Dans le projet 6, la qualité de ces données était vérifiée manuellement, une seule fois, et les lignes en anomalie étaient supprimées avant le calcul des indicateurs présentés au CODIR.

Ce projet transforme ces vérifications en 15 règles de validation automatiques et réutilisables, écrites avec Pandera, selon un principe simple : **signaler sans supprimer**.

## Principaux résultats

| Résultat | Valeur |
|---|---|
| Anomalies détectées sur l'export d'octobre | 1 170, réparties sur 15 règles |
| Taux de détection sur un export volontairement dégradé | 100 % (13 erreurs injectées) |
| Chiffre d'affaires web corrigé | 143 680,10 € (P6 : 143 324,10 €) |
| Part du CA des 20 % meilleurs produits | 40,2 % (P6 : 44,1 %) |
| Ventes à perte non repérées dans le P6 | 4 produits, −6 425,51 € de marge latente |

Le « produit fantôme » signalé dans le projet 6 était en réalité un effet de bord de son propre nettoyage : un produit existant, dont le stock était saisi à -1, avait été supprimé avec ses 8 ventes.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `bottleneck_qualite.ipynb` | Livrable principal : règles, contrôle, rapport et recalcul des indicateurs |
| `bottleneck_qualite_construction.ipynb` | Journal de construction pas à pas des règles |
| `export_degrade.ipynb` | Création de l'export dégradé pour le test de robustesse |
| `test_chatgpt_promptB.ipynb` | Test du code généré par ChatGPT sur les données réelles |
| `test_great_expectations.ipynb` | Mini-test de Great Expectations, écarté au profit de Pandera |
| `journal_ia.md` | Traçabilité des essais IA : outils, prompts, résultats, décisions |
| `erp.xlsx`, `web.xlsx`, `liaison.xlsx` | Exports d'octobre (données fictives) |
| `*_degrade.xlsx`, `injections.csv` | Export dégradé et journal des erreurs injectées |
| `rapport_anomalies.xlsx` | Rapport d'anomalies destiné à l'équipe métier |
| `requirements.txt` | Versions figées des dépendances |

## Exécution

Environnement testé : Python 3.13 sur macOS.

1. Cloner le dépôt, puis installer les dépendances : `pip install -r requirements.txt`
2. Ouvrir `bottleneck_qualite.ipynb` dans Jupyter.
3. Lancer *Kernel > Restart Kernel and Run All Cells*.

Le notebook relit les fichiers Excel du dossier, applique les règles, régénère `rapport_anomalies.xlsx` et recalcule les indicateurs. L'export dégradé est déjà fourni ; pour le recréer, exécuter `export_degrade.ipynb`, dont le tirage est reproductible (`random_state=42`).

## Choix techniques

Pandera a été retenu après une veille comparant l'approche manuelle, Pandera et Great Expectations, puis confirmé par un mini-test : Great Expectations détecte les mêmes erreurs, mais il est plus lourd à mettre en place et compte trois fois plus de dépendances pour un contrôle mensuel de trois fichiers. Le code généré par ChatGPT à partir d'un prompt structuré a été testé sur les données réelles : il annonçait 5 916 anomalies contre 1 170 réelles, et une construction guidée, testée règle par règle, a été préférée. Le détail est tracé dans `journal_ia.md`.

## Limites

Les règles n'ont été testées que sur un mois de données réelles. La règle sur les ventes à perte nécessite la liste des déstockages validés par l'équipe métier, et le rapport ne distingue pas encore les anomalies d'origine de leurs effets de cascade.
