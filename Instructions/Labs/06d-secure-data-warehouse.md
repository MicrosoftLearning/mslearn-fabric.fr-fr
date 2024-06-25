---
lab:
  title: Sécuriser un entrepôt de données Microsoft Fabric
  module: Secure a Microsoft Fabric data warehouse
---

# Sécuriser des données dans un entrepôt de données

Les autorisations Microsoft Fabric et les autorisations SQL granulaires fonctionnent ensemble pour régir l’accès à l’entrepôt et les autorisations utilisateur. Dans cet exercice, vous allez sécuriser les données en utilisant des autorisations granulaires, la sécurité au niveau des colonnes, la sécurité au niveau des lignes et le masquage dynamique des données.

> **Remarque** : Pour effectuer les exercices de ce labo, vous aurez besoin de deux utilisateurs : un utilisateur avec le rôle Administrateur d’espace de travail et un autre avec le rôle Viewer d’espace de travail. Pour attribuer des rôles à des espaces de travail, consultez [ Accorder l’accès à votre espace de travail ](
https://learn.microsoft.com/fabric/get-started/give-access-workspaces
).

Ce labo est d’une durée de **45** minutes environ.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Data Warehouse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-empty-workspace.png)

> **Remarque** : Quand vous créez un espace de travail, vous devenez automatiquement membre du rôle Administrateur d’espace de travail. Vous pouvez ajouter un deuxième utilisateur de votre environnement au rôle Viewer d’espace de travail pour tester les fonctionnalités configurées dans ces exercices. Pour ce faire, sélectionnez **Gérer l’accès** dans l’espace de travail, puis **Ajouter des personnes ou des groupes**. Ceci permet au deuxième utilisateur de voir le contenu de l’espace de travail.

## Créer un entrepôt de données

Ensuite, créez un entrepôt de données dans l’espace de travail que vous avez créé. La page d’accueil Data Warehouse comprend un raccourci permettant de créer un entrepôt :

1. Dans la page d’accueil **Synapse Data Warehouse**, créez un **entrepôt** avec le nom de votre choix.

    Au bout d’une minute environ, un nouvel entrepôt est créé :

    ![Capture d’écran d’un nouvel entrepôt.](./Images/new-empty-data-warehouse.png)

## Appliquer un masquage dynamique des données aux colonnes d’une table

Les règles de masquage dynamique des données sont appliquées sur des colonnes individuelles au niveau de la table afin que toutes les requêtes soient affectées par le masquage. Les utilisateurs qui n’ont pas d’autorisations explicites pour visualiser les données confidentielles voient des valeurs masquées dans les résultats des requêtes, tandis que ceux qui ont l’autorisation explicite de visualiser ces données les voient en clair. Il existe quatre types de masques : par défaut, e-mail, chaîne aléatoire et personnalisée. Dans cet exercice, vous allez appliquer un masque par défaut, un masque de messagerie et un masque de chaîne personnalisé.

1. Dans votre entrepôt, sélectionnez la vignette **T-SQL** et remplacez le code SQL par défaut par les instructions T-SQL suivantes pour créer une table et insérer et afficher des données.  

    ```tsql
    CREATE TABLE dbo.Customers
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
    
    INSERT dbo.Customers (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
    
    SELECT * FROM dbo.Customers;
    
    ```
    Quand des utilisateurs qui ne sont pas autorisés à voir des données non masquées interrogent la table, la colonne **FirstName** va afficher la première lettre de la chaîne avec XXXXXXX, sans aucun des derniers caractères. La colonne **Phone** va afficher xxxx. La colonne **Email** va afficher la première lettre de l’adresse e-mail suivie de `XXX@XXX.com`. Cette approche garantit que les données sensibles restent confidentielles, tout en permettant aux utilisateurs restreints d’interroger la table.

2. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL, qui crée une table nommée **Customers** dans le schéma **dbo** de l’entrepôt de données.

3. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **dbo** > **Tables** et vérifiez que la table **Customers** a été créée. L’instruction `SELECT` retourne des données non masquées pour vous, car en tant que créateur de l’espace de travail, vous êtes membre du rôle Administrateur d’espace de travail, qui peut voir les données non masquées.

4. Connectez-vous en tant qu’utilisateur de test membre du rôle d’espace de travail **Viewer** et exécutez l’instruction T-SQL suivante.

    ```tsql
    SELECT * FROM dbo.Customers;
    ```
    L’utilisateur de test n’a pas reçu l’autorisation UNMASK : les données retournées pour les colonnes FirstName, Phone et Email sont donc masquées, car ces colonnes ont été définies avec un masque dans l’instruction `CREATE TABLE`.

