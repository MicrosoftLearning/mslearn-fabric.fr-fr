---
lab:
  title: Appliquer la sécurité du modèle sémantique
  module: Design and build tabular models
---

# Appliquer la sécurité des modèles

## Vue d’ensemble

**La durée estimée pour effectuer ce tutoriel est de 45 minutes.**

Dans ce labo, vous allez mettre à jour un modèle de données prédéveloppé pour mettre en place la sécurité. Plus précisément, les commerciaux de l’entreprise Adventure Works ne doivent voir que les données de vente liées à leur région de vente.

Dans ce labo, vous découvrez comment :

- Créer des rôles statiques

- Créer des rôles dynamiques

- Valider les rôles

- Mapper des principaux de sécurité à des rôles de modèle sémantique.

## Bien démarrer

Dans cet exercice, vous allez préparer votre environnement.

### Cloner le référentiel pour cette formation

1. Dans le menu Démarrer, ouvrez l’invite de commandes.

    ![](../images/command-prompt.png)

1. Dans la fenêtre d’invite de commandes, accédez au lecteur D en tapant :

    `d:` 

   Appuyez sur Entrée.

    ![](../images/command-prompt-2.png)


1. Dans la fenêtre d’invite de commandes, entrez la commande ci-après pour télécharger les fichiers de cours et les enregistrer dans un dossier appelé DP500.
    
    `git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst DP500`
   
1. Une fois le référentiel cloné, fermez la fenêtre d’invite de commandes. 
   
1. Ouvrez le lecteur D dans l’explorateur de fichiers pour vous assurer que les fichiers ont été téléchargés.

### Configurer Power BI Desktop

Dans cette tâche, vous allez configurer Power BI Desktop.

1. Pour ouvrir l’Explorateur de fichiers, dans la barre des tâches, sélectionnez le raccourci vers **Explorateur de fichiers**.

2. Accédez au dossier **D:\DP500\Allfiles\09\Starter**.

3. Pour ouvrir un fichier Power BI Desktop prédéveloppé, double-cliquez sur le fichier **Sales Analysis - Enforce model security.pbix**.

4. Si vous n’êtes pas déjà connecté, dans le coin supérieur droit de Power BI Desktop, sélectionnez **Se connecter**. Utilisez les informations d’identification du labo pour terminer le processus de connexion.

    ![](../images/dp500-enforce-model-security-image2.png)

5. Pour enregistrer le fichier, dans le ruban **Fichier**, sélectionnez **Enregistrer sous**.

6. Dans la fenêtre **Enregistrer sous**, accédez au dossier **D:\DP500\Allfiles\09\MySolution**.

7. Sélectionnez **Enregistrer**.

    *Vous allez mettre à jour la solution Power BI Desktop pour appliquer la sécurité au niveau des lignes.*

### Se connecter au service Power BI

Dans cette tâche, vous allez vous connecter au service Power BI, démarrer une licence d’évaluation et créer un espace de travail.

*Important : si vous avez déjà configuré Power BI dans votre environnement de machine virtuelle, passez à la tâche suivante.*

