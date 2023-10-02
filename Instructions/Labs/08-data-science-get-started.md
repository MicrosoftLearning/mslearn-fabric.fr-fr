---
lab:
  title: Entraîner un modèle de classification pour prédire l’attrition clients
  module: Get started with data science in Microsoft Fabric
---

# Utiliser des notebooks pour entraîner un modèle dans Microsoft Fabric

Dans ce labo, nous allons utiliser Microsoft Fabric pour créer un notebook et entraîner un modèle Machine Learning afin de prédire l’attrition clients. Nous allons utiliser Scikit-Learn pour entraîner le modèle et MLflow pour suivre ses performances. L’attrition clients est un problème commercial critique auquel de nombreuses entreprises sont confrontées, et sa prédiction peut aider les entreprises à conserver leurs clients et à augmenter leur chiffre d’affaires. En suivant ce labo, vous allez acquérir une expérience pratique en Machine Learning et en suivi des modèles, et vous découvrirez comment utiliser Microsoft Fabric pour créer un notebook pour vos projets.

Ce labo prend environ **45** minutes.

> **Remarque** : Vous aurez besoin d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI](./Images/new-workspace.png)

## Créer un lakehouse et charger des fichiers

Maintenant que vous disposez d’un espace de travail, il est temps de basculer vers l’expérience *Science des données* dans le portail et de créer un data lakehouse pour les fichiers de données que vous allez analyser.

1. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI** et basculez vers l’expérience **Engineering données**.
1. Dans la page d’accueil de l’**Engineering données**, créez un **Lakehouse** avec le nom de votre choix.

    Au bout d’une minute environ, un nouveau lakehouse sans **tables** ou **fichiers** sera créé. Vous devez ingérer certaines données dans le data lakehouse à des fins d’analyse. Il existe plusieurs façons de procéder, mais dans cet exercice vous allez simplement télécharger et extraire un dossier de fichiers texte de votre ordinateur local (ou machine virtuelle de laboratoire le cas échéant), puis les charger dans votre lakehouse.

1. Téléchargez et enregistrez le fichier CSV `churn.csv` pour cet exercice à partir de [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv).


1. Revenez à l’onglet du navigateur web contenant votre lakehouse puis, dans le menu **…** du nœud **Fichiers** dans le volet **Vue du lac**, sélectionnez **Charger** et **Charger des fichiers**, puis chargez le fichier **churn.csv** à partir de votre ordinateur local (ou de la machine virtuelle de labo, le cas échéant) dans le lakehouse.
6. Une fois les fichiers chargés, développez **Fichiers** et vérifiez que le fichier CSV a été chargé.

## Créer un notebook

Pour entraîner un modèle, vous pouvez créer un *notebook*. Les notebooks fournissent un environnement interactif dans lequel vous pouvez écrire et exécuter du code (dans plusieurs langages) en tant qu’*expériences*.

1. En bas à gauche du portail Power BI, sélectionnez l’icône **Engineering données** et basculez vers l’expérience **Science des données**.

1. Dans la page d’accueil de **Science des données**, créez un **notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

1. Sélectionnez la première cellule (qui est actuellement une cellule de *code*) puis, dans la barre d’outils dynamique en haut à droite, utilisez le bouton **M&#8595;** pour convertir la cellule en cellule *Markdown*.

    Lorsque la cellule devient une cellule Markdown, le texte qu’elle contient est affiché.

