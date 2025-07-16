Première ébauche

### Decathlon

- Développement de la plateforme d'ingestion Decathlon : Data Factory Ingest
- Plateforme utilisée par 160 équipes, 400 To quotidiens et 2000 tables ingérées quotidiennes
- Implémentation du back-end des ingestions Spark sur EMR
- Réduction des coûts et du nombre d'appels (- XXX %) en implémentant une sauvegarde temporaire des données sur HDFS plutôt que spark traite des données sur s3
- Mise en place d'une approche très générique 
- Optimisations de tables JDBC très volumineuses (>2To) et à fortes latences dûes à la distance géographique (Inde) à traiter en annule et remplace tous les jours, en donnant la possibilité de sauvegarder en .avro de manière séquentielle plutôt qu'en parquet qui entraine des erreurs mémoires
- Gestion des PII : Comment chiffrer à la colonne des données potentiellement très nestée in-place (ie sans rajout de colonne)
- Mise en place d'une stratégie cross-cloud de chiffrement de données HR

- Implémentation et déploiement de tests unitaires sur toutes les parties de l'ingestions
- Mise en place d'une génération d'une "Reject Table" : en cas d'erreur de schéma, teste le schéma fourni sur chaque fichier et on relève l'éventuelle erreur dans une table
- Airflow -> Génération des DAGs (python) + mise en place des "Dataset" (fonctionnalité airflow qui permet le lancement d'un dag quand un update est effectué sur un path s3)
- Gouvernance : chaque table correspond à un contrat d'ingestion (idcard) relié à plusieurs paramètres sur l'UI
- Définition d'une stratégie de migration + metamodel glue -> unity 
- POC pour écrire dans unity catalog depuis EMR
- Industrialisation d'une partie des traitements sur Autoloader (Outil d'ingestion sur Databricks)
- Décomissionnement d'EMR au profit de databricks
- Support

Moins sûr (à réviser / c'est pas moi qui ai fait / j'ai juste repris les trucs) : 
- Agent : mise en place de flux multiclouds
- Mise en place de datadog pour logs
- CI/CD
- Gouvernance : synchronisation des metadatas collibra vers le back-end

Bof à mettre / à reformuler  
- Beaucoup de projets très impactants -> habitude de travail sous pression => priorisation + écoute
- Implémentation de différents mode d'ingestions possibles : 
	- FULL : annule et remplace pour toutes les données
	- DIFF : n'ingère que les derniers fichiers depuis une certaine date et merge sur l'existant
	- APPEND : met bout à bout les derniers fichiers 
- Mise en place d'une stratégie pour réduire la dette technique


