---
lab:
  title: Créer et explorer un modèle sémantique
  module: Understand scalability in Power BI
---

# Créer et explorer un modèle sémantique

Dans cet exercice, vous allez utiliser Microsoft Fabric pour développer un modèle de données pour l’exemple NY Taxi dans un entrepôt de données.

Vous allez vous entraîner à :

- Créer un modèle sémantique personnalisé à partir d’un entrepôt de données Fabric.
- Créer des relations et organiser le diagramme du modèle.
- Explorer les données de votre modèle sémantique directement dans Fabric.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une [licence d’essai Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour effectuer cet exercice.

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Sur la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com), sélectionnez **Synapse Engineering données**.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
1. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
1. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide.

## Créer un entrepôt de données et charge un exemple de données

Maintenant que vous disposez d’un espace de travail, il est temps de créer un entrepôt de données. La page d’accueil Data Warehouse comprend un raccourci permettant de créer un entrepôt :

1. Dans la page d’accueil **Synapse Data Warehouse**, créez un **entrepôt** avec le nom de votre choix.

    Au bout d’une minute environ, un nouvel entrepôt est créé :
    
    ![Capture d’écran d’un nouvel entrepôt](./Images/new-data-warehouse2.png)

1. Au centre de l’interface utilisateur de l’entrepôt de données, vous verrez plusieurs façons de charger des données dans votre entrepôt. Sélectionnez **Exemple de données** pour charger les données de NYC Taxi dans votre entrepôt de données. Cette opération prend quelques minutes.

1. Une fois que les données de l’échantillon ont été chargées, utilisez le volet **Explorer** sur la gauche pour voir quelles tables et vues existent déjà dans l’entrepôt de données de l’échantillon.

1. Sélectionnez l’onglet **Rapports** du ruban, puis choisissez **Nouveau modèle sémantique**. Ceci vous permet de créer un modèle sémantique avec seulement des tables et des vues spécifiques de votre entrepôt de données, pour une utilisation par les équipes de données et l’entreprise pour créer des rapports.

1. Nommez le modèle sémantique **Taxi Revenue**, vérifiez qu’il se trouve dans l’espace de travail que vous venez de créer, puis sélectionnez les tables suivantes :
   - Date
   - Trajet
   - Pays
   - Météo
     
   ![Capture d’écran de l’interface de Nouveau modèle sémantique avec quatre tables sélectionnées](./Images/new-semantic-model.png)
     
## Créer des relations entre les tables

Vous allez maintenant créer des relations entre les tables pour analyser et visualiser avec précision vos données. Si vous avez l’habitude de créer des relations dans Power BI Desktop, ceci vous semblera familier.

1. Revenez à votre espace de travail et vérifiez que vous voyez votre nouveau modèle sémantique, Taxi Revenue. Notez que le type d’élément est **Modèle sémantique** et non pas **Modèle sémantique (par défaut)**, qui est créé automatiquement quand vous créez un entrepôt de données.

     *Remarque : Un modèle sémantique par défaut est créé automatiquement quand vous créez un point de terminaison d’analyse Entrepôt de données ou SQL dans Microsoft Fabric, et il hérite de la logique métier du lakehouse ou de l’entrepôt de données parent. Un modèle sémantique que vous créez vous-même, comme nous l’avons fait ici, est un modèle personnalisé que vous pouvez concevoir et modifier en fonction de vos besoins et de vos préférences spécifiques. Vous pouvez créer un modèle sémantique personnalisé en utilisant Power BI Desktop, le service Power BI ou d’autres outils qui se connectent à Microsoft Fabric.*

1. Sélectionnez **Ouvrir un modèle données à partir du ruban**.
   
    À présent, vous allez créer des relations entre les tables. Si vous avez l’habitude de créer des relations dans Power BI Desktop, ceci vous semblera familier.

    *En passant en revue le concept de schéma en étoile, nous allons organiser les tables de notre modèle en une table de faits et en tables de dimension. Dans ce modèle, la table **Trip** est notre table de faits, et nos dimensions sont **Date**, **Geography**et **Weather**.*

