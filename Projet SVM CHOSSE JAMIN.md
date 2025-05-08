# Prédiction de la qualité du vin

## Ce qu'on a fait

### 1. Analyse exploratoire des données

Les variables de notre jeu de données étaient bien codées et ne présentaient pas de valeurs manquantes. 

Après une étude des statistiques descriptives et des représentations graphiques, nous avons remarqué que plusieurs variables présentent une distribution bimodale.
Cela suggère la présence possible de deux sous-populations distinctes de vins. 

De façon générale, nous observons :
- Une forte dispersion des variables chimiques,
- Peu de corrélations fortes avec la variable cible (qualité).

Cela indique qu’aucune variable seule ne permet de prédire correctement la qualité du vin.  
→ Cela justifie le recours à **des méthodes multivariées**.


Nous avons utilisé des techniques de **clustering** afin de vérifier si les distributions bimodales reflètent la présence de deux types de vins (rouge/blanc ou doux/sec).

#### Méthodes utilisées :
- K-means clustering
- Méthode du coude
- Score de silhouette
- Analyse en Composantes Principales

#### Résultats :
- La méthode du coude ne révèle pas de point d’inflexion clair, mais la courbe diminue nettement jusqu’à k = 4.
- Le score de silhouette est maximal pour k = 2, suggérant que 2 clusters sont optimaux.
- L’ACP confirme une séparation nette :  PC1 : 59.9 %, PC2 : 8.7 %, soit 68.6 % de variance expliquée.

#### Interprétation des clusters :
- **Cluster 1** : vins moins acides, moins sucrés, faibles en soufre, pH plus élevé.
- **Cluster 2** : vins plus acides, plus sucrés, plus concentrés en soufre, pH plus bas.

#### Conclusion intermédiaire

Il existe **deux sous-populations de vins** bien distinctes dans le dataset.  


### 2. Modélisation

#### Séparation des données
- Division en un **jeu d'entraînement (80%)** et un **jeu de test (20%)**.

#### Modèles évalués
- Régression linéaire
- Random Forest
- SVM (Support Vector Machine)
- XGBoost
- Gradient Boosting Regressor
- Bagging Regressor
- KNN (K-Nearest Neighbors)
- MLP Regressor (Réseau de neurones)

#### Critère de sélection
- Comparaison des modèles selon le score R2.

#### Modèle retenu
- **Random Forest** obtient le **meilleur score R2** et est sélectionné pour l’optimisation.


### 3. Optimisation du modèle Random Forest

#### Amélioration des performances après tunage
L’optimisation des hyperparamètres du modèle Random Forest a permis d’obtenir un léger gain en performance :

- **MSE** : réduit de 0.510 à 0.499  
- **R2** : augmenté de 0.485 à 0.496  

#### Analyse partielle (PDP) des variables les plus importantes

Les graphiques de dépendance partielle (PDP) permettent de visualiser l’effet moyen d’une variable sur la prédiction du modèle, toutes choses égales par ailleurs. Voici une synthèse des tendances observées pour les variables les plus influentes :

1. **Densité** : Une densité faible est associée à une qualité inférieure. L’effet devient positif pour des valeurs modérées, avant de se stabiliser. Cela suggère qu’une certaine densité optimale pourrait améliorer la perception de la qualité.

2. **Dioxyde de soufre total** : L’impact reste globalement neutre ou très modéré. 

3. **Dioxyde de soufre libre** : L’effet sur la qualité est croissant jusqu’à un certain seuil, au-delà duquel l’influence se stabilise. Un bon niveau de dioxyde de soufre libre semble bénéfique, probablement en lien avec ses propriétés de conservation, notamment dans les vins blancs.



#### Importance par permutation

L’analyse par permutation, qui consiste à mesurer la dégradation des performances du modèle lorsqu’on perturbe aléatoirement chaque variable, confirme les résultats précédents : 

- Confirme l’importance des variables :
    - **Densité** : confirmée comme la variable la plus influente sur la qualité prédite.
    - **Chlorures** : exerce un effet négatif notable sur la qualité, ce que l’analyse PDP avait déjà suggéré.
    - **Dioxyde de soufre libre** : son rôle protecteur semble déterminant jusqu’à un certain seuil.

- Impact très limité sur la prédiction des variables:
    - **pH** et **acidité fixe** : leur influence est marginale dans le modèle.
    - **Cluster** : cette variable, issue d’un regroupement automatique (k-means), ne semble pas apporter d’information supplémentaire pertinente ici.


#### Conclusion intermédiaire
- La **qualité perçue du vin** dépend de **facteurs chimiques** clés, comme la densité ou la teneur en dioxyde de soufre.

#### Explicabilité par individu

##### ICE Plots  
Les ICE plots permettent d’analyser l’effet individuel de variables comme la densité, le dioxyde de soufre total, le sucre résiduel, ou les chlorures sur les prédictions. Les courbes sont globalement horizontales, indiquant un effet faible de ces variables, mais avec quelques cas individuels montrant une influence plus marquée. Certaines variables présentent des effets non linéaires (ex. sucre résiduel, chlorures), possiblement liés à la nature des vins (blancs, rouges, secs, moelleux).

##### Interprétation locale avec LIME  
Pour une observation avec une prédiction très faible (0.01), LIME met en évidence des variables influentes. L’alcool, les sulfites libres, le sucre résiduel et l’acide citrique poussent la prédiction vers le haut, tandis que la densité, les sulfites totaux et l’acidité volatile ont un effet inverse. Ces effets peuvent varier selon le type de vin et soulignent l'intérêt d'une analyse locale complémentaire à l'analyse globale.

