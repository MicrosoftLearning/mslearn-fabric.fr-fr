---
lab:
  title: Utiliser Data Activator dans Fabric
  module: Get started with Data Activator in Microsoft Fabric
---

# Utiliser Data Activator dans Fabric

Dans Microsoft Fabric, un entrepôt de données fournit une base de données relationnelle pour l’analytique à grande échelle. Contrairement au point de terminaison SQL en lecture seule par défaut pour les tables définies dans un lakehouse, un entrepôt de données fournit une sémantique SQL complète, y compris la possibilité d’insérer, de mettre à jour et de supprimer des données dans les tables.

Ce labo prend environ **30** minutes.

> **Remarque** : Vous devez disposer d’une licence Microsoft Fabric pour effectuer cet exercice. Consultez [Bien démarrer avec Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) pour plus d’informations sur l’activation d’une licence d’essai Fabric gratuite. Vous aurez besoin pour cela d’un compte *scolaire* ou *professionnel* Microsoft. Si vous n’en avez pas, vous pouvez vous [inscrire à un essai de Microsoft Office 365 E3 ou version ultérieure](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Créer un espace de travail

Avant d’utiliser des données dans Fabric, créez un espace de travail avec l’essai gratuit de Fabric activé.

1. Connectez-vous à [Microsoft Fabric](https://app.fabric.microsoft.com) à l’adresse `https://app.fabric.microsoft.com` et sélectionnez **Power BI**.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec le nom de votre choix et sélectionnez un mode de licence qui inclut la capacité Fabric (*Essai*, *Premium* ou *Fabric*).
4. Lorsque votre nouvel espace de travail s’ouvre, il doit être vide, comme illustré ici :

    ![Capture d’écran d’un espace de travail vide dans Power BI.](./Images/new-workspace.png)
