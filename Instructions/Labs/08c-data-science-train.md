---
lab:
  title: "Former et suivre un modèle dans Microsoft\_Fabric"
  module: Train and track machine learning models with MLflow in Microsoft Fabric
---

# Entraîner et suivre des modèles Machine Learning avec MLflow dans Microsoft Fabric

Dans ce labo, vous allez effectuer l'apprentissage d’un modèle Machine Learning pour prédire une mesure quantitative de diabète. Vous allez effectuer l'apprentissage d’un modèle de régression avec scikit-learn, puis suivre et comparer vos modèles avec MLflow.

En suivant ce labo, vous allez acquérir une expérience pratique du Machine Learning et du suivi des modèles, et apprendre à utiliser des *notebooks*, des *expériences* et des *modèles* dans Microsoft Fabric.

Ce labo prend environ **25** minutes.

> **Remarque** : Vous aurez besoin d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des modèles dans Fabric, créez un espace de travail en activant l’essai gratuit de Fabric.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)

## Créer un notebook

Pour entraîner un modèle, vous pouvez créer un *notebook*. Les notebooks fournissent un environnement interactif dans lequel vous pouvez écrire et exécuter du code (dans plusieurs langues).

1. En bas à gauche du portail Fabric, sélectionnez l’icône **Power BI** et basculez vers l’expérience **Science des données**.

1. Dans la page d’accueil de **Science des données**, créez un **notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

1. Sélectionnez la première cellule (qui est actuellement une cellule de *code*) puis, dans la barre d’outils dynamique en haut à droite, utilisez le bouton **M&#8595;** pour convertir la cellule en cellule *Markdown*.

    Lorsque la cellule devient une cellule Markdown, le texte qu’elle contient est affiché.

1. Utilisez le bouton **&#128393;** (Modifier) pour basculer la cellule en mode édition, puis supprimez le contenu et entrez le texte suivant :

    ```text
   # Train a machine learning model and track with MLflow
    ```

## Charger des données dans un DataFrame

Vous êtes maintenant prêt à exécuter du code pour obtenir des données et former un modèle. Vous allez utiliser le [jeu de données diabetes](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) à partir d’Azure Open Datasets. Après avoir chargé les données, vous allez convertir les données en dataframe Pandas, qui est une structure courante pour l’utilisation des données dans les lignes et les colonnes.

1. Dans votre notebook, utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant :

    ```python
    # Azure storage access info for open dataset diabetes
    blob_account_name = "azureopendatastorage"
    blob_container_name = "mlsamples"
    blob_relative_path = "diabetes"
    blob_sas_token = r"" # Blank since container is Anonymous access
    
    # Set Spark config to access  blob storage
    wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
    spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
    print("Remote blob path: " + wasbs_path)
    
    # Spark read parquet, note that it won't load any data yet by now
    df = spark.read.parquet(wasbs_path)
    ```

1. Utilisez le bouton **&#9655; Exécuter la cellule** à gauche de la cellule pour l’exécuter. Vous pouvez également appuyer `SHIFT` + `ENTER` sur votre clavier pour exécuter une cellule.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans cette session, le pool Spark doit être démarré. Cela signifie que la première exécution dans la session peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant :

    ```python
    display(df)
    ```

1. Une fois la commande de la cellule exécutée, examinez la sortie sous la cellule, qui doit être similaire à ceci :

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|O|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32,1|101.0|157|93,2|38.0|4.0|4,8598|87|151|
    |48|1|21,6|87,0|183|103,2|70.0|3.0|3,8918|69|75|
    |72|2|30.5|93.0|156|93,6|41,0|4.0|4,6728|85 %|141|
    |24|1|25,3|84.0|198|131,4|40,0|5.0|4,8903|89|206|
    |50|1|23.0|101.0|192|125,4|52,0|4.0|4,2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    La sortie affiche les lignes et les colonnes du jeu de données diabetes.

1. Les données sont chargées en tant que trame de données Spark. Scikit-learn s’attend à ce que le jeu de données d’entrée soit un dataframe Pandas. Exécutez le code ci-dessous pour convertir votre jeu de données en dataframe Pandas :

    ```python
    import pandas as pd
    df = df.toPandas()
    df.head()
    ```

## Entraîner un modèle Machine Learning

Maintenant que vous avez chargé les données, vous pouvez les utiliser pour former un modèle Machine Learning et prédire une mesure quantitative du diabète. Vous allez former un modèle de régression à l’aide de la bibliothèque scikit-learn et suivre le modèle avec MLflow.

1. Exécutez le code suivant pour fractionner les données en un jeu de données de formation et de test, et pour séparer les fonctionnalités de l’étiquette que vous souhaitez prédire :

    ```python
    from sklearn.model_selection import train_test_split
    
    print("Splitting data...")
    X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
    
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   import mlflow
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
    ```

    Le code crée une expérience MLflow nommée `experiment-diabetes`. Vos modèles seront suivis dans cette expérience.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
    from sklearn.linear_model import LinearRegression
    
    with mlflow.start_run():
       mlflow.autolog()
    
       model = LinearRegression()
       model.fit(X_train, y_train)
    
       mlflow.log_param("estimator", "LinearRegression")
    ```

    Le code entraîne un modèle de régression à l’aide de la régression linéaire. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow. En outre, vous journaliserez un paramètre appelé `estimator`, avec la valeur `LinearRegression`.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
    from sklearn.tree import DecisionTreeRegressor
    
    with mlflow.start_run():
       mlflow.autolog()
    
       model = DecisionTreeRegressor(max_depth=5) 
       model.fit(X_train, y_train)
    
       mlflow.log_param("estimator", "DecisionTreeRegressor")
    ```

    Le code entraîne un modèle de régression à l’aide du régresseur de l’arbre de décision. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow. En outre, vous journaliserez un paramètre appelé `estimator`, avec la valeur `DecisionTreeRegressor`.

