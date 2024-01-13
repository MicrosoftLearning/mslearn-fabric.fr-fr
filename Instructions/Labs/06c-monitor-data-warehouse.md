---
lab:
  title: Surveiller un entrepôt de données dans Microsoft Fabric
  module: Monitor a data warehouse in Microsoft Fabric
---

# Surveiller un entrepôt de données dans Microsoft Fabric

Dans Microsoft Fabric, un entrepôt de données fournit une base de données relationnelle pour l’analytique à grande échelle. Les entrepôts de données dans Microsoft Fabric incluent des vues de gestion dynamiques que vous pouvez utiliser pour surveiller l’activité et les requêtes.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous avez besoin d’un compte *scolaire* ou *professionnel* Microsoft pour réaliser cet exercice. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou supérieur](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

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

## Explorer les vues de gestion dynamiques

Les entrepôts de données Microsoft Fabric incluent des vues de gestion dynamiques (DMV), que vous pouvez utiliser pour identifier l’activité actuelle dans l’instance d’entrepôt de données.

1. Dans la page de l’entrepôt de données **sample-dw**, dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL**.
1. Dans le volet de nouvelle requête vide, entrez le code Transact-SQL suivant pour interroger la vue de gestion dynamique **sys.dm_exec_connections** :

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL et voir les résultats, qui incluent les détails de toutes les connexions à l’entrepôt de données.
1. Modifiez le code SQL pour interroger la vue de gestion dynamique **sys.dm_exec_sessions** comme suit :

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. Exécutez la requête modifiée et visualisez les résultats, qui montrent les détails de toutes les sessions authentifiées.
1. Modifiez le code SQL pour interroger la vue de gestion dynamique **sys.dm_exec_requests** comme suit :

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. Exécutez la requête modifiée et visualisez les résultats, qui montrent les détails de toutes les requêtes exécutées dans l’entrepôt de données.
1. Modifiez le code SQL pour joindre les vues de gestion dynamiques et retourner des informations sur les requêtes en cours d’exécution dans la même base de données, comme suit :

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. Exécutez la requête modifiée et visualisez les résultats, qui montrent les détails de toutes les requêtes en cours d’exécution dans la base de données (y compris celle-ci).
1. Dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL** pour ajouter un deuxième onglet de requête. Ensuite, dans le nouvel onglet de requête vide, exécutez le code suivant :

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. Laissez la requête en cours d’exécution, revenez à l’onglet contenant le code pour interroger les vues de gestion dynamiques, puis réexécutez-la. Cette fois, les résultats doivent inclure la deuxième requête en cours d’exécution dans l’autre onglet. Notez le temps écoulé pour cette requête.
1. Attendez quelques secondes, puis réexécutez le code pour interroger à nouveau les vues de gestion dynamiques. Le temps écoulé pour la requête dans l’autre onglet doit avoir augmenté.
1. Revenez au deuxième onglet de requête où la requête est toujours en cours d’exécution, puis sélectionnez **X Annuler** pour l’annuler.
1. De retour sous l’onglet avec le code pour interroger les vues de gestion dynamiques, réexécutez la requête pour vérifier que la deuxième requête n’est plus en cours d’exécution.
1. Fermez tous les onglets de requête.

> **Informations complémentaires** : Pour plus d’informations sur l’utilisation des vues de gestion dynamiques, consultez [Surveiller les connexions, les sessions et les demandes en utilisant des vues de gestion dynamiques](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv) dans la documentation Microsoft Fabric.

## Explorer les insights des requêtes

Les entrepôts de données Microsoft Fabric fournissent des *insights des requêtes* – un ensemble spécial de vues qui contiennent des détails sur les requêtes exécutées dans votre entrepôt de données.

1. Dans la page de l’entrepôt de données **sample-dw**, dans la liste déroulante **Nouvelle requête SQL**, sélectionnez **Nouvelle requête SQL**.
1. Dans le volet de nouvelle requête vide, entrez le code Transact-SQL suivant pour interroger la vue **exec_requests_history** :

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. Utilisez le bouton **&#9655; Exécuter** pour exécuter le script SQL et voir les résultats, qui incluent les détails des requêtes exécutées précédemment.
1. Modifiez le code SQL pour interroger la vue **frequently_run_queries**, comme suit :

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. Exécutez la requête modifiée et visualisez les résultats, qui montrent les détails des requêtes exécutées fréquemment.
1. Modifiez le code SQL pour interroger la vue **long_running_queries**, comme suit :

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. Exécutez la requête modifiée et visualisez les résultats, qui montrent les détails de toutes les requêtes et leur durée.

> **Informations complémentaires** : Pour plus d’informations sur l’utilisation des insights des requêtes, consultez [Insights des requêtes dans l’entreposage de données Fabric](https://learn.microsoft.com/fabric/data-warehouse/query-insights) dans la documentation Microsoft Fabric.


## Nettoyer les ressources

Dans cet exercice, vous avez utilisé des vues de gestion dynamiques et des insights des requêtes pour surveiller l’activité dans un entrepôt de données Microsoft Fabric.

Si vous avez terminé d’explorer votre entrepôt de données, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu  **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
