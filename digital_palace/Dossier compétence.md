## Decathlon - Senior Data Engineer - Freelance
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
- Mise en place d'une politique Role Based Access Control sur les bases 

Stack technique : Snowflake, Python, Apache Spark, PostgreSQL, Docker, Jenkins, Fuzzy Logic (FuzzyWuzzy), Pandas, NumPy, Apache Airflow

Équipe : 1 PO, 1 Tech Lead, 3 Data Engineers


----

## Pernod Ricard - Formateur Data - CDI
Avril 2019 - Octobre 2019 (6 mois)

Contexte : Formation d'équipes techniques chez Pernod Ricard, groupe français leader mondial des vins et spiritueux. Dans le cadre de la transformation digitale du groupe et de l'adoption de nouvelles technologies data, mission de formation des équipes techniques à un projet data end-to-end via Databricks. Projet basé sur un cas d'usage concret et stratégique : développement d'un modèle de prédiction de catégorie de bars (lounge, animé, club) non encore licenciés chez Pernod-Ricard pour optimiser la répartition des forces commerciales sur le terrain et identifier de nouveaux points de vente potentiels.

#### Conception pédagogique et formation technique
- Développement d'un projet data complet de A à Z sur cas d'usage métier réel
- Formation aux algorithmes de base de Machine Learning
- Formation à la création, évaluation, validation et optimisation de modèles
- Introduction à la visualisation de données pour visualiser géographiquement les différents bars
- Préparation complète des supports de TP, visuels et code sous-jacent

#### Pipeline de données et traitement
- Scraping via API Google Places pour collecte de données géolocalisées
- Transformation, nettoyage des données et jointure sur données internes via Spark
- Sauvegarde dans datalake et industrialisation des Notebooks via Databricks
- Mise en place de visualisations géographiques pour aide à la décision commerciale

#### Impact métier
- Optimisation de la répartition des forces commerciales sur le terrain
- Identification de bars potentiels non encore licenciés chez Pernod-Ricard
- Montée en compétences des équipes techniques sur l'écosystème data moderne

Stack technique : Databricks, DBFS, Python, Apache Spark, Scikit-learn, Power BI

Équipe : 1 Tech Lead, 1 Formateur

---- 

## MAIF - Data Engineer - CDI
Janvier 2019 - Janvier 2020 (1 an)

Contexte : Industrialisation de projets data science chez MAIF, mutuelle d'assurance française de millions de sociétaires. Problématique critique : de nombreux projets data science développés par les équipes restaient cantonnés à des notebooks expérimentaux sans passage en production, générant un ROI nul malgré des investissements importants en ressources et temps. Mission double : industrialiser ces projets pour les mettre en production et développer un système de scoring d'appétence des sociétaires pour optimiser les propositions commerciales. Enjeu stratégique : transformer les POCs data science en solutions productives générant de la valeur métier concrète pour les 20 agences du réseau.

#### Industrialisation et mise en production
- Transition des algorithmes de Python (pandas, scikit-learn) vers PySpark pour traiter des volumes de données réels
- Mise en production effective par 15 data scientists répartis dans 4 équipes
- Pickling et versioning des modèles avec observations de data drift
- Déploiement de solutions robustes et scalables en remplacement des notebooks expérimentaux

#### Pipeline de données et scoring
- Développement de pipelines de data cleaning et d'extraction Spark
- Mise en place d'un système de scoring d'appétence des sociétaires
- Optimisation des traitements pour gérer des volumes de données en production
- Automatisation des processus de traitement et de mise à jour des modèles

#### Impact métier et collaboration
- Facilitation des propositions commerciales dans 20 agences
- Amélioration de l'adéquation des produits proposés aux sociétaires
- Collaboration étroite avec les équipes data science pour industrialiser leurs travaux

Stack technique : PySpark, Jenkins, Docker, Zeppelin

Équipe : 1 PO, 2 Data Scientists, 2 Data Engineers