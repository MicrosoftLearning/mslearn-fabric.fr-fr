---
lab:
  title: Créer une architecture en médaillon dans un lakehouse Microsoft Fabric
  module: Organize a Fabric lakehouse using medallion architecture design
---

# Créer une architecture en médaillon dans un lakehouse Microsoft Fabric

Dans cet exercice, vous allez créer une architecture en médaillon dans un lakehouse Fabric à l’aide de notebooks. Vous allez créer un espace de travail, créer un lakehouse, charger les données dans la couche bronze, transformer les données et les charger dans la table Delta argent, transformer davantage les données et les charger dans les tables Delta or, puis explorer le jeu de données et créer des relations.

Cet exercice devrait prendre environ **40** minutes.

> **Remarque** : Vous devez disposer d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail et activer la modification du modèle de données

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace-medallion.png)
5. Accédez aux paramètres de l’espace de travail et activez la fonctionnalité d’évaluation **Modification du modèle de données**. Cela vous permet de créer des relations entre des tables dans votre lakehouse.

    ![Capture d’écran de la page des paramètres de l’espace de travail dans Power BI.](./Images/workspace-settings.png)

    > **Remarque** : vous devrez peut-être actualiser l’onglet du navigateur après avoir activé la fonctionnalité d’évaluation.
## Créer un lakehouse et charger les données dans la couche bronze

Maintenant que vous disposez d’un espace de travail, il est temps de passer à l’expérience *Engineering données* dans le portail Fabric et de créer un data lakehouse pour les données que vous allez analyser.

1. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI** et basculez vers l’expérience **Engineering données**.

2. Dans la page d’accueil d’**Engineering données Synapse**, créez un **Lakehouse** avec le nom de votre choix.

    Au bout d’une minute environ, un nouveau lakehouse vide est créé. Vous devez ingérer certaines données dans le data lakehouse à des fins d’analyse. Il existe plusieurs façons de faire cela mais dans cet exercice, vous allez simplement télécharger un fichier texte sur votre ordinateur local (ou sur votre machine virtuelle de labo le cas échéant), puis le charger dans votre lakehouse.

3. Téléchargez le fichier de données pour cet exercice à partir de `https://github.com/MicrosoftLearning/dp-data/blob/main/orders.zip`. Extrayez des fichiers et enregistrez-les avec leur nom d’origine sur votre ordinateur local (ou votre machine virtuelle de labo le cas échéant). Il doit y avoir 3 fichiers contenant des données de vente pour 3 ans : 2019.csv, 2020.csv et 2021.csv.

4. Retournez à l’onglet du navigateur web contenant votre lakehouse et, dans le menu  **...** du dossier **Fichiers** dans le volet **Explorateur**, sélectionnez **Nouveau sous-dossier**, puis créez un sous-dossier nommé **bronze**.

5. Dans le menu **...** du dossier **bronze**, sélectionnez **Charger** et **Charger des fichiers**, puis chargez les 3 fichiers (2019.csv, 2020.csv et 2021.csv) depuis votre ordinateur local (ou votre machine virtuelle de labo le cas échéant) dans le lakehouse. Utilisez la touche MAJ pour charger les 3 fichiers à la fois.
   
6. Une fois les fichiers chargés, sélectionnez le dossier **bronze**, et vérifiez que les fichiers ont été chargés, comme illustré ici :

    ![Capture d’écran du fichier products.csv chargé dans un lakehouse.](./Images/bronze-files.png)

## Transformer les données et les charger dans une table Delta argent

Maintenant que vous avez des données dans la couche bronze de votre lakehouse, vous pouvez utiliser un notebook pour les transformer et les charger dans une table Delta dans la couche argent. 

1. Dans la page **Accueil**, lors de l’affichage du contenu du dossier **bronze** dans votre lac de données, dans le menu **Ouvrir un notebook**, sélectionnez **Nouveau notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

2. Lorsque le notebook s’ouvre, nommez-le **Transformer les données pour la couche argent** en sélectionnant le texte **Notebook xxxx** en haut à gauche du notebook et en entrant le nouveau nom.

    ![Capture d’écran d’un nouveau notebook nommé Sales.](./Images/sales-notebook-rename.png)

