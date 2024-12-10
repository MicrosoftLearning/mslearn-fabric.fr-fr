---
lab:
  title: Bien démarrer avec Eventstream dans Microsoft Fabric
  module: Get started with Eventstream in Microsoft Fabric
---
# Bien démarrer avec Eventstream dans Microsoft Fabric

Eventstream est une fonctionnalité de Microsoft Fabric qui capture, transforme et route les événements en temps réel vers différentes destinations avec une expérience sans code. Vous pouvez ajouter des sources de données d’événement, des destinations de routage, ainsi que le processeur d’événements lorsque la transformation est nécessaire, au flux d’événements. EventStore de Microsoft Fabric est une option de supervision qui tient à jour les événements du cluster et fournit un moyen de comprendre l’état de votre cluster ou de votre charge de travail à un moment donné. Vous pouvez interroger le service EventStore à propos des événements qui sont disponibles pour chaque entité et type d’entité du cluster. Cela signifie que vous pouvez rechercher des événements à différents niveaux, tels que les clusters, les nœuds, les applications, les services, les partitions et les réplicas de partition. Le service EventStore a également la possibilité de mettre en corrélation les événements du cluster. En examinant les événements écrits en même temps à partir de différentes entités dont les conséquences peuvent être mutuelles, le service EventStore peut lier ces événements pour aider à identifier les causes des activités du cluster. L’agrégation et la collecte d’événements avec EventFlow constituent une autre option pour superviser et diagnostiquer des clusters Microsoft Fabric.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une [licence d’essai Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

   ![Capture d’écran d’un espace de travail vide dans Power BI](./Images/new-workspace.png)
5. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI**, puis passez à l’expérience **Real-Time Intelligence**.

## Créer un organisateur d’événements Real-Time Intelligence

1. Sur la page d’accueil de Real-Time Intelligence dans Microsoft Fabric, créez un **Eventhouse** en lui donnant un nom unique de votre choix.
1. Fermez toutes les invites ou conseils affichés jusqu’à ce que le nouvel eventhouse vide soit visible :

    ![Capture d’écran d’un nouvel eventhouse](./Images/create-eventhouse.png)

## Créer une base de données KQL

1. Dans le tableau de bord **Organisateur d’événements Real-Time Intelligence**, cochez la case **Base de données KQL +**.
1. Vous pouvez créer une **Nouvelle base de données (par défaut)**, ou créer un **Nouveau raccourci de base de données (abonné)**.

    >**Remarque :** la fonctionnalité de base de données d’abonné vous permet d’attacher une base de données située dans un cluster différent à votre cluster Azure Data Explorer. La base de données d’abonné est jointe en mode lecture seule, ce qui permet d’afficher les données et d’exécuter des requêtes sur les données ingérées dans la base de données du responsable. La base de données d’abonné synchronise les modifications apportées aux bases de données de responsable. En raison de la synchronisation, il existe un décalage de données de quelques secondes à quelques minutes au niveau de la disponibilité des données. La durée du décalage dépend de la taille globale des métadonnées de la base de données du responsable. Les bases de données de responsable et d’abonné utilisent le même compte de stockage pour extraire les données. Le stockage appartient à la base de données de responsable. La base de données d’abonné affiche les données sans qu’il soit nécessaire de les ingérer. Étant donné que la base de données jointe est une base de données en lecture seule, les données, les tables et les stratégies de la base de données ne peuvent pas être modifiées, à l’exception de la stratégie de mise en cache, des principaux et des autorisations.

1. Créez une base de données et nommez-la `Eventhouse-DB`.

## Créer un flux d’événements

1. Dans la page principale de votre base de données KQL, sélectionnez **Obtenir des données**.
2. Pour la source de données, sélectionnez **Eventstream** > **Nouvel evenstream**. Nommez l’eventstream `bicycle-data`.

    La création de votre flux d’événements dans l’espace de travail ne prend que quelques instants. Une fois l’opération effectuée, vous êtes automatiquement redirigé vers l’éditeur principal, prêt à commencer à intégrer des sources dans votre flux d’événements.

    ![Capture d’écran d’un nouvel eventstream.](./Images//name-eventstream.png)

## Établir une source d’Eventstream

1. Dans le canevas d’eventstream, sélectionnez **Utiliser des exemples de données**.
2. Nommez la source `Bicycles` et sélectionnez **Vélos** comme exemple de données.

    Votre flux sera mappé et sera automatiquement affiché sur le **canevas d’eventstream**.

   ![Passer en revue le canevas d’eventstream](./Images/real-time-intelligence-eventstream-sourced.png)

## Ajouter une destination

1. Dans la liste déroulante **Transformer des événements ou ajouter une destination**, sélectionnez **Eventhouse**.
1. Dans le volet **Eventhouse**, configurez les options suivantes.
   - **Mode d’ingestion des données :** traitement des événements avant l’ingestion
   - **Nom de la destination :**`Bicycle-database`
   - **Espace de travail :***sélectionnez l’espace de travail que vous avez créé au début de cet exercice.*
   - **Eventhouse** : *sélectionnez votre eventhouse*
   - **Base de données KQL :** Eventhouse-DB
   - **Table de destination :** créez une table nommée `bike-count`
   - **Format de données d’entrée :** JSON

   ![Eventstream de base de données KQL avec modes d’ingestion](./Images/kql-database-event-processing-before-ingestion.png)

1. Dans le volet **Eventhouse**, sélectionnez **Enregistrer**. 
1. Dans la barre d’outils, sélectionnez **Publier**.
1. Attendez environ une minute que la destination des données devienne active.

## Afficher les données capturées

L’eventstream que vous avez créé prend les données de l’exemple de source de données de vélos et les charge dans la base de données dans votre eventhouse. Vous pouvez afficher les données capturées en interrogeant la table dans la base de données.

1. Dans la barre de menus de gauche, sélectionnez votre base de données **Eventhouse-DB**.
1. Dans le menu **...** de la base de données KQL **Eventhouse-DB**, sélectionnez **Interroger des données**.
1. Dans le volet de requête, modifiez le premier exemple de requête, comme illustré ici :

    ```kql
    ['bike-count']
    | take 100
    ```

1. Sélectionnez le code de requête et exécutez-le pour afficher 100 lignes de données depuis la table.

    ![Capture d’écran d’une requête KQL.](./Images/kql-query.png)

## Transformer des données d’événements

Les données que vous avez capturées ne sont pas modifiées à partir de la source. Dans de nombreux scénarios, vous pouvez transformer les données dans l’eventstream avant de les charger dans une destination.

1. Dans la barre de menus de gauche, sélectionnez l’eventstream **Bike-data**.
1. Dans la barre d’outils, sélectionnez **Modifier** pour modifier l’eventstream.

1. Dans le menu **Transformer des événements**, sélectionnez **Regrouper par** pour ajouter un nouveau nœud **Regrouper par** à l’eventstream.
1. Faites glisser une connexion de la sortie du nœud **Bicycle-data** vers l’entrée du nouveau nœud **Regrouper par**, puis utilisez l’icône *crayon* dans le nœud **Regrouper par** pour la modifier.

   ![Ajoutez Regrouper par à l’événement de transformation.](./Images/eventstream-add-aggregates.png)

1. Configurez les propriétés de la section de paramètres **Regrouper par** :
    - **Nom de l’opération :** GroupByStreet
    - **Type d’agrégat :***sélectionnez* Sum
    - **Champ :***sélectionnez* No_Bikes *Sélectionnez ensuite **Ajouter** pour créer la fonction* SUM_No_Bikes
    - **Regrouper les agrégations par (facultatif) :** Street
    - **Fenêtre temporelle** : bascule
    - **Durée** : 5 secondes
    - **Décalage** : 0 seconde

    > **Remarque** : cette configuration entraîne le calcul par l’eventstream du nombre total de vélos dans chaque rue toutes les 5 secondes.
      
1. Enregistrez la configuration et revenez au canevas d’Eventstream, où une erreur est indiquée (car vous devez stocker la sortie de la transformation Regrouper par quelque part !).

1. Utilisez l’icône **+** à droite du nœud **GroupByStreet** pour ajouter un nouveau nœud **Eventhouse**.
1. Configurez le nouveau nœud d’eventhouse avec les options suivantes :
   - **Mode d’ingestion des données :** traitement des événements avant l’ingestion
   - **Nom de la destination :**`Bicycle-database`
   - **Espace de travail :***sélectionnez l’espace de travail que vous avez créé au début de cet exercice.*
   - **Eventhouse** : *sélectionnez votre eventhouse*
   - **Base de données KQL :** Eventhouse-DB
   - **Table de destination :** créez une table nommée `bikes-by-street`
   - **Format de données d’entrée :** JSON

   ![Capture d’écran d’une table pour les données regroupées.](./Images/group-by-table.png)

1. Dans le volet **Eventhouse**, sélectionnez **Enregistrer**. 
1. Dans la barre d’outils, sélectionnez **Publier**.
1. Attendez environ une minute que les modifications deviennent actives.

## Afficher les données transformées

Vous pouvez maintenant afficher les données de vélos qui ont été transformées et chargées dans une table par votre eventstream.

1. Dans la barre de menus de gauche, sélectionnez votre base de données **Eventhouse-DB**.
1. Dans le menu **...** de la base de données KQL **Eventhouse-DB**, sélectionnez **Interroger des données**.
1. Dans le volet de requête, modifiez un exemple de requête, comme illustré ici :

    ```kql
    ['bikes-by-street']
    | take 100
    ```

1. Sélectionnez le code de requête et exécutez-le pour afficher les 100 premières lignes de la table.

    ![Capture d’écran d’une requête KQL.](./Images/kql-group-query.png)

    > **Conseil** : vous pouvez également interroger la table à l’aide de la syntaxe SQL. Par exemple, essayez la requête `SELECT TOP 100 * FROM bikes-by-street`.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un eventhouse et rempli des tables dans sa base de données à l’aide d’un eventstream.

Lorsque vous avez terminé d’explorer votre base de données KQL, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail.
2. Dans la barre d’outils, sélectionnez **Paramètres de l’espaces de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
.
