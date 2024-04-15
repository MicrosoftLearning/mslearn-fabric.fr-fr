---
lab:
  title: "Bien démarrer avec la science des données dans Microsoft\_Fabric"
  module: Get started with data science in Microsoft Fabric
---

# Bien démarrer avec la science des données dans Microsoft Fabric

Dans ce labo, vous allez ingérer des données, explorer les données d’un notebook, traiter les données avec Data Wrangler et entraîner deux types de modèles. En effectuant toutes ces étapes, vous serez en mesure d’explorer les fonctionnalités de science des données dans Microsoft Fabric.

En suivant ce labo, vous allez acquérir une expérience pratique du Machine Learning et du suivi des modèles, et apprendre à utiliser des *notebooks*, *Data Wrangler*, *des expériences*et des *modèles* dans Microsoft Fabric.

Ce labo prend environ **20** minutes.

> **Remarque** : Vous devez disposer d’un [essai gratuit Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Accédez à la page d’accueil de Microsoft Fabric à [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com) dans un navigateur.
1. **Science des données Synapse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un notebook

Pour exécuter du code, vous pouvez créer un *notebook*. Les notebooks fournissent un environnement interactif dans lequel vous pouvez écrire et exécuter du code (dans plusieurs langues).

1. Sur la page d’accueil de **Synapse Science des données**, créez un **Notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

1. Sélectionnez la première cellule (qui est actuellement une cellule de *code*) puis, dans la barre d’outils dynamique en haut à droite, utilisez le bouton **M&#8595;** pour convertir la cellule en cellule *Markdown*.

    Lorsque la cellule devient une cellule Markdown, le texte qu’elle contient est affiché.

1. Utilisez le bouton **&#128393;** (Modifier) pour basculer la cellule en mode édition, puis supprimez le contenu et entrez le texte suivant :

    ```text
   # Data science in Microsoft Fabric
    ```

## Obtenir les données

Vous êtes maintenant prêt à exécuter du code pour obtenir des données et former un modèle. Vous allez utiliser le [jeu de données diabetes](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) à partir d’Azure Open Datasets. Après avoir chargé les données, vous allez convertir les données en dataframe Pandas, qui est une structure courante pour l’utilisation des données dans les lignes et les colonnes.

1. Dans votre notebook, utilisez l’icône **+ Code** sous la dernière sortie de cellule pour ajouter une nouvelle cellule de code au notebook.

    > **Conseil** : Pour afficher l’icône **+ Code**, déplacez la souris juste en dessous et à gauche de la sortie de la cellule active. Sinon, dans la barre de menus, sous l’onglet **Modifier**, sélectionnez **+ Ajouter une cellule de code**.

1. Entrez le code suivant dans une nouvelle cellule de code :

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

1. Il existe deux onglets en haut du tableau affiché : **Tableau** et **Graphique**. Sélectionnez un **graphique**.
1. Sélectionnez les **options Afficher** en haut à droite du graphique pour modifier la visualisation.
1. Remplacez le graphique par les paramètres suivants :
    * **Type de graphique** : `Box plot`
    * **Clé** : *laissez ce champ vide*
    * **Valeurs** : `Y`
1. Sélectionnez **Appliquer** pour afficher la nouvelle visualisation et explorer la sortie.

## Préparer les données

Maintenant que vous avez ingéré et exploré les données, vous pouvez les transformer. Vous pouvez exécuter du code dans un notebook ou utiliser Data Wrangler pour générer du code pour vous.

1. Les données sont chargées en tant que trame de données Spark. Pour lancer data Wrangler, vous devez convertir les données en dataframe Pandas. Exécutez la cellule suivante dans votre notebook :

    ```python
   df = df.toPandas()
   df.head()
    ```

1. Sélectionnez **Données** dans le ruban du notebook, puis la liste déroulante **Transformer DataFrame dans Data Wrangler**.
1. Sélectionnez le jeu de données `df`. Lorsque Data Wrangler est lancé, il génère une vue d’ensemble descriptive du dataframe dans le panneau **Résumé**.

    Actuellement, la colonne d’étiquette est `Y`, qui est une variable continue. Pour entraîner un modèle Machine Learning qui prédit Y, vous devez entraîner un modèle de régression. Les valeurs (prédites) de Y peuvent être difficiles à interpréter. Au lieu de cela, nous pourrions explorer la formation d’un modèle de classification qui prédit si quelqu’un est à faible risque ou à risque élevé de développer le diabète. Pour pouvoir entraîner un modèle de classification, vous devez créer une colonne d’étiquette binaire basée sur les valeurs de `Y`.

1. Sélectionnez la colonne `Y` dans Data Wrangler. Notez qu’il y a une diminution de la fréquence pour le bin `220-240`. Le 75e centile `211.5` s’aligne approximativement sur la transition des deux régions dans l’histogramme. Utilisons cette valeur comme seuil pour les risques faibles et élevés.
1. Accédez au panneau **Opérations**, développez **Formules**, puis sélectionnez **Créer une colonne à partir de la formule**.
1. Créez une colonne avec les paramètres suivants :
    * **Nom de la colonne** : `Risk`
    * **Formule de colonne** : `(df['Y'] > 211.5).astype(int)`
1. Passez en revue la nouvelle colonne `Risk` qui est ajoutée à l’aperçu. Vérifiez que le nombre de lignes avec une valeur `1` doit être d’environ 25 % de toutes les lignes (car il s’agit du 75e centile de `Y`).
1. Sélectionnez **Appliquer**.
1. Sélectionnez **Ajouter du code au notebook**.
1. Exécutez la cellule avec le code généré par Data Wrangler.
1. Exécutez le code suivant dans une nouvelle cellule pour vérifier que la colonne `Risk` est mise en forme comme prévu :

    ```python
   df_clean.describe()
    ```

## Effectuer l’apprentissage de modèles Machine Learning

Maintenant que vous avez chargé les données, vous pouvez les utiliser pour entraîner un modèle Machine Learning et prédire le diabète. Nous pouvons entraîner deux types de modèles différents avec notre jeu de données : un modèle de régression (prédiction `Y`) ou un modèle de classification (prédiction `Risk`). Vous allez entraîner un modèle à l’aide de la bibliothèque Scikit-Learn et suivre le modèle avec MLflow.

### Entraîner un modèle de régression

1. Exécutez le code suivant pour fractionner les données en un jeu de données d’entraînement et de test, et pour séparer les fonctionnalités de l’étiquette `Y` que vous souhaitez prédire :

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   import mlflow
   experiment_name = "diabetes-regression"
   mlflow.set_experiment(experiment_name)
    ```

    Le code crée une expérience MLflow nommée `diabetes-regression`. Vos modèles seront suivis dans cette expérience.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    ```

    Le code entraîne un modèle de régression à l’aide de la régression linéaire. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow.

### Entraîner un modèle de classification

1. Exécutez le code suivant pour fractionner les données en un jeu de données d’entraînement et de test, et pour séparer les fonctionnalités de l’étiquette `Risk` que vous souhaitez prédire :

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Risk'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   import mlflow
   experiment_name = "diabetes-classification"
   mlflow.set_experiment(experiment_name)
    ```

    Le code crée une expérience MLflow nommée `diabetes-classification`. Vos modèles seront suivis dans cette expérience.

1. Ajoutez une nouvelle cellule de code au notebook, entrez le code suivant, puis exécutez-le :

    ```python
   from sklearn.linear_model import LogisticRegression
    
   with mlflow.start_run():
       mlflow.sklearn.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)
    ```

    Le code entraîne un modèle de classification à l’aide de la régression logistique. Les paramètres, les métriques et les artefacts sont automatiquement enregistrés avec MLflow.

## Explorer vos expériences

Microsoft Fabric effectue le suivi de toutes vos expériences et vous permet de les explorer visuellement.

1. Accédez à votre espace de travail à partir de la barre de menu du hub sur la gauche.
1. Sélectionnez l’expérience `diabetes-regression` pour l’ouvrir.

    > **Conseil :** Si vous ne voyez aucune exécution d’expérience journalisée, actualisez la page.

1. Passez en revue les **métriques d’exécution** pour explorer la précision de votre modèle de régression.
1. Revenez à la page d’accueil et sélectionnez l’expérience `diabetes-classification` pour l’ouvrir.
1. Passez en revue les **métriques d’exécution** pour explorer la précision du modèle de classification. Notez que le type de métriques est différent à mesure que vous avez entraîné un autre type de modèle.

## Enregistrer le modèle

Après avoir comparé les modèles Machine Learning que vous avez entraînés pour les différentes exécutions d’expériences, vous pouvez choisir le modèle le plus performant. Pour utiliser le modèle le plus performant, enregistrez le modèle et utilisez-le afin de générer des prédictions.

1. Sélectionnez **Enregistrer en tant que modèle ML** dans le ruban d’expérience.
1. Sélectionnez **Créer un modèle ML** dans la fenêtre contextuelle nouvellement ouverte.
1. Sélectionnez le dossier `model`.
1. Nommez le modèle `model-diabetes`, puis sélectionnez **Enregistrer**.
1. Sélectionnez **Afficher le modèle ML** dans la notification qui s’affiche en haut à droite de votre écran lors de la création du modèle. Vous pouvez également actualiser la fenêtre. Le modèle enregistré est lié sous **Versions de modèle ML**.

Notez que le modèle, l’expérience et l’exécution de l’expérience sont liés, ce qui vous permet d’examiner la façon dont le modèle est entraîné.

## Enregistrer le notebook et mettre fin à la session Spark

Maintenant que vous avez terminé l’entraînement et l’évaluation des modèles, vous pouvez enregistrer le notebook avec un nom significatif et mettre fin à la session Spark.

1. Dans la barre de menus du notebook, utilisez l’icône ⚙️ **Paramètres** pour afficher les paramètres du notebook.
2. Définissez **Entraîner et comparer des modèles** comme **Nom** du notebook, puis fermez le volet des paramètres.
3. Dans le menu du notebook, sélectionnez **Arrêter la session** pour mettre fin à la session Spark.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un notebook et entraîné un modèle Machine Learning. Vous avez utilisé Scikit-Learn pour former le modèle et MLflow pour suivre ses performances.

Si vous avez terminé d’explorer votre modèle et vos expériences, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans la page de l’espace de travail, sélectionnez **Paramètres de l’espace de travail**.
3. En bas de la section **Général**, sélectionnez **Supprimer cet espace de travail**.