2. Sélectionnez la cellule existante dans le notebook, qui contient du code simple commenté. Mettez en surbrillance et supprimez ces deux lignes ; vous n’aurez pas besoin de ce code.
   
   > **Remarque** : les notebooks vous permettent d’exécuter du code dans divers langages, notamment Python, Scala et SQL. Dans cet exercice, vous allez utiliser PySpark et SQL. Vous pouvez également ajouter des cellules markdown pour fournir du texte et des images mis en forme pour documenter votre code.

3. Collez le code suivant dans la cellule :

    ```python
    from pyspark.sql.types import *
    
    # Create the schema for the table
    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    # Import all files from bronze folder of lakehouse
    df = spark.read.format("csv").option("header", "true").schema(orderSchema).load("Files/bronze/*.csv")
    
    # Display the first 10 rows of the dataframe to preview your data
    display(df.head(10))
    ```

4. Utilisez le bouton **&#9655;** (*Exécuter la cellule*) à gauche de la cellule pour exécuter le code.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans ce notebook, une session Spark doit être démarrée. Cela signifie que la première exécution peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

5. Une fois la commande de la cellule exécutée, examinez la sortie sous la cellule, qui doit être similaire à ceci :

    | Index | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | Courrier | Élément | Quantité | UnitPrice | Taxe |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO49172 | 1 | 2021-01-01 | Brian Howard | brian23@adventure-works.com | Road-250 Red, 52 | 1 | 2443,35 | 195,468 |
    | 2 |  SO49173 | 1 | 2021-01-01 | Linda Alvarez | Mountain-200 Silver, 38 | 1 | 2071.4197 | 165,7136 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    Le code que vous avez exécuté a chargé les données des fichiers CSV du dossier **bronze** dans une trame de données Spark, puis a affiché les premières lignes de la trame de données.

    > **Remarque** : vous pouvez effacer, masquer et redimensionner automatiquement le contenu de la sortie de cellule en sélectionnant le menu **...** en haut à gauche du volet de sortie.

6. Vous allez maintenant ajouter des colonnes pour la validation et le nettoyage des données, à l’aide d’une trame de données PySpark pour ajouter des colonnes et mettre à jour les valeurs de certaines des colonnes existantes. Utilisez le bouton + pour ajouter un nouveau bloc de code et ajoutez le code suivant à la cellule :

    ```python
    from pyspark.sql.functions import when, lit, col, current_timestamp, input_file_name
    
    # Add columns FileName, IsFlagged, CreatedTS and ModifiedTS for data validation and tracking
    df = df.withColumn("FileName", input_file_name())
    df = df.withColumn("IsFlagged", when(col("OrderDate") < '2019-08-01',True).otherwise(False))
    df = df.withColumn("CreatedTS", current_timestamp()).withColumn("ModifiedTS", current_timestamp())
    df = df.withColumn("CustomerID", lit(None).cast("BigInt"))
    df = df.withColumn("ItemID", lit(None).cast("BigInt"))
    
    # Update CustomerName to "Unknown" if CustomerName null or empty
    df = df.withColumn("CustomerName", when((col("CustomerName").isNull() | (col("CustomerName")=="")),lit("Unknown")).otherwise(col("CustomerName")))
    ```

    La première ligne du code que vous avez exécuté importe les fonctions nécessaires à partir de PySpark. Vous ajoutez ensuite de nouvelles colonnes à la trame de données afin de pouvoir suivre le nom du fichier source (si l’ordre a été marqué comme étant avant l’exercice concerné) et quand la ligne a été créée et modifiée. 
    
    Vous ajoutez également des colonnes pour CustomerID et ItemID, qui seront renseignées ultérieurement.
    
    Enfin, vous mettez à jour la colonne CustomerName pour « Inconnu » si elle a une valeur nulle ou est vide.

7. Exécutez la cellule pour exécuter le code à l’aide du bouton **&#9655;** (*Exécuter la cellule*).

