---
lab:
  title: Sécuriser des données dans un entrepôt de données
  module: Get started with data warehouses in Microsoft Fabric
---

# Sécuriser des données dans un entrepôt de données

Les autorisations Microsoft Fabric et les autorisations SQL granulaires fonctionnent ensemble pour régir l’accès à l’entrepôt et les autorisations utilisateur. Dans cet exercice, vous allez sécuriser les données à l’aide d’autorisations granulaires, de sécurité au niveau des colonnes, de sécurité au niveau des lignes et de masquage dynamique des données.

Ce labo prend environ **45** minutes.

> **Remarque** : Vous devez disposer d’une [licence d’essai Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Data Warehouse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-empty-workspace.png)

## Créer un entrepôt de données

Ensuite, vous allez créer un entrepôt de données dans l’espace de travail que vous venez de créer. La page d’accueil Data Warehouse comprend un raccourci permettant de créer un entrepôt :

1. Dans la page d’accueil **Synapse Data Warehouse**, créez un **entrepôt** avec le nom de votre choix.

    Au bout d’une minute environ, un nouvel entrepôt est créé :

    ![Capture d’écran d’un nouvel entrepôt.](./Images/new-empty-data-warehouse.png)

## Appliquer un masquage dynamique des données aux colonnes d’une table

Les règles de masquage dynamique des données sont appliquées sur des colonnes individuelles au niveau de la table afin que toutes les requêtes soient affectées par le masquage. Les utilisateurs qui n’ont pas d’autorisations explicites pour afficher les données confidentielles voient les valeurs masquées dans les résultats de requête, contrairement à ceux disposant d’une autorisation d’affichage des données. Il existe quatre types de masques : par défaut, adresse e-mail, chaîne aléatoire et personnalisée. Dans cet exercice, vous allez appliquer un masque par défaut, un masque d’adresse e-mail et un masque de chaîne personnalisé.

1. Dans votre entrepôt, sélectionnez la vignette **T-SQL** et remplacez le code SQL par défaut par les instructions T-SQL suivantes pour créer une table et insérer et afficher des données.  Les masques appliqués dans l’instruction `CREATE TABLE` effectuent les opérations suivantes :

    ```sql
    CREATE TABLE dbo.Customer
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
    GO
    --Users restricted from seeing masked data will see the following when they query the table
    --The FirstName column shows the first letter of the string with XXXXXXX and none of the last characters.
    --The Phone column shows xxxx
    --The Email column shows the first letter of the email address followed by XXX@XXX.com.
    
    INSERT dbo.Customer (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
    GO

    SELECT * FROM dbo.Customer;
    GO
    ```

2. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL, qui crée une table nommée **Client** dans le schéma **dbo** de l’entrepôt de données.

3. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **dbo** > **Tables** et vérifiez que la table **Client** a été créée. L’instruction SELECT retourne des données non masquées, car vous êtes connecté en tant qu’administrateur d’espace de travail qui peut voir les données non masquées.

4. Connectez-vous en tant qu’utilisateur de test membre du rôle d’espace de travail **viewer** et exécutez l’instruction T-SQL suivante.

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    Cet utilisateur n’a pas reçu l’autorisation UNMASK afin que les données retournées pour les colonnes FirstName, Phone et Email soient masquées, car ces colonnes ont été définies avec un masque dans l’instruction `CREATE TABLE`.

5. Reconnectez-vous en tant qu’administrateur de l’espace de travail et exécutez les données T-SQL suivantes pour supprimer les données de l’utilisateur de test.

    ```sql
    GRANT UNMASK ON dbo.Customer TO [testUser@testdomain.com];
    GO
    ```

6. Connectez-vous à nouveau en tant qu’utilisateur de test et exécutez l’instruction T-SQL suivante.

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    Les données sont retournées non masquées, car l’utilisateur de test a reçu l’autorisation `UNMASK`.

## Appliquer la sécurité au niveau des lignes

La sécurité au niveau des lignes (RLS) peut être utilisée pour limiter l’accès aux lignes en fonction de l’identité ou du rôle de l’utilisateur exécutant une requête.  Dans cet exercice, vous limiterez l’accès aux lignes en créant une stratégie de sécurité et un prédicat de sécurité défini comme une TVF inline.

1. Dans l’entrepôt que vous avez créé dans le dernier exercice, sélectionnez la liste déroulante **Nouvelle requête SQL**.  Sous la liste déroulante sous l’en-tête **Vide**, sélectionnez **Nouvelle requête SQL**.

2. Créer une table et y insérer des données. Pour que vous puissiez tester la sécurité au niveau des lignes dans une étape ultérieure, remplacez « testuser1@mydomain.com » par un nom d’utilisateur de votre environnement et remplacez « testuser2@mydomain.com » par votre nom d’utilisateur.
    ```sql
    CREATE TABLE dbo.Sales  
    (  
        OrderID INT,  
        SalesRep VARCHAR(60),  
        Product VARCHAR(10),  
        Quantity INT  
    );
    GO
     
    --Populate the table with 6 rows of data, showing 3 orders for each test user. 
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, 'testuser1@mydomain.com', 'Valve', 5),   
    (2, 'testuser1@mydomain.com', 'Wheel', 2),   
    (3, 'testuser1@mydomain.com', 'Valve', 4),  
    (4, 'testuser2@mydomain.com', 'Bracket', 2),   
    (5, 'testuser2@mydomain.com', 'Wheel', 5),   
    (6, 'testuser2@mydomain.com', 'Seat', 5);  
    GO
   
    SELECT * FROM dbo.Sales;  
    GO
    ```

3. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL, qui crée une table nommée **Ventes** dans le schéma **dbo** de l’entrepôt de données.

4. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **dbo** > **Tables** et vérifiez que la table **Ventes** a été créée.
5. Créez un schéma, un prédicat de sécurité défini en tant que fonction et une stratégie de sécurité.  

    ```sql
    --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
    CREATE SCHEMA rls;
    GO
    
    --Create the security predicate defined as an inline table-valued function. A predicate evalutes to true (1) or false (0). This security predicate returns 1, meaning a row is accessible, when a row in the SalesRep column is the same as the user executing the query.

    --Create a function to evaluate which SalesRep is querying the table
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO 
    
    --Create a security policy to invoke and enforce the function each time a query is run on the Sales table. The security policy has a Filter predicate that silently filters the rows available to read operations (SELECT, UPDATE, and DELETE). 
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
6. Use the **&#9655; Run** button to run the SQL script
7. Then, in the **Explorer** pane, expand **Schemas** > **rls** > **Functions**, and verify that the function has been created.
7. Confirm that you're logged as another user by running the following T-SQL.

    ```sql
    SELECT USER_NAME();
    GO
5. Query the sales table to confirm that row-level security works as expected. You should only see data that meets the conditions in the security predicate defined for the user you're logged in as.

    ```sql
    SELECT * FROM dbo.Sales;
    GO

## Implement column-level security

Column-level security allows you to designate which users can access specific columns in a table. It is implemented by issuing a GRANT statement on a table specifying a list of columns and the user or role that can read them. To streamline access management, assign permissions to roles in lieu of individual users. In this exercise, you will create a table, grant access to a subset of columns on the table, and test that restricted columns are not viewable by a user other than yourself.

1. In the warehouse you created in the earlier exercise, select the **New SQL Query** dropdown.  Under the dropdown under the header **Blank**, select **New SQL Query**.  

2. Create a table and insert data into the table.

 ```sql
    CREATE TABLE dbo.Orders
    (   
        OrderID INT,   
        CustomerID INT,  
        CreditCard VARCHAR(20)      
        );
    GO

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');
    GO

    SELECT * FROM dbo.Orders;
    GO
 ```

3. Refuser l’autorisation d’afficher une colonne dans la table. Transact SQL ci-dessous empêche « <testuser@mydomain.com> » de voir la colonne CreditCard dans la table Orders. Dans l’instruction `DENY` ci-dessous, remplacez testuser@mydomain.com par un nom d’utilisateur dans votre système qui dispose des autorisations de visionneuse sur l’espace de travail.

 ```sql
DENY SELECT ON dbo.Orders (CreditCard) TO [testuser@mydomain.com];
 ```

4. Testez la sécurité au niveau des colonnes en vous connectant à Fabric en tant qu’utilisateur auquel vous avez refusé les autorisations de sélection.

5. Interrogez la table Orders pour confirmer que la sécurité au niveau des colonnes fonctionne comme prévu. La requête suivante retourne uniquement les colonnes OrderID et CustomerID, et non la colonne CrediteCard.  

    ```sql
    SELECT * FROM dbo.Orders;
    GO

    --You'll receive an error because access to the CreditCard column has been restricted.  Try selecting only the OrderID and CustomerID fields and the query will succeed.

    SELECT OrderID, CustomerID from dbo.Orders
    ```

## Configurer des autorisations granulaires SQL à l’aide de T-SQL

L’entrepôt Fabric dispose d’un modèle d’autorisations qui vous permet de contrôler l’accès aux données au niveau de l’espace de travail et au niveau de l’élément. Lorsque vous avez besoin d’un contrôle plus précis de ce que les utilisateurs peuvent faire avec des éléments sécurisables dans un entrepôt Fabric, vous pouvez utiliser les commandes de langage de contrôle de données SQL standard (DCL) `GRANT`, `DENY` et `REVOKE`. Dans cet exercice, vous allez créer des objets, les sécuriser avec `GRANT` et `DENY` puis exécuter des requêtes pour afficher l’effet de l’application d’autorisations granulaires.

1. Dans l’entrepôt que vous avez créé dans l’exercice précédent, sélectionnez la liste déroulante **Nouvelle requête SQL**.  Sous l’en-tête **Vide**, sélectionnez **Nouvelle requête SQL**.  

2. Créez une procédure stockée et une table.

 ```
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO
  
    CREATE TABLE dbo.Parts
    (
        PartID INT,
        PartName VARCHAR(25)
    );
    GO
    
    INSERT dbo.Parts (PartID, PartName) VALUES
    (1234, 'Wheel'),
    (5678, 'Seat');
    GO  
    
    --Execute the stored procedure and select from the table and note the results you get because you're a member of the Workspace Admin. Look for output from the stored procedure on the 'Messages' tab.
      EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
    GO
  ```

3. Autorisations `DENY SELECT` suivantes sur la table pour un utilisateur avec qui est membre du rôle Workspace Viewer et `GRANT EXECUTE` sur la procédure pour le même utilisateur.

 ```sql
    DENY SELECT on dbo.Parts to [testuser@mydomain.com];
    GO

    GRANT EXECUTE on dbo.sp_PrintMessage to [testuser@mydomain.com];
    GO

 ```

4. Connectez-vous à Fabric en tant qu’utilisateur que vous avez spécifié dans les instructions DENY et GRANT ci-dessus à la place de [testuser@mydomain.com]. Testez ensuite les autorisations granulaires que vous venez d’appliquer en exécutant la procédure stockée et en interrogeant la table.  

 ```sql
    EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
 ```

## Nettoyer les ressources

Dans cet exercice, vous avez appliqué le masquage dynamique des données aux colonnes d’une table, appliqué la sécurité au niveau des lignes, implémenté la sécurité au niveau des colonnes et configuré des autorisations granulaires SQL à l’aide de T-SQL.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
