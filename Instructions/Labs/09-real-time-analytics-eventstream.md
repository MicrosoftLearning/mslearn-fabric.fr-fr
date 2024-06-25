---
lab:
  title: Bien démarrer avec Eventstream dans Microsoft Fabric
  module: Get started with Eventstream in Microsoft Fabric
---
# Bien démarrer avec Eventstream dans Real-Time Intelligence

Eventstream est une fonctionnalité de Microsoft Fabric qui capture, transforme et route les événements en temps réel vers différentes destinations avec une expérience sans code. Vous pouvez ajouter des sources de données d’événement, des destinations de routage, ainsi que le processeur d’événements lorsque la transformation est nécessaire, au flux d’événements. EventStore de Microsoft Fabric est une option de supervision qui tient à jour les événements du cluster et fournit un moyen de comprendre l’état de votre cluster ou de votre charge de travail à un moment donné. Vous pouvez interroger le service EventStore à propos des événements qui sont disponibles pour chaque entité et type d’entité du cluster. Cela signifie que vous pouvez rechercher des événements à différents niveaux, tels que les clusters, les nœuds, les applications, les services, les partitions et les réplicas de partition. Le service EventStore a également la possibilité de mettre en corrélation les événements du cluster. En examinant les événements écrits en même temps à partir de différentes entités dont les conséquences peuvent être mutuelles, le service EventStore peut lier ces événements pour aider à identifier les causes des activités du cluster. L’agrégation et la collecte d’événements avec EventFlow constituent une autre option pour superviser et diagnostiquer des clusters Microsoft Fabric.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une [licence d’essai Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

   ![Capture d’écran d’un espace de travail vide dans Power BI](./Images/new-workspace.png)
5. En bas à gauche du portail Power BI, sélectionnez l’icône **Power BI**, puis passez à l’expérience **Real-Time Intelligence**.

## Scénario

Avec les flux d’événements Fabric, vous pouvez gérer facilement vos données d’événement dans un même endroit. Vous pouvez collecter, transformer et envoyer des données d’événement en temps réel dans différentes destinations au format souhaité. Vous pouvez également connecter vos flux d’événements à Azure Event Hubs, à la base de données KQL et à Lakehouse sans aucune difficulté.

Ce labo est basé sur des exemples de données de streaming appelées « Stock Market Data » (Données boursières). L’exemple de données Stock Market est un jeu de données d’une bourse avec une colonne de schéma prédéfini telle que l’heure, le symbole, le prix, le volume, et plus encore. Vous allez utiliser cet exemple de données pour simuler des événements en temps réel des cours des actions et les analyser avec différentes destinations, telles que la base de données KQL.

Utilisez les fonctionnalités de streaming et d’interrogation de Real-Time Intelligence pour répondre à des questions clés sur les statistiques boursières. Dans ce scénario, nous allons tirer pleinement parti de l’Assistant au lieu de créer manuellement certains composants sans aide extérieure, comme la base de données KQL.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

- Créer un Eventhouse
- Créer une base de données KQL
- Activer la copie de données vers OneLake
- Créer un flux d’événements
- Streamer des données d’un flux d’événements vers votre base de données KQL
- Explorer des données avec KQL et SQL\

## Créer un organisateur d’événements Real-Time Intelligence

1. Sélectionnez l’option Real-Time Intelligence dans Microsoft Fabric.
1. Sélectionnez Organisateur d’événements dans la barre de menus, puis donnez un nom à votre organisateur d’événements.
    
    ![Image de la création d’un organisateur d’événements](./Images/create-eventhouse.png)

## Créer une base de données KQL

1. Dans le tableau de bord **Organisateur d’événements Real-Time Intelligence**, cochez la case **Base de données KQL +**.
1. Vous pouvez nommer votre base de données et sélectionner une **Nouvelle base de données (par défaut)**, ou créer un **Nouveau raccourci de base de données (abonné)**.
1. Sélectionnez **Créer**.

     >[!Note]
     > La fonctionnalité de base de données d’abonné vous permet d’attacher une base de données située dans un cluster différent à votre cluster Azure Data Explorer. La base de données d’abonné est jointe en mode lecture seule, ce qui permet d’afficher les données et d’exécuter des requêtes sur les données ingérées dans la base de données du responsable. La base de données d’abonné synchronise les modifications apportées aux bases de données de responsable. En raison de la synchronisation, il existe un décalage de données de quelques secondes à quelques minutes au niveau de la disponibilité des données. La durée du décalage dépend de la taille globale des métadonnées de la base de données du responsable. Les bases de données de responsable et d’abonné utilisent le même compte de stockage pour extraire les données. Le stockage appartient à la base de données de responsable. La base de données d’abonné affiche les données sans qu’il soit nécessaire de les ingérer. Étant donné que la base de données jointe est une base de données en lecture seule, les données, les tables et les stratégies de la base de données ne peuvent pas être modifiées, à l’exception de la stratégie de mise en cache, des principaux et des autorisations.

   ![Image du choix de la base de données kql](./Images/create-kql-database-eventhouse.png)

4. Vous êtes invité à donner un **Nom** à la base de données KQL

   ![Image du nom de la base de données kql](./Images/name-kqldatabase.png)

5. Donnez à la base de données KQL un nom dont vous vous souviendrez, par exemple **Eventhouse-HR**, puis appuyez sur **Créer**.

6. Dans le panneau **Détails de la base de données**, sélectionnez l’icône de crayon pour activer la disponibilité dans OneLake.

   [ ![Image de l’activation d’onlake](./Images/enable-onelake-availability.png) ](./Images/enable-onelake-availability-large.png)

7. Veillez à basculer le bouton sur **Actif**, puis sélectionnez **Terminé**.

   ![Image de l’activation de la touche bascule onelake](./Images/enable-onelake-toggle.png)

## Créer un flux d’événements

1. Dans la barre de menus, sélectionnez **Real-Time Intelligence** (l’icône ressemble au ![logo Real-Time Intelligence](./Images/rta_logo.png))
2. Sous **Nouveau**, sélectionnez **Flux d’événements (préversion)**

   ![Image du choix eventstream](./Images/select-eventstream.png)

3. Vous êtes invité à **nommer** votre flux d’événements. Donnez à l’EventStream un nom dont vous vous souviendrez, par exemple ***MyStockES**, puis cliquez sur le bouton **Créer**.

   ![Image du nom eventstream](./Images/name-eventstream.png)

4. **Nommez** le **Nouvel Eventstream**, sélectionnez l’option **Fonctionnalités améliorées (préversion)**, puis sélectionnez le bouton **Créer**.

     >[!Remarque :] La création de votre flux d’événements dans l’espace de travail ne prend que quelques instants. Une fois l’opération effectuée, vous êtes automatiquement redirigé vers l’éditeur principal, prêt à commencer à intégrer des sources dans votre flux d’événements.

## Établir une source d’Eventstream

1. Dans le canevas du flux d’événements, sélectionnez **Nouvelle source** dans la liste déroulante, puis sélectionnez **Exemples de données**.

    [ ![Image de l’utilisation d’un exemple de données](./Images/eventstream-select-sample-data.png) ](./Images/eventstream-select-sample-data-large.png#lightbox)

2.  Dans **Ajouter une source**, attribuez un nom à votre source, puis sélectionnez **Bicycles Reflex compatible)
1.  Cliquez sur le bouton **Ajouter**.

    ![Sélectionner et nommer un flux d’événements lié à l’exemple de données](./Images/eventstream-sample-data.png)

1. Une fois que vous avez sélectionné le bouton **Ajouter**, votre flux est mappé, et vous êtes automatiquement redirigé vers le **canevas d’Eventstream**.

   [ ![Passer en revue le canevas d’eventstream](./Images/real-time-intelligence-eventstream-sourced.png) ](./Images/real-time-intelligence-eventstream-sourced-large.png#lightbox)

3. Saisissez les valeurs de vos données d’échantillonnage comme indiqué dans le tableau suivant, puis sélectionnez **Ajouter et configurer**.
 
 > [!REMARQUE :] Une fois que vous avez créé l’exemple de source de données, vous voyez qu’il est ajouté à votre Eventstream sur le canevas en mode d’édition. Pour implémenter cet exemple de données récemment ajouté, sélectionnez **Publier**.

## Ajouter une activité Transformer des événements ou ajouter une destination

1. Après la publication, vous pouvez sélectionner **Transformer des événements ou ajouter une destination**, puis l’option **Base de données KQL**.

   [ ![définir une base de données KQL en tant que destination d’Eventstream](./Images/select-kql-destination.png) ](./Images/select-kql-destination-large.png)


2. Vous voyez un nouveau panneau latéral s’ouvrir, et vous offrir de nombreuses options. Entrez les détails nécessaires de votre base de données KQL.

   [ ![Eventstream de base de données KQL avec modes d’ingestion](./Images/kql-database-event-processing-before-ingestion.png) ](./Images/kql-database-event-processing-before-ingestion.png)

    - **Mode d’ingestion des données :** Il existe deux façons d’ingérer des données dans une base de données KQL :
        - ***Ingestion directe*** : ingestion des données directement dans une table KQL sans transformation.
        - ***Traitement des événements avant l’ingestion***: transformation des données avec le processeur d’événements avant d’envoyer à une table KQL.      
        
        > [!WARNING]
        > **Avertissement :** Vous **NE POUVEZ PAS** modifier le mode d’ingestion une fois la destination de la base de données KQL ajoutée à l’Eventstream.     

   - **Nom de la destination** : entrez un nom pour cette nouvelle destination Evenstream, par exemple "kql-dest."
   - **Espace de travail** : l’emplacement où se trouve votre base de données KQL.
   - **Base de données KQL** : nom de votre base de données KQL.
   - **Table de destination** : nom de votre table KQL. Vous pouvez également entrer un nom pour créer une table, par exemple « bike-count ».
   - **Format des données d’entrée :** Choisissez JSON en tant que format de données pour votre table KQL.


3. Cliquez sur **Enregistrer**. 
4. Cliquez sur **Publier**.

## Transformer les événements

1. Dans le canevas d’**Eventstream**, sélectionnez **Transformer les événements**.

    A. Sélectionnez **Regrouper par**.

    B. Sélectionnez **Modifier**, représenté par l’icône de ***crayon***.

    C. Remplir les propriétés de la section de paramètres **Regrouper par**

    [ ![Ajouter Regrouper par à l’événement de transformation.](./Images/eventstream-add-aggregates.png) ](./Images/eventstream-add-aggregates-large.png)

2. Une fois que vous avez créé l’événement de transformation **Regrouper par**, vous devez le connecter de l’**Eventstream** à **Regrouper par**. Pour ce faire, sans utiliser du code, cliquez sur le point situé à droite de l’**Eventstream**, puis faites-le glisser vers le point situé à gauche de la nouvelle zone **Regrouper par**.

   [ ![Ajouter un lien entre l’Eventstream et Regrouper par.](./Images/group-by-drag-connectors.png) ](./Images/group-by-drag-connectors-large.png)

3. De la même manière, vous pouvez pointer sur la flèche entre le **flux d’événements** et ***kql_dest***, puis sélectionner la ***poubelle***

   [ ![Supprimer un lien entre deux événements](./Images/delete-flow-arrows.png) ](./Images/delete-flow-arrows-large.png)

    > [!REMARQUE :] Chaque fois que vous ajoutez ou supprimez des connecteurs, vous devez reconfigurer les objets de destination.



## Requêtes KQL

Une requête KQL (Kusto Query Language, langage de requête Kusto) est une requête en lecture seule de traitement de données et de retour de résultats. La demande est formulée en texte brut en utilisant un modèle de flux de données facile à lire, à créer et à automatiser. Les requêtes s’exécutent toujours dans le contexte d’une table ou d’une base de données particulière. Une requête se compose au minimum d’une référence de données source et d’un ou de plusieurs opérateurs de requête appliqués de manière séquentielle, indiqués visuellement par l’utilisation d’une barre verticale (|) pour délimiter les opérateurs. Pour plus d’informations sur le langage de requête, consultez la [vue d’ensemble du langage de requête Kusto (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext).

> **Remarque** : L’éditeur KQL est fourni avec la mise en évidence de la syntaxe et d’Intellisense, ce qui vous permet d’acquérir rapidement des connaissances sur le langage de requête Kusto (KQL).

1. Accédez à la base de données KQL qui vient d’être créée et remplie de données :

    A.  Sélectionnez **kql_dest** 

    B. Sélectionnez le lien hypertexte **Ouvrir l’élément**, situé sur la ligne **Élément connexe**

   [ ![Supprimer un lien entre deux événements](./Images/navigate-to-data.png) ](./Images/navigate-to-data-large.png)

1. Dans l’arborescence des données, sélectionnez le menu Plus [...] pour la table ***Bike_sum***. Sélectionnez ensuite Interroger la table > Afficher 100 enregistrements.

   [ ![Supprimer un lien entre deux événements](./Images/kql-query-sample.png) ](./Images/kql-query-sample-large.png)

3. L’exemple de requête s’ouvre dans le volet **Explorer vos données** avec le contexte de table déjà renseigné. Cette première requête utilise l’opérateur take pour retourner un nombre restreint d’enregistrements, et est utile pour obtenir un premier aperçu de la structure des données et des valeurs possibles. Les exemples de requêtes renseignées automatiquement sont exécutés automatiquement. Les résultats de la requête s’affichent dans le volet des résultats.

   ![Image des résultats de la requête KQL](./Images/kql-query-results.png)

4. Retournez à l’arborescence des données afin de sélectionner la requête suivante, qui utilise l’opérateur summarize pour compter le nombre d’enregistrements ingérés par intervalles de 15 minutes.

   ![Image des résultats de la requête KQL](./Images/kql-query-results-15min-intervals.png)

> **Remarque** : Vous pouvez voir un avertissement indiquant que vous avez dépassé les limites de requête. Ce comportement varie en fonction de la quantité de données diffusées en streaming dans votre base de données.

Vous pouvez continuer à naviguer à l’aide des fonctions de requête intégrées pour vous familiariser avec vos données.

## Interroger avec Copilot

L’éditeur de requête prend en charge l’utilisation de T-SQL en plus de son langage KQL (Kusto Query Language) de requête principale. T-SQL peut être utile pour les outils qui ne peuvent pas utiliser KQL. Pour plus d’informations, consultez [Interroger des données à l’aide de T-SQL](https://learn.microsoft.com/en-us/azure/data-explorer/t-sql).

1. De retour dans l’arborescence Données, sélectionnez le **menu Plus** [...] dans la table MyStockData. Sélectionnez **Interroger la table > SQL > Afficher 100 enregistrements**.

   [ ![Image de l’exemple de requête SQL](./Images/sql-query-sample.png) ](./Images/sql-query-sample-large.png)

2. Placez votre curseur n’importe où dans la requête, puis sélectionnez **Exécuter** ou appuyez sur **Maj + Entrée**.

   ![Image des résultats de la requête sql](./Images/sql-query-results.png)

Vous pouvez continuer à naviguer à l’aide des fonctions intégrées et à vous familiariser avec les données à l’aide de SQL ou de KQL. 

## Fonctionnalités utilisant les ensembles de requêtes

Les ensembles de requêtes dans les bases de données KQL (Langage de requête Kusto) sont utilisés à diverses fins, principalement pour l’exécution des requêtes ainsi que pour la visualisation et la personnalisation des résultats des requêtes sur les données d’une base de données KQL. Ils constituent un composant clé des fonctionnalités d’interrogation de données de Microsoft Fabric, car ils permettent aux utilisateurs d’effectuer les tâches suivantes :

 - **Exécuter des requêtes :** Exécutez des requêtes KQL pour récupérer des données à partir d’une base de données KQL.
 - **Personnaliser les résultats :** Visualisez et modifiez les résultats des requêtes pour faciliter l’analyse et l’interprétation des données.
 - **Enregistrer et partager des requêtes :** Créez plusieurs onglets au sein d’un ensemble de requêtes afin d’enregistrer les requêtes pour les utiliser plus tard, ou les partager avec d’autres utilisateurs dans le cadre d’une exploration collaborative des données.
 - **Prendre en charge les fonctions SQL :** Parallèlement à l’utilisation de KQL pour la création de requêtes, les ensembles de requêtes prennent également en charge de nombreuses fonctions SQL, ce qui offre une certaine flexibilité dans l’interrogation des données.
 - **Tirer profit de Copilot :** Une fois que vous avez enregistré les requêtes en tant qu’ensemble de requêtes KQL, vous pouvez ensuite visualiser les résultats

L’enregistrement d’un ensemble de requêtes est simple, et comporte plusieurs approches. 

1. Dans votre **base de données KQL**, quand vous utilisez l’outil **Explorer vos données**, il vous suffit de sélectionner **Enregistrer en tant qu’ensemble de requêtes KQL**

   ![Enregistrer l’ensemble de requêtes KQL à partir de Explorer vos données](./Images/save-as-queryset.png)

2. Une autre approche consiste à utiliser la page de destination de Real-Time Intelligence en sélectionnant le bouton **Ensemble de requêtes KQL** de la page, puis en nommant votre **ensemble de requêtes**

   ![Créer un ensemble de requêtes KQL à partir de la page de destination de Real-Time Intelligence](./Images/select-create-new-queryset.png)

3. Une fois que vous êtes sur la **page de destination de l’ensemble de requêtes**, vous pouvez voir un bouton **Copilot** dans la barre d’outils. Sélectionnez-le pour ouvrir le **volet Copilot**, et poser des questions sur les données.

    [ ![Ouvrir Copilot à partir de la barre de menus](./Images/open-copilot-in-queryset.png) ](./Images/open-copilot-in-queryset-large.png)

4. Dans le **volet Copilot**, tapez simplement votre question. **Copilot** génère la requête KQL, et vous permet de ***copier*** ou d’***insérer** cette requête dans la fenêtre de votre ensemble de requêtes. 

    [ ![écrire une requête avec Copilot en posant une question](./Images/copilot-queryset-results.png) ](./Images/copilot-queryset-results-large.png)

5. À ce stade, vous pouvez utiliser des requêtes individuelles dans des tableaux de bord ou des rapports Power BI à l’aide des boutons **Épingler au tableau de bord** ou **Générer un rapport Power BI**.

## Nettoyer les ressources

Dans cet exercice, vous avez créé une base de données KQL et configuré un streaming continu avec un flux d’événements. Après cela, vous avez interrogé les données à l’aide de KQL et de SQL. Lorsque vous avez terminé d’explorer votre base de données KQL, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.
1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
.