8. Ensuite, vous allez utiliser SQL magic pour créer votre trame de données nettoyée en tant que nouvelle table appelée sales_silver dans la base de données des ventes au format Delta Lake. Créez un bloc de code et ajoutez le code suivant à la cellule :

    ```python
     %%sql
    
    -- Create sales_silver table 
    CREATE TABLE sales.sales_silver (
        SalesOrderNumber string
        , SalesOrderLineNumber int
        , OrderDate date
        , CustomerName string
        , Email string
        , Item string
        , Quantity int
        , UnitPrice float
        , Tax float
        , FileName string
        , IsFlagged boolean
        , CustomerID bigint
        , ItemID bigint
        , CreatedTS date
        , ModifiedTS date
    ) USING delta;
    ```

    Ce code utilise la commande `%sql` magic pour exécuter des instructions SQL. La première instruction crée une base de données nommée **sales**. La deuxième instruction crée une table nommée **sales_silver** dans la base de données **sales**, à l’aide du format Delta Lake et de la trame de données que vous avez créée dans le bloc de code précédent.

9. Exécutez la cellule pour exécuter le code à l’aide du bouton **&#9655;** (*Exécuter la cellule*).

10. Sélectionnez le bouton **...** dans la section Tables du volet Explorateur lakehouse, puis sélectionnez **Actualiser**. Vous devrez maintenant voir la nouvelle table **sales_silver** répertoriée. L’icône triangle indique qu’il s’agit d’une table Delta.

    ![Capture d’écran de la table sales_silver dans un lakehouse.](./Images/sales-silver-table.png)

    > **Remarque** : si vous ne voyez pas la nouvelle table, attendez quelques secondes, puis sélectionnez **Actualiser** à nouveau ou actualisez l’onglet du navigateur.

11. Vous allez maintenant effectuer une opération upsert sur une table Delta, mettant à jour les enregistrements existants en fonction de conditions spécifiques et insérant de nouveaux enregistrements lorsqu’aucune correspondance n’est trouvée. Ajoutez un nouveau bloc de code et collez le code suivant :

    ```python
    # Update existing records and insert new ones based on a condition defined by the columns SalesOrderNumber, OrderDate, CustomerName, and Item.

    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/sales_silver')
    
    dfUpdates = df
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.SalesOrderNumber = updates.SalesOrderNumber and silver.OrderDate = updates.OrderDate and silver.CustomerName = updates.CustomerName and silver.Item = updates.Item'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "SalesOrderNumber": "updates.SalesOrderNumber",
          "SalesOrderLineNumber": "updates.SalesOrderLineNumber",
          "OrderDate": "updates.OrderDate",
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "Item": "updates.Item",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax",
          "FileName": "updates.FileName",
          "IsFlagged": "updates.IsFlagged",
          "CustomerID": "updates.CustomerID",
          "ItemID": "updates.ItemID",
          "CreatedTS": "updates.CreatedTS",
          "ModifiedTS": "updates.ModifiedTS"
        }
      ) \
      .execute()
    ```
    Cette opération est importante, car elle vous permet de mettre à jour les enregistrements existants dans la table en fonction des valeurs de colonnes spécifiques et d’insérer de nouveaux enregistrements lorsqu’aucune correspondance n’est trouvée. Il s’agit d’une exigence courante lorsque vous chargez depuis un système source des données qui peuvent contenir des mises à jour de lignes existantes et de nouvelles lignes.

Vous disposez maintenant dans votre table Delta argent de données qui sont prêtes pour une transformation et une modélisation ultérieures.
    

## Transformer les données pour la couche or

Vous avez réussi à extraire des données de votre couche bronze, à les transformer et à les charger dans une table Delta argent. Vous allez maintenant utiliser un nouveau notebook pour transformer davantage les données, les modéliser en schéma en étoile et les charger dans des tables Delta or.

Notez que vous auriez pu effectuer toutes ces opérations dans un seul notebook, mais pour les besoins de cet exercice, vous utilisez des notebooks distincts pour illustrer le processus de transformation des données de la couche bronze vers la couche argent, puis de la couche argent vers la couche or. Cela peut faciliter le débogage, la résolution des problèmes et la réutilisation.