5. Reconnectez-vous en tant qu’Administrateur d’espace de travail (vous-même) et exécutez les instructions T-SQL suivantes afin de supprimer le masquage des données pour l’utilisateur de test. Remplacez `<username>@<your_domain>.com` par le nom de l’utilisateur que vous testez, qui est membre du rôle d’espace de travail **Viewer**. 

    ```tsql
    GRANT UNMASK ON dbo.Customers TO [<username>@<your_domain>.com];
    ```

6. Connectez-vous à nouveau en tant qu’utilisateur de test et exécutez l’instruction T-SQL suivante.

    ```tsql
    SELECT * FROM dbo.Customers;
    ```

    Les données sont retournées non masquées, car l’utilisateur de test a reçu l’autorisation `UNMASK`.

## Appliquer la sécurité au niveau des lignes

La sécurité au niveau des lignes (RLS) peut être utilisée pour limiter l’accès aux lignes en fonction de l’identité ou du rôle de l’utilisateur exécutant une requête. Dans cet exercice, vous limitez l’accès aux lignes en créant une stratégie de sécurité et un prédicat de sécurité défini comme une TVF inline.

1. Dans l’entrepôt que vous avez créé dans le dernier exercice, sélectionnez la liste déroulante **Nouvelle requête SQL**.  Sous l’en-tête **Vide**, sélectionnez **Nouvelle requête SQL**.

2. Créer une table et y insérer des données. Pour pouvoir tester la sécurité au niveau des lignes dans une étape ultérieure, remplacez `<username1>@<your_domain>.com` par un nom d’utilisateur de votre environnement, et remplacez `<username2>@<your_domain>.com` par votre nom d’utilisateur.

    ```tsql
    CREATE TABLE dbo.Sales  
    (  
        OrderID INT,  
        SalesRep VARCHAR(60),  
        Product VARCHAR(10),  
        Quantity INT  
    );
     
    --Populate the table with 6 rows of data, showing 3 orders for each test user. 
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, '<username1>@<your_domain>.com', 'Valve', 5),   
    (2, '<username1>@<your_domain>.com', 'Wheel', 2),   
    (3, '<username1>@<your_domain>.com', 'Valve', 4),  
    (4, '<username2>@<your_domain>.com', 'Bracket', 2),   
    (5, '<username2>@<your_domain>.com', 'Wheel', 5),   
    (6, '<username2>@<your_domain>.com', 'Seat', 5);  
     
    SELECT * FROM dbo.Sales;  
    ```

3. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL, qui crée une table nommée **Ventes** dans le schéma **dbo** de l’entrepôt de données.

4. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **dbo** > **Tables** et vérifiez que la table **Ventes** a été créée.
5. Créez un schéma, un prédicat de sécurité défini en tant que fonction et une stratégie de sécurité.  

    ```tsql
    --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
    CREATE SCHEMA rls;
    GO
    
    /*Create the security predicate defined as an inline table-valued function.
    A predicate evaluates to true (1) or false (0). This security predicate returns 1,
    meaning a row is accessible, when a row in the SalesRep column is the same as the user
    executing the query.*/

    --Create a function to evaluate who is querying the table
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO

    /*Create a security policy to invoke and enforce the function each time a query is run on the Sales table.
    The security policy has a filter predicate that silently filters the rows available to 
    read operations (SELECT, UPDATE, and DELETE). */
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
    ```

6. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL.
7. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **rls** > **Fonctions** et vérifiez que la fonction a été créée.
8. Connectez-vous à Fabric en tant qu’utilisateur par lequel vous avez remplacé `<username1>@<your_domain>.com`, dans l’instruction `INSERT` pour la table Sales. Vérifiez que vous êtes connecté en tant que cet utilisateur en exécutant le T-SQL suivant.

    ```tsql
    SELECT USER_NAME();
    ```

9. Interrogez la table **Sales** pour vérifier que la sécurité au niveau des lignes fonctionne comme prévu. Vous devez voir seulement les données qui répondent aux conditions du prédicat de sécurité défini pour l’utilisateur sous lequel vous êtes connecté.

    ```tsql
    SELECT * FROM dbo.Sales;
    ```

## Implémenter la sécurité au niveau des colonnes

La sécurité au niveau des colonnes vous permet de désigner les utilisateurs qui peuvent accéder à des colonnes spécifiques d’une table. Elle est implémentée en émettant une instruction `GRANT` ou `DENY` sur une table en spécifiant une liste de colonnes et l’utilisateur ou le rôle qui peuvent ou ne peuvent pas les lire. Pour simplifier la gestion des accès, affectez des autorisations aux rôles au lieu de le faire à des utilisateurs individuels. Dans cet exercice, vous allez créer une table, accorder l’accès à un sous-ensemble de colonnes sur la table et tester que les colonnes restreintes ne peuvent pas être visualisées par un utilisateur autre que vous-même.

