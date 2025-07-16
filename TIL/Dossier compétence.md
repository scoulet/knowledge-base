### Decathlon - Data Engineer Senior Freelance
Mai 2021 - Présent (4 ans 3 mois)

Contexte : Développement de la plateforme d'ingestion de Decathlon : Data Factory Ingest à l'échelle du groupe. Solution critique utilisée par 160 équipes à travers le monde, traitant 400 To quotidiens et 2000 tables ingérées quotidiennement.

#### Architecture et optimisation technique
- Implémentation du back-end des ingestions Spark sur EMR avec prise de décisions architecturales
- Mise en place d'une approche générique permettant la scalabilité sur 160 équipes
- Réduction des coûts et du nombre d'appels par implémentation d'une sauvegarde temporaire des données sur HDFS au lieu d'un traitement direct des données sur S3
- Optimisation de tables JDBC volumineuses (>2To) avec fortes latences dues à la distance géographique (Inde), traitées en mode "annule et remplace" quotidien, par sauvegarde séquentielle en format .avro plutôt qu'en parquet pour éviter les erreurs mémoire
- Industrialisation d'une partie des traitements sur Autoloader (outil d'ingestion Databricks)
- Décommissionnement d'EMR au profit de Databricks

#### Sécurité et gouvernance
- Gestion des PII : chiffrement au niveau colonne de données nestées complexes sans ajout de colonne
- Mise en place d'une stratégie cross-cloud de chiffrement de données HR
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
- Support technique aux équipes utilisatrices dans un contexte international
- Gestion de projets critiques avec priorisation et coordination d'équipes métiers multiples

Stack technique : Spark, Scala, AWS (S3, EMR, Glue), Airflow, Databricks, Delta Lake, Unity Catalog, Autoloader, Python, Datadog, Collibra

Équipe : X Data Engineers, X Front-end, X Fullstack Engineer, 1 PO, 1 Lead Tech