1. Revenez à la page d’accueil **Engineering données** et créez un notebook appelé **Transformer les données pour la couche or**.

2. Dans le volet Explorateur lakehouse, ajoutez votre lakehouse **Sales** en sélectionnant **Ajouter**, puis en sélectionnant le lakehouse **Sales** que vous avez créé précédemment. Vous devrez voir la table **sales_silver** répertoriée dans la section **Tables** du volet Explorateur.

3. Dans le bloc de code existant, supprimez le texte réutilisable et ajoutez le code suivant pour charger des données dans votre trame de données et commencer à créer votre schéma en étoile :

    ```python
    # Load data to the dataframe as a starting point to create the gold layer
    df = spark.read.table("Sales.sales_silver")
    ```

4. Ajoutez un nouveau bloc de code et collez le code suivant pour créer votre table de dimension Date :

    ```python
        %%sql
    -- Create Date_gold dimension table
    CREATE TABLE IF NOT EXISTS sales.dimdate_gold (
        OrderDate date
        , Day int
        , Month int
        , Year int
        , `mmmyyyy` string
        , yyyymm string
    ) USING DELTA;
    
    ```
    > **Remarque** : vous pouvez exécuter la commande `display(df)` à tout moment pour vérifier la progression de votre travail. Dans ce cas, vous exécutez « display(dfdimDate_gold) » pour afficher le contenu de la trame de données dimDate_gold.

5. Dans un nouveau bloc de code, ajoutez le code suivant pour mettre à jour la dimension Date à mesure de l’intégration de nouvelles données :

    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/dimdate_gold')
    
    dfUpdates = dfdimDate_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.OrderDate = updates.OrderDate'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "OrderDate": "updates.OrderDate",
          "Day": "updates.Day",
          "Month": "updates.Month",
          "Year": "updates.Year",
          "mmmyyyy": "updates.mmmyyyy",
          "yyyymm": "yyyymm"
        }
      ) \
      .execute()
    ```
5. Nous allons maintenant créer notre table de dimension Client. Ajoutez un nouveau bloc de code et collez le code suivant :

    ```python
   %%sql
    -- Create Customer dimension table
    CREATE TABLE sales.dimCustomer_gold (
        CustomerName string
        , Email string
        , First string
        , Last string
        , CustomerID BIGINT
    ) USING DELTA;
    ```
    
6. Dans un nouveau bloc de code, ajoutez le code suivant pour mettre à jour la dimension Client à mesure de l’intégration de nouvelles données :

    ```python
    from pyspark.sql.functions import col, split

    # Create Customer_gold dataframe

    dfdimCustomer_silver = df.dropDuplicates(["CustomerName","Email"]).select(col("CustomerName"),col("Email")) \
        .withColumn("First",split(col("CustomerName"), " ").getItem(0)) \
        .withColumn("Last",split(col("CustomerName"), " ").getItem(1)) \
    ```

     Ici, vous avez créé une nouvelle trame de données dfdimCustomer_silver en effectuant diverses transformations telles que la suppression des doublons, la sélection de colonnes spécifiques et le fractionnement de la colonne « CustomerName » pour créer des colonnes de nom « First » et « Last ». Le résultat est une trame de données avec des données client nettoyées et structurées, y compris des colonnes de nom « First » et « Last » distinctes extraites de la colonne « CustomerName ».

7. Ensuite, nous allons créer la colonne ID pour nos clients. Dans un nouveau bloc de code, collez les éléments suivants :

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col, when

    dfdimCustomer_temp = spark.sql("SELECT * FROM dimCustomer_gold")
    CustomerIDCounters = spark.sql("SELECT COUNT(*) AS ROWCOUNT, MAX(CustomerID) AS MAXCustomerID FROM dimCustomer_gold")
    MAXCustomerID = CustomerIDCounters.select((when(col("ROWCOUNT")>0,col("MAXCustomerID"))).otherwise(0)).first()[0]
    
    dfdimCustomer_gold = dfdimCustomer_silver.join(dfdimCustomer_temp,(dfdimCustomer_silver.CustomerName == dfdimCustomer_temp.CustomerName) & (dfdimCustomer_silver.Email == dfdimCustomer_temp.Email), "left_anti")
    
    dfdimCustomer_gold = dfdimCustomer_gold.withColumn("CustomerID",monotonically_increasing_id() + MAXCustomerID)
    
    ```
    Ici, vous nettoyez et transformez les données client (dfdimCustomer_silver) en effectuant une jointure anti gauche pour exclure les doublons qui existent déjà dans la table dimCustomer_gold, puis en générant des valeurs CustomerID uniques à l’aide de la fonction monotonically_increasing_id().

