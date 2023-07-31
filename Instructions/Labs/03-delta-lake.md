---
lab:
  title: Utiliser des tables delta dans Apache Spark
  module: Work with Delta Lake tables in Microsoft Fabric
---

# Utiliser des tables delta dans Apache Spark

Les tables d’un lakehouse Microsoft Fabric sont basées sur le format de fichier open source *Delta Lake*, pour Apache Spark. Delta Lake ajoute la prise en charge de la sémantique relationnelle pour les opérations de données par lots et de streaming, et permet la création d’une architecture Lakehouse, dans laquelle Apache Spark peut être utilisé pour traiter et interroger des données dans des tables basées sur des fichiers sous-jacents dans un lac de données.

Cet exercice devrait prendre environ **40** minutes.

> **Remarque** : Vous devez disposer d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)

## Créer un lakehouse et charger des données

Maintenant que vous disposez d’un espace de travail, il est temps de passer à l’expérience *Engineering données* dans le portail et de créer un data lakehouse pour les données que vous allez analyser.

1. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI** et passez à l’expérience **Engineering données**.

2. Dans la page d’accueil d’**Engineering données Synapse**, créez un **Lakehouse** avec le nom de votre choix.

    Au bout d’une minute environ, un nouveau lakehouse vide est créé. Vous devez ingérer certaines données dans le data lakehouse à des fins d’analyse. Il existe plusieurs façons de faire cela mais dans cet exercice, vous allez simplement télécharger un fichier texte sur votre ordinateur local (ou sur votre machine virtuelle de labo le cas échéant), puis le charger dans votre lakehouse.

3. Téléchargez le fichier de données pour cet exercice depuis `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv`, et enregistrez-le en tant que **products.csv** sur votre ordinateur local (ou votre machine virtuelle de labo le cas échéant).

4. Retournez à l’onglet du navigateur web contenant votre lakehouse et, dans le menu  **...** du dossier **Fichiers** dans le volet **Explorateur**, sélectionnez **Nouveau sous-dossier**, puis créez un sous-dossier nommé **products**.

5. Dans le menu **...** du dossier **products**, sélectionnez **Charger** et **Charger des fichiers**, puis chargez le fichier **products.csv** depuis votre ordinateur local (ou votre machine virtuelle de labo le cas échéant) dans le lakehouse.
6. Une fois le fichier chargé, sélectionnez le dossier **products** et vérifiez que le fichier **products.csv** a été chargé, comme montré ici :

    ![Capture d’écran du fichier products.csv chargé dans un lakehouse.](./Images/products-file.png)

## Explorer les données dans un dataframe

1. Dans la page **Accueil**, quand vous visualisez le contenu du dossier **products** dans votre lac de données, dans le menu **Ouvrir un notebook**, sélectionnez **Nouveau notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du texte *markdown* (du texte mis en forme).

2. Sélectionnez la cellule existante dans le notebook, qui contient du code simple, puis utilisez son icône **&#128465;** (*Supprimer*) en haut à droite pour le supprimer, car vous n’aurez pas besoin de ce code.
3. Dans le volet **Explorateur Lakehouse** à gauche, développez **Fichiers** et sélectionnez **products** pour afficher un nouveau volet montrant le fichier**products.csv** que vous avez chargé précédemment :

    ![Capture d’écran d’un notebook avec un volet Fichiers.](./Images/notebook-products.png)

4. Dans le menu **…** pour **products.csv**, sélectionnez **Charger des données** > **Spark**. Une nouvelle cellule de code contenant le code suivant doit être ajoutée au notebook :

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

    > **Conseil** : Vous pouvez masquer le volet contenant les fichiers à gauche en utilisant son icône **<<** . Ceci vous aidera à vous concentrer sur le notebook.

5. Utilisez le bouton **&#9655;** (*Exécuter la cellule*) à gauche de la cellule pour l’exécuter.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans ce notebook, une session Spark doit être démarrée. Cela signifie que la première exécution peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

