---
lab:
  title: "Utiliser SQL\_Database dans Microsoft\_Fabric"
  module: Get started with SQL Database in Microsoft Fabric
---

# Utiliser SQL Database dans Microsoft Fabric

SQL Database dans Microsoft Fabric est une base de données transactionnelle conviviale basée sur Azure SQL Database, qui vous permet de créer facilement votre base de données opérationnelle dans Fabric. Une base de données SQL dans Fabric utilise le moteur SQL Database comme Azure SQL Database.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’un [essai gratuit Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Dans la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric`, sélectionnez **Synapse Data Warehouse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer une base de données SQL avec des échantillons de données

Maintenant que vous disposez d’un espace de travail, vous pouvez créer une base de données SQL.

1. Sélectionnez **+ Créer** dans le volet de gauche.
1. Accédez à la section **Bases de données**, puis sélectionnez **Base de données SQL**.
1. Entrez **AdventureWorksLT** comme nom de base de données, puis sélectionnez **Créer**.
1. Une fois la base de données créée, vous pouvez y charger des échantillons de données à partir de la carte **Échantillons de données**.

    En une minute environ, votre base de données se remplit d’échantillons de données pour votre scénario.

    ![Capture d’écran d’une nouvelle base de données chargée d’échantillons de données.](./Images/sql-database-sample.png)

## Interroger une base de données SQL

L’éditeur de requête SQL prend en charge IntelliSense, la complétion de code, la mise en surbrillance de la syntaxe, l’analyse côté client et la validation. Vous pouvez exécuter des instructions DDL (Data Definition Language), DML (Data Manipulation Language) et DCL (Data Control Language).

1. Dans la page de base de données **AdventureWorksLT**, accédez à **Accueil**, puis sélectionnez **Nouvelle requête**.

1. Dans le volet de nouvelle requête vide, entrez et exécutez le code T-SQL suivant :

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    Cette requête joint les tables `Product` et les `ProductCategory` pour afficher le nom des produits, leurs catégories et leurs prix catalogue, triés par ordre décroissant.

1. Dans un nouvel éditeur de requête, entrez et exécutez le code T-SQL suivant :

    ```sql
   SELECT 
        c.FirstName,
        c.LastName,
        soh.OrderDate,
        soh.SubTotal
    FROM 
        SalesLT.Customer c
    INNER JOIN 
        SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    ORDER BY 
        soh.OrderDate DESC;
    ```

    Cette requête récupère une liste de clients, ainsi que leurs dates de commande et leurs sous-totaux, triés par date de commande dans l’ordre décroissant. 

1. Fermez tous les onglets de requête.

## Intégrer des données à des sources de données externes

Vous allez intégrer des données externes sur les jours fériés avec la commande client. Ensuite, vous allez identifier les commandes qui coïncident avec les jours fériés, en fournissant des informations sur la façon dont ces derniers peuvent avoir un impact sur les activités de vente.

1. Accédez à **Accueil**, puis sélectionnez **Nouvelle requête**.

1. Dans le volet de nouvelle requête vide, entrez et exécutez le code T-SQL suivant :

    ```sql
    CREATE TABLE SalesLT.PublicHolidays (
        CountryOrRegion NVARCHAR(50),
        HolidayName NVARCHAR(100),
        Date DATE,
        IsPaidTimeOff BIT
    );
    ```

    Cette requête crée la table `SalesLT.PublicHolidays` en préparation de l’étape suivante.

1. Dans un nouvel éditeur de requête, entrez et exécutez le code T-SQL suivant :

    ```sql
    INSERT INTO SalesLT.PublicHolidays (CountryOrRegion, HolidayName, Date, IsPaidTimeOff)
    SELECT CountryOrRegion, HolidayName, Date, IsPaidTimeOff
    FROM OPENROWSET 
    (BULK 'abs://holidaydatacontainer@azureopendatastorage.blob.core.windows.net/Processed/*.parquet'
    , FORMAT = 'PARQUET') AS [PublicHolidays]
    WHERE countryorRegion in ('Canada', 'United Kingdom', 'United States')
        AND YEAR([date]) = 2024
    ```
    
    Cette requête lit les données de congés des fichiers Parquet dans Stockage Blob Azure, les filtre pour inclure uniquement les jours fériés au Canada, au Royaume-Uni et aux États-Unis pour l’année 2024, puis insère ces données filtrées dans la table `SalesLT.PublicHolidays`.    

1. Dans un éditeur de requête nouveau ou existant, entrez et exécutez le code T-SQL suivant.

    ```sql
    -- Insert new addresses into SalesLT.Address
    INSERT INTO SalesLT.Address (AddressLine1, City, StateProvince, CountryRegion, PostalCode, rowguid, ModifiedDate)
    VALUES
        ('123 Main St', 'Seattle', 'WA', 'United States', '98101', NEWID(), GETDATE()),
        ('456 Maple Ave', 'Toronto', 'ON', 'Canada', 'M5H 2N2', NEWID(), GETDATE()),
        ('789 Oak St', 'London', 'England', 'United Kingdom', 'EC1A 1BB', NEWID(), GETDATE());
    
    -- Insert new orders into SalesOrderHeader
    INSERT INTO SalesLT.SalesOrderHeader (
        SalesOrderID, RevisionNumber, OrderDate, DueDate, ShipDate, Status, OnlineOrderFlag, 
        PurchaseOrderNumber, AccountNumber, CustomerID, ShipToAddressID, BillToAddressID, 
        ShipMethod, CreditCardApprovalCode, SubTotal, TaxAmt, Freight, Comment, rowguid, ModifiedDate
    )
    VALUES
        (1001, 1, '2024-12-25', '2024-12-30', '2024-12-26', 1, 1, 'PO12345', 'AN123', 1, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), 'Ground', '12345', 100.00, 10.00, 5.00, 'New Order 1', NEWID(), GETDATE()),
        (1002, 1, '2024-11-28', '2024-12-03', '2024-11-29', 1, 1, 'PO67890', 'AN456', 2, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), 'Air', '67890', 200.00, 20.00, 10.00, 'New Order 2', NEWID(), GETDATE()),
        (1003, 1, '2024-02-19', '2024-02-24', '2024-02-20', 1, 1, 'PO54321', 'AN789', 3, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Sea', '54321', 300.00, 30.00, 15.00, 'New Order 3', NEWID(), GETDATE()),
        (1004, 1, '2024-05-27', '2024-06-01', '2024-05-28', 1, 1, 'PO98765', 'AN321', 4, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Ground', '98765', 400.00, 40.00, 20.00, 'New Order 4', NEWID(), GETDATE());
    ```

    Ce code ajoute de nouvelles adresses et commandes à la base de données, en simulant des commandes fictives provenant de différents pays.

1. Dans un éditeur de requête nouveau ou existant, entrez et exécutez le code T-SQL suivant.

    ```sql
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    ```

    Prenez un moment pour observer les résultats, en notant comment la requête identifie les commandes commerciales qui coïncident avec les jours fériés dans les pays respectifs. Vous obtenez ainsi des informations précieuses sur les modèles de commande et les impacts potentiels des jours fériés sur les activités de vente.

1. Fermez tous les onglets de requête.

## Sécuriser les données

Supposons qu’un groupe spécifique d’utilisateurs ne doit avoir accès qu’aux données des États-Unis pour générer des rapports.

Créons une vue basée sur la requête utilisée plus tôt et ajoutons-y un filtre.

1. Dans le volet de nouvelle requête vide, entrez et exécutez le code T-SQL suivant :

    ```sql
    CREATE VIEW SalesLT.vw_SalesOrderHoliday AS
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    WHERE a.CountryRegion = 'United Kingdom';
    ```

1. Dans un éditeur de requête nouveau ou existant, entrez et exécutez le code T-SQL suivant.

    ```sql
    -- Create the role
    CREATE ROLE SalesOrderRole;
    
    -- Grant select permission on the view to the role
    GRANT SELECT ON SalesLT.vw_SalesOrderHoliday TO SalesOrderRole;
    ```

    Tout utilisateur ajouté en tant que membre au rôle `SalesOrderRole` aura accès uniquement à la vue filtrée. Si un utilisateur dans ce rôle tente d’accéder à d’autres objets utilisateur, il reçoit un message d’erreur similaire à celui-ci :

    ```
    Msg 229, Level 14, State 5, Line 1
    The SELECT permission was denied on the object 'ObjectName', database 'DatabaseName', schema 'SchemaName'.
    ```

> **Informations supplémentaires** : voir [Qu’est-ce que Microsoft Fabric ?](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview) dans la documentation Microsoft Fabric pour en savoir plus sur les autres composants disponibles dans la plateforme.

Dans cet exercice, vous avez créé et importé des données externes, puis vous les avez interrogées et sécurisées dans une base de données SQL de Microsoft Fabric.

## Nettoyer les ressources

Si vous avez terminé d’explorer votre base de données, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
