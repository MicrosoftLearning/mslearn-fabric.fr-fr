---
lab:
  title: Implémenter des pipelines de déploiement dans Microsoft Fabric
  module: Implement CI/CD in Microsoft Fabric
---

# Implémenter des pipelines de déploiement dans Microsoft Fabric

Les pipelines de déploiement dans Microsoft Fabric vous permettent d’automatiser le processus de copie des modifications apportées au contenu dans des éléments Fabric entre des environnements, par exemple, de développement, de test et de production. Vous pouvez utiliser des pipelines de déploiement pour développer et tester du contenu avant que celui-ci soit diffusé aux utilisateurs finaux. Dans cet exercice, vous allez créer un pipeline de déploiement et attribuer des étapes au pipeline. Ensuite, vous allez créer du contenu dans un espace de travail de développement et utiliser des pipelines de déploiement pour le déployer entre les étapes de pipelines Développement, Test et Production.

> **Note** : pour effectuer cet exercice, vous devez être membre du rôle Administrateur de l’espace de travail Fabric. Pour attribuer des rôles, consultez [Rôles dans les espaces de travail dans Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/get-started/roles-workspaces).

Ce labo dure environ **20** minutes.

## Créer des espaces de travail

Créez trois espaces de travail avec la version d’évaluation de Fabric activée.

1. Dans la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric), à l’adresse `https://app.fabric.microsoft.com/home?experience=fabric`, sélectionnez **Microsoft Fabric**, puis **Engineering données** dans la barre de menu inférieure gauche.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail nommé Developpement et sélectionnez un mode de licence qui inclut la fonctionnalité Fabric (*Évaluation*, *Premium* ou *Fabric*).
4. Répétez les étapes 1 et 2, en créant deux espaces de travail supplémentaires nommés Test et Production. Vos espaces de travail sont ainsi : Développement, Test et Production.
5. Sélectionnez l’icône **Espaces de travail** dans la barre de menu à gauche et vérifiez qu’il y a bien trois espaces de travail nommés : Développement, Test et Production.

> **Note** : si vous êtes invité à entrer un nom unique pour les espaces de travail, ajoutez un ou plusieurs nombres aléatoires après les mots Développement, Test ou Production.

## Créer un pipeline de déploiement

Ensuite, créez un pipeline de déploiement.

1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.
2. Sélectionnez **Pipelines de déploiement**, puis **Nouveau pipeline**.
3. Dans la fenêtre **Ajouter un nouveau pipeline de déploiement**, donnez au pipeline un nom unique.
4. Acceptez les valeurs par défaut dans la fenêtre **Personnaliser vos étapes**.  

   ![Capture d’écran des étapes de pipeline.](./Images/customize-stages.png)

5. Sélectionnez **Créer**.

## Attribuer des espaces de travail aux étapes d’un pipeline de déploiement

Attribuez des espaces de travail aux étapes du pipeline de déploiement.

1. Dans la barre de menu de gauche, sélectionnez le pipeline que vous avez créé. 
2. Dans la fenêtre qui s’affiche, cliquez sur le mot **Sélectionner** sous chaque étape de déploiement, puis sélectionnez le nom de l’espace de travail correspondant au nom de l’étape.
3. Sélectionnez **Attribuer un espace de travail** pour chaque étape de déploiement.

  ![Capture d'écran du pipeline de déploiement.](./Images/deployment-pipeline.png)

## Créer du contenu

Les éléments Fabric n’ont pas encore été créés dans vos espaces de travail. Ensuite, créez un lakehouse dans l’espace de travail Développement.

1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.
2. Sélectionnez l’espace de travail **Développement**.
3. Sélectionnez **Nouvel élément**.
4. Dans la fenêtre qui s’affiche, sélectionnez **Lakehouse** et dans la fenêtre **Nouveau lakehouse**, nommez le lakehouse **LabLakehouse**.
5. Sélectionnez **Créer**.
6. Dans la fenêtre de l’explorateur de lakehouse, sélectionnez **Démarrer avec des exemples de données** pour remplir le nouveau lakehouse avec des données.

  ![Capture d’écran de l’explorateur de lakehouse.](./Images/lakehouse-explorer.png)

8. Dans la barre de menu de gauche, sélectionnez le pipeline que vous avez créé.
9. Dans l’étape **Développement**, cliquez sur le **>** jusqu’à ce que vous voyiez **Lakehouses**. Le lakehouse s’affiche comme un nouveau contenu dans l’étape Développement. Entre les étapes **Développement** et **Test**, il y a un **X** orange dans un cercle. Le **X** orange indique que les étapes Développement et Test ne sont pas synchronisées.
10. Sélectionnez la flèche vers le bas sous le **X** orange pour comparer le contenu dans les environnements Développement et Test. Sélectionnez **Comparer**. LabLakehouse existe uniquement dans l’étape Développement.  

  ![Capture d’écran du pipeline de déploiement montrant des disparités de contenu entre les étapes.](./Images/lab-pipeline-compare.png)

## Déployer du contenu entre les étapes

Déployez le lakehouse à partir de l’étape **Développement** vers les étapes **Test** et **Production**.
1. Sélectionnez le bouton **Déployer** dans l’étape **Développement** du pipeline pour copier le lakehouse dans son état actuel vers l’étape suivante. 
2. Dans la fenêtre **Déployer vers l’étape suivante**, sélectionnez **Déployer**.
3. Il y a un X orange entre les étapes Test et Production. Sélectionnez la flèche vers le bas sous le X orange. Le lakehouse existe dans les étapes Développement et Test, mais pas encore dans l’étape Production.
4. Dans l’étape **Test**, sélectionnez **Déployer**.
5. Dans la fenêtre **Déployer vers l’étape suivante**, sélectionnez **Déployer**. La coche verte entre les étapes indique que toutes les étapes sont synchronisées et contiennent le même contenu.
6. L’utilisation de pipelines de déploiement pour déployer entre les étapes met également à jour le contenu dans les espaces de travail qui correspondent à l’étape de déploiement. Nous allons le vérifier.
7. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.
8. Sélectionnez l’espace de travail **Test**. C’est ici que le lakehouse a été copié.
9. Ouvrez l’espace de travail **Production** via l’icône **Espaces de travail** du menu de gauche. Le lakehouse a également été copié dans l’espace de travail Production.

## Nettoyage

Dans cet exercice, vous avez créé un pipeline de déploiement et attribué des étapes au pipeline. Ensuite, vous avez créé du contenu dans un espace de travail de développement et l’avez déployé entre les étapes du pipeline à l’aide de pipelines de déploiement.

- Dans la barre de navigation de gauche, sélectionnez les icônes des espaces de travail pour afficher tous les éléments qu’ils contiennent.
- Dans le menu de la barre d’outils du haut, sélectionnez Paramètres de l’espace de travail.
- Dans la section Général, sélectionnez Supprimer cet espace de travail.