---
lab:
  title: Analyser des données dans un entrepôt de données
  module: Get started with data warehouses in Microsoft Fabric
---

# Analyser des données dans un entrepôt de données

Dans Microsoft Fabric, un entrepôt de données fournit une base de données relationnelle pour l’analytique à grande échelle. Contrairement au point de terminaison SQL en lecture seule par défaut pour les tables définies dans un lakehouse, un entrepôt de données fournit une sémantique SQL complète, y compris la possibilité d’insérer, de mettre à jour et de supprimer des données dans les tables.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous avez besoin d’un compte *scolaire* ou *professionnel* Microsoft pour réaliser cet exercice. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Activer une version d’évaluation de Microsoft Fabric

1. Après avoir ouvert un compte Microsoft Fabric, accédez au portail Microsoft Fabric à l’adresse [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com).
1. Sélectionnez l’icône **Gestionnaire de comptes** (l’image de l’*utilisateur* en haut à droite)
1. Dans le menu du gestionnaire de comptes, sélectionnez **Démarrer la version d’évaluation** pour démarrer un essai de Microsoft Fabric.
1. Après avoir effectué la mise à niveau vers Microsoft Fabric, accédez à la page d’accueil en sélectionnant **Page d’accueil de l’infrastructure**.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Data Warehouse**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un entrepôt de données

Maintenant que vous disposez d’un espace de travail, il est temps de créer un entrepôt de données. La page d’accueil Data Warehouse comprend un raccourci permettant de créer un entrepôt :

1. Dans la page d’accueil **Synapse Data Warehouse**, créez un **entrepôt** avec le nom de votre choix.

    Au bout d’une minute environ, un nouvel entrepôt est créé :

    ![Capture d’écran d’un nouvel entrepôt.](./Images/new-data-warehouse.png)

## Créer des tables et insérer des données

Un entrepôt est une base de données relationnelle dans laquelle vous pouvez définir des tables et d’autres objets.

1. Dans votre nouvel entrepôt, sélectionnez la vignette **Créer des tables avec T-SQL** et remplacez le code SQL par défaut par l’instruction CREATE TABLE suivante :

    ```sql
   CREATE TABLE dbo.DimProduct
   (
       ProductKey INTEGER NOT NULL,
       ProductAltKey VARCHAR(25) NULL,
       ProductName VARCHAR(50) NOT NULL,
       Category VARCHAR(50) NULL,
       ListPrice DECIMAL(5,2) NULL
   );
   GO
    ```

2. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL, qui crée une table nommée **DimProduct** dans le schéma **dbo** de l’entrepôt de données.
3. Utilisez le bouton **Actualiser** dans la barre d’outils pour actualiser la vue. Ensuite, dans le volet **Explorateur**, développez **Schémas** > **dbo** > **Tables** et vérifiez que la table **DimProduct** a été créée.
4. Sous l’onglet du menu **Accueil**, utilisez le bouton **Nouvelle requête SQL** pour créer une requête, puis entrez l’instruction INSERT suivante :

    ```sql
   INSERT INTO dbo.DimProduct
   VALUES
   (1, 'RING1', 'Bicycle bell', 'Accessories', 5.99),
   (2, 'BRITE1', 'Front light', 'Accessories', 15.49),
   (3, 'BRITE2', 'Rear light', 'Accessories', 15.49);
   GO
    ```

5. Exécutez la nouvelle requête pour insérer trois lignes dans la table **DimProduct**.
6. Une fois la requête terminée, sélectionnez l’onglet **Données** en bas de la page dans l’entrepôt de données. Dans le volet **Explorateur**, sélectionnez la table **DimProduct** et vérifiez que les trois lignes ont été ajoutées à la table.
7. Sous l’onglet du menu **Accueil**, utilisez le bouton **Nouvelle requête SQL** pour créer une requête. Ensuite, copiez et collez le code Transact-SQL depuis [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt) dans le nouveau volet de requête.
<!-- I had to remove the GO command in this query as well -->
8. Exécutez la requête, qui crée un schéma d’entrepôt de données simple et charge des données. L’exécution du script doit prendre environ 30 secondes.
9. Utilisez le bouton **Actualiser** dans la barre d’outils pour actualiser la vue. Ensuite, dans le volet **Explorateur**, vérifiez que le schéma **dbo** dans l’entrepôt de données contient maintenant les quatre tables suivantes :
    - **DimCustomer**
    - **DimDate**
    - **DimProduct**
    - **FactSalesOrder**

    > **Conseil** : Si le chargement du schéma prend un certain temps, actualisez simplement la page du navigateur.

## Définir un modèle de données

