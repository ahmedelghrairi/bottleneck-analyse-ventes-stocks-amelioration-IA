# Journal des expérimentations IA — Mission 1 (BottleNeck)

## Règles que je m'impose
- Tout prompt utilisé pour ce projet est copié ici, y compris ceux dont le résultat est écarté.
- Aucune donnée réelle de BottleNeck n'est envoyée à un outil IA en ligne : je transmets la structure (noms de colonnes, types, valeurs fictives), jamais les fichiers.
- Tout code proposé par une IA est testé avant d'être intégré au notebook.

## Critères d'évaluation utilisés
Exactitude après test · Exhaustivité · Temps passé · Lisibilité et maintenabilité · Reproductibilité · Sécurité / RGPD · Sobriété

---

## Essai n°1
- **Date** : 30 Août 2026
- **Objectif** : choisir l'outil pour automatiser les contrôles qualité des données
- **Outil** : Claude (Anthropic), version web
- **Prompt exact** : « Agis en tant qu'architecte Data expérimenté. Je dois automatiser les contrôles de qualité de données sur un volume restreint (3 fichiers CSV/DataFrames pandas en mémoire, pas d'entrepôt de données type Snowflake/BigQuery).
Compare brièvement les options (dbt, Pandera, Great Expectations, pandas pur) et recommande la solution la plus sobre et pertinente.
Format attendu : Un tableau comparatif rapide (forces/faiblesses/prérequis) suivi d'une recommandation finale argumentée. »

- **Variante testée** : aucune, question ouverte
- **Résultat obtenu** : comparaison de quatre options (dbt, Pandera, Great Expectations, contrôles pandas). dbt écarté parce qu'il suppose un entrepôt de données ; Great Expectations écarté parce que surdimensionné pour trois fichiers.
- **Critères retenus pour juger** : pertinence au format des données (tableaux pandas en mémoire), lisibilité pour un profil non technique, sobriété
- **Décision** : gardé. Pandera comme solution principale, contrôles pandas en complément pour les règles inter-tables
- **Pourquoi** : la réponse correspondait à ce que je connaissais de dbt (vu en formation sur Snowflake) et l'argument de l'entrepôt était vérifiable

## Essai n°2
- **Date** : 01 septembre 2026
- **Objectif** : générer le schéma Pandera des règles de l'ERP
- **Outil** : Claude (Anthropic)
- **Prompt exact** : « Agis en tant qu'ingénieur Data spécialisé en validation de données. Analyse le notebook joint et génère un schéma de validation Pandera (DataFrameSchema) robuste pour les données ERP.
Contraintes techniques :
Règles métier à couvrir : Détecte les prix négatifs, les stocks négatifs et les incohérences de statut.»

- **Variante testée** : v1 : colonnes texte déclarées avec le type `str`. v2 : colonnes texte déclarées sans type, contrôlées uniquement par des règles `Check`
- **Résultat obtenu** : la v1 dépend de la version de pandas (type `object` en pandas 2, type `str` en pandas 3) et risque d'échouer selon la machine. La v2 fonctionne sur les deux et retrouve les trois prix négatifs, les deux stocks négatifs et les deux incohérences de statut
- **Critères retenus pour juger** : exactitude sur les cas connus, reproductibilité sur une autre machine
- **Décision** : gardé v2, écarté v1
- **Pourquoi** : la reproductibilité est un critère d'acceptation du cahier des charges ; un schéma qui casse selon la version de pandas ne l'est pas

## Essai n°3
- **Date** : 01 septembre 2026
- **Objectif** : transformer la sortie technique de Pandera en un rapport lisible (une ligne par règle, avec nombre de lignes et exemples)
- **Outil** : Claude (Anthropic)
- **Prompt exact** : « À partir des résultats d'exécution du schéma Pandera, écris une fonction Python qui transforme les erreurs techniques en un rapport de synthèse lisible par un profil non technique.
Exigences de restitution :
Presente un tableau récapitulatif : Règle | Nombre de lignes en anomalie | Exemples d'identifiants.
Attention au comptage : Pour les règles de table multi-colonnes (ex: cohérence statut / quantité), regroupe les anomalies par règle uniquement, de manière à compter chaque ligne unique en défaut et non le nombre de colonnes impactées. »