1. Dans un navigateur Web, accédez à [https://powerbi.com](https://powerbi.com/).

2. Utilisez les informations d’identification du labo pour terminer le processus de connexion.

    *Important : vous devez utiliser les mêmes informations d’identification que celles utilisées pour vous connecter à partir de Power BI Desktop.*

3. En haut à droite, sélectionnez l’icône de profil, puis sélectionnez **Démarrer la version d’évaluation**.

    ![](../images/dp500-enforce-model-security-image3.png)

4. Lorsque vous y êtes invité, sélectionnez **Démarrer la version d’évaluation**.

5. Effectuez les tâches restantes pour terminer la configuration de la version d’évaluation.

    *Conseil : l’expérience du navigateur web de Power BI est appelée **service Power BI**.*

### Créer un espace de travail

Dans cette tâche, vous allez créer un espace de travail.

1. Dans le service Power BI, pour créer un espace de travail, dans le volet **Navigation** (situé à gauche), sélectionnez **Espaces de travail**, puis **Créer un espace de travail**.

    ![](../images/dp500-enforce-model-security-image5.png)


2. Dans le volet **Créer un espace de travail** (situé à droite), dans la zone **Nom de l’espace de travail**, saisissez un nom pour l’espace de travail.

    *Ce nom doit être unique au sein du locataire.*

    ![](../images/dp500-enforce-model-security-image6.png)

3. Sélectionnez **Enregistrer**.

    *Une fois créé, l’espace de travail est ouvert. Dans un exercice ultérieur, vous allez publier un modèle sémantique dans cet espace de travail.*

### Examiner le modèle de données

Dans cette tâche, vous allez passer en revue le modèle de données.

1. Dans Power BI Desktop, à gauche, passez à la vue **Modèle**.

    ![](../images/dp500-enforce-model-security-image8.png)


2. Utilisez le diagramme de modèle pour passer en revue la conception du modèle.

    ![](../images/dp500-enforce-model-security-image9.png)

    *Le modèle comprend six tables de dimension et une table de faits. La table de faits **Sales** stocke les détails de la commande client. Il s’agit d’une conception classique de schéma en étoile.*

3. Développez la table **Sales Territory**.

    ![](../images/dp500-enforce-model-security-image10.png)

4. Notez que la table inclut une colonne **Region**.

    *La colonne **Region** stocke les régions de ventes d’Adventure Works. Au sein de cette organisation, les commerciaux sont autorisés à voir uniquement les données relatives à leur région de vente. Dans ce labo, vous allez implémenter deux techniques de sécurité au niveau des lignes différentes pour appliquer des autorisations de données.*

## Créer des rôles statiques

Dans cet exercice, vous allez créer et valider des rôles statiques, puis voir comment mapper des principaux de sécurité aux rôles de modèle sémantique.

### Créer des rôles statiques

Dans cette tâche, vous allez créer deux rôles statiques.

1. Passez à l’affichage **Report**.

    ![](../images/dp500-enforce-model-security-image11.png)

2. Dans le visuel de graphique à barres empilées, dans la légende, notez qu’il est possible (pour l’instant) de voir un grand nombre de régions.

    ![](../images/dp500-enforce-model-security-image12.png)

    *Pour l’instant, le graphique semble trop chargé. C’est parce que toutes les régions sont visibles. Lorsque la solution applique la sécurité au niveau des lignes, le consommateur de rapports ne voit qu’une seule région.*


3. Sous l’onglet de ruban **Modélisation**, dans le groupe **Sécurité**, sélectionnez **Gérer les rôles** pour ajouter un rôle de sécurité.

    ![](../images/dp500-enforce-model-security-image13.png)

4. Dans la fenêtre **Gérer les rôles**, sélectionnez **Créer**.

    ![](../images/dp500-enforce-model-security-image14.png)

5. Pour nommer le rôle, remplacez le texte sélectionné par **Australia**, puis appuyez sur **Entrée**.

    ![](../images/dp500-enforce-model-security-image15.png)


6. Dans la liste **Tables**, pour la table **Sales Territory**, sélectionnez les points de suspension, puis **Ajouter un filtre** > **[Region].**

    ![](../images/dp500-enforce-model-security-image16.png)

7. Dans la zone **Expression DAX de filtre de table**, remplacez **Valeur** par **Australia**.

    ![](../images/dp500-enforce-model-security-image17.png)

    *Cette expression filtre la colonne **Region** en fonction de la valeur **Australia**.*

8. Pour créer un autre rôle, appuyez sur **Créer**.

    ![](../images/dp500-enforce-model-security-image18.png)


9. Répétez les étapes de cette tâche pour créer un rôle nommé **Canada** qui filtre la colonne **Region** par **Canada**.

    ![](../images/dp500-enforce-model-security-image19.png)

    *Dans ce labo, vous allez créer uniquement deux rôles. Toutefois, n’oubliez pas que dans une solution réelle, un rôle doit être créé pour chacune des onze régions d’Adventure Works.*

10. Sélectionnez **Enregistrer**.

    ![](../images/dp500-enforce-model-security-image20.png)

### Valider les rôles statiques

Dans cette tâche, vous allez valider l’un des rôles statiques.

1. Sous l’onglet de ruban **Modélisation**, dans le groupe **Sécurité**, sélectionnez **Afficher comme**.

    ![](../images/dp500-enforce-model-security-image21.png)


2. Dans la fenêtre **Afficher comme rôles**, sélectionnez le rôle **Australia**.

    ![](../images/dp500-enforce-model-security-image22.png)

3. Cliquez sur **OK**.

    ![](../images/dp500-enforce-model-security-image23.png)

4. Sur la page du rapport, notez que le visuel de graphique à barres empilées affiche uniquement les données pour l’Australie.

    ![](../images/dp500-enforce-model-security-image24.png)

5. En haut du rapport, la bannière jaune qui confirme le rôle appliqué apparaît.

    ![](../images/dp500-enforce-model-security-image25.png)

6. Pour arrêter l’affichage à partir du rôle, à droite de la bannière jaune, sélectionnez **Arrêter l’affichage**.

    ![](../images/dp500-enforce-model-security-image26.png)

### Publier l’état

Dans cette tâche, vous allez publier le rapport.

1. Enregistrez le fichier Power BI Desktop.

    ![](../images/dp500-enforce-model-security-image27.png)
 

2. Pour publier le rapport, sous l’onglet de ruban **Accueil**, sélectionnez le bouton **Publier**.

    ![](../images/dp500-enforce-model-security-image28.png)

3. Dans la fenêtre **Publier sur Power BI**, sélectionnez votre espace de travail, puis **Sélectionner**.

    ![](../images/dp500-enforce-model-security-image29.png)

4. Une fois la publication réussie, sélectionnez **OK**.

    ![](../images/dp500-enforce-model-security-image30.png)

### Configurer la sécurité au niveau des lignes (*facultatif*)

Dans cette tâche, vous allez voir comment configurer la sécurité au niveau des lignes dans le service Power BI. 

Cette tâche s’appuie sur l’existence d’un groupe de sécurité **Salespeople_Australia** dans le locataire dans lequel vous travaillez. Ce groupe de sécurité N’existe PAS automatiquement dans le locataire. Si vous disposez d’autorisations sur votre locataire, vous pouvez suivre les étapes ci-dessous. Si vous utilisez un locataire fourni pour la formation, vous n’avez pas les autorisations appropriées pour créer des groupes de sécurité. Lisez les tâches, mais notez que vous ne pourrez pas les terminer en l’absence du groupe de sécurité. **Après la lecture, passez à la tâche de nettoyage.**

1. Basculez vers le service Power BI (navigateur web).

2. Dans la page d’arrivée de l’espace de travail, notez le modèle sémantique **Analyse des ventes – Appliquer le modèle de sécurité**.

    ![](../images/dp500-enforce-model-security-image31.png)


3. Placez le curseur sur le modèle sémantique et, quand les points de suspension apparaissent, sélectionnez-les, puis sélectionnez **Sécurité**.

    ![](../images/dp500-enforce-model-security-image32.png)

    *L’option **Sécurité** prend en charge le mappage des principaux de sécurité Microsoft Azure Active Directory (Azure AD), qui incluent des groupes de sécurité et des utilisateurs.*

4. À gauche, examinez la liste des rôles ; vous verrez qu’**Australia** est sélectionné.

    ![](../images/dp500-enforce-model-security-image33.png)

5. Dans la zone **Membres**, commencez à entrer **Salespeople_Australia**. 

    *Les étapes 5 à 8 sont uniquement destinées à des fins de démonstration, car elles s’appuient sur la création ou l’existence d’un groupe de sécurité Salespeople_Australia. Si vous disposez des autorisations et des connaissances nécessaires pour créer des groupes de sécurité, n’hésitez pas à continuer. Sinon, passez à la tâche de nettoyage.*

    ![](../images/dp500-enforce-model-security-image34.png)

6. Sélectionnez **Ajouter**.

    ![](../images/dp500-enforce-model-security-image35.png)

7. Pour terminer le mappage de rôle, sélectionnez **Enregistrer**.

    ![](../images/dp500-enforce-model-security-image36.png)

    *À présent, tous les membres du groupe de sécurité **Salespeople_Australia** sont mappés au rôle **Australia**, ce qui limite l’accès aux données pour afficher uniquement les ventes australiennes.*

    *Dans une solution réelle, chaque rôle doit être mappé à un groupe de sécurité.*

    *Cette approche de conception est simple et efficace quand des groupes de sécurité existent pour chaque région. Il existe cependant des inconvénients : créer et configurer demande plus de travail. Elle nécessite également de mettre à jour et de republier le modèle sémantique quand de nouvelles régions sont intégrées.*

    *Dans l’exercice suivant, vous allez créer un rôle dynamique piloté par les données. Cette approche de conception peut aider à résoudre ces inconvénients.*

8. Pour revenir à la page d’accueil de l’espace de travail, dans le volet **Navigation**, sélectionnez l’espace de travail.


### Nettoyer la solution

Dans cette tâche, vous allez nettoyer la solution en supprimant le modèle sémantique et les rôles de modèle.

1. Pour supprimer le modèle sémantique, placez le curseur sur le modèle sémantique et, quand les points de suspension apparaissent, sélectionnez-les, puis sélectionnez **Supprimer**.

    ![](../images/dp500-enforce-model-security-image37.png)

    *Vous allez republier un modèle sémantique révisé dans l’exercice suivant.*

2. Lorsque vous êtes invité à confirmer la suppression, sélectionnez **Supprimer**.

    ![](../images/dp500-enforce-model-security-image38.png)

3. Revenez à Power BI Desktop.
 

4. Pour supprimer les rôles de sécurité, sous l’onglet de ruban **Modélisation**, dans le groupe **Sécurité**, sélectionnez **Gérer les rôles**.

    ![](../images/dp500-enforce-model-security-image39.png)

5. Dans la fenêtre **Gérer les rôles**, pour supprimer le premier rôle, sélectionnez **Supprimer**.

    ![](../images/dp500-enforce-model-security-image40.png)

6. Lorsque vous êtes invité à confirmer la suppression, sélectionnez **Oui, supprimer**.

    ![](../images/dp500-enforce-model-security-image41.png)

7. Supprimez également le deuxième rôle.

8. Sélectionnez **Enregistrer**.

    ![](../images/dp500-enforce-model-security-image42.png)


## Créer un rôle dynamique

Dans cet exercice, vous allez ajouter une table au modèle, créer et valider un rôle dynamique, puis mapper un principal de sécurité au rôle de modèle sémantique.

### Créer la table Salesperson

Dans cette tâche, vous allez ajouter la table **Salesperson** au modèle.

1. Basculez vers la vue **Modèle**.

    ![](../images/dp500-enforce-model-security-image43.png)

2. Sous l’onglet de ruban **Accueil**, dans le groupe **Requêtes**, sélectionnez l’icône **Transformer les données**.

    ![](../images/dp500-enforce-model-security-image44.png)


3. Dans la fenêtre **Éditeur Power Query**, dans le volet **Requêtes** (situé à gauche), cliquez avec le bouton droit sur la requête **Customer**, puis sélectionnez **Dupliquer**.

    ![](../images/dp500-enforce-model-security-image45.png)

    *Étant donné que la requête **Customer** inclut déjà des étapes pour connecter l’entrepôt de données, la duplication est un moyen efficace de commencer le développement d’une nouvelle requête.*

4. Dans le volet **Paramètres de la requête** (situé à droite), dans la zone **Nom**, remplacez le texte par **Salesperson**.

    ![](../images/dp500-enforce-model-security-image46.png)


5. Dans la liste **Étapes appliquées**, cliquez avec le bouton droit sur l’étape **Supprimer d’autres colonnes** (troisième étape), puis sélectionnez **Supprimer jusqu’à la fin**.

    ![](../images/dp500-enforce-model-security-image47.png)

6. Lorsque vous êtes invité à confirmer la suppression, sélectionnez **Supprimer**.

    ![](../images/dp500-enforce-model-security-image48.png)

7. Pour sourcer des données à partir d’une autre table d’entrepôt de données, dans la liste **Étapes appliquées**, dans l’étape **Navigation** (deuxième étape), sélectionnez l’icône d’engrenage (située à droite).

    ![](../images/dp500-enforce-model-security-image49.png)

8. Dans la fenêtre **Navigation**, sélectionnez la table **DimEmployee**.

    ![](../images/dp500-enforce-model-security-image50.png)


9. Cliquez sur **OK**.

    ![](../images/dp500-enforce-model-security-image51.png)

10. Pour supprimer des colonnes inutiles, sous l’onglet de ruban **Accueil**, dans le groupe **Gérer les colonnes**, sélectionnez l’icône **Choisir les colonnes**.

    ![](../images/dp500-enforce-model-security-image52.png)

11. Dans la fenêtre **Choisir des colonnes**, décochez l’élément **(Sélectionner toutes les colonnes)**.

    ![](../images/dp500-enforce-model-security-image53.png)

12. Cochez les trois colonnes suivantes :

    - EmployeeKey

    - SalesTerritoryKey

    - EmailAddress

13. Cliquez sur **OK**.

    ![](../images/dp500-enforce-model-security-image54.png)

14. Pour renommer la colonne **EmailAddress**, double-cliquez sur l’en-tête de colonne **EmailAddress**.

15. Remplacez le texte par **UPN**, puis appuyez sur **Entrée**.

    *UPN est l’acronyme de User Principal Name (nom d’utilisateur principal). Les valeurs de cette colonne correspondent aux noms de compte Azure AD.*

    ![](../images/dp500-enforce-model-security-image55.png)

16. Pour charger la table dans le modèle, sous l’onglet de ruban **Accueil**, sélectionnez l’icône **Fermer &amp; appliquer**.

    ![](../images/dp500-enforce-model-security-image56.png)

17. Lorsque la table a été ajoutée au modèle, notez qu’une relation avec la table **Sales Territory** a été créée automatiquement.

### Configurer la relation

Dans cette tâche, vous allez configurer les propriétés de la nouvelle relation.

1. Cliquez avec le bouton droit sur la relation entre les tables **Salesperson** et **Sales Territory**, puis sélectionnez **Propriétés**.

    ![](../images/dp500-enforce-model-security-image57.png)


2. Dans la liste déroulante **Direction du filtre croisé** de la fenêtre **Modifier la relation**, sélectionnez **Les deux**.

3. Cochez la case **Appliquer le filtre de sécurité dans les deux directions**.

    ![](../images/dp500-enforce-model-security-image58.png)

    *Étant donné qu’il existe une relation un-à-plusieurs entre la table **Sales Territory** et la table **Salesperson**, les filtres se propagent uniquement de la table **Sales Territory** à la table **Salesperson**. Pour forcer la propagation dans l’autre sens, la direction du filtrage croisé doit être définie sur les deux.*

4. Cliquez sur **OK**.

    ![](../images/dp500-enforce-model-security-image59.png)

5. Pour masquer la table, en haut à droite de la table **Salesperson**, sélectionnez l’icône d’œil.

    ![](../images/dp500-enforce-model-security-image60.png)

    *L’objectif de la table **Salesperson** est d’appliquer des autorisations de données. Lorsqu’ils sont masqués, les créateurs de rapports et l’expérience Q&A ne voient pas la table ou ses champs.*
 

### Créer un rôle dynamique

Dans cette tâche, vous allez créer un rôle dynamique, qui applique des autorisations basées sur des données dans le modèle.

1. Passez à l’affichage **Report**.

    ![](../images/dp500-enforce-model-security-image61.png)

2. Sous l’onglet de ruban **Modélisation**, dans le groupe **Sécurité**, sélectionnez **Gérer les rôles** pour ajouter un rôle de sécurité.

    ![](../images/dp500-enforce-model-security-image62.png)

3. Dans la fenêtre **Gérer les rôles**, sélectionnez **Créer**.

    ![](../images/dp500-enforce-model-security-image63.png)

4. Pour nommer le rôle, remplacez le texte sélectionné par **Salespeople**.

    ![](../images/dp500-enforce-model-security-image64.png)

    *Cette fois, un seul rôle doit être créé.*

5. Ajoutez un filtre à la colonne **UPN** de la table **Salesperson**.

    ![](../images/dp500-enforce-model-security-image65.png)

6. Dans la zone **Expression DAX de filtre de table**, remplacez **"Value"** par **USERPRINCIPALNAME()**.

    ![](../images/dp500-enforce-model-security-image66.png)

    *Cette expression filtre la colonne **UPN** par la fonction USERPRINCIPALNAME, qui retourne le nom d’utilisateur principal (UPN) de l’utilisateur authentifié.*

    *Lorsque l’UPN filtre la table **Salesperson**, elle filtre la table **Sales Territory**, qui filtre à son tour la table **Sales**. De cette manière, l’utilisateur authentifié voit uniquement les données de vente de sa région.*

7. Sélectionnez **Enregistrer**.

    ![](../images/dp500-enforce-model-security-image67.png)

### Valider le rôle dynamique

Dans cette tâche, vous allez valider le rôle dynamique.

1. Sous l’onglet de ruban **Modélisation**, dans le groupe **Sécurité**, sélectionnez **Afficher comme**.

    ![](../images/dp500-enforce-model-security-image68.png)


2. Dans la fenêtre **Afficher comme rôles**, cochez **Autre utilisateur**, puis dans la zone correspondante, saisissez : **michael9@adventure-works.com**.

    ![](../images/dp500-enforce-model-security-image69.png)

    *À des fins de test, **Autre utilisateur** est la valeur qui sera renvoyée par la fonction USERPRINCIPALNAME. Notez que ce vendeur est affecté à la région **Northeast**.*

3. Sélectionnez le rôle **Salespeople**.

    ![](../images/dp500-enforce-model-security-image70.png)

4. Cliquez sur **OK**.

    ![](../images/dp500-enforce-model-security-image71.png)

5. Sur la page de rapport, notez que le visuel de graphique à barres empilées affiche uniquement les données de la région Nord-est.

    ![](../images/dp500-enforce-model-security-image72.png)

6. En haut du rapport, la bannière jaune qui confirme le rôle appliqué apparaît.

    ![](../images/dp500-enforce-model-security-image73.png)


7. Pour arrêter l’affichage à partir du rôle, à droite de la bannière jaune, sélectionnez **Arrêter l’affichage**.

    ![](../images/dp500-enforce-model-security-image74.png)

### Finaliser la conception

Dans cette tâche, vous allez finaliser la conception en publiant le rapport et en mappant un groupe de sécurité au rôle.

*Les étapes décrites dans cette tâche sont délibérément brèves. Pour plus d’informations, reportez-vous aux étapes de tâche de l’exercice précédent.*

1. Enregistrez le fichier Power BI Desktop.

    ![](../images/dp500-enforce-model-security-image75.png)

2. Publiez le rapport sur l’espace de travail que vous avez créé au début du labo. 

3. Fermez Power BI Desktop.

4. Basculez vers le service Power BI (navigateur web).

5. Accédez aux paramètres pour le modèle sémantique **Analyse des ventes – Appliquer la sécurité du modèle**.

6. Mappez le groupe de sécurité **Salespeople** au rôle **Salespeople**.

    ![](../images/dp500-enforce-model-security-image76.png)

    *À présent, tous les membres du groupe de sécurité **Salespeople** sont mappés au rôle **Salespeople**. Si l’utilisateur authentifié est représenté par une ligne dans la table **Salesperson**, le territoire de vente affecté sera utilisé pour filtrer la table des ventes.*

    *Cette approche de conception est simple et efficace lorsque le modèle de données stocke les valeurs de nom d’utilisateur principal (UPN). Lorsque des commerciaux sont ajoutés ou supprimés, ou qu’ils sont affectés à différents territoires de vente, cette approche de conception fonctionne simplement.*
