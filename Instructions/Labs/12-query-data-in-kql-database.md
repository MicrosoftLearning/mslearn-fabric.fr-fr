---
lab:
  title: Interroger des données dans une base de données KQL
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# Bien démarrer avec l’interrogation d’une base de données Kusto dans Microsoft Fabric
KQL Queryset est un outil qui vous permet d’exécuter des requêtes, mais également de modifier et d’afficher les résultats des requêtes à partir d’une base de données KQL. Vous pouvez lier chaque onglet dans KQL Queryset à une base de données KQL différente et enregistrer vos requêtes pour une utilisation ultérieure ou les partager avec d’autres personnes pour l’analyse des données. Vous pouvez également basculer la base de données KQL pour n’importe quel onglet, ce qui vous permet de comparer les résultats de la requête à partir de diverses sources de données.

Pour créer des requêtes, KQL Queryset utilise le langage Kusto Query qui est compatible avec de nombreuses fonctions SQL. Pour en savoir plus sur le [langage kusto query (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext), 

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI](./Images/new-workspace.png)

Dans ce labo, vous allez utiliser l’Analyse en temps réel (RTA) dans Fabric pour créer une base de données KQL à partir d’un exemple d’eventstream. L’Analyse en temps réel fournit facilement un exemple de jeu de données à utiliser pour explorer les fonctionnalités de l’analyse en temps réel (RTA). Vous allez utiliser cet exemple de données pour créer des requêtes et des ensembles de requêtes KQL | SQL qui analysent certaines données en temps réel et permettent une utilisation supplémentaire dans les processus en aval.


## Scénario
Dans ce scénario, vous êtes un analyste chargé d’interroger un exemple de jeu de données que vous allez implémenter à partir de l’environnement Fabric.



Une requête Kusto est un moyen de lire des données, de les traiter et d’en afficher les résultats. La requête est écrite en texte brut facile à utiliser. Une requête Kusto peut avoir une ou plusieurs instructions qui montrent les données sous forme de table ou de graphique.

Une instruction de table a certains opérateurs qui fonctionnent sur les données de la table. Chaque opérateur prend une table comme entrée et fournit une table en tant que sortie. Les opérateurs sont joints par un canal (|). Les données passent d’un opérateur à l’autre. Chaque opérateur modifie les données d’une manière ou d’une autre, puis les transmet.

Vous pouvez l’imaginer comme un entonnoir dans lequel vous commencez avec une table entière de données. Chaque opérateur filtre, trie ou résume les données. L’ordre des opérateurs est important, car ils travaillent les uns après les autres. À la fin de l’entonnoir, vous obtenez une sortie finale.

Ces opérateurs sont spécifiques à KQL, mais peuvent être similaires à SQL ou à un autre langage.