8. Vous allez maintenant vous assurer que votre table client reste à jour à mesure de l’intégration de nouvelles données. Dans un nouveau bloc de code, collez les éléments suivants :

    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/dimcustomer_gold')
    
    dfUpdates = dfdimCustomer_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.CustomerName = updates.CustomerName AND silver.Email = updates.Email'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "First": "updates.First",
          "Last": "updates.Last",
          "CustomerID": "updates.CustomerID"
        }
      ) \
      .execute()
    ```
9. Vous allez maintenant répéter ces étapes pour créer votre dimension produit. Dans un nouveau bloc de code, collez les éléments suivants :

    ```python
    %%sql
    -- Create Product dimension table
    CREATE TABLE sales.dimProduct_gold (
        Item string
        , ItemID BIGINT
    ) USING DELTA;
    ```    
10. Ajoutez un autre bloc de code pour créer la trame de données customer_gold. Vous l’utiliserez ultérieurement sur la jointure Sales.
    
    ```python
    from pyspark.sql.functions import col, split, lit

    # Create Customer_gold dataframe, this dataframe will be used later on on the Sales join
    
    dfdimProduct_silver = df.dropDuplicates(["Item"]).select(col("Item")) \
        .withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    # display(dfdimProduct_gold)
            ```

11. Now you'll prepare to add new products to the dimProduct_gold table. Add the following syntax to a new code block:

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col

    dfdimProduct_temp = spark.sql("SELECT * FROM dimProduct_gold")
    Product_IDCounters = spark.sql("SELECT COUNT(*) AS ROWCOUNT, MAX(ItemID) AS MAXProductID FROM dimProduct_gold")
    MAXProduct_ID = Product_IDCounters.select((when(col("ROWCOUNT")>0,col("MAXProductID"))).otherwise(0)).first()[0]
    
    
    dfdimProduct_gold = dfdimProduct_gold.withColumn("ItemID",monotonically_increasing_id() + MAXProduct_ID)
    
    #display(dfdimProduct_gold)

12.  Similar to what you've done with your other dimensions, you need to ensure that your product table remains up-to-date as new data comes in. In a new code block, paste the following:
    
    ```python
    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'abfss://Learn@onelake.dfs.fabric.microsoft.com/Sales.Lakehouse/Tables/dimproduct_gold')
    
    dfUpdates = dfdimProduct_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.ItemName = updates.ItemName AND silver.ItemInfo = updates.ItemInfo'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "ItemName": "updates.ItemName",
          "ItemInfo": "updates.ItemInfo",
          "ItemID": "updates.ItemID"
        }
      ) \
      .execute()
    ```

    Cela calcule l’ID de produit disponible suivant en fonction des données actuelles dans la table, affecte ces nouveaux ID aux produits, puis affiche les informations de produit mises à jour (si la commande d’affichage n’est pas commentée).

Maintenant que vos dimensions sont générées, la dernière étape consiste à créer la table de faits.

13. Dans un nouveau bloc de code, collez le code suivant pour créer la table de faits :

    ```python
       %%sql
    -- Create Date_gold dimension table if not exist
    CREATE TABLE IF NOT EXISTS sales.factsales_gold (
        CustomerID BIGINT
        , ItemID BIGINT
        , OrderDate date
        , Quantity INT
        , UnitPrice float
        , Tax float
    ) USING DELTA;
    ```
14. Dans un nouveau bloc de code, collez le code suivant pour créer une nouvelle trame de données afin de combiner les données de vente avec les informations client et produit, notamment l’ID client, l’ID d’article, la date de commande, la quantité, le prix unitaire et les taxes :

    ```python
    from pyspark.sql.functions import col

    dfdimCustomer_temp = spark.sql("SELECT * FROM dimCustomer_gold")
    dfdimProduct_temp = spark.sql("SELECT * FROM dimProduct_gold")
    
    df = df.withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    # Create Sales_gold dataframe
    
    dffactSales_gold = df.alias("df1").join(dfdimCustomer_temp.alias("df2"),(df.CustomerName == dfdimCustomer_temp.CustomerName) & (df.Email == dfdimCustomer_temp.Email), "left") \
            .join(dfdimProduct_temp.alias("df3"),(df.ItemName == dfdimProduct_temp.ItemName) & (df.ItemInfo == dfdimProduct_temp.ItemInfo), "left") \
        .select(col("df2.CustomerID") \
            , col("df3.ItemID") \
            , col("df1.OrderDate") \
            , col("df1.Quantity") \
            , col("df1.UnitPrice") \
            , col("df1.Tax") \
        ).orderBy(col("df1.OrderDate"), col("df2.CustomerID"), col("df3.ItemID"))
    
    
    display(dffactSales_gold)
    ```

15. Vous allez maintenant vous assurer que les données de vente restent à jour en exécutant le code suivant dans un nouveau bloc de code :
    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://Learn@onelake.dfs.fabric.microsoft.com/Sales.Lakehouse/Tables/factsales_gold')
    
    dfUpdates = dffactSales_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.OrderDate = updates.OrderDate AND silver.CustomerID = updates.CustomerID AND silver.ItemID = updates.ItemID'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerID": "updates.CustomerID",
          "ItemID": "updates.ItemID",
          "OrderDate": "updates.OrderDate",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax"
        }
      ) \
      .execute()
    ```
     Ici, vous utilisez l’opération de fusion de Delta Lake pour synchroniser et mettre à jour la table factsales_gold avec les nouvelles données de vente (dffactSales_gold). L’opération compare la date de commande, l’ID client et l’ID d’articles entre les données existantes (table argent) et les nouvelles données (met à jour la trame de données), mettant à jour les lignes correspondantes et insérant de nouvelles lignes si nécessaire.

