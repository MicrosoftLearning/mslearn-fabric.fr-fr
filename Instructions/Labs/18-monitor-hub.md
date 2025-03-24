---
lab:
  title: Surveiller l’activité Fabric dans le hub de surveillance
  module: Monitoring Fabric
---

# Surveiller l’activité Fabric dans le hub de surveillance

Le *hub de surveillance* dans Microsoft Fabric fournit un emplacement central où vous pouvez surveiller l’activité. Vous pouvez utiliser le hub de surveillance pour passer en revue les événements liés aux éléments que vous avez l’autorisation d’afficher.

Ce labo prend environ **30** minutes.

> **Remarque** : pour terminer cet exercice, vous avez besoin d’un [locataire Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail dans un locataire avec la fonctionnalité Fabric activée.

1. Accédez à la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric` dans un navigateur et connectez-vous avec vos informations d’identification Fabric.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence dans la section **Avancé** qui comprend la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un lakehouse

Maintenant que vous disposez d’un espace de travail, il est temps de créer un data lakehouse pour vos données.

1. Sélectionnez **Créer** dans la barre de menus de gauche. Dans la page *Nouveau*, sous la section *Engineering données*, sélectionnez **Lakehouse**. Donnez-lui un nom unique de votre choix.

    >**Note** : si l’option **Créer** n’est pas épinglée à la barre latérale, vous devez d’abord sélectionner l’option avec des points de suspension (**...**).

    Au bout d’une minute environ, un nouveau lakehouse est créé :

    ![Capture d’écran d’un nouveau lakehouse.](./Images/new-lakehouse.png)

1. Affichez le nouveau lakehouse et notez que le volet **Explorateur de lakehouse** à gauche vous permet de parcourir les tables et les fichiers présents dans le lakehouse :

    Actuellement, il n’y a pas de tables ou de fichiers dans le lakehouse.

## Créer et surveiller un flux de données

Dans Microsoft Fabric, vous pouvez utiliser un flux de données (Gen2) pour ingérer des données à partir d’un large éventail de sources. Dans cet exercice, vous allez utiliser un flux de données pour obtenir des données à partir d’un fichier CSV et les charger dans une table de votre lakehouse.

1. Dans la **page d’accueil** de votre lakehouse, dans le menu **Obtenir des données**, sélectionnez **Nouveau Dataflow Gen2**.

   Un nouveau flux de donnée nommé **Dataflow 1** est créé et ouvert.

    ![Capture d’écran d’un nouveau flux de données.](./Images/new-data-flow.png)

1. En haut à gauche de la page de flux de données, sélectionnez **Dataflow 1** pour afficher ses détails et renommer le flux de données en **Obtenir des données produit**.
1. Dans le concepteur de flux de données, sélectionnez **Importer à partir d’un fichier Texte/CSV**. Exécutez ensuite l’Assistant Obtenir des données pour créer une connexion de données en vous liant à `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv` à l’aide de l’authentification anonyme. Après avoir terminé de suivre l’assistant, un aperçu des données s’affiche dans le concepteur de flux de données comme suit :

    ![Capture d’écran d’une requête de flux de données.](./Images/data-flow-query.png)

1. Publiez le flux de données.
1. Dans la barre de navigation de gauche, sélectionnez **Surveiller** pour afficher le hub de surveillance et observez que votre flux de données est en cours (si ce n’est pas le cas, actualisez l’affichage jusqu’à ce que vous le voyiez).

    ![Capture d’écran du hub de surveillance avec un flux de données en cours.](./Images/monitor-dataflow.png)

1. Attendez quelques secondes, puis actualisez la page jusqu’à ce que l’état du flux de données soit **Réussi**.
1. Dans le volet de navigation, sélectionnez votre lakehouse. Développez ensuite le dossier **Tables** pour vérifier qu’une table nommée **Produits** a été créée et chargée par le flux de données (vous devrez peut-être actualiser le dossier **Tables**).

    ![Capture d’écran de la table Produits dans la page du lakehouse.](./Images/products-table.png)

## Créer et surveiller un notebook Spark

Dans Microsoft Fabric, vous pouvez utiliser des notebooks pour exécuter du code Spark.

1. Dans le volet de navigation, sélectionnez **Accueil**. Dans la page d’accueil Ingénieurs de données, créez un **Notebook**.

    Un notebook nommé **Notebook 1** est créé et ouvert.

    ![Capture d’écran d’un nouveau notebook.](./Images/new-notebook.png)

1. En haut à gauche du notebook, sélectionnez **Notebook 1** pour afficher ses détails et remplacez son nom par **Demander des produits**.
1. Dans l’éditeur de notebook, dans le volet **Explorateur**, sélectionnez **Lakehouses** et ajoutez le lakehouse que vous avez créé précédemment.
1. Dans le menu **...** de la table **Produits**, sélectionnez **Charger des données** > **Spark**. Cela ajoute une cellule de code au notebook, comme illustré ici :

    ![Capture d’écran d’un notebook avec du code pour interroger une table.](./Images/load-spark.png)

1. Utilisez le bouton **&#9655; Exécuter tout** pour exécuter toutes les cellules du notebook. Le démarrage de la session Spark prend un certain temps, puis les résultats de la requête s’affichent sous la cellule de code.

    ![Capture d’écran d’un notebook avec des résultats de la requête.](./Images/notebook-output.png)

1. Dans la barre d’outils, utilisez le bouton **&#9723;** (*Arrêter la session*) pour arrêter la session Spark.
1. Dans la barre de navigation, sélectionnez **Surveiller** pour afficher le hub de surveillance, puis notez que l’activité du notebook est répertoriée.

    ![Capture d’écran du hub de surveillance avec une activité de notebook.](./Images/monitor-notebook.png)

## Surveiller l’historique d’un élément

Certains éléments d’un espace de travail peuvent être exécutés plusieurs fois. Vous pouvez utiliser le hub de surveillance pour afficher son historique.

1. Dans la barre de navigation, revenez à la page de votre espace de travail. Utilisez ensuite le bouton **&#8635;** (*Actualiser maintenant*) de votre flux de données **Obtenir des données produit** pour le réexécuter.
1. Dans le volet de navigation, sélectionnez la page **Surveiller** pour afficher le hub de surveillance et vérifiez que le flux de données est en cours.
1. Dans le menu **...** du flux de données **Obtenir des données produit**, sélectionnez **Exécutions historiques** pour afficher l’historique des exécutions du flux de données :

    ![Capture d’écran de l’affichage des exécutions historiques du hub de surveillance.](./Images/historical-runs.png)

1. Dans le menu **...** pour l’une des exécutions historiques, sélectionnez **Afficher les détails** pour afficher les détails de l’exécution.
1. Fermez le volet **Détails** et utilisez le bouton **Retour à l’affichage principal** pour revenir à la page principale du hub de surveillance.

## Personnaliser les vues du hub de surveillance

Dans cet exercice, vous n’avez exécuté que quelques activités. Il doit donc être assez facile de trouver des événements dans le hub de surveillance. Toutefois, dans un environnement réel, vous devrez peut-être effectuer une recherche dans un grand nombre d’événements. L’utilisation de filtres et d’autres personnalisations d’affichage peut faciliter cette opération.

1. Dans le hub de surveillance, utilisez le bouton **Filtrer** pour appliquer le filtre suivant :
    - **État** : Réussite
    - **Type d’élément** : Dataflow Gen2

    Avec le filtre appliqué, seules les exécutions réussies de flux de données sont répertoriées.

    ![Capture d’écran du hub de surveillance avec un filtre appliqué.](./Images/monitor-filter.png)

1. Utilisez le bouton **Options de colonne** pour inclure les colonnes suivantes dans l’affichage (utilisez le bouton **Appliquer** pour appliquer les modifications) :
    - Nom de l’activité
    - État
    - Type d'élément
    - Heure de début
    - Soumis par
    - Emplacement
    - Heure de fin
    - Durée
    - Type d’actualisation

    Vous devrez peut-être faire défiler l’écran horizontalement pour voir toutes les colonnes.

    ![Capture d’écran du hub de surveillance avec des colonnes personnalisées.](./Images/monitor-columns.png)

## Nettoyer les ressources

Dans cet exercice, vous avez créé un lachouse, un dataflow et un notebook Spark ; et vous avez utilisé le hub de surveillance pour afficher l’activité des éléments.

Si vous avez terminé d’explorer votre lakehouse, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
