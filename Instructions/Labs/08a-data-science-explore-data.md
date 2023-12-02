---
lab:
  title: Créer et utiliser des notebooks pour l’exploration des données
  module: Explore data for data science with notebooks in Microsoft Fabric
---

# Utiliser des notebooks pour explorer les données dans Microsoft Fabric

Dans ce labo, nous allons utiliser des notebooks pour l’exploration des données. Les notebooks sont un outil puissant pour l’exploration et l’analyse interactives des données. Au cours de cet exercice, nous allons apprendre à créer et à utiliser des notebooks pour explorer un jeu de données, générer des statistiques récapitulatives et créer des visualisations pour mieux comprendre les données. À la fin de ce labo, vous aurez une solide compréhension de l’utilisation des notebooks pour l’exploration et l’analyse des données.

Ce labo prend environ **30** minutes.

> **Remarque**: Vous avez besoin d’un compte *scolaire* ou *professionnel* Microsoft pour réaliser cet exercice. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Activer une version d’évaluation de Microsoft Fabric

1. Après avoir ouvert un compte Microsoft Fabric, accédez au portail Microsoft Fabric à l’adresse [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com).
1. Sélectionnez l’icône **Gestionnaire de comptes** (l’image de l’*utilisateur* en haut à droite)
1. Dans le menu du gestionnaire de comptes, sélectionnez **Démarrer la version d’évaluation** pour démarrer un essai de Microsoft Fabric.
1. Après avoir effectué la mise à niveau vers Microsoft Fabric, accédez à la page d’accueil en sélectionnant **Page d’accueil de l’infrastructure**.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Science des données**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un notebook

Pour entraîner un modèle, vous pouvez créer un *notebook*. Les notebooks fournissent un environnement interactif dans lequel vous pouvez écrire et exécuter du code (dans plusieurs langages) en tant qu’*expériences*.

1. Sur la page d’accueil de **Synapse Science des données**, créez un **Notebook**.

    Après quelques secondes, un nouveau notebook contenant une seule *cellule* s’ouvre. Les notebooks sont constitués d’une ou plusieurs cellules qui peuvent contenir du *code* ou du *Markdown* (texte mis en forme).

1. Sélectionnez la première cellule (qui est actuellement une cellule de *code*) puis, dans la barre d’outils dynamique en haut à droite, utilisez le bouton **M&#8595;** pour convertir la cellule en cellule *Markdown*.

    Lorsque la cellule devient une cellule Markdown, le texte qu’elle contient est affiché.

1. Utilisez le bouton **&#128393;** (Modifier) pour basculer la cellule en mode édition, puis supprimez le contenu et entrez le texte suivant :

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ``` 

## Charger des données dans un DataFrame

Vous êtes maintenant prêt à exécuter du code pour obtenir des données. Vous allez utiliser le [**jeu de données diabetes**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) à partir d’Azure Open Datasets. Après avoir chargé les données, vous allez convertir les données en dataframe Pandas, qui est une structure courante pour l’utilisation des données dans les lignes et les colonnes.

1. Dans votre bloc-notes, utilisez l’icône **+ Code** sous la dernière cellule pour ajouter une nouvelle cellule de code au bloc-notes. Entrez le code suivant pour charger le jeu de données dans un dataframe.

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
    df = df.toPandas()
    df.head()
    ```

## Vérifier la forme des données

Maintenant que vous avez chargé les données, vous pouvez case activée la structure du jeu de données, comme le nombre de lignes et de colonnes, les types de données et les valeurs manquantes.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant :

    ```python
    # Display the number of rows and columns in the dataset
    print("Number of rows:", df.shape[0])
    print("Number of columns:", df.shape[1])

    # Display the data types of each column
    print("\nData types of columns:")
    print(df.dtypes)
    ```

    Le jeu de données contient **442 lignes** et **11 colonnes**. Cela signifie que vous avez 442 exemples et 11 fonctionnalités ou variables dans votre jeu de données. La variable `SEX` contient probablement des données catégorielles ou de chaînes.

## Vérifier les données manquantes

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant :

    ```python
    missing_values = df.isnull().sum()
    print("\nMissing values per column:")
    print(missing_values)
    ```

    Le code recherche les valeurs manquantes. Notez qu’il n’y a pas de données manquantes dans le jeu de données.

## Générer des statistiques descriptives pour les variables numériques

