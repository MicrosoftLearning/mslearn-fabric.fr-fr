---
lab:
  title: Prétraiter des données avec Data Wrangler dans Microsoft Fabric
  module: Preprocess data with Data Wrangler in Microsoft Fabric
---

# Utiliser des notebooks pour entraîner un modèle dans Microsoft Fabric

Dans ce labo, vous allez apprendre à utiliser Data Wrangler dans Microsoft Fabric pour prétraiter des données, et générer du code à l’aide d’une bibliothèque d’opérations courantes de science des données.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

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

1. TODO : Téléchargez et enregistrez le fichier CSV `dominicks_OJ.csv` pour cet exercice à partir de [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/XXXXX.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/XXXXX.csv).


1. Revenez à l’onglet du navigateur web contenant votre lakehouse puis, dans le menu **…** du nœud **Fichiers** dans le volet **Vue du lac**, sélectionnez **Charger** et **Charger des fichiers**, puis chargez le fichier **dominicks_OJ.csv** à partir de votre ordinateur local (ou de la machine virtuelle de labo, le cas échéant) dans le lakehouse.
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
    df = pd.read_csv("/lakehouse/default/" + "Files/dominicks_OJ.csv") 
    display(df.head(5))
    ```

    > **Conseil** : Vous pouvez masquer le volet contenant les fichiers à gauche en utilisant son icône **<<** . Cela vous aidera à vous concentrer sur le notebook.

1. Utilisez le bouton **&#9655; Exécuter la cellule** à gauche de la cellule pour l’exécuter.

    > **Remarque** : Comme c’est la première fois que vous exécutez du code Spark dans cette session, le pool Spark doit être démarré. Cela signifie que la première exécution dans la session peut prendre environ une minute. Les exécutions suivantes seront plus rapides.

## Afficher des statistiques récapitulatives

Lorsque Data Wrangler est lancé, il génère une vue d’ensemble descriptive du dataframe dans le panneau Résumé. 

1. Sélectionnez **Données** dans le menu supérieur, puis la liste déroulante **Data Wrangler** pour parcourir le jeu de données `df`.

    ![Capture d’écran de l’option de lancement de Data Wrangler.](./Images/launch-data-wrangler.png)

1. Sélectionnez la colonne **Large HH** et observez combien il est facile de déterminer la distribution des données de cette caractéristique.

    ![Capture d’écran de la page de Data Wrangler montrant la distribution des données pour une colonne particulière.](./Images/data-wrangler-distribution.png)

    Notez que cette caractéristique suit une distribution normale.

1. Vérifiez le panneau latéral Résumé, et observez les plages de centiles. 

    ![Capture d’écran de la page de Data Wrangler montrant les détails du panneau Résumé.](./Images/data-wrangler-summary.png)

    Vous pouvez constater que la plupart des données se situent entre **0,098** et **0,132**, et que 50 % des valeurs de données se trouvent dans cette plage.

## Mettre en forme des données de texte

Appliquons maintenant quelques transformations à la caractéristique **Brand**.

1. Dans la page **Data Wrangler**, sélectionnez la caractéristique `Brand`.

1. Accédez au panneau **Opérations**, développez **Rechercher et remplacer**, puis sélectionnez **Rechercher et remplacer**.

1. Dans le panneau **Rechercher et remplacer**, modifiez les propriétés suivantes :
    
    - **Ancienne valeur :** "."
    - **Nouvelle valeur :** " " (caractère d’espace)

    ![Capture d’écran de la page de Data Wrangler montrant le panneau Rechercher et remplacer.](./Images/data-wrangler-find.png)

    Les résultats de l’opération sont affichés automatiquement dans la grille d’affichage.

1. Sélectionnez **Appliquer**.

1. Revenez au panneau **Opérations** et développez **Format**.

1. Sélectionnez **Convertir du texte en majuscule**.

1. Dans le panneau **Convertir du texte en majuscules**, sélectionnez **Appliquer**.

1. Sélectionnez **Ajouter du code au notebook**. En outre, vous pouvez également enregistrer le jeu de données transformé en tant que fichier .csv.

    Notez que le code est automatiquement copié dans la cellule du notebook, et qu’il est prêt à être utilisé.

1. Exécutez le code.

> **Important :** Le code généré ne remplace pas le dataframe d’origine. 

Vous avez appris à générer facilement du code et à manipuler des données de texte à l’aide d’opérations Data Wrangler. 

## Appliquer une transformation d’encodeur one-hot

Maintenant, nous allons générer le code pour appliquer une transformation d’encodeur one-hot en tant qu’étape de prétraitement.

1. Sélectionnez **Données** dans le menu supérieur, puis la liste déroulante **Data Wrangler** pour parcourir le jeu de données `df`.

1. Dans le panneau **Opérations**, développez **Formules**.

1. Sélectionnez **Un encodeur à chaud**.

1. Dans le panneau **Un encodeur à chaud**, sélectionnez **Appliquer**.

    Accédez à la fin de la grille d’affichage Data Wrangler. Notez que trois nouvelles caractéristiques ont été ajoutées, et que la caractéristique `Brand` a été supprimée.

1. Sélectionnez **Ajouter du code au notebook**.

1. Exécutez le code.

## Opérations de tri et de filtrage

1. Sélectionnez **Données** dans le menu supérieur, puis la liste déroulante **Data Wrangler** pour parcourir le jeu de données `df`.

1. Dans le panneau **Opérations**, développez **Trier et filtrer**.

1. Sélectionnez **Filtrer**.

1. Dans le panneau **Filtre**, ajoutez la condition suivante :
    
    - **Colonne cible :** Store
    - **Opération :** Égal à
    - **Valeur :** 2

1. Sélectionnez **Appliquer**.

    Observez les modifications apportées à la grille d’affichage de Data Wrangler.

1. De retour au panneau **Opérations**, développez **Trier et filtrer**.

1. Sélectionner **Trier les valeurs**.

1. Dans le panneau **Price**, ajoutez la condition suivante :
    
    - **Nom de la colonne :** Price
    - **Ordre de tri :** Décroissant

1. Sélectionnez **Appliquer**.

    Observez les modifications apportées à la grille d’affichage de Data Wrangler.

## Agréger les données

1. De retour au panneau **Opérations**, sélectionnez **Regrouper et agréger**.

1. Dans la propriété **Colonnes de regroupement :** , sélectionnez la caractéristique `Store`.

1. Sélectionnez **Ajouter une agrégation**.

1. Dans la propriété **Colonne à agréger**, sélectionnez la caractéristique `Quantity`.

1. Sélectionnez **Nombre** pour la propriété **Type d’agrégation**.

1. Sélectionnez **Appliquer**. 

    Observez les modifications apportées à la grille d’affichage de Data Wrangler.

## Parcourir et supprimer les étapes

Supposez que vous avez commis une erreur et que vous devez supprimer l’agrégation créée à l’étape précédente. Suivez ces étapes pour la supprimer :

1. Développez le panneau **Étapes de nettoyage**.

1. Sélectionnez l’étape **Regrouper et agréger**.

1. Sélectionnez l’icône de suppression pour supprimer l’étape.

    ![Capture d’écran de la page de Data Wrangler montrant le panneau Rechercher et remplacer.](./Images/data-wrangler-delete.png)

    > **Important :** L’affichage de grille et le résumé sont limités à l’étape actuelle.

    Notez que les modifications sont rétablies à l’étape précédente, qui est l’étape **Trier les valeurs**.

1. Sélectionnez **Ajouter du code au notebook**.

1. Exécutez le code.

Vous avez généré le code pour certaines des opérations de prétraitement, et vous l’avez enregistré dans le notebook en tant que fonction, que vous pouvez ensuite réutiliser ou modifier en fonction des besoins.

## Enregistrer le notebook et mettre fin à la session Spark

Maintenant que vous avez terminé d’utiliser les données, vous pouvez enregistrer le notebook avec un nom explicite et mettre fin à la session Spark.

1. Dans la barre de menus du notebook, utilisez l’icône ⚙️ **Paramètres** pour afficher les paramètres du notebook.
2. Définissez **Prétraiter des données avec Data Wrangler** comme **Nom** du notebook, puis fermez le volet des paramètres.
3. Dans le menu du notebook, sélectionnez **Arrêter la session** pour mettre fin à la session Spark.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un notebook et utilisé Data Wrangler pour explorer et prétraiter des données pour un modèle Machine Learning.

Si vous avez terminé d’explorer les étapes de prétraitement, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
