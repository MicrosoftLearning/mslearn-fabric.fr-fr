---
lab:
  title: Interroger des données dans une base de données KQL
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# Bien démarrer avec l’interrogation d’une base de données Kusto dans Microsoft Fabric
KQL Queryset est un outil qui vous permet d’exécuter des requêtes, mais également de modifier et d’afficher les résultats des requêtes à partir d’une base de données KQL. Vous pouvez lier chaque onglet dans KQL Queryset à une base de données KQL différente et enregistrer vos requêtes pour une utilisation ultérieure ou les partager avec d’autres personnes pour l’analyse des données. Vous pouvez également basculer la base de données KQL pour n’importe quel onglet, ce qui vous permet de comparer les résultats de la requête à partir de diverses sources de données.

Pour créer des requêtes, KQL Queryset utilise le langage Kusto Query qui est compatible avec de nombreuses fonctions SQL. Pour en savoir plus sur le [langage kusto query (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext), 

Ce labo prend environ **25** minutes.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)

Dans ce labo, vous allez utiliser l’Analyse en temps réel (RTA) dans Fabric pour créer une base de données KQL à partir d’un exemple d’eventstream. L’Analyse en temps réel fournit facilement un exemple de jeu de données à utiliser pour explorer les fonctionnalités de l’analyse en temps réel (RTA). Vous allez utiliser cet exemple de données pour créer des requêtes et des ensembles de requêtes KQL | SQL qui analysent certaines données en temps réel et permettent une utilisation supplémentaire dans les processus en aval.

## Créer une base de données KQL

1. Dans **Real-Time Analytics**, cochez la case **Base de données KQL**.

   ![Image du choix de la base de données kql](./Images/select-kqldatabase.png)

2. Vous êtes invité à donner un **Nom** à la base de données KQL

   ![Image du nom de la base de données kql](./Images/name-kqldatabase.png)

3. Donnez à la base de données KQL un nom dont vous vous souviendrez, par exemple **MyStockData**, puis appuyez sur **Créer**.

4. Dans le panneau **Détails de la base de données**, sélectionnez l’icône de crayon pour activer la disponibilité dans OneLake.

   ![Image de l’activation de onelake](./Images/enable-onelake-availability.png)

5. Sélectionnez la zone **exemple de données** dans les options ***Commencer par obtenir des données***.
 
   ![Image des options de sélection avec exemple de données mis en évidence](./Images/load-sample-data.png)

6. Choisissez la zone **Analytique des métriques** dans les options d’exemples de données.

   ![Image du choix des données analytiques pour le labo](./Images/create-sample-data.png)

7. Une fois les données chargées, vérifiez qu’elles sont chargées dans la base de données KQL. Pour ce faire, sélectionnez les points de suspension à droite de la table, accédez à **Interroger la table** et sélectionnez **Afficher 100 enregistrements**.

    ![Image de la sélection des 100 premiers fichiers de la table RawServerMetrics](./Images/rawservermetrics-top-100.png)

> **REMARQUE** : La première fois que vous effectuez cette opération, l’allocation des ressources de calcul peut prendre plusieurs secondes.

## Scénario
Dans ce scénario, vous êtes analyste et vous êtes chargé d’interroger un exemple de jeu de données de métriques brutes d’une instance SQL Server hypothétique que vous allez implémenter à partir de l’environnement Fabric. Vous utilisez KQL et T-SQL pour interroger ces données et collecter des informations afin d’obtenir des insights instructifs sur les données.

