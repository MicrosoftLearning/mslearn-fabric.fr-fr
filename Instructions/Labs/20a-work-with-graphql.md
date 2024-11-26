---
lab:
  title: "Utiliser l’API pour GraphQL dans Microsoft\_Fabric"
  module: Get started with GraphQL in Microsoft Fabric
---

# Utiliser l’API pour GraphQL dans Microsoft Fabric

L’API Microsoft Fabric pour GraphQL est une couche d’accès aux données qui permet d’interroger rapidement et efficacement plusieurs sources de données à l’aide d’une technologie API largement adoptée et familière. L’API vous permet d’extraire les spécificités des sources de données principales afin de pouvoir vous concentrer sur la logique de votre application et fournir toutes les données dont un client a besoin dans un seul appel. GraphQL utilise un langage de requête simple et des jeux de résultats facilement manipulés, ce qui réduit le temps nécessaire aux applications pour accéder à vos données dans Fabric.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’un [essai gratuit Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Dans la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric`.
1. Dans la barre de menus à gauche, sélectionnez **Nouvel espace de travail**.
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer une base de données SQL avec des échantillons de données

Maintenant que vous disposez d’un espace de travail, vous pouvez créer une base de données SQL.

1. Sur le portail Fabric, sélectionnez **+ Nouvel élément** dans le volet de gauche.
1. Accédez à la section **Stocker des données**, puis sélectionnez **Base de données SQL**.
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

1. Fermez tous les onglets de requête.

## Créer une API pour GraphQL

Tout d’abord, vous allez configurer un point de terminaison GraphQL pour exposer les données de commande. Ce point de terminaison vous permet d’interroger les commandes en fonction de différents paramètres, tels que la date, le client et le produit.

1. Sur le portail Fabric, accédez à votre espace de travail, puis sélectionnez **+ Nouvel élément**.
1. Accédez à la section **Développer des données**, puis sélectionnez **API pour GraphQL**.
1. Indiquez un nom, puis sélectionnez **Créer**.
1. Dans la page principale de votre API pour GraphQL, sélectionnez **Sélectionner une source de données**.
1. Si vous êtes invité à choisir une option de connectivité, sélectionnez **Se connecter aux sources de données Fabric avec l’authentification unique (SSO)**.
1. Dans la page **Choisir les données que vous souhaitez connecter**, sélectionnez la base de données `AdventureWorksLT` précédemment créée.
1. Sélectionnez **Connecter**.
1. Dans la page **Choisir des données**, sélectionnez la table `SalesLT.Product`. 
1. Affichez un aperçu des données, puis sélectionnez **Charger**.
1. Sélectionnez **Copier le point de terminaison** et notez le lien d’URL publique. Nous n’avons pas besoin de cela, mais c’est là que vous allez copier votre adresse API.

## Désactiver les mutations

Maintenant que notre API est créée, nous voulons uniquement exposer les données des ventes pour les opérations de lecture dans ce scénario.

1. Dans l’**Explorateur de schémas** de votre API pour GraphQL, développez **Mutations**.
1. Sélectionnez **...** (points de suspension) en regard de chaque mutation, puis sélectionnez **Désactiver**.

Cela empêche toute modification ou mise à jour des données via l’API. Cela signifie que les données seront en lecture seule et que les utilisateurs pourront uniquement afficher ou interroger les données, mais pas y apporter de modifications.

## Interroger des données à l’aide de GraphQL

Nous allons maintenant interroger les données à l’aide de GraphQL pour rechercher tous les produits dont les noms commencent par *« HL Road Frame ».*

1. Dans l’éditeur de requête GraphQL, entrez et exécutez la requête suivante.

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

Dans cette requête, les produits sont le type principal, et il inclut des champs pour `ProductModelID`, `Name`, `ListPrice`, `Color`, `Size` et `ModifiedDate`. Cette requête renvoit une liste de produits dont les noms commencent par *« HL Road Frame ».*

> **Informations supplémentaires** : voir [Qu’est-ce que l’API Microsoft Fabric pour GraphQL ?](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview) dans la documentation Microsoft Fabric pour en savoir plus sur les autres composants disponibles sur la plateforme.

Dans cet exercice, vous avez créé, interrogé et exposé des données à partir d’une base de données SQL à l’aide de GraphQL dans Microsoft Fabric.

## Nettoyer les ressources

Si vous avez terminé d’explorer votre base de données, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.