## Utiliser MLflow pour rechercher et afficher vos expériences

Une fois que vous avez entraîné et suivi des modèles avec MLflow, vous pouvez utiliser la bibliothèque MLflow pour récupérer vos expériences et leurs détails.

1. Pour lister toutes les expériences, utilisez le code suivant :

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. Vous pouvez récupérer une expérience spécifique par son nom :

    ```python
   experiment_name = "experiment-diabetes"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. Avec un nom d’expérience, vous pouvez récupérer tous les travaux de cette expérience :

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. Pour comparer plus facilement les exécutions de travaux et les sorties, vous pouvez configurer la recherche afin que les résultats soient classés. Par exemple, la cellule suivante classe les résultats selon la valeur `start_time` et affiche un maximum de `2` résultats :

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. Pour finir, vous pouvez tracer les métriques d’évaluation de plusieurs modèles les unes à côté des autres afin de comparer facilement les modèles :

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_r2_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_r2_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("R2 score")
   ax.set_title("R2 score by Estimator")
   for i, v in enumerate(df_results["metrics.training_r2_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    La sortie doit ressembler à l’image suivante :

    ![Capture d’écran des métriques d’évaluation tracées.](./Images/plotted-metrics.png)

## Explorer vos expériences

Microsoft Fabric effectue le suivi de toutes vos expériences et vous permet de les explorer visuellement.

1. Accédez à votre espace de travail à partir de la barre de menu du hub sur la gauche.
1. Sélectionnez l’expérience `experiment-diabetes` pour l’ouvrir.

    > **Conseil :** Si vous ne voyez aucune exécution d’expérience journalisée, actualisez la page.

1. Sélectionnez l’onglet **Affichage**.
1. Sélectionnez **Liste d’exécutions**.
1. Sélectionnez les deux dernières exécutions en cochant chaque case.
    Vos deux dernières exécutions seront comparées l’une à l’autre dans le volet **Comparaison des métriques**. Par défaut, les métriques sont tracées en fonction du nom d’exécution.
1. Sélectionnez le bouton **&#128393;** (Modifier) du graphe affichant l’erreur absolue moyenne de chaque exécution.
1. Sélectionnez `bar` comme **type de visualisation**.
1. Sélectionnez `estimator` comme **axe x**.
1. Sélectionnez **Remplacer** et explorez le nouveau graphe.
1. Si vous le souhaitez, vous pouvez répéter ces étapes pour les autres graphiques dans le volet **comparaison des métriques**.

En traçant les mesures de performance par estimateur journalisé, vous pouvez examiner l’algorithme qui a abouti à un meilleur modèle.

## Enregistrer le modèle

Après avoir comparé les modèles Machine Learning que vous avez entraînés pour les différentes exécutions d’expériences, vous pouvez choisir le modèle le plus performant. Pour utiliser le modèle le plus performant, enregistrez le modèle et utilisez-le afin de générer des prédictions.

1. Dans la vue d’ensemble de l’expérience, vérifiez que l’onglet **Affichage** est sélectionné.
1. Sélectionnez **Détails de l’exécution**.
1. Sélectionnez l’exécution avec le score R2 le plus élevé.
1. Sélectionnez **Enregistrer** dans la zone **Enregistrer l’exécution en tant que modèle**.
1. Sélectionnez **Créer un modèle** dans la fenêtre contextuelle nouvellement ouverte.
1. Sélectionnez le dossier `model` .
1. Nommez le modèle `model-diabetes`, puis sélectionnez **Enregistrer**.
1. Sélectionnez **Afficher le modèle** dans la notification qui s’affiche en haut à droite de votre écran lors de la création du modèle. Vous pouvez également actualiser la fenêtre. Le modèle enregistré est lié sous **Version modèle**.

Notez que le modèle, l’expérience et l’exécution de l’expérience sont liés, ce qui vous permet d’examiner la façon dont le modèle est entraîné.

## Enregistrer le notebook et mettre fin à la session Spark

Maintenant que vous avez terminé l’entraînement et l’évaluation des modèles, vous pouvez enregistrer le notebook avec un nom significatif et mettre fin à la session Spark.

1. Retournez à votre notebook et, dans la barre de menus du notebook, utilisez l’icône ⚙️ **Paramètres** pour afficher ses paramètres.
2. Définissez **Entraîner et comparer des modèles** comme **Nom** du notebook, puis fermez le volet des paramètres.
3. Dans le menu du notebook, sélectionnez **Arrêter la session** pour mettre fin à la session Spark.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un notebook et entraîné un modèle Machine Learning. Vous avez utilisé Scikit-Learn pour former le modèle et MLflow pour suivre ses performances.

Si vous avez terminé d’explorer votre modèle et vos expériences, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
