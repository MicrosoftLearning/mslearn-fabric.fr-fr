---
lab:
  title: Ingérer des données avec des notebooks Spark et Microsoft Fabric
  module: Ingest data with Spark and Microsoft Fabric notebooks
---

# Ingérer des données avec des notebooks Spark et Microsoft Fabric

Dans ce labo, vous allez créer un notebook Microsoft Fabric et utiliser PySpark pour vous connecter à un chemin de Stockage Blob Azure. Vous allez ensuite charger les données dans un lakehouse à l’aide d’optimisations d’écriture.

Ce labo prend environ **30** minutes.

Pour cette expérience, nous allons générer le code sur plusieurs cellules de code de notebook, ce qui peut ne pas refléter la façon dont vous allez procéder dans votre environnement. Toutefois, cela peut être utile pour le débogage.

Étant donné que nous travaillons également avec un exemple de jeu de données, l’optimisation ne reflète pas ce que vous pouvez voir en production à grande échelle. Cependant, vous pouvez tout de même constater des améliorations et l’optimisation est fondamentale quand chaque milliseconde compte.

> **Note** : vous devez disposer d’une **licence Microsoft Fabric** pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite.
>
> Vous avez également besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail et une destination lakehouse

Commencez par créer un espace de travail avec la version d’essai Fabric activée, un nouveau lakehouse et un dossier de destination dans le lakehouse.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez l’expérience **Engineering données Synapse**.

    ![Capture d’écran de l’expérience Engineering données Synapse](Images/data-engineering-home.png)

1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.

1. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).

1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide avec un diamant à côté du nom de l’espace de travail, comme illustré ici :

    ![Capture d’écran d’un nouvel espace de travail vide](Images/new-workspace.png)

1. Dans votre espace de travail, sélectionnez **+ Nouveau > Lakehouse**, fournissez un nom, puis appuyez sur **Créer**.

    > :memo: **Note :** la création d’un lakehouse sans **tables** ou **fichiers** peut prendre quelques minutes.

    ![Capture d’écran d’un nouveau lakehouse](Images/new-lakehouse.png)

1. Dans **Fichiers**, sélectionnez **[...]** pour créer un **Nouveau sous-dossier** nommé **RawData**.

1. À partir de l’explorateur Lakehouse dans le lakehouse, sélectionnez **Fichiers > ... > Propriétés**.

1. Copiez le **chemin ABFS** du dossier **RawData** dans un bloc-notes vide pour une utilisation ultérieure. Cela doit ressembler à ce qui suit :  `abfss://{workspace_name}@onelake.dfs.fabric.microsoft.com/{lakehouse_name}.Lakehouse/Files/{folder_name}/{file_name}`

Vous devez maintenant disposer d’un espace de travail avec un lakehouse et un dossier de destination RawData.

## Créer un notebook Fabric et charger des données externes

Créez un nouveau notebook Fabric et connectez-vous à une source de données externe avec PySpark.

1. Dans le menu supérieur du lakehouse, sélectionnez **Ouvrir le notebook > Nouveau notebook**. Celui-ci s’ouvrira une fois créé.

    > :bulb: **Conseil :** vous avez accès à l’explorateur Lakehouse à partir de ce notebook et vous pouvez actualiser pour voir la progression à mesure que vous accomplissez cet exercice.

1. Dans la cellule par défaut, notez que le code est défini sur **PySpark (Python)** .

1. Insérez le code suivant dans la cellule de code, ce qui permet les actions suivantes :
    1. Déclarer les paramètres pour la chaîne de connexion
    1. Générer la chaîne de connexion
    1. Lire les données dans un DataFrame

    ```Python
    # Azure Blob Storage access info
    blob_account_name = "azureopendatastorage"
    blob_container_name = "nyctlc"
    blob_relative_path = "yellow"
    
    # Construct connection path
    wasbs_path = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}'
    print(wasbs_path)
    
    # Read parquet data from Azure Blob Storage path
    blob_df = spark.read.parquet(wasbs_path)
    ```

1. Sélectionnez **&#9655; Run Cell** à côté de la cellule de code pour vous connecter et lire les données dans un DataFrame.

    **Résultat attendu :** votre commande doit réussir et imprimer `wasbs://nyctlc@azureopendatastorage.blob.core.windows.net/yellow`

    > :memo: **Note :** une session Spark démarre à la première exécution de code. Son exécution peut donc prendre plus de temps.

1. Pour écrire les données dans un fichier, vous avez maintenant besoin de ce **chemin ABFS** pour votre dossier **RawData**.

1. Insérez le code suivant dans une **nouvelle cellule de code** :

    ```python
        # Declare file name    
        file_name = "yellow_taxi"
    
        # Construct destination path
        output_parquet_path = f"**InsertABFSPathHere**/{file_name}"
        print(output_parquet_path)
        
        # Load the first 1000 rows as a Parquet file
        blob_df.limit(1000).write.mode("overwrite").parquet(output_parquet_path)
    ```

1. Votre **output_parquet_path** doit ressembler à :  `abfss://Spark@onelake.dfs.fabric.microsoft.com/DPDemo.Lakehouse/Files/RawData/yellow_taxi`

