# Fiche de soutenance — M113

Cette fiche sert à préparer la soutenance. Le plan dure environ 12 minutes : 1 min d'introduction, 3 min par partie, 2 min de conclusion. Les figures à montrer se trouvent toutes dans `results/` et dans le rapport.

## 0. Introduction (1 min) — un membre au choix

- **Problème.** Peut-on repérer à l'avance les étudiants qui vont abandonner ?
- **Données.** 4 424 étudiants d'un établissement portugais (UCI) et 36 variables : démographie, situation financière, admission, notes des deux premiers semestres.
- **Cible à 3 classes.** Dropout (32 %), Enrolled (18 %), Graduate (50 %). Les classes sont déséquilibrées, d'où le choix du **F1 macro**, qui donne le même poids aux trois classes.
- **Organisation.** Une partie par membre. La partie 1 prépare les données pour tout le monde (dossier `shared/`), et les trois parties utilisent les mêmes étudiants et les mêmes variables.

## 1. Arbre de décision — Charaf (3 min)

À montrer : `comparaison_arbres.png`, puis `importances_variables.png`.

1. **Préparation.**
   - Découpage 60 % train, 20 % validation, 20 % test, stratifié, avec la graine 42.
   - Les variables catégorielles sont encodées en one-hot, ce qui donne 225 colonnes.
   - Aucune valeur manquante.
2. **Arbre CART codé en NumPy, sans scikit-learn.**
   - À chaque nœud, on teste toutes les coupures possibles et on garde celle qui réduit le plus l'impureté de Gini, G = 1 − Σ p².
3. **Amélioration : le pré-élagage.**
   - L'arbre sans limite apprend tout le train par cœur (F1 = 1,00) mais tombe à 0,59 en validation : c'est du surapprentissage.
   - En limitant la profondeur à 8 et en imposant au moins 5 étudiants par feuille, on obtient **0,674 en validation (+8 points)**, avec 5 fois moins de nœuds.
4. **Sélection des variables.**
   - On garde les variables qui totalisent 90 % de l'importance : il en reste **11**.
   - La plus importante, de loin (58 %), est le **nombre d'unités validées au 2ᵉ semestre**.

## 2. Régression logistique et SVM — Chemseddine (3 min)

À montrer : le tableau des résultats sur le test, `confusion_matrices.png`, puis `roc_curves.png`.

1. **Mise à l'échelle.** On standardise avec un scaler appris sur le train seulement, pour ne rien apprendre du test.
2. **Trois modèles**, chacun réglé sur la validation (14 configurations au total) :
   - la régression logistique ;
   - le SVM linéaire ;
   - le SVM à noyau RBF (C et gamma).
3. **Amélioration : `class_weight="balanced"`.** Les classes rares, surtout `Enrolled`, reçoivent plus de poids. Cela aide les deux modèles linéaires (+2 points) mais pas le RBF.
4. **Résultats sur le test, mesurés une seule fois :**
   - régression logistique pondérée : **F1 macro 0,707** (meilleur modèle) ;
   - SVM linéaire : 0,696 ;
   - SVM RBF : 0,646.
5. **Message.**
   - Dropout et Graduate sont bien reconnus.
   - **Enrolled est la classe difficile** : le RBF n'en retrouve que 24 %, le SVM linéaire 67 %.

## 3. K-means — Adam (3 min)

À montrer : `choix_k.png`, `centroides_heatmap.png`, `labels_par_cluster.png`, puis `pca_clusters.png`.

1. **Principe.** On cherche des groupes d'étudiants **sans donner les étiquettes** à l'algorithme.
2. **Choix de k.** On teste k de 2 à 10. La meilleure silhouette est obtenue pour **k = 3** (0,445), et Davies-Bouldin confirme ce choix.
3. **Amélioration et code personnalisé.**
   - On compare l'initialisation aléatoire à k-means++ sur 20 graines. À k = 3, les deux donnent le même résultat. À k = 8, k-means++ est meilleur et plus stable.
   - Nous avons aussi recodé K-means en NumPy : il donne exactement le même résultat que scikit-learn.
