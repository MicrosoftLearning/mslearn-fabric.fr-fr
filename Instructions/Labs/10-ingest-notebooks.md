---
lab:
  title: Ingérer des données avec des notebooks Spark et Microsoft Fabric
  module: Ingest data with Spark and Microsoft Fabric notebooks
---

# Ingérer des données avec des notebooks Spark et Microsoft Fabric

Dans ce labo, vous allez créer un notebook Microsoft Fabric et utiliser PySpark pour vous connecter à un chemin de Stockage Blob Azure. Vous allez ensuite charger les données dans un lakehouse à l’aide d’optimisations d’écriture.

Ce labo prend environ **30** minutes.

Pour cette expérience, vous allez générer le code sur plusieurs cellules de code de notebook, ce qui peut ne pas refléter la façon dont vous allez procéder dans votre environnement ; toutefois, il peut être utile pour le débogage.

Étant donné que vous travaillez également avec un exemple de jeu de données, l’optimisation ne reflète pas ce que vous pouvez voir en production à grande échelle. Toutefois, vous pouvez toujours voir des améliorations et lorsque chaque milliseconde compte, l’optimisation est la clé.

> **Remarque** : Vous devez disposer d’une [licence d’essai Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) à l’adresse `https://app.fabric.microsoft.com/home?experience=fabric`, sélectionnez **Engineering données**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un espace de travail et une destination lakehouse

Commencez par créer un lakehouse et un dossier de destination dans le lakehouse.

1. Dans votre espace de travail, sélectionnez **+ Nouvel élément > Lakehouse**, fournissez un nom, puis appuyez sur **Créer**.

    > **Note :** la création d’un lakehouse sans **tables** ou **fichiers** peut prendre quelques minutes.

    ![Capture d’écran d’un nouveau lakehouse](Images/new-lakehouse.png)

1. Dans **Fichiers**, sélectionnez **[...]** pour créer un **Nouveau sous-dossier** nommé **RawData**.

1. Depuis l’explorateur Lakehouse dans le lakehouse, sélectionnez **RawData > ... > Propriétés**.

1. Copiez le **chemin ABFS** du dossier **RawData** dans un bloc-notes vide pour une utilisation ultérieure. Cela doit ressembler à ce qui suit :  `abfss://{workspace_name}@onelake.dfs.fabric.microsoft.com/{lakehouse_name}.Lakehouse/Files/{folder_name}/{file_name}`

Vous devez maintenant disposer d’un espace de travail avec un lakehouse et un dossier de destination RawData.

## Créer un notebook Fabric et charger des données externes

Créez un nouveau notebook Fabric et connectez-vous à une source de données externe avec PySpark.

1. Dans le menu supérieur du lakehouse, sélectionnez **Ouvrir le notebook > Nouveau notebook**. Celui-ci s’ouvrira une fois créé.

    >  **Conseil :** vous avez accès à l’explorateur Lakehouse à partir de ce notebook et vous pouvez actualiser pour voir la progression à mesure que vous accomplissez cet exercice.

1. Dans la cellule par défaut, notez que le code est défini sur **PySpark (Python)** .

1. Insérez le code suivant dans la cellule de code, ce qui permet les actions suivantes :
    - Déclarer les paramètres pour la chaîne de connexion
    - Générer la chaîne de connexion
    - Lire les données dans un DataFrame

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

    > **Note:** Une session Spark démarre à la première exécution de code, ce qui peut prendre plus de temps.

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

1. Ajoutez votre chemin **RawData** ABFS et sélectionnez **&#9655; cellule d’exécution** pour écrire 1000 lignes dans un fichier yellow_taxi.parquet.

1. Votre **output_parquet_path** doit ressembler à :  `abfss://Spark@onelake.dfs.fabric.microsoft.com/DPDemo.Lakehouse/Files/RawData/yellow_taxi`

1. Pour confirmer le chargement des données à partir de l’explorateur Lakehouse, sélectionnez **Fichiers > ... > Actualiser**.

Vous devez maintenant voir votre nouveau dossier **RawData** avec un « fichier » **yellow_taxi.parquet** *qui s’affiche sous la forme d’un dossier contenant des fichiers de partition*.

## Transformer et charger des données dans une table Delta

Il est probable que votre tâche d’ingestion de données ne se termine pas par le simple chargement d’un fichier. Les tables Delta dans un lakehouse permettent une interrogation et un stockage évolutifs et flexibles. Nous allons donc également en créer une.

1. Créez une nouvelle cellule de code, puis insérez le code suivant :

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet(output_parquet_path)   
    
    # Add dataload_datetime column with current timestamp
    filtered_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    filtered_df = filtered_df.filter(col("storeAndFwdFlag").isNotNull())
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi"
    filtered_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(filtered_df.limit(1))
    ```

1. Sélectionnez **&#9655; Run Cell** à côté de la cellule de code.

    - Cela ajoute une colonne timestamp **dataload_datetime** pour journaliser le moment auquel les données ont été chargées dans une table Delta
    - Filtrer les valeurs NULL dans **storeAndFwdFlag**
    - Charger des données filtrées dans une table Delta
    - Afficher une seule ligne pour la validation

1. Passez en revue et confirmez les résultats affichés, qui ressemblent à l’image suivante :

    ![Capture d’écran d’une sortie réussie affichant une seule ligne](Images/notebook-transform-result.png)

Vous avez réussi à vous connecter à des données externes, à les écrire dans un fichier parquet, à charger les données dans un DataFrame, à les transformer et à les charger dans une table Delta.

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

1. Sélectionnez **&#9655; Run Cell** à côté de la cellule de code.

     De nombreux analystes de données sont plus à l’aise avec la syntaxe SQL. Spark SQL est une API de langage SQL dans Spark que vous pouvez utiliser pour exécuter des instructions SQL, ou même pour conserver des données dans des tables relationnelles.

   Le code que vous venez d’exécuter crée une *vue* relationnelle des données dans un dataframe, puis utilise la bibliothèque **spark.sql** pour incorporer la syntaxe Spark SQL dans votre code Python et interroger la vue et retourner les résultats sous forme de dataframe.

## Nettoyer les ressources

Dans cet exercice, vous avez utilisé des notebooks avec PySpark dans Fabric pour charger des données et les enregistrer dans Parquet. Vous avez ensuite utilisé ce fichier Parquet pour transformer davantage les données. Enfin, vous avez utilisé SQL pour interroger les tables Delta.

Une fois que vous avez fini d’explorer, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
1. Sélectionnez **Paramètres de l’espace de travail** et, dans la section**Général**, faites défiler vers le bas et sélectionnez **Supprimer cet espace de travail**.
1. Sélectionnez **Supprimer** pour supprimer l’espace de travail.
