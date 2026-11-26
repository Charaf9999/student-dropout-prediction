# Prédiction du décrochage et de la réussite des étudiants

Mini-projet M113 (Apprentissage automatique), Master IDIA, FST Tanger.

Le jeu de données vient de l'UCI : *Predict Students' Dropout and Academic Success*. Il décrit 4 424 étudiants. La cible `Target` vaut **Dropout** (abandon), **Enrolled** (toujours inscrit) ou **Graduate** (diplômé).

| Partie | Sujet | Membre | Notebook |
| --- | --- | --- | --- |
| 1 | Préparation des données, arbre CART en NumPy, sélection des variables | Charaf Eddine Ifrinchaou | `01_preparation_arbre.ipynb` |
| 2 | Régression logistique et SVM (linéaire, RBF) | Chemseddine Ayoub | `classification.ipynb` |
| 3 | K-means et profils d'étudiants | Adam Chaara | `03_kmeans.ipynb` |

## Livrables

| Fichier | Contenu |
| --- | --- |
| `Rapport_M113_MiniProjet.docx` / `.pdf` | Rapport final du groupe |
| `M113_MiniProjet.ipynb` | Notebook final : les trois parties dans l'ordre |
| `PRESENTATION.md` | Fiche pour préparer la soutenance |
| `rapport_partie1.md`, `rapport_partie2.md`, `rapport_partie3.md` | Contributions détaillées de chaque membre |

## Organisation du dépôt

```
data/raw/data.csv        fichier UCI original
shared/                  données communes écrites par la partie 1 (contrat version 1)
results/task1/           arbre : comparaison, importances, figures
results/task2/           classification : validation, test, prédictions, figures
results/task3/           K-means : choix de k, initialisations, profils, figures
```

Toutes les parties utilisent les mêmes données :
- même découpage stratifié 60/20/20 avec la graine 42, soit 2 653 étudiants en train, 883 en validation et 888 en test ;
- mêmes 11 variables, choisies par l'arbre.

## Exécuter le projet

1. Installer les bibliothèques : `python -m pip install -r requirements.txt`.
2. Ouvrir le dossier `student-dropout-prediction` dans VS Code. Il faut ouvrir ce dossier-là, pas le dossier parent.
3. Ouvrir `M113_MiniProjet.ipynb`, choisir l'environnement Python avec **Select Kernel**, puis lancer **Restart** et **Run All**. L'exécution prend environ 1 minute.

Le projet peut aussi s'exécuter en ligne de commande, depuis le dossier du projet :

```bash
python -m jupyter nbconvert --to notebook --execute --inplace M113_MiniProjet.ipynb
```

Les notebooks individuels s'exécutent dans l'ordre `01` → `classification` → `03`. Toutes les graines sont fixées, donc les fichiers de `results/` sont reproduits à l'identique. Seules changent les durées mesurées.

## Résultats principaux

| Méthode | Résultat |
| --- | --- |
| Arbre CART (NumPy) | Le pré-élagage (profondeur 8, au moins 5 étudiants par feuille) fait passer le F1 macro de validation de 0,593 à 0,674. 11 variables gardent 90 % de l'importance. La plus importante est le nombre d'unités validées au 2ᵉ semestre. |
| Régression logistique pondérée | Meilleur F1 macro sur le test : 0,707 (exactitude 0,747). |
| SVM linéaire pondéré / SVM RBF | F1 macro sur le test : 0,696 / 0,646. Le SVM linéaire a le meilleur rappel sur `Enrolled` (0,67). |
| K-means (k = 3) | Silhouette de 0,445. Un groupe à risque compte 83 % de décrocheurs, soit 2,6 fois le taux global. Il se retrouve sur la validation et le test. |

**Limite principale.** Les variables du 2ᵉ semestre sont utilisées, donc l'analyse est rétrospective. La classe `Enrolled` reste la plus difficile à reconnaître pour les trois méthodes.

## Sources

- Realinho, V., Vieira Martins, M., Machado, J., & Baptista, L. (2021). *Predict Students' Dropout and Academic Success*. UCI Machine Learning Repository. https://doi.org/10.24432/C5MC89 (licence CC BY 4.0).
- Realinho, V., Machado, J., Baptista, L., & Martins, M. V. (2022). *Predicting Student Dropout and Academic Success*. Data, 7(11), 146. https://doi.org/10.3390/data7110146

La bibliographie complète se trouve dans le rapport.
