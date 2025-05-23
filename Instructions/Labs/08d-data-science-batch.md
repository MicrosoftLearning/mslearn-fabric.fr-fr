---
lab:
  title: "Générer des prédictions par lots en tirant parti d’un modèle déployé dans Microsoft\_Fabric"
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# Générer des prédictions par lots en tirant parti d’un modèle déployé dans Microsoft Fabric

Dans ce labo, vous allez utiliser un modèle Machine Learning pour prédire une mesure quantitative de diabète.

En effectuant ce labo, vous allez acquérir une expérience pratique de la génération de prédictions et de la visualisation des résultats.

Ce labo prend environ **20** minutes.

> **Remarque** : Vous devez disposer d’un [essai gratuit Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Accédez à la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric` dans un navigateur et connectez-vous avec vos informations d’identification Fabric.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un notebook

Vous utilisez un *notebook* pour effectuer l’apprentissage et utiliser un modèle dans cet exercice.

1. Sélectionnez **Créer** dans la barre de menus de gauche. Dans la page *Nouveau*, sous la section *Science des données*, sélectionnez **Notebook**. Donnez-lui un nom unique de votre choix.

    >**Note** : si l’option **Créer** n’est pas épinglée à la barre latérale, vous devez d’abord sélectionner l’option avec des points de suspension (**...**).

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

1. Sélectionnez la première cellule (qui est actuellement une cellule de *code*) puis, dans la barre d’outils dynamique en haut à droite, utilisez le bouton **M&#8595;** pour convertir la cellule en cellule *Markdown*.

    Lorsque la cellule devient une cellule Markdown, le texte qu’elle contient est affiché.

1. Si nécessaire, utilisez le bouton **&#128393;** (Modifier) pour basculer la cellule en mode d’édition, puis supprimez le contenu et entrez le texte suivant :

    ```text
   # Train and use a machine learning model
    ```

## Entraîner un modèle Machine Learning

Tout d’abord, effectuons l’apprentissage du modèle Machine Learning qui utilise un algorithme de *régression* pour prédire une réponse d’intérêt pour des patients diabétiques (une mesure quantitative de la progression de la maladie un an après la base de référence)

1. Dans votre bloc-notes, utilisez l’icône **+ Code** sous la dernière cellule pour ajouter une nouvelle cellule de code au bloc-notes.

    > **Conseil** : pour afficher l’icône **+ Code** , déplacez la souris juste en dessous et à gauche de la sortie de la cellule active. Sinon, dans la barre de menus, sous l’onglet **Modifier**, sélectionnez **+ Ajouter une cellule de code**.

1. Entrez le code suivant pour charger et préparer des données et les utiliser pour effectuer l’apprentissage d’un modèle.

    ```python
   import pandas as pd
   import mlflow
   from sklearn.model_selection import train_test_split
   from sklearn.tree import DecisionTreeRegressor
   from mlflow.models.signature import ModelSignature
   from mlflow.types.schema import Schema, ColSpec

   # Get the data
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r""
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   df = spark.read.parquet(wasbs_path).toPandas()

   # Split the features and label for training
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

   # Train the model in an MLflow experiment
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
   with mlflow.start_run():
       mlflow.autolog(log_models=False)
       model = DecisionTreeRegressor(max_depth=5)
       model.fit(X_train, y_train)
       
       # Define the model signature
       input_schema = Schema([
           ColSpec("integer", "AGE"),
           ColSpec("integer", "SEX"),\
           ColSpec("double", "BMI"),
           ColSpec("double", "BP"),
           ColSpec("integer", "S1"),
           ColSpec("double", "S2"),
           ColSpec("double", "S3"),
           ColSpec("double", "S4"),
           ColSpec("double", "S5"),
           ColSpec("integer", "S6"),
        ])
       output_schema = Schema([ColSpec("integer")])
       signature = ModelSignature(inputs=input_schema, outputs=output_schema)
   
       # Log the model
       mlflow.sklearn.log_model(model, "model", signature=signature)
    ```

1. Utilisez le bouton **&#9655; Exécuter la cellule** à gauche de la cellule pour l’exécuter. Vous pouvez également appuyer sur **MAJ** + **ENTRÉE** sur votre clavier pour exécuter une cellule.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans cette session, le pool Spark doit être démarré. Cela signifie que la première exécution dans la session peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant afin d’inscrire le modèle entraîné par l’expérience dans la cellule précédente :

    ```python
   # Get the most recent experiement run
   exp = mlflow.get_experiment_by_name(experiment_name)
   last_run = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=1)
   last_run_id = last_run.iloc[0]["run_id"]

   # Register the model that was trained in that run
   print("Registering the model from run :", last_run_id)
   model_uri = "runs:/{}/model".format(last_run_id)
   mv = mlflow.register_model(model_uri, "diabetes-model")
   print("Name: {}".format(mv.name))
   print("Version: {}".format(mv.version))
    ```

    Votre modèle est désormais enregistré dans votre espace de travail en tant que **diabetes-model**. Vous pouvez éventuellement utiliser la fonctionnalité de navigation dans votre espace de travail pour rechercher le modèle dans l’espace de travail et l’explorer en utilisant l’interface utilisateur.

## Créer un jeu de données test dans un lakehouse

Pour utiliser le modèle, vous allez avoir besoin d’un jeu de données des informations du patient pour lequel vous devez prédire un diagnostic de diabète. Vous allez créer ce jeu de données en tant que table dans un lakehouse Microsoft Fabric.

1. Dans l’éditeur Notebook, dans le volet **Explorer** sur la gauche, sélectionnez **+ Source de données** pour ajouter un Lakehouse.
1. Sélectionnez **Nouveau lakehouse**, puis **Ajouter** et créez un **Lakehouse** avec un nom valide de votre choix.
1. Au moment de l’invitation vous demandant d’arrêter la session active, sélectionnez **Arrêter maintenant** pour redémarrer le notebook.
1. Une fois le lakehouse créé et attaché à votre notebook, ajoutez une nouvelle cellule de code pour exécuter le code suivant afin de créer un jeu de données et l’enregistrer dans la table d’un lakehouse :

    ```python
   from pyspark.sql.types import IntegerType, DoubleType

   # Create a new dataframe with patient data
   data = [
       (62, 2, 33.7, 101.0, 157, 93.2, 38.0, 4.0, 4.8598, 87),
       (50, 1, 22.7, 87.0, 183, 103.2, 70.0, 3.0, 3.8918, 69),
       (76, 2, 32.0, 93.0, 156, 93.6, 41.0, 4.0, 4.6728, 85),
       (25, 1, 26.6, 84.0, 198, 131.4, 40.0, 5.0, 4.8903, 89),
       (53, 1, 23.0, 101.0, 192, 125.4, 52.0, 4.0, 4.2905, 80),
       (24, 1, 23.7, 89.0, 139, 64.8, 61.0, 2.0, 4.1897, 68),
       (38, 2, 22.0, 90.0, 160, 99.6, 50.0, 3.0, 3.9512, 82),
       (69, 2, 27.5, 114.0, 255, 185.0, 56.0, 5.0, 4.2485, 92),
       (63, 2, 33.7, 83.0, 179, 119.4, 42.0, 4.0, 4.4773, 94),
       (30, 1, 30.0, 85.0, 180, 93.4, 43.0, 4.0, 5.3845, 88)
   ]
   columns = ['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']
   df = spark.createDataFrame(data, schema=columns)

   # Convert data types to match the model input schema
   df = df.withColumn("AGE", df["AGE"].cast(IntegerType()))
   df = df.withColumn("SEX", df["SEX"].cast(IntegerType()))
   df = df.withColumn("BMI", df["BMI"].cast(DoubleType()))
   df = df.withColumn("BP", df["BP"].cast(DoubleType()))
   df = df.withColumn("S1", df["S1"].cast(IntegerType()))
   df = df.withColumn("S2", df["S2"].cast(DoubleType()))
   df = df.withColumn("S3", df["S3"].cast(DoubleType()))
   df = df.withColumn("S4", df["S4"].cast(DoubleType()))
   df = df.withColumn("S5", df["S5"].cast(DoubleType()))
   df = df.withColumn("S6", df["S6"].cast(IntegerType()))

   # Save the data in a delta table
   table_name = "diabetes_test"
   df.write.format("delta").mode("overwrite").saveAsTable(table_name)
   print(f"Spark dataframe saved to delta table: {table_name}")
    ```

1. Une fois le code terminé, sélectionnez les **...** à côté de **Tables** dans le volet **Explorateur Lakehouse**, puis **Actualiser**. La table **diabetes_test** doit s’afficher.
1. Développez la table **diabetes_test** dans le volet gauche pour afficher tous les champs qu’elle comprend.

## Appliquer le modèle pour générer des prédictions

Maintenant vous pouvez utiliser le modèle entraîné précédemment pour générer des prévisions de progression du diabète pour les lignes de données de patients dans votre table.

1. Ajoutez une nouvelle cellule de code, puis exécutez le code suivant :

    ```python
   import mlflow
   from synapse.ml.predict import MLFlowTransformer

   ## Read the patient features data 
   df_test = spark.read.format("delta").load(f"Tables/{table_name}")

   # Use the model to generate diabetes predictions for each row
   model = MLFlowTransformer(
       inputCols=["AGE","SEX","BMI","BP","S1","S2","S3","S4","S5","S6"],
       outputCol="predictions",
       modelName="diabetes-model",
       modelVersion=1)
   df_test = model.transform(df)

   # Save the results (the original features PLUS the prediction)
   df_test.write.format('delta').mode("overwrite").option("mergeSchema", "true").saveAsTable(table_name)
    ```

1. Une fois le code terminé, sélectionnez les **...** à côté de la table **diabetes_test** dans le volet **Explorateur Lakehouse**, puis **Actualiser**. Un nouveau champ **prévisions** a été ajouté.
1. Ajoutez une nouvelle cellule de code au notebook, puis faites-y glisser la table **diabetes_test**. Le code nécessaire à l’affichage du contenu de la table s’affiche. Exécutez la cellule pour afficher les données.

## Nettoyer les ressources

Dans cet exercice, vous avez utilisé un modèle pour générer des prédictions par lots.

Si vous avez fini d’explorer le notebook, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
