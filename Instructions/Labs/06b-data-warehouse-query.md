---
lab:
  title: Interroger un entrepôt de données dans Microsoft Fabric
  module: Query a data warehouse in Microsoft Fabric
---

# Interroger un entrepôt de données dans Microsoft Fabric

Dans Microsoft Fabric, un entrepôt de données fournit une base de données relationnelle pour l’analytique à grande échelle. L’ensemble complet d’expériences intégrées à l’espace de travail Microsoft Fabric permet aux clients de réduire leur temps d’insights en ayant un modèle sémantique facilement consommable et toujours connecté intégré à Power BI en mode DirectLake. 

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’un [essai gratuit Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Data Warehouse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un exemple d’entrepôt de données

Maintenant que vous disposez d’un espace de travail, il est temps de créer un entrepôt de données.

1. En bas à gauche, vérifiez que l’expérience **Data Warehouse** est sélectionnée.
1. Dans la page **Accueil**, sélectionnez **Exemple d’entrepôt**, puis créez un entrepôt de données nommé **sample-dw**.

    Au bout d’environ une minute, un nouvel entrepôt sera créé et rempli avec des exemples de données pour un scénario d’analyse de trajet en taxi.

    ![Capture d’écran d’un nouvel entrepôt.](./Images/sample-data-warehouse.png)

## Interroger l’entrepôt de données

L’éditeur de requête SQL prend en charge IntelliSense, la complétion de code, la mise en surbrillance de la syntaxe, l’analyse côté client et la validation. Vous pouvez exécuter des instructions DDL (Data Definition Language), DML (Data Manipulation Language) et DCL (Data Control Language).

1. Dans la page de l’entrepôt de données **sample-dw**, dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL**.

1. Dans le volet de nouvelle requête vide, entrez le code Transact-SQL suivant :

    ```sql
    SELECT 
    D.MonthName, 
    COUNT(*) AS TotalTrips, 
    SUM(T.TotalAmount) AS TotalRevenue 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.MonthName;
    ```

1. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL et afficher les résultats qui doivent montrer le nombre de trajets et le revenu total par mois.

1. Entrez le code Transact-SQL suivant :

    ```sql
   SELECT 
    D.DayName, 
    AVG(T.TripDurationSeconds) AS AvgDuration, 
    AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. Exécutez la requête modifiée et affichez les résultats qui montrent la durée moyenne d’un trajet et la distance par jour de la semaine.

1. Entrez le code Transact-SQL suivant :

    ```sql
    SELECT TOP 10 
    G.City, 
    COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.PickupGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    
    SELECT TOP 10 
        G.City, 
        COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.DropoffGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    ```

1. Exécutez la requête modifiée et affichez les résultats qui montrent les 10 principaux emplacements de départ et d’arrivée les plus connus.

1. Fermez tous les onglets de requête.

## Vérifier la cohérence des données

La vérification de la cohérence des données est importante pour veiller à ce que les données soient exactes et fiables pour les analyses et les prises de décision. Des données incohérentes peuvent entraîner des analyses incorrectes et des résultats trompeurs. 

Interrogeons votre entrepôt de données pour vérifier la cohérence.

1. Dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL**.

1. Dans le volet de nouvelle requête vide, entrez le code Transact-SQL suivant :

    ```sql
    -- Check for trips with unusually long duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds > 86400; -- 24 hours
    ```

1. Exécutez la requête modifiée et visualisez les résultats qui montrent les informations sur tous les trajets d’une durée exceptionnellement longue.

1. Dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL** pour ajouter un deuxième onglet de requête. Ensuite, sous le nouvel onglet de requête vide, exécutez le code suivant :

    ```sql
    -- Check for trips with negative trip duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

1. Dans le volet de nouvelle requête vide, entrez et exécutez le code Transact-SQL suivant :

    ```sql
    -- Remove trips with negative trip duration
    DELETE FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

    > **Remarque :** Il existe plusieurs façons de gérer des données incohérentes. Plutôt que de les supprimer, une autre solution consiste à les remplacer par une valeur différente, telle que la moyenne ou la médiane.

1. Fermez tous les onglets de requête.

## Enregistrer en tant que vue

Supposons que vous deviez filtrer certains trajets pour un groupe d’utilisateurs qui utilisent les données afin de générer des rapports.

Créons une vue basée sur la requête utilisée plus tôt et ajoutons-y un filtre.

1. Dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL**.

1. Dans le volet de nouvelle requête vide, entrez à nouveau et exécutez le code Transact-SQL suivant :

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. Modifiez la requête pour ajouter `WHERE D.Month = 1`. Cette opération va filtrer les données pour inclure uniquement les enregistrements du mois de janvier. La requête finale doit se présenter comme suit :

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    WHERE D.Month = 1
    GROUP BY D.DayName
    ```

1. Sélectionnez le texte de l’instruction SELECT dans votre requête. Puis à côté du bouton **&#9655; Exécuter**, sélectionnez **Enregistrer en tant que vue**.

1. Créez une vue nommée **vw_JanTrip**.

1. Dans l’**Explorateur**, accédez à **Schémas >> dbo >> Vues**. Notez la vue *vw_JanTrip* que vous venez de créer.

1. Fermez tous les onglets de requête.

> **Informations complémentaires** : Consultez [Interroger en utilisant l’éditeur de requête SQL](https://learn.microsoft.com/fabric/data-warehouse/sql-query-editor) dans la documentation Microsoft Fabric pour obtenir plus d’informations sur l’interrogation d’un entrepôt de données.

## Nettoyer les ressources

Dans cet exercice, vous avez utilisé des requêtes pour obtenir des insights sur les données d’un entrepôt de données Microsoft Fabric.

Si vous avez terminé d’explorer votre entrepôt de données, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans la page de l’espace de travail, sélectionnez **Paramètres de l’espace de travail**.
3. En bas de la section **Général**, sélectionnez **Supprimer cet espace de travail**.
