---
lab:
  title: "Bien démarrer avec Real-Time Intelligence dans Microsoft\_Fabric"
  module: Get started with Real-Time Intelligence in Microsoft Fabric
---

# Bien démarrer avec Real-Time Intelligence dans Microsoft Fabric

Microsoft Fabric fournit un hub en temps réel dans lequel vous pouvez créer des solutions analytiques pour les flux de données en temps réel. Dans cet exercice, vous allez explorer certaines des principales fonctionnalités d’intelligence en temps réel dans Microsoft Fabric afin de vous familiariser avec elles.

Ce labo prend environ **30** minutes.

> **Remarque** : pour effectuer cet exercice, vous avez besoin d’un [locataire Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, vous devez créer un espace de travail dans un locataire avec la fonctionnalité Fabric activée.

1. Dans la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric`, sélectionnez **Real-Time Intelligence**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

    ![Capture d’écran d’un espace de travail vide dans Fabric.](./Images/new-workspace.png)

## Créer un eventhouse

Maintenant que vous disposez d’un espace de travail, vous pouvez commencer à créer les éléments Fabric dont vous aurez besoin pour votre solution d’intelligence en temps réel. Nous allons commencer par créer un eventhouse, qui contient une base de données KQL pour vos données en temps réel.

1. Dans la barre de menus de gauche, sélectionnez **Accueil**, puis, dans la page d’accueil de Real-Time Intelligence, créez un **Eventhouse**, en lui donnant un nom unique de votre choix.
1. Fermez toutes les invites ou conseils affichés jusqu’à ce que le nouvel eventhouse vide soit visible :

    ![Capture d’écran d’un nouvel eventhouse](./Images/create-eventhouse.png)

1. Dans le volet de gauche, notez que votre eventhouse contient une base de données KQL portant le même nom que l’eventhouse. Vous pouvez créer des tables pour vos données en temps réel dans cette base de données ou créer des bases de données supplémentaires si nécessaire.
1. Sélectionnez la base de données et notez qu’il existe un *ensemble de requêtes* associé. Ce fichier contient des exemples de requêtes KQL que vous pouvez utiliser pour commencer à interroger les tables de votre base de données.

    Toutefois, il n’existe actuellement aucune table à interroger. Nous allons résoudre ce problème en ingérant certaines données dans la base de données à l’aide d’un eventstream.

## Créer un Eventstream

1. Dans la page principale de votre base de données KQL, sélectionnez **Obtenir des données**.
2. Pour la source de données, sélectionnez **Eventstream** > **Nouvel evenstream**. Nommez l’eventstream `stock-stream`.

    La création de votre eventstream dans l’espace de travail ne prend que quelques instants. Une fois l’opération effectuée, vous êtes automatiquement redirigé vers l’éditeur principal, prêt à commencer à intégrer des sources dans votre flux d’événements.

    ![Capture d’écran d’un nouvel eventstream.](./Images//name-eventstream.png)

1. Dans le canevas d’eventstream, sélectionnez **Utiliser des exemples de données**.
1. Nommez la source `Stock`, puis sélectionnez l’exemple de données **Stock Market**.

    Votre flux sera mappé et sera automatiquement affiché sur le **canevas d’eventstream**.

   ![Capture d’écran du canevas d’eventstream.](./Images/new-stock-stream.png)

1. Dans la liste déroulante **Transformer des événements ou ajouter une destination**, dans la section **Destinations**, sélectionnez **Eventhouse**.
1. Dans le volet **Eventhouse**, configurez les options suivantes.
   - **Mode d’ingestion des données :** traitement des événements avant l’ingestion
   - **Nom de la destination :**`stock-table`
   - **Espace de travail :***sélectionnez l’espace de travail que vous avez créé au début de cet exercice.*
   - **Eventhouse** : *sélectionnez votre eventhouse*
   - **Base de données KQL :***sélectionnez la base de données KQL de votre eventhouse*
   - **Table de destination :** créez une table nommée `stock`
   - **Format de données d’entrée :** JSON

   ![Eventstream de base de données KQL avec modes d’ingestion](./Images/configure-destination.png)

1. Dans le volet **Eventhouse**, sélectionnez **Enregistrer**.
1. Dans la barre d’outils, sélectionnez **Publier**.
1. Attendez environ une minute que la destination des données devienne active.

    Dans cet exercice, vous avez créé un eventstream très simple qui capture des données en temps réel et les charge dans une table. Dans une solution réelle, vous ajouteriez généralement des transformations pour agréger les données sur des fenêtres temporelles (par exemple, pour capturer le prix moyen de chaque action sur des périodes de cinq minutes).

    Examinons maintenant comment interroger et analyser les données capturées.

## Interroger les données capturées

L’eventstream capture les données boursières en temps réel et les charge dans une table de votre base de données KQL. Vous pouvez interroger cette table pour afficher les données capturées.

1. Dans la barre de menus de gauche, sélectionnez la base de données de votre eventhouse.
1. Sélectionnez l’*ensemble de requêtes* de votre base de données.
1. Dans le volet de requête, modifiez le premier exemple de requête, comme illustré ici :

    ```kql
    stock
    | take 100
    ```

1. Sélectionnez le code de requête et exécutez-le pour afficher 100 lignes de données depuis la table.

    ![Capture d’écran d’une requête KQL.](./Images/kql-stock-query.png)

1. Passez en revue les résultats, puis modifiez la requête pour récupérer le prix moyen de chaque symbole boursier au cours des 5 dernières minutes :

    ```kql
    stock
    | where ["time"] > ago(5m)
    | summarize avgPrice = avg(todecimal(bidPrice)) by symbol
    | project symbol, avgPrice
    ```

1. Sélectionnez la requête modifiée et exécutez-la pour afficher les résultats.
1. Attendez quelques secondes et réexécutez-la, en notant que les prix moyens changent à mesure que de nouvelles données sont ajoutées à la table à partir du flux en temps réel.

## Créer des tableaux de bord en temps réel

Maintenant que vous disposez d’une table remplie par le flux de données, vous pouvez utiliser un tableau de bord en temps réel pour visualiser les données.

1. Dans l’éditeur de requête, sélectionnez la requête KQL que vous avez utilisée pour récupérer les prix moyens des actions pendant les cinq dernières minutes.
1. Sélectionnez **Épingler au tableau de bord** dans la barre d’outils. Épinglez ensuite la requête **dans un nouveau tableau de bord** avec les paramètres suivants :
    - **Nom du tableau de bord** : `Stock Dashboard`
    - **Nom de la vignette** : `Average Prices`
1. Créez le tableau de bord et ouvrez-le. Il doit se présenter comme suit :

    ![Capture d’écran d’un nouveau tableau de bord.](./Images/stock-dashboard-table.png)

1. En haut du tableau de bord, passez du mode **Affichage** au mode **Édition**.
1. Sélectionnez l’icône **Modifier** (*crayon*) pour la vignette **Prix moyens**.
1. Dans le volet **Mise en forme visuelle**, remplacez le **visuel** *Table* par *Histogramme* :

    ![Capture d’écran d’une vignette de tableau de bord en cours de modification.](./Images/edit-dashboard-tile.png)

1. En haut du tableau de bord, sélectionnez **Appliquer les modifications** et affichez votre tableau de bord modifié :

    ![Capture d’écran d’un tableau de bord avec une vignette de graphique.](./Images/stock-dashboard-chart.png)

    Vous disposez maintenant d’une visualisation dynamique de vos données boursières en temps réel.

## Créer une alerte

L’intelligence en temps réel dans Microsoft Fabric inclut une technologie nommée *Activator*, qui peut déclencher des actions basées sur des événements en temps réel. Nous allons l’utiliser pour vous avertir lorsque le prix moyen des actions augmente d’un montant spécifique.

1. Dans la fenêtre du tableau de bord contenant votre visualisation du prix des actions, dans la barre d’outils, sélectionnez **Définir une alerte**.
1. Dans le volet **Définir une alerte**, créez une alerte avec les paramètres suivants :
    - **Exécuter la requête toutes les** : 5 minutes
    - **Vérifier** : sur chaque événement Regroupé par
    - **Champ de regroupement** : symbole
    - **Quand** : avgPrice
    - **Condition** : Augmente de
    - **Valeur** : 100
    - **Action** : Envoyer un e-mail
    - **Emplacement d’enregistrement** :
        - **Espace de travail** : *votre espace de travail*
        - **Élément** : Créer un élément
        - **Nouveau nom d’élément** : *un nom unique de votre choix*

    ![Capture d’écran des paramètres d’alerte.](./Images/configure-activator.png)

1. Créez l’alerte et attendez qu’elle soit enregistrée. Fermez ensuite le volet confirmant qu’elle a été créée.
1. Dans la barre de menus de gauche, sélectionnez la page de votre espace de travail (enregistrez les modifications non enregistrées dans votre tableau de bord si vous y êtes invité).
1. Dans la page de l’espace de travail, affichez les éléments que vous avez créés dans cet exercice, y compris l’activateur de votre alerte.
1. Ouvrez l’activateur et, dans sa page, sous le nœud **avgPrice**, sélectionnez l’identificateur unique de votre alerte. Affichez ensuite son onglet **Historique**.

    Votre alerte n’a peut-être pas été déclenchée, auquel cas l’historique ne contiendra aucune donnée. Si le prix moyen des actions change de plus de 100, l’activateur vous envoie un e-mail et l’alerte est enregistrée dans l’historique.

## Nettoyer les ressources

Dans cet exercice, vous avez créé un eventhouse, ingéré des données en temps réel à l’aide d’un eventstream, interrogé les données ingérées dans une table de base de données KQL, créé un tableau de bord en temps réel pour visualiser les données en temps réel et configuré une alerte à l’aide de l’activateur.

Si vous avez fini d’explorer l’intelligence en temps réel dans Fabric, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail.
2. Dans la barre d’outils, sélectionnez **Paramètres de l’espaces de travail**.
3. Dans la section **Général**, sélectionnez **Supprimer cet espace de travail**.