Maintenant, nous allons générer des statistiques descriptives pour comprendre la distribution des variables numériques.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant.

    ```python
    df.describe()
    ```

    L’âge moyen est d’environ 48,5 ans, avec un écart type de 13,1 ans. La personne la plus jeune a 19 ans et la plus âgée a 79 ans. La moyenne `BMI` est d’environ 26,4, ce qui se situe dans la catégorie de **surpoids** selon les [normes de l’OMS](https://www.who.int/health-topics/obesity#tab=tab_1). Le minimum `BMI` est de 18 et le maximum est de 42,2.

## Tracer la distribution des données

Nous allons vérifier la `BMI` fonctionnalité et tracer sa distribution pour mieux comprendre ses caractéristiques.

1. Ajoutez une autre cellule de code au notebook. Ensuite, entrez le code suivant dans cette cellule et exécutez-le.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    import numpy as np
    
    # Calculate the mean, median of the BMI variable
    mean = df['BMI'].mean()
    median = df['BMI'].median()
    
    # Histogram of the BMI variable
    plt.figure(figsize=(8, 6))
    plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
    plt.title('BMI Distribution')
    plt.xlabel('BMI')
    plt.ylabel('Frequency')
    
    # Add lines for the mean and median
    plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
    plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
    # Add a legend
    plt.legend()
    plt.show()
    ```

    À partir de ce graphique, vous pouvez observer la plage et la distribution de dans le jeu de `BMI` données. Par exemple, la plupart des `BMI` données sont comprises entre 23.2 et 29.2, et les données sont correctement asymétriques.

## Effectuer une analyse multivariée

Nous allons générer des visualisations telles que des nuages de points et des tracés de boîtes pour découvrir les modèles et les relations au sein des données.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns

    # Scatter plot of Quantity vs. Price
    plt.figure(figsize=(8, 6))
    sns.scatterplot(x='BMI', y='Y', data=df)
    plt.title('BMI vs. Target variable')
    plt.xlabel('BMI')
    plt.ylabel('Target')
    plt.show()
    ```
    
    Nous pouvons voir qu’à mesure que la variable `BMI` augmente, la variable cible augmente également, ce qui indique une relation linéaire positive entre ces deux variables.

1. Ajoutez une autre cellule de code au notebook. Ensuite, entrez le code suivant dans cette cellule et exécutez-le.

    ```python
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    fig, ax = plt.subplots(figsize=(7, 5))
    
    # Replace numeric values with labels
    df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
    sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
    ax.set_title('Blood pressure across Gender')
    plt.tight_layout()
    plt.show()
    ```

    Ces observations suggèrent qu’il existe des différences dans les profils de tension artérielle chez les patients masculins et féminins. En moyenne, les femmes ont une pression artérielle plus élevée que les patients masculins.

1. L’agrégation des données peut les rendre plus faciles à gérer pour la visualisation et l’analyse. Ajoutez une autre cellule de code au notebook. Ensuite, entrez le code suivant dans cette cellule et exécutez-le.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # Calculate average BP and BMI by SEX
    avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
    # Bar chart of the average BP and BMI by SEX
    ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
    # Add title and labels
    plt.title('Avg. Blood Pressure and BMI by Gender')
    plt.xlabel('Gender')
    plt.ylabel('Average')
    
    # Display actual numbers on the bar chart
    for p in ax.patches:
        ax.annotate(format(p.get_height(), '.2f'), 
                    (p.get_x() + p.get_width() / 2., p.get_height()), 
                    ha = 'center', va = 'center', 
                    xytext = (0, 10), 
                    textcoords = 'offset points')
    
    plt.show()
    ```

    Ce graphique montre que la pression artérielle moyenne est plus élevée chez les femmes que chez les patients masculins. En outre, il montre que l’indice de masse corporelle (IMC) moyen est légèrement plus élevé chez les femmes que chez les hommes.

1. Ajoutez une autre cellule de code au notebook. Ensuite, entrez le code suivant dans cette cellule et exécutez-le.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    plt.figure(figsize=(10, 6))
    sns.lineplot(x='AGE', y='BMI', data=df, ci=None)
    plt.title('BMI over Age')
    plt.xlabel('Age')
    plt.ylabel('BMI')
    plt.show()
    ```

    Le groupe d’âge de 19 à 30 ans a les valeurs moyennes d’IMC les plus faibles, tandis que l’IMC moyen le plus élevé se trouve dans le groupe d’âge de 65 à 79 ans. De plus, observez que l’IMC moyen pour la plupart des groupes d’âge se situe dans la plage de surpoids.

## Analyse des corrélations

Calculons les corrélations entre différentes fonctionnalités pour comprendre leurs relations et leurs dépendances.

1. Utilisez l’icône **+ Code** sous la sortie de cellule pour ajouter une nouvelle cellule de code au notebook, puis entrez le code suivant.

    ```python
    df.corr(numeric_only=True)
    ```

1. Une carte thermique est un outil utile pour visualiser rapidement la force et la direction des relations entre les paires de variables. Il peut mettre en évidence des corrélations positives ou négatives fortes et identifier des paires qui n’ont aucune corrélation. Pour créer une carte thermique, ajoutez une autre cellule de code au notebook, puis entrez le code suivant.

    ```python
    plt.figure(figsize=(15, 7))
    sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    les variables `S1` et `S2` ont une corrélation positive élevée de **0,89**, ce qui indique qu’elles évoluent dans la même direction. En cas d’augmentation`S1`, `S2` a également tendance à augmenter, et vice versa. En outre, `S3` et `S4` ont une forte corrélation négative de **-0,73**. Cela signifie qu’à mesure qu’augmente `S3` , `S4` tend à diminuer.

## Enregistrer le notebook et mettre fin à la session Spark

Maintenant que vous avez fini d'explorer les données, vous pouvez enregistrer le carnet de notes avec un nom significatif et terminer la session Spark.

1. Dans la barre de menus du notebook, utilisez l’icône ⚙️ **Paramètres** pour afficher les paramètres du notebook.
2. Définissez le **Nom** du notebook sur **Explorer les commandes client**, puis fermez le volet des paramètres.
3. Dans le menu du notebook, sélectionnez **Arrêter la session** pour mettre fin à la session Spark.

## Nettoyer les ressources

Dans cet exercice, vous avez créé et utilisé des notebooks pour l’exploration des données. Vous avez également exécuté du code pour calculer des statistiques récapitulatives et créer des visualisations pour mieux comprendre les modèles et les relations dans les données.

Si vous avez terminé d’explorer votre modèle et vos expériences, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
