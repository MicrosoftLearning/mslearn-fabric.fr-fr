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

1. Accédez à la [page d’accueil de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) sur `https://app.fabric.microsoft.com/home?experience=fabric` dans un navigateur et connectez-vous avec vos informations d’identification Fabric.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un nouvel espace de travail nommé Developpement et sélectionnez un mode de licence qui inclut la fonctionnalité Fabric (*Évaluation*, *Premium* ou *Fabric*).
4. Répétez les étapes 1 et 2, en créant deux espaces de travail supplémentaires nommés Test et Production. Vos espaces de travail sont ainsi : Développement, Test et Production.
5. Sélectionnez l’icône **Espaces de travail** dans la barre de menu à gauche et vérifiez qu’il y a bien trois espaces de travail nommés : Développement, Test et Production.

> **Note** : si vous êtes invité à entrer un nom unique pour les espaces de travail, ajoutez un ou plusieurs nombres aléatoires après les mots Développement, Test ou Production.

## Créer un pipeline de déploiement

Ensuite, créez un pipeline de déploiement.

1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.
2. Sélectionnez **Pipelines de déploiement**, puis **Nouveau pipeline**.
3. Dans la fenêtre **Ajouter un nouveau pipeline de déploiement**, donnez au pipeline un nom unique, puis sélectionnez **Suivant**.
4. Dans la fenêtre du nouveau pipeline, sélectionnez **Créer et continuer**.

## Attribuer des espaces de travail aux étapes d’un pipeline de déploiement

Attribuez des espaces de travail aux étapes du pipeline de déploiement.

1. Dans la barre de menu de gauche, sélectionnez le pipeline que vous avez créé. 
2. Dans la fenêtre qui s’affiche, développez les options sous **Attribuer un espace de travail** pour chaque étape de déploiement, puis sélectionnez le nom de l’espace de travail correspondant au nom de l’étape.
3. Cochez la case **Attribuer** pour chaque étape de déploiement.

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

7. Sélectionnez l’exemple **NYCTaxi**.
8. Dans la barre de menu de gauche, sélectionnez le pipeline que vous avez créé.
9. Sélectionnez l’étape **Développement**, puis, sous le canevas du pipeline de déploiement, vous verrez le lakehouse que vous avez créé en tant qu’élément d’étape. Dans le bord gauche de l’étape **Test**, il y a un **X** dans un cercle. Le **X** indique que les étapes Développement et Test ne sont pas synchronisées.
10. Sélectionnez l’étape **Test** et, sous le canevas du pipeline de déploiement, vous verrez que le lakehouse que vous avez créé n’est qu’un élément d’étape dans la source, ce qui, dans ce cas, fait référence à l’étape **Développement**.  

  ![Capture d’écran du pipeline de déploiement montrant des disparités de contenu entre les étapes.](./Images/lab-pipeline-compare.png)

## Déployer du contenu entre les étapes

Déployez le lakehouse à partir de l’étape **Développement** vers les étapes **Test** et **Production**.
1. Sélectionnez l’étape **Test** dans le canevas du pipeline de déploiement.
1. Sous le canevas du pipeline de déploiement, cochez la case en regard de l’élément Lakehouse. Sélectionnez le bouton **Déployer** pour copier le Lakehouse dans son état actuel dans l’étape **Test**.
1. Dans la fenêtre **Déployer vers l’étape suivante** qui s’affiche, sélectionnez **Déployer**.
 Il y a maintenant un X dans un cercle dans la phase de production dans le canevas du pipeline de déploiement. Le lakehouse existe dans les étapes Développement et Test, mais pas encore dans l’étape Production.
1. Sélectionnez l’étape **Production** dans le canevas de déploiement.
1. Sous le canevas du pipeline de déploiement, cochez la case en regard de l’élément Lakehouse. Sélectionnez ensuite le bouton **Déployer** pour copier le lakehouse dans son état actuel dans l’étape **Production**.
1. Dans la fenêtre **Déployer vers l’étape suivante** qui s’affiche, sélectionnez **Déployer**. La coche verte entre les étapes indique que toutes les étapes sont synchronisées et contiennent le même contenu.
1. L’utilisation de pipelines de déploiement pour déployer entre les étapes met également à jour le contenu dans les espaces de travail qui correspondent à l’étape de déploiement. Nous allons le vérifier.
1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**.
1. Sélectionnez l’espace de travail **Test**. C’est ici que le lakehouse a été copié.
1. Ouvrez l’espace de travail **Production** via l’icône **Espaces de travail** du menu de gauche. Le lakehouse a également été copié dans l’espace de travail Production.

## Nettoyage

Dans cet exercice, vous avez créé un pipeline de déploiement et attribué des étapes au pipeline. Ensuite, vous avez créé du contenu dans un espace de travail de développement et l’avez déployé entre les étapes du pipeline à l’aide de pipelines de déploiement.

- Dans la barre de navigation de gauche, sélectionnez les icônes des espaces de travail pour afficher tous les éléments qu’ils contiennent.
- Dans le menu de la barre d’outils du haut, sélectionnez Paramètres de l’espace de travail.
- Dans la section Général, sélectionnez Supprimer cet espace de travail.