##### Interprétation locale avec SHAP  
Les valeurs de SHAP confirment les résultats de LIME et permettent une interprétation plus robuste. Elles quantifient précisément l’effet marginal de chaque variable sur la prédiction d’un individu. Pour l’observation analysée, l'alcool, le dioxyde de soufre libre et l'acide citrique contribuent positivement à la prédiction, tandis que la densité et le dioxyde de soufre total ont un impact négatif marqué. SHAP permet également de visualiser ces effets via des graphiqes de contribution individuels ou globaux, utiles pour détecter les patterns récurrents.




## Ce qui a marché

- Optimisation du modèle Random Forest : L’optimisation des hyperparamètres du modèle Random Forest a permis d’obtenir une légère amélioration des performances, avec un MSE réduit et un R2 augmenté. Le modèle a donc bien réagi à l’ajustement des paramètres.

- Analyse de l'importante des variables par permutation : L’analyse par permutation a confirmé l'importance des variables clés pour la prédiction de la qualité du vin, comme la densité et le dioxyde de soufre libre, ce qui renforce la validité du modèle et des résultats observés.

## Ce qui n’a pas marché

- Difficultés d’interprétation avec les clusters : L’utilisation des clusters issus du K-means n’a pas apporté une amélioration significative pour la prédiction de la qualité du vin. Bien que l’analyse suggère la présence de deux sous-populations distinctes de vins, l’ajout de la variable cluster dans le modèle n’a pas amélioré la performance du modèle de manière notable, ce qui montre que le type de vin (rouge, blanc, sec, doux) ne suffit pas à lui seul pour prédire la qualité de manière significative.

- Malgré le fait que le cours soit centré sur les modèles SVM, ce n’est finalement pas cette méthode qui a permis d’obtenir les meilleures performances. Le modèle SVM s’est révélé moins performant en termes de R2 que le modèle Random Forest. Cette situation n’est pas totalement surprenante : le Random Forest est particulièrement efficaces lorsqu’il s’agit de capturer des relations non linéaires et des interactions complexes entre variables. À l’inverse, les SVM peuvent se montrer moins robustes lorsque les données présentent du bruit, des non-linéarités ou des variables peu discriminantes. 


## Résultats

L'analyse approfondie de la qualité du vin à travers un modèle prédictif met en évidence une interaction complexe de facteurs physico-chimiques. Parmi les variables les plus influentes, la densité (density), le dioxyde de soufre total (total_sulfur_dioxide), et le dioxyde de soufre libre (free_sulfur_dioxide) se distinguent. La densité, qui reflète la masse volumique du vin, joue un rôle crucial dans sa structure et sa perception en bouche, avec des variations dans son impact selon le type de vin : une faible densité peut indiquer un vin dilué, une densité modérée est généralement positive, et une densité élevée est caractéristique des vins moelleux. Les composés soufrés, utilisés comme conservateurs, soulignent l'importance de la stabilité chimique et de la protection contre l'oxydation et les bactéries. D'autres variables, telles que le sucre résiduel (residual_sugar) et les chlorures (chlorides), contribuent également à la prédiction de la qualité, bien que dans une moindre mesure. Le sucre résiduel, indicateur de la douceur, a un effet positif, particulièrement dans les vins moelleux, tandis que les chlorures, reflétant la teneur en sel, ont tendance à avoir un impact négatif sur la qualité.

L'examen des effets individuels des variables, à travers les Partial Dependence Plots (PDP) et Individual Conditional Expectation (ICE) plots, révèle des relations nuancées. La densité présente une relation non linéaire avec la qualité, où son influence varie selon le type de vin. Le dioxyde de soufre libre améliore la qualité jusqu'à un certain seuil, tandis que le dioxyde de soufre total a un effet relativement faible. Le sucre résiduel montre une corrélation positive, mais son impact dépend fortement du type de vin. Les chlorures, en général, ont un effet négatif. 

Les techniques d'explication des prédictions, telles que LIME (Local Interpretable Model-agnostic Explanations) et SHAP (SHapley Additive exPlanations), confirment ces observations. LIME indique comment des variables comme l'alcool, le dioxyde de soufre libre, le sucre résiduel et l'acide citrique peuvent augmenter la qualité prédite, tandis que la densité, le dioxyde de soufre total et l'acidité volatile peuvent la diminuer. Les graphiques SHAP soulignent l'importance de l'alcool, des sulfates et des niveaux de soufre, tout en mettant en évidence que leur influence varie selon le type de vin.

## Conclusion

 En conclusion, la qualité du vin est déterminée par un équilibre délicat de nombreux facteurs physico-chimiques, et cet équilibre est fortement influencé par le type de vin. La densité et les composés soufrés sont des déterminants majeurs, mais leur effet est modulé par des caractéristiques telles que la douceur, l'acidité et la teneur en alcool, qui sont perçues différemment dans les vins blancs, rouges, secs ou moelleux. 

Les modèles prédictifs de la qualité du vin gagneraient donc en robustesse et en pertinence en intégrant explicitement cette dimension catégorielle. Une telle approche permettrait des prédictions plus précises et des recommandations plus ciblées pour optimiser la qualité du vin en fonction de ses spécificités.  