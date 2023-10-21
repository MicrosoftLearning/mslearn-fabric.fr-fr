---
lab:
  title: Générer et enregistrer des prédictions par lots
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# Générer et enregistrer des prédictions par lots

Dans ce labo, vous allez utiliser un modèle Machine Learning pour prédire une mesure quantitative de diabète. Vous allez utiliser la fonction PREDICT dans Fabric pour générer les prédictions avec un modèle inscrit.

En effectuant ce labo, vous allez acquérir une expérience pratique de la génération de prédictions et de la visualisation des résultats.

Ce labo prend environ **45** minutes.

> **Remarque** : Vous aurez besoin d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des modèles dans Fabric, créez un espace de travail en activant l’essai gratuit de Fabric.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)

## Charger le bloc-notes

Pour ingérer des données, former et inscrire un modèle, vous allez exécuter les cellules d’un notebook. Vous pouvez charger le notebook sur votre espace de travail.

1. En bas à gauche du portail Fabric, sélectionnez l’icône **Engineering données** et basculez vers l’expérience **Science des données**.
1. Dans la page d’accueil **Science des données**, sélectionnez **Importer un notebook**.

    Vous recevez une notification une fois que le notebook est importé.

1. Accédez au notebook importé nommé `Generate-Predictions`.

1. Lisez attentivement les instructions dans le notebook et exécutez chaque cellule individuellement.

## Nettoyer les ressources

Dans cet exercice, vous avez utilisé un modèle pour générer des prédictions par lots.

Si vous avez fini d’explorer le notebook, vous pouvez supprimer l’espace de travail que vous avez créé pour cet exercice.

1. Dans la barre de gauche, sélectionnez l’icône de votre espace de travail pour afficher tous les éléments qu’il contient.
2. Dans le menu **...** de la barre d’outils, sélectionnez **Paramètres de l’espace de travail**.
3. Dans la section **Autre**, sélectionnez **Supprimer cet espace de travail**.