4. **Les trois profils :**
   - **Groupe 0, « en échec dès le S1 »** (18 %) : ces étudiants ne valident presque rien et ont plus de dettes. **83 % décrochent**, soit 2,6 fois la moyenne, et c'est vrai aussi sur la validation et le test.
   - **Groupe 1, « parcours régulier »** (76 %) : environ 5 unités validées par semestre, moyenne de 12,5/20. La plupart sont diplômés.
   - **Groupe 2, « forte charge »** (6 %) : deux fois plus d'unités, des étudiants plus âgés, plus souvent en cours du soir.
5. **Limite.**
   - K-means ne retrouve pas la classe `Enrolled`, qui reste mélangée dans le groupe 1.
   - En utilisant ses groupes pour prédire, on obtient un F1 macro de 0,47 contre 0,71 pour la régression logistique : le clustering décrit des profils mais ne remplace pas un classifieur.

## 4. Conclusion (2 min) — un membre au choix

- Les trois méthodes disent la même chose : **les résultats du premier semestre sont le meilleur signal d'alerte.**
- L'arbre **explique** (quelles variables comptent), la classification **prédit** (F1 de 0,71), et le clustering **décrit** des profils (un groupe à risque à 83 %).
- **Limites :**
  - les variables du 2ᵉ semestre rendent l'analyse rétrospective ;
  - la classe Enrolled reste difficile à reconnaître ;
  - un seul établissement et un seul découpage.
- **Perspectives :**
  - un modèle d'alerte précoce sans le 2ᵉ semestre ;
  - les forêts aléatoires ;
  - la validation croisée.

## Questions probables et réponses courtes

| Question | Réponse |
| --- | --- |
| Pourquoi le F1 macro et pas l'exactitude ? | Les classes sont déséquilibrées. L'exactitude peut être bonne même si on rate presque tous les `Enrolled`. Le F1 macro donne le même poids aux trois classes. |
| Pourquoi trois jeux (train, validation, test) ? | Le train sert à apprendre, la validation à choisir les paramètres, et le test sert une seule fois, à la fin, pour un score honnête. |
| Pourquoi standardiser ? | La régression logistique, les SVM et K-means utilisent des distances ou des coefficients sensibles à l'échelle : sans standardisation, la note d'admission (sur 200) écraserait le nombre d'unités (0 à 20). L'arbre n'en a pas besoin. |
| Pourquoi le scaler est-il appris sur le train seulement ? | Pour ne rien apprendre du test. Sinon, il y aurait une fuite de données. |
| Qu'est-ce que l'impureté de Gini ? | C'est la probabilité de mal classer un étudiant tiré au hasard dans le nœud. Elle vaut 0 si le nœud est pur. |
| Pourquoi le SVM RBF fait-il moins bien ? | Il favorise les grandes classes et ne retrouve que 24 % des `Enrolled`. Ici, des frontières linéaires suffisent. |
| Que mesure la silhouette ? | Si chaque point est plus proche de son groupe que du groupe voisin. Elle va de −1 à 1. Une valeur de 0,45 indique des groupes réels mais qui se touchent. |
| Qu'apporte k-means++ ? | Il choisit des centres de départ éloignés les uns des autres. Les résultats sont plus stables quand k est grand (à k = 8 : ARI 0,91 contre 0,74). |
| Peut-on utiliser ce modèle dès l'inscription ? | Pas tel quel, car il utilise les notes du 2ᵉ semestre. C'est la principale perspective du projet. |
| Qu'avez-vous codé vous-mêmes ? | L'arbre CART complet en NumPy (partie 1) et K-means en NumPy (partie 3). Les deux sont vérifiés sur des exemples, et K-means est aussi comparé à scikit-learn. |

## Chiffres à retenir

| Élément | Valeur |
| --- | --- |
| Effectifs | 4 424 étudiants (2 653 en train, 883 en validation, 888 en test) |
| Variables | 36 variables au départ, 225 colonnes après encodage, 11 retenues |
| Arbre | F1 macro de validation : 0,674 (contre 0,593 sans élagage) |
| Meilleur classifieur | Régression logistique pondérée, F1 macro de test 0,707 |
| K-means | k = 3 ; groupe à risque à 83 % de décrocheurs |
