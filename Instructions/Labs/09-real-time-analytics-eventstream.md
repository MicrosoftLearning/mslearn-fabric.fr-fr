---
lab:
  title: Bien démarrer avec Real-Time Analytics dans Microsoft Fabric
  module: Get started with real-time analytics in Microsoft Fabric
---
# Bien démarrer avec EventStream dans Real-Time Analytics (RTA)

Les flux d’événements sont une fonctionnalité de Microsoft Fabric qui capture, transforme et route les événements en temps réel vers différentes destinations avec une expérience sans code. Lorsque vous créez un élément EventStream dans le portail, il s’agit d’une instance de flux d’événements Fabric (également appelé « eventstream »). Vous pouvez ajouter des sources de données d’événement, des destinations de routage, ainsi que le processeur d’événements lorsque la transformation est nécessaire, au flux d’événements. EventStore d’Azure Service Fabric est une option de supervision qui tient à jour les événements du cluster et fournit un moyen de comprendre l’état de votre cluster ou de vos charges de travail à un moment donné. Vous pouvez interroger le service EventStore à propos des événements qui sont disponibles pour chaque entité et type d’entité du cluster. Cela signifie que vous pouvez rechercher des événements à différents niveaux, tels que le cluster, les nœuds, les applications, les services, les partitions et les réplicas de partition. Le service EventStore a également la possibilité de mettre en corrélation les événements du cluster. En examinant les événements écrits en même temps à partir de différentes entités dont les conséquences peuvent être mutuelles, le service EventStore est en mesure de lier ces événements pour aider à identifier les causes des activités du cluster. L’agrégation et la collecte d’événements à l’aide d’EventFlow constituent une autre option pour la supervision et le diagnostic de clusters Azure Service Fabric.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une licence Microsoft Fabric pour effectuer cet exercice. Pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite, consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial). Pour cela, vous avez besoin d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com`, puis sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Quand votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

   ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)
5. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI**, puis passez à l’expérience **Real-Time Analytics**.

## Scénario

Avec les flux d’événements Microsoft Fabric, vous pouvez gérer facilement vos données d’événement dans un même emplacement. Vous pouvez collecter, transformer et envoyer des données d’événement en temps réel dans différentes destinations au format souhaité. Vous pouvez également connecter vos flux d’événements à Azure Event Hubs, à la base de données KQL et à Lakehouse sans aucune difficulté.

Ce labo est basé sur des exemples de données de streaming appelées « Stock Market Data » (Données boursières). L’exemple de données Stock Market est un jeu de données d’une bourse avec une colonne de schéma prédéfini telle que l’heure, le symbole, le prix, le volume, et plus encore. Vous allez utiliser cet exemple de données pour simuler des événements en temps réel des cours des actions et les analyser avec différentes destinations, telles que la base de données KQL.

Vous utilisez les fonctionnalités de streaming et de requête de Real-Time Analytics pour répondre à des questions clés sur les statistiques boursières, et vous avez la possibilité d’utiliser ses résultats pour créer des rapports Power BI. Dans ce scénario, nous allons tirer pleinement parti de l’Assistant au lieu de créer manuellement certains composants sans aide extérieure, comme la base de données KQL.

Dans ce tutoriel, vous allez apprendre à :

- Créer une base de données KQL
- Activer la copie de données vers OneLake
- Créer un flux d’événements
- Diffuser en streaming des données d’EventStream vers votre base de données KQL
- Explorer des données avec KQL et SQL

## Créer une base de données KQL

1. Dans **Real-Time Analytics**, cochez la case **Base de données KQL**.

![choisir Base de données KQL](./Images/select-kqldatabase.png)

2. Vous êtes invité à donner un **Nom** à la base de données KQL

![donner un nom à la base de données KQL](./Images/name-kqldatabase.png)

3. Donnez à la base de données KQL un nom dont vous vous souviendrez, par exemple **MyStockData**, puis appuyez sur **Créer**.

Nous allons ensuite activer la disponibilité dans OneLake

1. Dans le panneau **Détails de la base de données**, sélectionnez l’icône en forme de crayon.

![activer OneLake](./Images/enable-onelake-availability.png)

2. Veillez à basculer le bouton sur **Actif**, puis sélectionnez **Terminé**.

![activer le bouton bascule Onelake](./Images/enable-onelake-toggle.png)

## Créer un flux d’événements

1. Dans la barre de menus, sélectionnez **Real-Time Analytics** (l’icône ressemble au ![logo RTA](./Images/rta_logo.png)).
2. Sous **Nouveau**, sélectionnez **Flux d’événements (préversion)**

![choisir Flux d’événements](./Images/select-eventstream.png)

3. Vous êtes invité à **nommer le flux d’événements**

![nommer le flux d’événements](./Images/name-eventstream.png)

4. Donnez au flux d’événements un nom dont vous vous souviendrez, par exemple ***MyStockES**, puis appuyez sur le bouton **Créer**.

## Données du flux d’événements - Source

1. Dans le canevas du flux d’événements, sélectionnez **Nouvelle source** dans la liste déroulante, puis sélectionnez **Exemples de données**.

![canevas du flux d’événements](./Images/real-time-analytics-canvas.png)

2. Entrez les valeurs de vos exemples de données, comme indiqué dans le tableau suivant, puis sélectionnez **Ajouter et configurer**.

| Champ       | Valeur recommandée |
| ----------- | ----------------- |
| Nom de la source | StockData         |
| Exemple de données | Marché boursier      |

## Données du flux d’événements - Destination

1. Dans le canevas du flux d’événements, sélectionnez **Nouvelle destination**, puis sélectionnez **Base de données KQL**.

![destination du flux d’événements](./Images/new-kql-destination.png)

2. Dans la configuration de la base de données KQL, utilisez le tableau suivant pour effectuer la configuration.

| Champ            | Valeur recommandée                              |
| ---------------- | ---------------------------------------------- |
| Nom de la destination | MyStockData                                    |
| Espace de travail        | Espace de travail dans lequel vous avez créé une base de données KQL |
| Base de données KQL     | MyStockData                                    |

3. Sélectionnez **Ajouter et configurer**.

## Configurer l’ingestion des données

1. Dans la page de boîte de dialogue **Ingérer des données**, sélectionnez **Nouvelle table**, puis entrez MyStockData.

![insérer des données boursières](./Images/ingest-stream-data-to-kql.png)

2. Sélectionnez **Suivant : Source**.
3. Dans la page **Source**, confirmez le **Nom de la connexion de données**, puis sélectionnez **Suivant : Schéma**.

![nom de source de données](./Images/ingest-data.png)

4. Les données entrantes n’étant pas compressées pour les exemples de données, conservez le type de compression comme Non compressé.
5. Dans la liste déroulante **Format des données**, sélectionnez **JSON**.

![Passer au format JSON](./Images/injest-as-json.png)

6. Après cela, il peut être nécessaire de remplacer certains ou tous les types de données de votre flux entrant par votre ou vos tables de destination.
7. Pour accomplir cette tâche, sélectionnez la **flèche vers le bas > Changer le type de données**. Vérifiez ensuite que les colonnes reflètent le type de données correct :

![changer les types de données](./Images/change-data-type-in-es.png)

8. Quand vous avez terminé, sélectionnez **Suivant : Résumé**.

Attendez que toutes les étapes soient marquées par des coches vertes. Vous devez voir le titre de la page **Ingestion continue à partir d’un flux d’événements établie**. Après cela, sélectionnez **Fermer** pour revenir à votre page de flux d’événements.

> !Remarque : Il peut être nécessaire d’actualiser la page pour afficher votre table une fois que la connexion de flux d’événements a été créée et établie

## Requêtes KQL

Une requête KQL (Kusto Query Language, langage de requête Kusto) est une requête en lecture seule de traitement de données et de retour de résultats. La demande est formulée en texte brut en utilisant un modèle de flux de données facile à lire, à créer et à automatiser. Les requêtes s’exécutent toujours dans le contexte d’une table ou d’une base de données particulière. Une requête se compose au minimum d’une référence de données source et d’un ou de plusieurs opérateurs de requête appliqués de manière séquentielle, indiqués visuellement par l’utilisation d’une barre verticale (|) pour délimiter les opérateurs. Pour plus d’informations sur le langage de requête, consultez la [vue d’ensemble du langage de requête Kusto (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext).

> ! Remarque : L’éditeur KQL est fourni avec la mise en surbrillance de la syntaxe et d’Intellisense, ce qui vous permet d’acquérir rapidement des connaissances sur le langage de requête Kusto (KQL).

1. Accédez à la base de données KQL que vous venez de créer et d’activer, nommée ***MyStockData***.
2. Dans l’arborescence Données, sélectionnez le menu Plus [...] dans la table MyStockData. Sélectionnez ensuite Interroger la table > Afficher 100 enregistrements.

![Jeu de requêtes KQL](./Images/kql-query-sample.png)

3. L’exemple de requête s’ouvre dans le volet **Explorer vos données** avec le contexte de table déjà renseigné. Cette première requête utilise l’opérateur take pour retourner un nombre restreint d’enregistrements, et est utile pour obtenir un premier aperçu de la structure des données et des valeurs possibles. Les exemples de requêtes renseignées automatiquement sont exécutés automatiquement. Les résultats de la requête s’affichent dans le volet des résultats.

![Résultats de requête SQL](./Images/kql-query-results.png)

4. Retournez dans l’arborescence de données pour sélectionner la requête suivante, qui utilise l’opérateur where et l’opérateur between pour retourner les enregistrements ingérés au cours des dernières 24 heures.

![Résultats de requête KQL des dernières 24 heures](./Images/kql-query-results-last24.png)

> !Remarque : Notez que les volumes des données de streaming dépassent les limites de requête. Ce comportement peut varier en fonction de la quantité de données diffusées en streaming dans votre base de données.
> Vous pouvez continuer à naviguer à l’aide des fonctions de requête intégrées pour vous familiariser avec vos données.

## Exemples de requêtes SQL

L’éditeur de requête prend en charge l’utilisation de T-SQL en plus de son langage KQL (Kusto Query Language) de requête principale. T-SQL peut être utile pour les outils qui ne peuvent pas utiliser KQL. Pour plus d’informations, consultez [Interroger des données à l’aide de T-SQL](https://learn.microsoft.com/en-us/azure/data-explorer/t-sql).

1. De retour dans l’arborescence Données, sélectionnez le **menu Plus** [...] dans la table MyStockData. Sélectionnez **Interroger la table > SQL > Afficher 100 enregistrements**.

![exemple de requête SQL](./Images/sql-query-sample.png)

2. Placez votre curseur n’importe où dans la requête, puis sélectionnez **Exécuter** ou appuyez sur **Maj + Entrée**.

![Résultats de la requête SQL](./Images/sql-query-results.png)

Vous pouvez continuer à naviguer à l’aide des fonctions intégrées et à vous familiariser avec les données à l’aide de SQL ou de KQL. Cela met fin à la leçon.

## Nettoyer les ressources

1. Dans cet exercice, vous avez créé une base de données KQL et configuré le streaming continu avec EventStream. Après cela, vous avez interrogé les données à l’aide de KQL et de SQL.
2. Si vous avez terminé d’explorer votre base de données KQL, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice. 
3. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail.
4. Dans le menu ... de la barre d’outils, sélectionnez Paramètres de l’espace de travail.
5. Dans la section Autre, sélectionnez Supprimer cet espace de travail.