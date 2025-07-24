### Decathlon - Senior Data Engineer - Freelance
Mai 2021 - Présent (4 ans 3 mois)

Contexte : Développement de la plateforme d'ingestion de Decathlon : Data Factory Ingest à l'échelle du groupe. Solution critique utilisée par 160 équipes à travers le monde, traitant 400 To quotidiens et 2000 tables ingérées quotidiennement.

#### Architecture et optimisation technique
- Implémentation du back-end des ingestions Spark sur EMR avec prise de décisions architecturales
- Mise en place d'une approche générique permettant la scalabilité sur 160 équipes
- Réduction des coûts et du nombre d'appels par implémentation d'une sauvegarde temporaire des données sur HDFS au lieu d'un traitement direct des données sur S3
- Migration de Datalake vers Lakehouse
- Optimisation de tables JDBC volumineuses (>2To) avec fortes latences dues à la distance géographique (Inde), traitées en mode "annule et remplace" quotidien, par sauvegarde séquentielle en format .avro plutôt qu'en parquet pour éviter les erreurs mémoire
- Industrialisation d'une partie des traitements sur Autoloader (outil d'ingestion Databricks)
- Décommissionnement d'EMR au profit de Databricks

#### Sécurité et gouvernance
- Gestion des PII : chiffrement au niveau colonne de données nestées complexes sans ajout de colonne
- Mise en place d'une stratégie cross-cloud de chiffrement de données confidentielles (Ressources Humaines)
- Gouvernance : mapping de chaque table avec un contrat d'ingestion (idcard) paramétré via l'interface utilisateur
- Synchronisation des métadonnées Collibra vers le back-end

#### Orchestration et qualité
- Airflow : génération automatique des DAGs (Python) et mise en place des "Dataset" (fonctionnalité Airflow permettant le déclenchement automatique d'un DAG lors d'une mise à jour sur un path S3)
- Implémentation d'une "Reject Table" : validation de schéma par fichier et remontée des erreurs dans une table dédiée
- Implémentation et déploiement de tests unitaires sur l'ensemble des composants d'ingestion
- Développement de différents modes d'ingestion :
	 - FULL : remplacement complet des données
	 - DIFF : ingestion incrémentale avec merge sur l'existant
	 - APPEND : concaténation des nouveaux fichiers

#### Infrastructure et DevOps
- Configuration et déploiement de Datadog pour la centralisation des logs
- Mise en place d'une CI/CD pour l'automatisation des déploiements

#### Migration et évolution
- Définition d'une stratégie de migration metamodel Glue vers Unity Catalog
- Réalisation d'un POC pour écriture dans Unity Catalog depuis EMR
- Propositions et mise en place de stratégies de réduction de la dette technique

#### Collaboration et méthodes
- Support technique aux équipes utilisatrices dans un contexte international
- Gestion de projets critiques avec priorisation et coordination d'équipes métiers
- Organisation en agile avec sprints de 2 semaines

Stack technique : Spark, Scala, AWS (S3, EMR, Glue), Airflow, Databricks, Delta Lake, Unity Catalog, Autoloader, Python, Datadog, Collibra

Équipe : 3 Data Engineers, 1 Front-end, 1 Fullstack Engineer, 1 PO, 1 Lead Tech

------

## EPISEN - Professeur de Big Data en école d'ingénieur - Freelance
Novembre 2022 - Présent (2 ans 9 mois)

Contexte : Vacataire à l'EPISEN (École Publique d'Ingénieurs de la Santé et du Numérique), école d'ingénieurs publique de l'Université Paris-Est Créteil spécialisée dans la formation de 250 élèves ingénieurs combinant santé et numérique. Intervention sur l'UE "Data Mining, Warehouse, Viz, Big data, NoSQL pour les systèmes de santé".

#### Formation théorique et pratique
- 10h CM : concepts fondamentaux du Big Data et écosystème technologique
- 10h TP : mise en pratique sur des cas d'usage concrets
- Préparation des supports de cours et de TP adaptés

#### Programme pédagogique
- Première formation à l'écosystème big data (5V, types de données, Datalake, DWH) 
- Formation et mise en pratique des systèmes distribués (Apache Spark)
- Acculturation au streaming event-driven (Apache Kafka)
- Introduction au data mining, à des projets bout en bout data science, théorie et techniques de base de Machine Learning

Compétences développées : Pédagogie, vulgarisation technique

---- 

## AAA-Data (Automobile) - Data Engineer - CDI
Janvier 2020 - Mai 2021 (1 an 5 mois)

Contexte : Migration du système d'information de prédiction on-premise vers Snowflake chez AAA DATA, leader français de l'exploitation des données automobiles depuis 60 ans. Position de monopole historique (licence ANTS 7M€/an) menacée par de nouveaux entrants cloud. Objectif : moderniser la stack pour sécuriser la position marché.

#### Architecture et pipeline de données
- Supervision technique de 2 data engineers junior avec mentorat quotidien.
- Implémentation de pipelines de traitement : 
	- Réception fichier ministère (ANTS)
	- Correction automatique des données manuscrites
	- Enrichissement avec bases constructeurs
	- Sauvegarde en vues statistiques vendues aux constructeurs et concessions automobiles.
- Réduction drastique du temps d'ingestion des données du ministère de 13h à 6 minutes.
- Jointures et correction en logique floue (fuzzy logic) pour réconciliation automatique de données hétérogènes.

#### Collaboration et méthodes
- Contact quotidien avec les équipes métiers pour expression de besoins et validation
- Organisation en agile avec sprints de 2 semaines

#### Migration cloud et modernisation
- Transition complète d'une architecture on-premise vers Snowflake sans interruption de service
- Optimisation des performances par parallélisation des traitements
- Réduction de la dette technique et modernisation des processus ETL

Stack technique : Snowflake, Python, Apache Spark, PostgreSQL, Docker, Jenkins, Fuzzy Logic (FuzzyWuzzy), Pandas, NumPy, Apache Airflow

Équipe : 1 PO, 1 Tech Lead, 3 Data Engineers


---- 
## Pernod Ricard - Formateur - CDI
Avril 2019 - Octobre 2019 (6 mois)

Contexte : Formation d'équipes techniques à un projet data end-to-end via Databricks basé sur un cas d'usage concret et applicable au contexte Pernod Ricard : mise en place de prédiction de catégorie de bar pour répartir aux mieux les forces commerciales ; 

- Scraping via API Google Places
- Transformation, nettoyage des données, jointure sur données internes via Spark et sauvegarde dans datalake
- Industrialisation de ces Notebooks via Databricks
- Formation aux algorithmes de base de Machine Learning
- Formation à la création, évaluation, validation et optimisation de modèles
- Préparation de tous les supports de TP, de tous les visuels et de tous le code sous-jacent

Stack technique : Databricks, DBFS, Python, Apache Spark, Scikit, Power BI

Equipe : 1 Tech Lead, 1 Formateur