Vous disposez maintenant d’une couche or organisée et modélisée qui peut être utilisée pour la création de rapports et l’analyse.

## Créer un jeu de données

Dans votre espace de travail, vous pouvez désormais utiliser la couche or pour créer un rapport et analyser les données. Vous pouvez accéder au jeu de données directement dans votre espace de travail afin de créer des relations et des mesures pour la génération de rapports.

Remarque : vous ne pouvez pas utiliser le jeu de données par défaut créé automatiquement lorsque vous créez un lakehouse. Vous devez créer un jeu de données qui inclut les tables or que vous avez créées dans cet exercice, à partir de l’Explorateur lakehouse.

1. Dans votre espace de travail, accédez à votre lakehouse **Sales**.
2. Sélectionnez **Nouveau jeu de données Power BI** dans le ruban de la vue Explorateur lakehouse.
3. Sélectionnez vos tables or transformées à inclure dans votre jeu de données, puis **Confirmer**.
   - dimdate_gold
   - dimcustomer_gold
   - dimproduct_gold
   - factsales_gold

    Cela ouvre le jeu de données dans Fabric, où vous pouvez créer des relations et des mesures.

4. Renommez votre jeu de données afin de l’identifier plus facilement. Sélectionnez le nom du jeu de données dans le coin supérieur gauche de la fenêtre. Renommez le jeu de données en **Sales_Gold**.

À partir de là, vous ou d’autres membres de votre équipe de données pouvez créer des rapports et des tableaux de bord en fonction des données de votre lakehouse. Ces rapports seront directement connectés à la couche or de votre lakehouse, de sorte à refléter en permanence les données les plus récentes.

## Nettoyer les ressources

Dans cet exercice, vous avez appris à créer une architecture en médaillon dans un lakehouse Microsoft Fabric.

Si vous avez terminé d’explorer votre lakehouse, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