Un entrepôt de données relationnelles se compose généralement de tables de *faits* et de *dimension*. Les tables de faits contiennent des mesures numériques que vous pouvez agréger pour analyser les performances de l’entreprise (par exemple le chiffre d’affaires), et les tables de dimension contiennent des attributs des entités par lesquelles vous pouvez agréger les données (par exemple produit, client ou temps). Dans un entrepôt de données Microsoft Fabric, vous pouvez utiliser ces clés pour définir un modèle de données qui encapsule les relations entre les tables.

1. En bas de la page de l’entrepôt de données, sélectionnez l’onglet **Modèle**.
2. Dans le volet du modèle, réorganisez les tables de votre entrepôt de données afin que la table **FactSalesOrder** se trouve au milieu, comme ceci :

    ![Capture d’écran de la page du modèle de l’entrepôt de données.](./Images/model-dw.png)

3. Faites glisser le champ **ProductKey** de la table **FactSalesOrder** et déposez-le sur le champ **ProductKey** de la table **DimProduct**. Vérifiez ensuite les détails des relations suivantes :
    - **Table 1** : FactSalesOrder
    - **Colonne** : ProductKey
    - **Table 2** : DimProduct
    - **Colonne** : ProductKey
    - **Cardinalité** : Plusieurs-à-un (*:1)
    - **Direction du filtre croisé** : Unique
    - **Rendre cette relation active** : Sélectionné
    - **Intégrité référentielle supposée** : Non sélectionné

4. Répétez le processus pour créer des relations plusieurs-à-un entre les tables suivantes :
    - **FactOrderSales.CustomerKey** &#8594; **DimCustomer.CustomerKey**
    - **FactOrderSales.SalesOrderDateKey** &#8594; **DimDate.DateKey**

    Quand toutes les relations ont été définies, le modèle doit se présenter comme ceci :

    ![Capture d’écran du modèle avec des relations.](./Images/dw-relationships.png)

## Interroger les tables de l’entrepôt de données

Comme l’entrepôt de données est une base de données relationnelle, vous pouvez utiliser SQL pour interroger ses tables.

### Interroger les tables de faits et de dimension

La plupart des requêtes dans un entrepôt de données relationnelles impliquent l’agrégation et le regroupement de données (en utilisant des fonctions d’agrégation et des clauses GROUP BY) entre des tables liées (en utilisant des clauses JOIN).

1. Créez une requête SQL et exécutez le code suivant :

    ```sql
   SELECT  d.[Year] AS CalendarYear,
            d.[Month] AS MonthOfYear,
            d.MonthName AS MonthName,
           SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   GROUP BY d.[Year], d.[Month], d.MonthName
   ORDER BY CalendarYear, MonthOfYear;
    ```

    Notez que les attributs de la dimension temps vous permettent d’agréger les mesures de la table de faits à plusieurs niveaux hiérarchiques, dans le cas présent année et mois. C’est un modèle courant dans les entrepôts de données.

2. Modifiez la requête comme suit pour ajouter une deuxième dimension à l’agrégation.

    ```sql
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

3. Exécutez la requête modifiée et examinez les résultats, qui incluent désormais le chiffre d’affaires agrégé par année, mois et région de vente.

## Créer une vue

Un entrepôt de données dans Microsoft Fabric offre la plupart des mêmes fonctionnalités que celles que vous pouvez utiliser dans les bases de données relationnelles. Par exemple, vous pouvez créer des objets de base de données comme des *vues* et des *procédures stockées* pour encapsuler la logique SQL.

1. Modifiez comme suit la requête que vous avez créée précédemment pour créer une vue (notez que vous devez supprimer la clause ORDER BY pour créer une vue).

    ```sql
   CREATE VIEW vSalesByRegion
   AS
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion;
    ```

2. Exécutez la requête pour créer la vue. Actualisez ensuite le schéma de l’entrepôt de données et vérifiez que la nouvelle vue est listée dans le volet **Explorateur**.
3. Créez une requête SQL et exécutez l’instruction SELECT suivante :

    ```SQL
   SELECT CalendarYear, MonthName, SalesRegion, SalesRevenue
   FROM vSalesByRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

### Créer une requête visuelle

Au lieu d’écrire du code SQL, vous pouvez utiliser le concepteur de requêtes graphique pour interroger les tables de votre entrepôt de données. Cette expérience est similaire à Power Query en ligne, où vous pouvez créer des étapes de transformation de données sans code. Pour les tâches plus complexes, vous pouvez utiliser le langage M (Mashup) de Power Query.

1. Dans le menu **Accueil**, sélectionnez **Nouvelle requête visuelle**.

1. Faites glisser **FactSalesOrder** sur le **canevas**. Notez qu’un aperçu de la table s’affiche dans le volet **Aperçu** en dessous.

1. Faites glisser **DimProduct** sur le **canevas**. Nous avons maintenant deux tables dans notre requête.