1. Utilisez le bouton **&#128393;** (Modifier) pour basculer la cellule en mode édition, puis supprimez le contenu et entrez le texte suivant :

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ``` 

## Charger des données dans un DataFrame

Vous êtes maintenant prêt à exécuter du code pour préparer des données et entraîner un modèle. Pour travailler avec des données, vous allez utiliser des *DataFrames*. Les DataFrames dans Spark sont similaires aux DataFrames Pandas dans Python, et fournissent une structure commune pour l’utilisation de données dans des lignes et des colonnes.

1. Dans le volet **Ajouter un lakehouse**, sélectionnez **Ajouter** pour ajouter un lakehouse.
1. Sélectionnez **Lakehouse existant**, puis **Ajouter**.
1. Sélectionnez le lakehouse que vous avez créé dans une section précédente.
1. Développez le dossier **Fichiers** afin que le fichier CSV soit listé en regard de l’éditeur de notebook.
1. Dans le menu **...** pour **churn.csv**, sélectionnez **Charger des données** > **Pandas**. Une nouvelle cellule de code contenant le code suivant doit être ajoutée au notebook :

    ```python
   import pandas as pd
   # Load data into pandas DataFrame from "/lakehouse/default/" + "Files/churn.csv"
   df = pd.read_csv("/lakehouse/default/" + "Files/churn.csv")
   display(df)
    ```

    > **Conseil** : Vous pouvez masquer le volet contenant les fichiers à gauche en utilisant son icône **<<** . Cela vous aidera à vous concentrer sur le notebook.

1. Utilisez le bouton **&#9655; Exécuter la cellule** à gauche de la cellule pour l’exécuter.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans cette session, le pool Spark doit être démarré. Cela signifie que la première exécution dans la session peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

1. Une fois la commande de la cellule exécutée, examinez la sortie sous la cellule, qui doit être similaire à ceci :

    |Index|CustomerID|years_with_company|total_day_calls|total_eve_calls|total_night_calls|total_intl_calls|average_call_minutes|total_customer_service_calls|age|churn|
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    |1|1000038|0|117|88|32|607|43.90625678|0.810828179|34|0|
    |2|1000183|1|164|102|22|40|49.82223317|0.294453889|35|0|
    |3|1000326|3|116|43|45|207|29.83377967|1.344657937|57|1|
    |4|1000340|0|92|24|11|37|31.61998183|0.124931779|34|0|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    La sortie affiche les lignes et les colonnes des données client du fichier churn.csv.

## Entraîner un modèle Machine Learning

Maintenant que vous avez chargé les données, vous pouvez les utiliser pour entraîner un modèle Machine Learning et prédire l’attrition clients. Vous allez entraîner un modèle à l’aide de la bibliothèque Scikit-Learn et suivre le modèle avec MLflow. 

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant :

    ```python
   from sklearn.model_selection import train_test_split

   print("Splitting data...")
   X, y = df[['years_with_company','total_day_calls','total_eve_calls','total_night_calls','total_intl_calls','average_call_minutes','total_customer_service_calls','age']].values, df['churn'].values
   
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Exécutez la cellule de code que vous avez ajoutée ; notez que vous omettez « CustomerID » du jeu de données et que vous fractionnez les données en un jeu de données d’entraînement et de test.
1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :
    
    ```python
   import mlflow
   experiment_name = "experiment-churn"
   mlflow.set_experiment(experiment_name)
    ```
    
    Le code crée une expérience MLflow nommée `experiment-churn`. Vos modèles seront suivis dans cette expérience.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   from sklearn.linear_model import LogisticRegression
   
   with mlflow.start_run():
       mlflow.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)

       mlflow.log_param("estimator", "LogisticRegression")
    ```
    
    Le code entraîne un modèle de classification à l’aide de la régression logistique. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow. En outre, vous journaliserez un paramètre appelé `estimator`, avec la valeur `LogisticRegression`.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   from sklearn.tree import DecisionTreeClassifier
   
   with mlflow.start_run():
       mlflow.autolog()

       model = DecisionTreeClassifier().fit(X_train, y_train)
   
       mlflow.log_param("estimator", "DecisionTreeClassifier")
    ```

    Le code effectue l’entraînement d’un modèle de classification à l’aide du classifieur d’arbre de décision. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow. En outre, vous journaliserez un paramètre appelé `estimator`, avec la valeur `DecisionTreeClassifier`.

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
   experiment_name = "experiment-churn"
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
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_accuracy_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_accuracy_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("Accuracy")
   ax.set_title("Accuracy by Estimator")
   for i, v in enumerate(df_results["metrics.training_accuracy_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    La sortie doit ressembler à l’image suivante :

    ![Capture d’écran des métriques d’évaluation tracées.](./Images/plotted-metrics.png)

## Explorer vos expériences

Microsoft Fabric effectue le suivi de toutes vos expériences et vous permet de les explorer visuellement.

1. Dans le volet gauche, accédez à votre espace de travail.
1. Sélectionnez l’expérience `experiment-churn` dans la liste.

    > **Conseil :** Si vous ne voyez aucune exécution d’expérience journalisée, actualisez la page.

1. Sélectionnez l’onglet **Affichage**.
1. Sélectionnez **Liste d’exécutions**. 
1. Sélectionnez les deux dernières exécutions en cochant chaque case.
    Vos deux dernières exécutions seront comparées l’une à l’autre dans le volet **Comparaison des métriques**. Par défaut, les métriques sont tracées en fonction du nom d’exécution. 
1. Sélectionnez le bouton **&#128393;** (Modifier) du graphe affichant la précision de chaque exécution. 
1. Sélectionnez `bar` comme **type de visualisation**. 
1. Sélectionnez `estimator` comme **axe x**. 
1. Sélectionnez **Remplacer** et explorez le nouveau graphe.

En traçant la précision par estimateur journalisé, vous pouvez examiner l’algorithme qui a abouti à un meilleur modèle.

## Enregistrer le modèle

Après avoir comparé les modèles Machine Learning que vous avez entraînés pour les différentes exécutions d’expériences, vous pouvez choisir le modèle le plus performant. Pour utiliser le modèle le plus performant, enregistrez le modèle et utilisez-le afin de générer des prédictions.

1. Dans la vue d’ensemble de l’expérience, vérifiez que l’onglet **Affichage** est sélectionné.
1. Sélectionnez **Détails de l’exécution**.
1. Sélectionnez l’exécution avec la précision la plus élevée. 
1. Sélectionnez **Enregistrer** dans la zone **Enregistrer en tant que modèle**.
1. Sélectionnez **Créer un modèle** dans la fenêtre contextuelle nouvellement ouverte.
1. Nommez le modèle `model-churn`, puis sélectionnez **Créer**. 
1. Sélectionnez **Afficher le modèle** dans la notification qui s’affiche en haut à droite de votre écran lors de la création du modèle. Vous pouvez également actualiser la fenêtre. Le modèle enregistré est lié sous **Version inscrite**. 

Notez que le modèle, l’expérience et l’exécution de l’expérience sont liés, ce qui vous permet d’examiner la façon dont le modèle est entraîné. 

## Enregistrer le notebook et mettre fin à la session Spark

Maintenant que vous avez terminé l’entraînement et l’évaluation des modèles, vous pouvez enregistrer le notebook avec un nom significatif et mettre fin à la session Spark.

1. Dans la barre de menus du notebook, utilisez l’icône ⚙️ **Paramètres** pour afficher les paramètres du notebook.
2. Définissez **Entraîner et comparer des modèles** comme **Nom** du notebook, puis fermez le volet des paramètres.
3. Dans le menu du notebook, sélectionnez **Arrêter la session** pour mettre fin à la session Spark.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un notebook et entraîné un modèle Machine Learning. Vous avez utilisé Scikit-Learn pour entraîner le modèle et MLflow pour suivre ses performances.

Si vous avez terminé d’explorer votre modèle et vos expériences, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
