---
lab:
  title: "Bien démarrer avec l’interrogation d’une base de données KQL dans Microsoft\_Fabric"
  module: Query data from a KQL database in Microsoft Fabric
---

# Bien démarrer avec l’interrogation d’une base de données KQL dans Microsoft Fabric

KQL Queryset est un outil qui vous permet d’exécuter des requêtes, mais également de modifier et d’afficher les résultats des requêtes à partir d’une base de données KQL. Vous pouvez lier chaque onglet dans KQL Queryset à une base de données KQL différente et enregistrer vos requêtes pour une utilisation ultérieure ou les partager avec d’autres personnes pour l’analyse des données. Vous pouvez également basculer la base de données KQL pour n’importe quel onglet, ce qui vous permet de comparer les résultats de la requête à partir de diverses sources de données.

Dans cet exercice, vous aurez le rôle d’un analyste chargé d’interroger un jeu de données sur les données de courses de taxi à NYC. Vous utilisez KQL pour interroger ces données et collecter des informations afin d’obtenir des insights informatifs sur les données.

> **Conseil** : pour créer des requêtes, KQL Queryset utilise le langage Kusto Query qui est compatible avec de nombreuses fonctions SQL. Pour en savoir plus sur KQL, consultez [Vue d’ensemble du Langage de requête Kusto (KQL)](https://learn.microsoft.com/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext).

Ce labo est d’une durée de **25** minutes environ.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec la capacité Fabric activée.

1. Dans la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric`, sélectionnez **Real-Time Intelligence**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un Eventhouse

1. Sur la page d’accueil de l’**Intelligence en temps réel**, créez un **eventhouse** avec le nom de votre choix. Lorsque l’eventhouse a été créé, fermez toutes les invites ou conseils affichés jusqu’à ce que la page de l’eventhouse soit visible :

   ![Capture d’écran d’un nouvel eventhouse.](./Images/create-eventhouse.png)
   
1. Dans le menu **...** de la base de données KQL créée dans l’eventhouse, sélectionnez **Obtenir des données** > **Échantillon**. Ensuite, choisissez l’exemple de données **Analytique des opérations automobiles**.

1. Une fois le chargement des données terminé, vérifiez qu’une table **Automobile** a été créée.

   ![Capture d’écran de la table Automobile dans une base de données eventhouse.](./Images/choose-automotive-operations-analytics.png)

## Interroger des données à l’aide de KQL

Le langage de requête Kusto (KQL) est un langage intuitif et complet que vous pouvez utiliser pour interroger une base de données KQL.

### Récupérer les données d’une table à l’aide de KQL

1. Dans le volet gauche de la fenêtre de l’eventhouse, sous votre base de données KQL, sélectionnez le fichier **queryset** par défaut. Ce fichier contient des exemples de requêtes KQL pour vous aider à démarrer.
1. Modifiez le premier exemple de requête comme suit.

    ```kql
    Automotive
    | take 100
    ```

    > **REMARQUE** : le caractère trait vertical est utilisé à deux fins dans KQL, notamment pour séparer des opérateurs de requête dans une instruction d’expression tabulaire. Il est également utilisé comme opérateur OR logique entre parenthèses carrées ou rondes pour indiquer que vous pouvez spécifier l’un des éléments séparés par le caractère du canal.

1. Sélectionnez le code de requête et exécutez-le pour renvoyer 100 lignes depuis la table.

   ![Capture d’écran de l’éditeur de requête KQL.](./Images/kql-take-100-query.png)

    Vous pouvez être plus précis en ajoutant des attributs spécifiques que vous souhaitez interroger à l’aide du mot clé `project`, puis en utilisant le mot clé `take` pour indiquer au moteur le nombre d’enregistrements à renvoyer.

1. Entrez la requête suivante, puis sélectionnez-la et exécutez-la.

    ```kql
    // Use 'project' and 'take' to view a sample number of records in the table and check the data.
    Automotive 
    | project vendor_id, trip_distance
    | take 10
    ```

    > **REMARQUE :** l’utilisation de // désigne un commentaire.

    Une autre pratique courante dans l’analyse consiste à renommer des colonnes dans notre ensemble de requêtes pour les rendre plus conviviales.

1. Essayez la requête suivante :

    ```kql
    Automotive 
    | project vendor_id, ["Trip Distance"] = trip_distance
    | take 10
    ```

### Résumer les données à l’aide de KQL

Vous pouvez utiliser le mot clé *summarize* avec une fonction pour agréger et manipuler des données d’autres façons.

1. Essayez la requête suivante, qui utilise la fonction **sum** pour résumer les données de trajet pour voir combien de kilomètres ont été parcourus au total :

    ```kql

    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance)
    ```

    Vous pouvez regrouper les données résumées selon une colonne ou une expression spécifiée.

1. Exécutez la requête suivante pour regrouper les distances de trajet par quartier, au sein du système Taxi de NY, pour déterminer la distance totale parcourue à partir de chaque quartier.

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = pickup_boroname, ["Total Trip Distance"]
    ```

    Les résultats incluent une valeur vide, ce qui n’est jamais bon pour les analyses.