2. Utilisez le bouton **(+)** sur la table **FactSalesOrder** qui est sur le canevas pour **Fusionner des requêtes**.
![Capture d’écran du canevas avec la table FactSalesOrder sélectionnée.](./Images/visual-query-merge.png)

1. Dans la fenêtre **Fusionner des requêtes**, sélectionnez **DimProduct** comme table appropriée pour la fusion. Sélectionnez **ProductKey** dans les deux requêtes, conservez le type de jointure **Externe gauche** par défaut, puis cliquez sur **OK**.

2. Dans l’**Aperçu**, notez que la nouvelle colonne **DimProduct** a été ajoutée à la table FactSalesOrder. Développez la colonne en cliquant sur la flèche à droite du nom de la colonne. Sélectionnez **ProductName**, puis cliquez sur **OK**.

    ![Capture d’écran du volet d’aperçu avec la colonne DimProduct développée, avec ProductName sélectionné.](./Images/visual-query-preview.png)

1. Si vous souhaitez examiner les données pour un seul produit, à la demande d’un responsable, vous pouvez maintenant utiliser la colonne **ProductName** pour filtrer les données dans la requête. Filtrez la colonne **ProductName** pour examiner seulement les données de **Cable Lock**.

1. À partir de là, vous pouvez analyser les résultats de cette requête en sélectionnant **Visualiser les résultats** ou **Ouvrir dans Excel**. Vous pouvez maintenant voir exactement ce que le responsable demandait : nous n’avons donc pas besoin d’analyser les résultats plus en détail.

### Visualiser vos données

Vous pouvez facilement visualiser les données dans une seule requête ou dans votre entrepôt de données. Avant de visualiser, masquez les colonnes et/ou les tables qui ne sont pas conviviales pour les concepteurs de rapports.

1. Dans le volet **Explorateur**, sélectionnez la vue **Modèle**. 

1. Masquez les colonnes suivantes dans vos tables de faits et de dimension qui ne sont pas nécessaires pour créer un rapport. Notez que cela ne supprime pas les colonnes du modèle, mais les masque simplement de la vue sur le canevas de rapport.
   1. FactSalesOrder
      - **SalesOrderDateKey**
      - **CustomerKey**
      - **ProductKey**
   1. DimCustomer
      - **CustomerKey**
      - **CustomerAltKey**
   1. DimDate
      - **DateKey**
      - **DateAltKey**
   1. DimProduct
      - **ProductKey**
      - **ProductAltKey** 

1. Vous êtes maintenant prêt à créer un rapport et à rendre ce jeu de données disponible pour d’autres utilisateurs. Dans le menu Accueil, sélectionnez **Nouveau rapport**. Ceci va ouvrir une nouvelle fenêtre, où vous pouvez créer un rapport Power BI.

1. Dans le volet **Données**, développez **FactSalesOrder**. Notez que les colonnes que vous avez masquées ne sont plus visibles. 

1. Sélectionnez **SalesTotal**. Ceci va ajouter la colonne au **Canevas de rapport**. Comme la colonne est une valeur numérique, le visuel par défaut est un **histogramme**.
1. Vérifiez que l’histogramme sur le canevas est actif (il doit avoir une bordure grise et des poignées), puis sélectionnez **Category** dans la table **DimProduct** pour ajouter une catégorie à votre histogramme.
1. Dans le volet **Visualisations**, changez le type de graphique d’histogramme en **graphique à barres groupées**. Ensuite, redimensionnez le graphique si nécessaire pour que les catégories soient lisibles.

    ![Capture d’écran du volet Visualisations avec le graphique à barres sélectionné.](./Images/visualizations-pane.png)

1. Dans le volet **Visualisations**, sélectionnez l’onglet **Mettre en forme votre visuel** et, dans le sous-onglet **Général**, dans la section **Titre**, remplacez **Texte**par **Total Sales by Category** (Ventes totales par catégorie).

1. Dans le menu **Fichier**, sélectionnez **Enregistrer**. Enregistrez ensuite le rapport sous le nom **Rapport des ventes** dans l’espace de travail que vous avez créé précédemment.

1. Dans le hub de menus à gauche, revenez à l’espace de travail. Notez que vous avez maintenant trois éléments enregistrés dans votre espace de travail : votre entrepôt de données, son jeu de données par défaut et le rapport que vous avez créé.

    ![Capture d’écran de l’espace de travail avec les trois éléments listés.](./Images/workspace-items.png)

## Nettoyer les ressources

Dans cet exercice, vous avez créé un entrepôt de données qui contient plusieurs tables. Vous avez utilisé SQL pour insérer des données dans les tables et les interroger. Vous avez aussi utilisé l’outil de requête visuelle. Enfin, vous avez amélioré le modèle de données pour le jeu de données par défaut de l’entrepôt de données et vous l’avez utilisé comme source pour un rapport.

Si vous avez terminé d’explorer votre entrepôt de données, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