1. Sélectionnez **&#9655; Run Cell** à côté de la cellule de code pour écrire 1 000 lignes dans un fichier yellow_taxi.parquet.

1. Pour confirmer le chargement des données à partir de l’explorateur Lakehouse, sélectionnez **Fichiers > ... > Actualiser**.

Vous devez maintenant voir votre nouveau dossier **RawData** avec un « fichier » **yellow_taxi.parquet** *qui s’affiche sous la forme d’un dossier contenant des fichiers de partition*.

## Transformer et charger des données dans une table Delta

Il est probable que votre tâche d’ingestion de données ne se termine pas par le simple chargement d’un fichier. Les tables Delta dans un lakehouse permettent une interrogation et un stockage évolutifs et flexibles. Nous allons donc également en créer une.

1. Créez une nouvelle cellule de code, puis insérez le code suivant :

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Add dataload_datetime column with current timestamp
    filtered_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    filtered_df = filtered_df.filter(raw_df["storeAndFwdFlag"].isNotNull())
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi"  # Replace with your desired table name
    filtered_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(filtered_df.limit(1))
    ```

1. Sélectionnez **&#9655; Run Cell** à côté de la cellule de code.

    * Cela ajoute une colonne timestamp **dataload_datetime** pour journaliser le moment auquel les données ont été chargées dans une table Delta
    * Filtrer les valeurs NULL dans **storeAndFwdFlag**
    * Charger des données filtrées dans une table Delta
    * Afficher une seule ligne pour la validation

1. Passez en revue et confirmez les résultats affichés, qui ressemblent à l’image suivante :

    ![Capture d’écran d’une sortie réussie affichant une seule ligne](Images/notebook-transform-result.png)

Vous avez réussi à vous connecter à des données externes, à les écrire dans un fichier parquet, à charger les données dans un DataFrame, à les transformer et à les charger dans une table Delta.

## Optimiser les écritures de tables Delta

Vous utilisez probablement le Big Data dans votre organisation et c’est pourquoi vous avez choisi les notebooks Fabric pour l’ingestion de données. Nous allons donc également expliquer comment optimiser l’ingestion et les lectures pour vos données. Tout d’abord, nous répétons les étapes de transformation et d’écriture dans une table Delta avec les optimisations d’écriture incluses.

1. Créez une nouvelle cellule de code et insérez le code suivant :

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet("**InsertYourABFSPathHere**")
    
    # Add dataload_datetime column with current timestamp
    opt_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    opt_df = opt_df.filter(opt_df["storeAndFwdFlag"].isNotNull())
    
    # Enable V-Order
    spark.conf.set("spark.sql.parquet.vorder.enabled", "true")
    
    # Enable automatic Delta optimized write
    spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", "true")
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi_opt"  # New table name
    opt_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(opt_df.limit(1))
    ```

1. Récupérez à nouveau votre **chemin ABFS** et mettez à jour le code dans le bloc **avant** d’exécuter la cellule.

1. Vérifiez que vous avez les mêmes résultats qu’avant le code d’optimisation.

À présent, prenez note des temps d’exécution pour les deux blocs de code. Vos temps peuvent varier, mais vous pouvez voir une nette amélioration des performances avec le code optimisé.

## Analyser les données de table Delta avec des requêtes SQL

Ce labo est axé sur l’ingestion des données, ce qui explique surtout le processus *d’extraction, de transformation et de chargement*, mais il est également utile pour vous permettre d’afficher un aperçu des données.

1. Créez une nouvelle cellule de code, puis insérez le code ci-dessous :

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi"
    table_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    table_df.createOrReplaceTempView("yellow_taxi_temp")
    
    # SQL Query
    table_df = spark.sql('SELECT * FROM yellow_taxi_temp')
    
    # Display 10 results
    display(table_df.limit(10))
    ```

1. Créez une autre cellule de code et insérez également le code suivant :

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi_opt"
    opttable_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    opttable_df.createOrReplaceTempView("yellow_taxi_opt")
    
    # SQL Query to confirm
    opttable_df = spark.sql('SELECT * FROM yellow_taxi_opt')
    
    # Display results
    display(opttable_df.limit(3))
    ```

1. Maintenant, sélectionnez **Tout exécuter** dans la barre de menus supérieure.

Cela exécute toutes les cellules de code et vous permet de voir le processus complet du début à la fin. Vous pourrez voir les temps d’exécution entre les blocs de code optimisés et non optimisés.

## Nettoyer les ressources

Dans cet exercice, vous avez découvert comment créer :

* Espaces de travail
* Des Lakehouses
* Des notebooks Fabric
* Du code PySpark pour :
  * Se connecter à des sources de données externes
  * Lire les données dans un DataFrame
  * Écrire des données DataFrame dans un fichier Parquet
  * Lire les données d'un fichier Parquet
  * Transformer des données dans un DataFrame
  * Charger des données DataFrame dans une table Delta
  * Optimiser les écritures de tables Delta
  * Interroger des données de table Delta avec SQL

Une fois que vous avez fini d’explorer, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