1. Créez une relation entre la table **Date** et la table **Trip** en utilisant la colonne **DateID**. **Sélectionnez la colonne DateID** dans la table **Date** et **faites-la glisser au-dessus de la colonne DateID dans la table Trip**. Vous pouvez aussi sélectionner **Gérer les relations** dans le ruban, puis **Nouvelle relation**.

1. Vérifiez que la relation est une relation **Un à plusieurs** de la table **Date** vers la table **Trip**.

1. Créez des relations vers la table de faits **Trip** depuis les dimensions **Geography** et **Weather**, en répétant l’étape ci-dessus. Vérifiez également que ces relations sont **Un à plusieurs**, avec une occurrence de la clé dans la table de dimension et plusieurs occurrences dans la table de faits. 

1. Faites glisser les tables dans une position telle que la table de faits **Trip** se trouve dans le bas du diagramme et que les tables restantes, qui sont des tables de dimension, se trouvent autour de la table de faits.

    ![Capture d’écran du diagramme du schéma en étoile](./Images/star-schema-diagram.png)

    *La création du modèle de schéma en étoile est maintenant terminée. De nombreuses configurations de modélisation peuvent désormais être appliquées, comme l’ajout de hiérarchies, de calculs et la définition de propriétés telles que la visibilité des colonnes.*

    > **Conseil** : Dans le volet Propriétés de la fenêtre, faites passer *Épingler les champs associés en haut de la carte* sur Oui. Ceci va vous aider (ainsi que d’autres utilisateurs créant des rapports à partir de ce modèle) à voir en un clin d’œil quels champs sont utilisés dans les relations. Vous pouvez également interagir avec les champs de vos tables en utilisant le volet des propriétés. Par exemple, si vous souhaitez vérifier que les types de données sont correctement définis, vous pouvez sélectionner un champ et examiner son format dans le volet des propriétés.

     ![Capture d’écran du volet des propriétés](./Images/properties-pane.png)

## Exploration de vos données

Vous disposez maintenant d’un modèle sémantique créé à partir de votre entrepôt, où des relations nécessaires pour la création de rapports ont été établies. Examinons les données en utilisant la fonctionnalité **Explorer les données**.

1. Revenez à votre espace de travail et sélectionnez votre **modèle sémantique Taxi Revenue**.

1. Dans la fenêtre, sélectionnez **Explorer ces données** dans le ruban. Ici, vous allez examiner vos données dans un format tabulaire. Ceci offre une expérience axée sur l’exploration de vos données sans créer un rapport Power BI complet.

1. Ajoutez **YearName** et **MonthName** aux lignes, puis explorez le **nombre moyen de passagers**, le **prix moyen d’un trajet**et la **durée moyenne d’un trajet** dans le champ des valeurs.

    *Quand vous faites un glisser-déposer d’un champ numérique dans le volet d’exploration, le comportement par défaut est de totaliser les nombres. Pour changer l’agrégation de **Total** en **Moyenne**, sélectionnez le champ, puis changez l’agrégation dans la fenêtre contextuelle.*

    ![Capture d’écran de la fenêtre Explorer les données, avec un visuel de matrice montrant les moyennes au fil du temps.](./Images/explore-data-fabric.png)

1. Pour examiner ces données sous forme de visuel au lieu de simplement une matrice, sélectionnez **Visuel** dans le bas de la fenêtre. Sélectionnez un graphique à barres pour visualiser rapidement ces données.

   *Un graphique à barres n’est pas la meilleure façon d’examiner ces données. Parcourez les différents visuels et les champs que vous examinez dans la section « Réorganiser les données » du volet Données sur le côté droit de l’écran.*

1. Vous pouvez maintenant enregistrer cette vue d’exploration dans votre espace de travail en cliquant sur le bouton **Enregistrer** dans le coin supérieur gauche. Vous avez aussi la possibilité de **Partager** la vue en sélectionnant **Partager** dans le coin supérieur droit. Ceci vous permet de partager l’exploration des données avec des collègues.

1. Une fois que vous avez enregistré votre exploration, revenez à votre espace de travail pour voir votre entrepôt de données, votre modèle sémantique par défaut, le modèle sémantique que vous avez créé et votre exploration.

    ![Capture d’écran d’un espace de travail dans Fabric montrant un entrepôt de données, un modèle sémantique par défaut, un modèle sémantique et une exploration de données.](./Images/semantic-model-workspace.png)