1. Modifiez la requête comme indiqué ici pour utiliser la fonction *case* avec les fonctions *isempty* et *isnull* pour regrouper tous les trajets pour lesquels le quartier est inconnu dans une catégorie ***Non identifié*** pour le suivi.

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    ```

### Trier les données à l’aide de KQL

Pour donner plus de sens à nos données, nous les trions généralement par colonne, et ce processus est effectué dans KQL avec un opérateur *sort by* ou *order by* (ils se comportent de la même façon).

1. Essayez la requête suivante :

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | sort by Borough asc
    ```

1. Modifiez la requête comme suit et réexécutez-la, et remarquez que l’opérateur *order by* fonctionne de la même façon que *sort by* :

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | order by Borough asc 
    ```

### Filtrer les données à l’aide de KQL

Dans KQL, la clause *where* est utilisée pour filtrer les données. Vous pouvez combiner des conditions dans une clause *where* à l’aide des opérateurs logiques *and* et *or*.

1. Exécutez la requête suivante pour filtrer les données de trajets afin d’inclure uniquement ceux qui sont partis de Manhattan :

    ```kql
    Automotive
    | where pickup_boroname == "Manhattan"
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | sort by Borough asc
    ```

## Interroger les données à l’aide de Transact-SQL

La base de données KQL ne prend pas en charge Transact-SQL de manière native, mais elle fournit un point de terminaison T-SQL qui émule Microsoft SQL Server et vous permet d’exécuter des requêtes T-SQL sur vos données. Le point de terminaison T-SQL présente certaines limitations et différences par rapport au SQL Server natif. Il ne prend par exemple pas en charge la création, la modification ou la suppression de tables, ni l’insertion, la mise à jour ou la suppression de données. Il ne prend pas non plus en charge certaines fonctions et syntaxe T-SQL non compatibles avec KQL. Il a été créé pour permettre aux systèmes (ne prenant pas en charge KQL) d’utiliser T-SQL pour interroger les données au sein d’une base de données KQL. Il est donc recommandé d’utiliser KQL comme langage de requête principal pour une base de données KQL, car il offre davantage de fonctionnalités et de performances que T-SQL. Vous pouvez également utiliser certaines fonctions SQL prises en charge par KQL, telles que count, sum, avg, min, max, etc.

### Récupérer des données d’une table à l’aide de Transact-SQL

1. Dans votre ensemble de requêtes, ajoutez et exécutez la requête Transact-SQL suivante : 

    ```sql  
    SELECT TOP 100 * from Automotive
    ```

1. Modifiez la requête comme suit pour récupérer des colonnes spécifiques :

    ```sql
    SELECT TOP 10 vendor_id, trip_distance
    FROM Automotive
    ```

1. Modifiez la requête pour attribuer un alias qui renomme **trip_distance** en utilisant un nom plus convivial.

    ```sql
    SELECT TOP 10 vendor_id, trip_distance as [Trip Distance]
    from Automotive
    ```

### Résumer les données à l’aide de Transact-SQL

1. Exécutez la requête suivante pour rechercher la distance totale parcourue :

    ```sql
    SELECT sum(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    ```

1. Modifiez la requête pour regrouper la distance totale par quartier de départ :

    ```sql
    SELECT pickup_boroname AS Borough, Sum(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY pickup_boroname
    ```

1. Modifiez la requête pour utiliser une instruction *CASE* afin de regrouper les trajets avec une origine inconnue dans une catégorie ***Non identifié*** pour le suivi. 

    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
               ELSE pickup_boroname
             END;
    ```

### Trier les données à l’aide de Transact-SQL

1. Exécutez la requête suivante pour trier les résultats groupés par quartier :
 
    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    ORDER BY Borough ASC;
    ```

### Filtrer les données à l’aide de Transact-SQL
    
1. Exécutez la requête suivante pour filtrer les données groupées afin que seules les lignes avec le quartier « Manhattan » soient incluses dans les résultats :

    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    HAVING Borough = 'Manhattan'
    ORDER BY Borough ASC;
    ```

## Nettoyer les ressources

Dans cet exercice, vous avez créé un eventhouse et interrogé les données à l’aide de KQL et SQL.

Lorsque vous avez terminé d’explorer votre base de données KQL, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail.
2. Dans la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
