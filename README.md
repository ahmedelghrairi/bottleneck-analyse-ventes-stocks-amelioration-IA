# BottleNeck : analyse des ventes et des stocks, version améliorée par l'IA

Reprise critique d'une analyse de données pour BottleNeck, un marchand de vin en ligne, réalisée pendant ma formation Data Analyst (OpenClassrooms). Ce dépôt contient la version 2 du notebook : la version initiale, livrée en février 2026, a été reprise et améliorée en m'appuyant sur l'intelligence artificielle, avec une règle simple : aucun code généré n'est intégré sans avoir été testé sur des cas dont le résultat est connu.

## Le contexte et le besoin

Les informations produits de BottleNeck sont réparties entre un ERP (références, prix, stocks) et un site WordPress (fiches, quantités vendues), reliés par une table de liaison tenue à la main. L'entreprise ne peut pas piloter son activité de façon fiable : impossible de savoir quelles gammes génèrent réellement le chiffre d'affaires, où dort l'argent immobilisé en stock, ou quels prix sont manifestement faux.

## Ce que fait le notebook

- Rapprochement des trois sources avec mesure du taux de correspondance (111 références sur 825 sans équivalent web, soit 13,5 %, avec alerte automatique en cas de hausse)
- Nettoyage documenté : chaque anomalie est tracée avec sa correction et une recommandation
- Analyses de pilotage : chiffre d'affaires (143 680,10 € sur octobre, vérifiable à l'euro près), répartition 80/20, valorisation du stock au prix de vente et au prix d'achat, taux de marge par catégorie, corrélations
- Détection des valeurs aberrantes par deux méthodes (Z-score et intervalle interquartile), chaque cas tranché entre erreur de saisie et produit haut de gamme

## Ce que la version 2 ajoute

- **Contrôles qualité automatisés à l'import** avec Pandera, choisi après une veille comparant quatre solutions (Pandera, contrôles pandas assistés par IA, Great Expectations, dbt). Le rapport de contrôles est lisible par une personne non technique.
- **Un contrôle inter-tables en pandas** pour le taux de correspondance ERP / web, que Pandera ne couvre pas
- **Une anomalie trouvée que la revue manuelle avait manquée** : deux quantités vendues négatives dans l'export web. C'est la preuve concrète de l'intérêt de l'automatisation.
- **Une section RGPD** : inventaire des 29 colonnes de l'export web, colonne post_author écartée par minimisation
- **La reproductibilité** : chemins relatifs, versions figées, le notebook s'exécute de bout en bout sur un poste vierge
- **Un tableau final récapitulant** anomalies, hypothèses, limites et biais

## Les données

Les trois fichiers sources (ERP, web, liaison) sont fournis dans le cadre de la formation et ne sont pas publiés dans ce dépôt. Le notebook attend trois fichiers Excel dans un dossier `data/` : l'export ERP, l'export web et la table de liaison.

## Limites assumées

Un seul mois de données, donc aucune tendance ni saisonnalité. TVA supposée uniforme à 20 %. Les corrections de prix et de stocks reposent sur des hypothèses explicitées : elles ne remplacent ni un inventaire ni un contrôle de saisie dans l'ERP. Une corrélation mesure une association, pas une cause.

## Exécution
```
pip install -r requirements.txt
```
Déposer les trois fichiers Excel dans `data/`, ouvrir le notebook et lancer « Redémarrer et tout exécuter ».

## Auteur

Ahmed El Ghrairi, Data Analyst à Marseille.
[Portfolio](https://ahmedelghrairi.github.io) · [LinkedIn](https://www.linkedin.com/in/ahmed-el-ghrairi-a581ba177/)