- **Variante testée** : v1 : regroupement des anomalies par colonne et par règle. v2 : les règles de table (qui portent sur plusieurs colonnes) regroupées par règle seule
- **Résultat obtenu** : la v1 comptait six fois la même anomalie pour la règle de cohérence statut / quantité, une fois par colonne de la table, alors qu'il n'y a que deux lignes en défaut. La v2 donne le bon comptage
- **Critères retenus pour juger** : exactitude, lisibilité du rapport
- **Décision** : gardé v2, écarté v1
- **Pourquoi** : un rapport qui gonfle les comptages perd toute crédibilité auprès de Nicolas

## Essai n°4
- **Date** : 3 septembre 2026
- **Objectif** : écrire les mêmes règles en pandas pur (option B) pour comparer avec Pandera
- **Outil** : Claude (Anthropic)
- **Prompt exact** : « Rédige les mêmes règles de contrôle qualité sous forme de fonctions pandas pures (sans bibliothèque externe), afin de comparer cette approche avec Pandera.
Contraintes de code :
Gestion des nuls : Pour l'unicité des identifiants, combine duplicated() et isna() afin de remonter aussi bien les doublons que les clés manquantes.
Lisibilité : Rends les masques booléens lisibles et commentés.
Sortie : Restitue un dictionnaire d'anomalies avec exactement la même structure de comptage que celle obtenue avec Pandera. »

- **Variante testée** : v1 : règle d'unicité écrite avec `duplicated()` seul. v2 : `duplicated() | isna()` pour prendre aussi les identifiants manquants
- **Résultat obtenu** : la v1 laissait passer un identifiant manquant sans le signaler. La v2 corrigée passe le test sur les cas connus et donne les mêmes comptages que Pandera sur les six règles communes
- **Critères retenus pour juger** : exactitude, lisibilité des règles, maintenabilité
- **Décision** : gardé v2 en complément uniquement, pour la règle inter-tables du taux de correspondance. Pandera reste la solution principale
- **Pourquoi** : les masques booléens se lisent mal (négations) et chaque nouvelle règle demande un nouveau test ; le schéma Pandera se lit comme une liste de contraintes

## Essai n°5
- **Date** : 3 septembre 2026
- **Objectif** : vérifier les deux options sur des cas dont le résultat est connu
- **Outil** : Claude (Anthropic) pour écrire les assertions, exécution sur les données réelles en local
- **Prompt exact** : « Rédige une suite de tests unitaires (avec assert Python) pour vérifier la cohérence des contrôles entre la version Pandera et la version pandas pur sur nos données réelles.
Objectif :
Vérifier que les deux approches identifient exactement le même nombre d'anomalies sur les cas connus (prix, stocks, statuts, identifiants).
Ajouter un contrôle sur la colonne des quantités vendues de l'export web pour détecter d'éventuelles valeurs négatives ou incohérentes non identifiées lors de la revue manuelle. »

- **Variante testée** : aucune
- **Résultat obtenu** : les deux options retrouvent 3 prix négatifs, 2 stocks négatifs, 2 incohérences et 3 identifiants hors format. Le contrôle révèle en plus deux quantités vendues négatives dans l'export web, sur des lignes sans code article, que la revue manuelle du projet 6 n'avait pas vues
- **Critères retenus pour juger** : exactitude
- **Décision** : gardé. Anomalie nouvelle ajoutée au tableau récapitulatif (ligne 7)
- **Pourquoi** : c'est la preuve que le contrôle automatisé apporte quelque chose que la revue manuelle ne fait pas