1. Dans l’entrepôt que vous avez créé dans l’exercice précédent, sélectionnez la liste déroulante **Nouvelle requête SQL**. Sous l’en-tête **Vide**, sélectionnez **Nouvelle requête SQL**.  

2. Créez une table et insérez des données dans la table.

    ```tsql
    CREATE TABLE dbo.Orders
    (   
        OrderID INT,   
        CustomerID INT,  
        CreditCard VARCHAR(20)      
    );

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');

    SELECT * FROM dbo.Orders;
     ```

3. Refuser l’autorisation d’afficher une colonne dans la table. L’instruction T-SQL empêche `<username>@<your_domain>.com` de voir la colonne CreditCard de la table Orders. Dans l’instruction `DENY`, remplacez `<username>@<your_domain>.com` par un nom d’utilisateur de votre système qui a des autorisations **Viewer** sur l’espace de travail.

     ```tsql
    DENY SELECT ON dbo.Orders (CreditCard) TO [<username>@<your_domain>.com];
     ```

4. Testez la sécurité au niveau des colonnes en vous connectant à Fabric en tant qu’utilisateur auquel vous avez refusé les autorisations de sélection.

5. Interrogez la table Orders pour confirmer que la sécurité au niveau des colonnes fonctionne comme prévu. La requête suivante va retourner seulement les colonnes OrderID et CustomerID, et non pas la colonne CreditCard.  

    ```tsql
    SELECT * FROM dbo.Orders;
    ```

    Vous recevrez une erreur car l’accès à la colonne CreditCard a été restreint.  Essayez en sélectionnant seulement les champs OrderID et CustomerID : la requête va réussir.

    ```tsql   
    SELECT OrderID, CustomerID from dbo.Orders
    ```

## Configurer des autorisations granulaires SQL à l’aide de T-SQL

Fabric a un modèle d’autorisations qui vous permet de contrôler l’accès aux données au niveau de l’espace de travail et au niveau de l’élément. Quand vous avez besoin d’un contrôle plus précis de ce que les utilisateurs peuvent faire avec des éléments sécurisables dans un entrepôt Fabric, vous pouvez utiliser les commandes du langage de contrôle de données SQL standard (DCL) `GRANT`, `DENY` et `REVOKE`. Dans cet exercice, vous allez créer des objets, les sécuriser avec `GRANT` et `DENY` puis exécuter des requêtes pour afficher l’effet de l’application d’autorisations granulaires.

1. Dans l’entrepôt que vous avez créé dans l’exercice précédent, sélectionnez la liste déroulante **Nouvelle requête SQL**. Sous l’en-tête **Vide**, sélectionnez **Nouvelle requête SQL**.  

2. Créez une procédure stockée et une table. Exécutez ensuite la procédure et interrogez la table.

     ```tsql
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO

    CREATE TABLE dbo.Parts
    (
        PartID INT,
        PartName VARCHAR(25)
    );
    
    INSERT dbo.Parts (PartID, PartName) VALUES
    (1234, 'Wheel'),
    (5678, 'Seat');
     GO
    
    /*Execute the stored procedure and select from the table and note the results you get
    as a member of the Workspace Admin role. Look for output from the stored procedure on 
    the 'Messages' tab.*/
    EXEC dbo.sp_PrintMessage;
    GO

    SELECT * FROM dbo.Parts
     ```

3. Ensuite, appliquez `DENY SELECT` sur la table à un utilisateur membre du rôle **Viewer d’espace de travail** et `GRANT EXECUTE` sur la procédure pour le même utilisateur. Remplacez `<username>@<your_domain>.com` par un nom d’utilisateur de votre environnement membre du rôle **Viewer d’espace de travail**. 

     ```tsql
    DENY SELECT on dbo.Parts to [<username>@<your_domain>.com];

    GRANT EXECUTE on dbo.sp_PrintMessage to [<username>@<your_domain>.com];
     ```

4. Connectez-vous à Fabric en tant que l’utilisateur que vous avez spécifié dans les instructions `DENY` et `GRANT` à la place de `<username>@<your_domain>.com`. Testez ensuite les autorisations granulaires que vous avez appliquées en exécutant la procédure stockée et en interrogeant la table.  

     ```tsql
    EXEC dbo.sp_PrintMessage;
    GO
   
    SELECT * FROM dbo.Parts;
     ```

## Nettoyer les ressources

Dans cet exercice, vous avez appliqué le masquage dynamique des données aux colonnes d’une table, appliqué la sécurité au niveau des lignes, implémenté la sécurité au niveau des colonnes et configuré des autorisations granulaires SQL en utilisant T-SQL.

1. Dans la barre de navigation de gauche, sélectionnez l’icône de votre espace de travail pour voir tous les éléments qu’il contient.
2. Dans le menu de la barre d’outils du haut, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