6. Une fois la commande de la cellule exécutée, examinez la sortie sous la cellule, qui doit être similaire à ceci :

    | Index | ProductID | ProductName | Catégorie | ListPrice |
    | -- | -- | -- | -- | -- |
    | 1 | 771 | Mountain-100 Silver, 38 | VTT | 3399.9900 |
    | 2 | 772 | Mountain-100 Silver, 42 | VTT | 3399.9900 |
    | 3 | 773 | Mountain-100 Silver, 44 | VTT | 3399.9900 |
    | ... | ... | ... | ... | ... |

## Créer des tables delta

Vous pouvez enregistrer le dataframe en tant que table delta en utilisant la méthode `saveAsTable`. Delta Lake prend en charge la création de tables *managées* et *externes*.

### Créer une table *managée*

Les tables *managées* sont des tables pour lesquelles les métadonnées de schéma et les fichiers de données sont gérés par Fabric. Les fichiers de données de la table sont créés dans le dossier **Tables**.

1. Sous les résultats retournés par la première cellule de code, utilisez le bouton **+ Code** pour ajouter une nouvelle cellule de code s’il n’en existe pas déjà. Entrez ensuite le code suivant dans la nouvelle cellule et exécutez-le :

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

2. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Tables**, sélectionnez **Actualiser**. Développez ensuite le nœud **Tables** et vérifiez que la table **managed_products** a été créée.

### Créer une table *externe*

Vous pouvez aussi créer des tables *externes* pour lesquelles les métadonnées de schéma sont définies dans le metastore pour le lakehouse, mais les fichiers de données sont stockés à un emplacement externe.

1. Ajoutez une nouvelle cellule de code, puis ajoutez-y le code suivant :

    ```python
   df.write.format("delta").saveAsTable("external_products", path="<abfs_path>/external_products")
    ```

2. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Files**, sélectionnez **Copier le chemin ABFS**.

    Le chemin ABFS est le chemin complet du dossier **Files** dans le stockage OneLake pour votre lakehouse, et il est similaire à ceci :

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files*

3. Dans le code que vous avez entré dans la cellule de code, remplacez **<abfs_path>** par le chemin que vous avez copié dans le Presse-papiers, pour que le code enregistre le dataframe en tant que table externe avec des fichiers de données dans un dossier nommé **external_products** à l’emplacement de votre dossier **Files**. Le chemin complet doit être similaire a ceci :

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products*

4. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Tables**, sélectionnez **Actualiser**. Développez ensuite le nœud **Tables** et vérifiez que la table **external_products** a été créée.

5. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Files**, sélectionnez **Actualiser**. Développez ensuite le nœud **Files** et vérifiez que le dossier **external_products** a été créé pour les fichiers de données de la table.

### Comparer les tables *managées* et les tables *externes*

Explorons les différences entre les tables managées et les tables externes.

1. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```sql
   %%sql

   DESCRIBE FORMATTED managed_products;
    ```

    Dans les résultats, affichez la propriété **Location** de la table, qui doit être un chemin vers le stockage OneLake pour le lakehouse se terminant par **/Tables/managed_products** (vous devrez peut-être élargir la colonne **Type de données** pour voir le chemin complet).

2. Modifiez la commande `DESCRIBE` pour afficher les détails de la table **external_products** comme indiqué ici :

    ```sql
   %%sql

   DESCRIBE FORMATTED external_products;
    ```

    Dans les résultats, examinez la propriété **Location** de la table, qui doit être un chemin vers le stockage OneLake pour le lakehouse se terminant par **/Files/external_products** (vous devrez peut-être élargir la colonne **Type de données** pour voir le chemin complet).

    Les fichiers pour la table managée sont stockés dans le dossier **Tables** dans le stockage OneLake pour le lakehouse. Dans le cas présent, un dossier nommé **managed_products** a été créé pour stocker les fichiers Parquet, et le dossier **delta_log** pour la table que vous avez créée.

3. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```sql
   %%sql

   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

4. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Tables**, sélectionnez **Actualiser**. Développez ensuite le nœud **Tables** et vérifiez qu’aucune table n’est listée.

5. Dans le volet **Explorateur Lakehouse**, développez le dossier **Files** et vérifiez que **external_products** n’a pas été supprimé. Sélectionnez ce dossier pour visualiser les fichiers de données Parquet et le dossier **_delta_log** pour les données qui se trouvaient auparavant dans la table **external_products**. Les métadonnées de table pour la table externe ont été supprimées, mais les fichiers n’ont pas été affectés.

### Utiliser SQL pour créer une table

1. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```sql
   %%sql

   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. Dans le volet **Explorateur Lakehouse**, dans le menu **…** du dossier **Tables**, sélectionnez **Actualiser**. Développez ensuite le nœud **Tables** et vérifiez qu’une nouvelle table nommée **products** est listée. Développez ensuite la table pour vérifier que son schéma correspond au dataframe d’origine qui a été enregistré dans le dossier **external_products**.

3. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```sql
   %%sql

   SELECT * FROM products;
   ```

## Explorer le versioning des tables

L’historique des transactions pour les tables delta est stocké dans des fichiers JSON, dans le dossier **delta_log**. Vous pouvez utiliser ce journal des transactions pour gérer le versioning des données.

1. Ajoutez une nouvelle cellule de code au notebook, puis exécutez le code suivant :

    ```sql
   %%sql

   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

    Ce code implémente une réduction de 10 % du prix des VTT (mountain bikes).

2. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```sql
   %%sql

   DESCRIBE HISTORY products;
    ```

    Les résultats montrent l’historique des transactions enregistrées pour la table.

3. Ajoutez une autre cellule de code et exécutez le code suivant :

    ```python
   delta_table_path = 'Files/external_products'

   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

    Les résultats montrent deux dataframes : un contenant les données après la réduction du prix et l’autre montrant la version d’origine des données.

## Utiliser des tables delta pour les données de streaming

Delta Lake prend en charge les données de streaming. Les tables delta peuvent être un *récepteur* ou une *source* pour des flux de données créés en utilisant l’API Spark Structured Streaming. Dans cet exemple, vous allez utiliser une table delta comme récepteur pour des données de streaming dans un scénario IoT (Internet des objets) simulé.

1. Ajoutez une nouvelle cellule de code dans le notebook. Ensuite, dans la nouvelle cellule, ajoutez le code suivant et exécutez-le :

    ```python
   from notebookutils import mssparkutils
   from pyspark.sql.types import *
   from pyspark.sql.functions import *

   # Create a folder
   inputPath = 'Files/data/'
   mssparkutils.fs.mkdirs(inputPath)

   # Create a stream that reads data from the folder, using a JSON schema
   jsonSchema = StructType([
   StructField("device", StringType(), False),
   StructField("status", StringType(), False)
   ])
   iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

   # Write some event data to the folder
   device_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''
   mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
   print("Source stream created...")
    ```

    Vérifiez que le message *Flux source créé...* est affiché. Le code que vous venez d’exécuter a créé une source de données de streaming basée sur un dossier dans lequel des données ont été enregistrées, représentant les lectures d’appareils IoT hypothétiques.

2. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

    Ce code écrit les données des appareils de streaming au format delta dans un dossier nommé **iotdevicedata**. En raison du chemin de l’emplacement du dossier dans le dossier **Tables**, une table sera créée automatiquement pour celui-ci.

3. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    Ce code interroge la table **IotDeviceData**, qui contient les données des appareils provenant de la source de streaming.

4. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Ce code écrit plus de données d’appareils hypothétiques dans la source de streaming.

5. Réexécutez la cellule contenant le code suivant :

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    Ce code interroge à nouveau la table **IotDeviceData**, qui doit maintenant inclure les données supplémentaires qui ont été ajoutées à la source de streaming.

6. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
   deltastream.stop()
    ```

    Ce code arrête le flux.

## Nettoyer les ressources

Dans cet exercice, vous avez découvert comment travailler avec des tables delta dans Microsoft Fabric.

Si vous avez terminé d’explorer votre lakehouse, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour visualiser tous les éléments qu’il contient.
2. Dans le menu  **…** de la barre d’outils, sélectionnez **Paramètres des espaces de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
