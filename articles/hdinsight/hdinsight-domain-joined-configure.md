---
title: "Configuration des clusters HDInsight joints à un domaine | Microsoft Docs"
description: "Découvrez comment configurer les clusters HDInsight joints à un domaine"
services: hdinsight
documentationcenter: 
author: saurinsh
manager: jhubbard
editor: cgronlun
tags: 
ms.assetid: 0cbb49cc-0de1-4a1a-b658-99897caf827c
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 11/02/2016
ms.author: saurinsh
translationtype: Human Translation
ms.sourcegitcommit: 86a0f6f2bc27f1411652b273325e73144582eee0
ms.openlocfilehash: b0122a87ec64d16d6e026f9b37a563125a5f1920


---
# <a name="configure-domain-joined-hdinsight-clusters-preview"></a>Configurer des clusters HDInsight joints à un domaine (version préliminaire)
Découvrez comment configurer un cluster Azure HDInsight avec Azure Active Directory (Azure AD) et [Apache Ranger](http://hortonworks.com/apache/ranger/)pour valoriser les stratégies d’authentification forte et de contrôle de l’accès enrichi basé sur les rôles.  Le cluster HDInsight peut être configuré uniquement sur les clusters Linux. Pour plus d’informations, consultez [Introduire des clusters HDInsight joints à un domaine](hdinsight-domain-joined-introduction.md).

Cet article est le premier didacticiel d’une série :

* Créez un cluster HDInsight connecté à Azure AD (via la fonctionnalité des services de domaines Azure Directory) avec Apache Ranger activé.
* Créez et appliquez les stratégies Hive via Apache Ranger et autorisez les utilisateurs (par exemple, les scientifiques de données) à se connecter à Hive à l’aide des outils ODBC, par exemple Excel, tableau, etc. Microsoft travaille à l’ajout d’autres charges de travail, comme HBase, Spark et Storm sur le cluster HDInsight joint à un domaine.

Voici un exemple de la topologie finale :

![Topologie HDInsight jointe à un domaine](./media/hdinsight-domain-joined-configure/hdinsight-domain-joined-topology.png)

Azure AD ne prenant en charge que les réseaux virtuels Classic et les clusters HDInsight Linux ne prenant en charge que les réseaux virtuels Azure Resource Manager, l’intégration HDInsight Azure AD requiert deux réseaux virtuels et une homologation mutuelle. Pour bénéficier d’une comparaison entre les deux modèles de déploiement, consultez [Déploiement Azure Resource Manager et déploiement classique : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/resource-manager-deployment-model.md). Les deux réseaux virtuels doivent se trouver dans la région de l’instance Azure AD DS.

Les noms de service Azure doivent être globalement uniques. Les noms suivants sont utilisés dans ce didacticiel. Contoso est un nom fictif. Vous devez remplacer *contoso* par un nom différent quand vous parcourez le didacticiel. 

**Noms :**

| Propriété | Valeur |
| --- | --- |
| Réseau virtuel Azure AD |contosoaadvnet |
| Groupe de ressources du réseau virtuel Azure AD |contosoaadrg |
| Machine virtuelle Azure AD |contosoaadadmin. Cette machine virtuelle est utilisée pour configurer la zone de l’unité d’organisation et du DNS renversé. |
| Annuaire Azure AD |contosoaaddirectory |
| Nom de domaine Azure AD |contoso (contoso.onmicrosoft.com) |
| Réseau virtuel HDInsight |contosohdivnet |
| Groupe de ressources du réseau virtuel HDInsight |contosohdirg |
| Cluster HDInsight |contosohdicluster |

Ce didacticiel vous indique la procédure de configuration d’un cluster HDInsight joint à un domaine. Chaque section comporte des liens vers d’autres articles avec des informations contextuelles supplémentaires.

## <a name="prerequisite"></a>Configuration requise :
* Familiarisez-vous avec les [services de domaine Azure AD](https://azure.microsoft.com/services/active-directory-ds/), et la structure de [tarification](https://azure.microsoft.com/pricing/details/active-directory-ds/).
* Assurez-vous que votre abonnement est dans la liste approuvée de cette version préliminaire publique. Pour ce faire, envoyez un e-mail à hdipreview@microsoft.com, avec votre ID d’abonnement.
* Un certificat SSL signé par une autorité signataire pour votre domaine. Le certificat est requis pour la configuration du LDAP sécurisé. Les certificats auto-signés ne peuvent pas être utilisés.

## <a name="procedures"></a>Procédures
1. Créez un réseau virtuel Azure pour votre instance Azure AD.  
2. Créez et configurez Azure AD et Azure AD AS.
3. Ajoutez une machine virtuelle au réseau virtuel Classic pour la création d’unité d’organisation. 
4. Créez une unité d’organisation pour Azure AD AS.
5. Créez un réseau virtuel HDInsight dans le mode Resource Management Azure.
6. Configurez les zones DNS inversé pour l’instance Azure AD AS.
7. Homologuez les deux réseaux virtuels.
8. Créez un cluster HDInsight.

> [!NOTE]
> Ce didacticiel part du principe que vous ne possédez pas d’instance Azure AD. Si vous en possédez un, vous pouvez passer la section de l’étape 2.
> 
> 

Il existe un script PowerShell qui automatise les étapes 3 à 7.  Pour plus d’informations, consultez la section [Configurer des clusters HDInsight joints à un domaine à l’aide d’Azure PowerShell](hdinsight-domain-joined-configure-use-powershell.md).

## <a name="create-an-azure-classic-vnet"></a>Créer un réseau virtuel Azure Classic
Dans cette section, vous créez un réseau virtuel classique à l’aide du portail Azure. Dans la section suivante, vous activez l’instance Azure AD DS pour Azure AD dans le réseau virtuel Classic. Pour plus d’informations sur la procédure suivante et l’utilisation d’autres méthodes de création de réseaux virtuels, consultez la section [Créer un réseau virtuel (classique) à l’aide du portail Azure](../virtual-network/virtual-networks-create-vnet-classic-portal.md).

**Pour créer un réseau virtuel classique**

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Cliquez sur **Nouveau** > **Mise en réseau** > **Réseau virtuel**.
3. Dans **Sélectionner un modèle de déploiement**, sélectionnez **Classique**, puis cliquez sur **Créer**.
4. Entrez ou sélectionnez les valeurs suivantes :
   
   * **Nom** : contosoaadvnet
   * **Espace d’adressage** : 10.1.0.0/16
   * **Nom du sous-réseau** : Subnet1
   * **Plage d’adresses de sous-réseau** : 10.1.0.0/24
   * **Abonnement** : (sélectionnez un abonnement utilisé pour la création de ce réseau virtuel.)
   * **ResourceGroup** : contosoaadrg
   * **Emplacement** : (Sélectionnez une région pour votre cluster HDInsight.)
     
     > [!IMPORTANT]
     > Vous devez choisir un emplacement qui prend en charge Azure AD DS. Pour plus d’informations, consultez [Disponibilité des produits par région](https://azure.microsoft.com/en-us/regions/services/). 
     > 
     > Le réseau virtuel classique et le réseau virtuel du groupe de ressources doivent se trouver dans la région de l’instance Azure AD DS.
     > 
     > 
5. Pour créer le réseau virtuel, cliquez sur **Créer**.

## <a name="create-and-configure-azure-ad-ds-for-your-azure-ad"></a>Créez et configurer Azure AD AS pour votre instance Azure AD
Dans cette section, vous allez :

1. Créer une application Azure AD.
2. Créer des utilisateurs Azure AD. Ces utilisateurs sont des utilisateurs du domaine. Vous utilisez le premier utilisateur pour la configuration du cluster HDInsight avec l’application Azure AD.  Les deux autres utilisateurs sont facultatifs pour ce didacticiel. Ils seront utilisés dans la section [Configuration de stratégies Hive pour les clusters HDInsight joints à un domaine](hdinsight-domain-joined-run-hive.md), au moment de la configuration des stratégies Apach Ranger.
3. Créez le groupe AAD DC Administrator, puis ajoutez l’utilisateur Azure AD au groupe. Cet utilisateur vous permet de créer l’unité d’organisation.
4. Activez les services de domaine Azure AD (Azure AD DS) pour l’instance Azure AD.
5. Configurez LDAPS pour l’instance Azure AD. Le protocole LDAP (Lightweight Directory Access Protocol) est utilisé pour la lecture et l’écriture sur Azure AD.

Si vous préférez utiliser une instance Azure AD existante, vous pouvez passer les étapes 1 et 2.

**Pour créer une instance Azure AD**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Nouveau** > **App Services** > **Active Directory** > **Répertoire** > **Création personnalisée**. 
2. Entrez ou sélectionnez les valeurs suivantes :
   
   * **Nom** : contosoaaddirectory
   * **Nom de domaine** : contoso.  Ce nom doit être globalement unique.
   * **Pays ou région** : sélectionnez votre pays ou région.
3. Cliquez sur **Terminé**.

**Créer un utilisateur Azure AD**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Active Directory** -> **contosoaaddirectory**. 
2. Cliquez sur **Utilisateurs** dans le menu supérieur.
3. Cliquez sur **Add User**.
4. Saisissez **Nom d’utilisateur**, puis cliquez sur **Suivant**. 
5. Configurez le profil utilisateur ; dans **Rôle**, sélectionnez **Administrateur global**, puis cliquez sur **Suivant**.  Le rôle d’administrateur global est requis pour la création des unités d’organisation.
6. Cliquez sur **Créer** pour récupérer un mot de passe temporaire.
7. Faites une copie du mot de passe, puis cliquez sur **Terminé**. Plus loin dans ce didacticiel, vous utiliserez cet utilisateur administrateur global afin de vous connecter à la machine virtuelle administrateur, pour la création d’une unité d’organisation et la configuration du DNS inversé.

Suivez la même procédure pour créer deux utilisateurs supplémentaires avec le rôle **Utilisateur**, hiveuser1 et hiveuser2. Les utilisateurs suivants seront utilisés dans [Configuration de stratégies Hive pour les clusters HDInsight joints à un domaine](hdinsight-domain-joined-run-hive.md).

**Pour créer le groupe AAD DC Administrators et ajouter un utilisateur Azure AD**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Active Directory** > **contosoaaddirectory**. 
2. Cliquez sur **Groupes** dans le menu supérieur.
3. Cliquez sur **Ajouter un groupe** ou ****.
4. Entrez ou sélectionnez les valeurs suivantes :
   
   * **Nom** : AAD DC Administrators.  Ne modifiez pas le nom du groupe.
   * **Type de groupe** : Sécurité.
5. Cliquez sur **Terminé**.
6. Cliquez sur **AAD DC Administrators** pour ouvrir le groupe.
7. Cliquez sur **Ajouter des membres**.
8. Sélectionnez le premier utilisateur créé lors de l’étape précédente, puis cliquez sur **Terminé**.
9. Répétez la procédure afin de créer un autre groupe appelé **HiveUsers**, puis ajoutez les deux utilisateurs Hive au groupe.

Pour plus d’informations, consultez [Services de domaine Azure AD (version préliminaire) - Créer le groupe « AAD DC Administrators »](../active-directory-domain-services/active-directory-ds-getting-started.md).

**Pour activer Azure AD DS pour votre instance Azure AD**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Active Directory** > **contosoaaddirectory**. 
2. Cliquez sur **Configurer** dans le menu supérieur.
3. Faites défiler jusqu’à **Services de domaine**, puis définissez les valeurs suivantes :
   
   * **Activer les services de domaine pour cet annuaire** : Oui.
   * **Nom de domaine DNS des services de domaine** : affiche le nom DNS par défaut du répertoire Azure. Par exemple, contoso.onmicrosoft.com.
   * **Connecter les services de domaine à ce réseau virtuel** : sélectionnez le réseeau virtuel Classic créé précédemment, comme **contosoaadvnet**.
4. Cliquez sur **Enregistrer** dans la partie inférieure de la page. L’indication **En attente ...** s’affichera en regard de **Activer les services de domaine pour cet annuaire**.  
5. Attendez que l’indication **En attente ...** disparaisse et que le champ **Adresse IP** soit renseigné. Deux adresses IP sont renseignées. Il s’agit des adresses IP des contrôleurs de domaine configurés par les services de domaine. Chaque adresse IP est visible une fois que le contrôleur de domaine correspondant est configuré et prêt. Notez les deux adresses IP. Vous en aurez besoin ultérieurement.

)Pour plus d’informations, consultez [Services de domaine Azure AD (version préliminaire) - Activer Azure AD Domain Services](../active-directory-domain-services/active-directory-ds-getting-started-enableaadds.md).

**Pour synchroniser le mot de passe**

Si vous utilisez votre propre domaine, vous devez synchroniser le mot de passe. Consultez [Enable password synchronization to Azure AD domain services for a cloud-only Azure AD directory (Activer la synchronisation des mots de passe sur les services de domaine Azure AD pour un répertoire Azure AD uniquement dans le cloud)](../active-directory-domain-services/active-directory-ds-getting-started-password-sync.md).

**Pour configurer LDAPS pour l’instance Azure AD**

1. Récupérez un certificat SSL signé par une autorité signataire pour votre domaine. Les certificats auto-signés ne peuvent pas être utilisés. Si vous ne pouvez pas obtenir dee certificat SSL, veuillez contacter hdipreview@microsoft.com pour solliciter une exception.
2. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Active Directory** > **contosoaaddirectory**. 
3. Cliquez sur **Configurer** dans le menu supérieur.
4. Faites défiler jusqu’à **services de domaine**.
5. Cliquez sur **Configurer le certificat**.
6. Suivez les instructions de configuration du fichier de certificat et du mot de passe. L’indication **En attente ...** s’affichera en regard de **Activer les services de domaine pour cet annuaire**.  
7. Attendez que l’indication **En attente...** disparaisse, puis que le champ **Certificat LDAP sécurisé** soit renseigné.  Cela peut prendre jusqu’à 10 minutes ou plus.

> [!NOTE]
> Si certaines tâches d’arrière-plan sont exécutées sur l’instance Azure AD DS, vous pouvez rencontrer une erreur lors du chargement du certificat - <i>Une opération est exécutée pour ce client. Veuillez réessayer plus tard</i>.  Si vous rencontrez cette erreur, veuillez réessayer plus tard. La configuration du second contrôleur de domaine peut prendre jusqu’à 3 heures.
> 
> 

Pour plus d’informations, consultez [Configurer le protocole LDAPS (LDAP sécurisé) pour un domaine géré par les services de domaine Azure AD](../active-directory-domain-services/active-directory-ds-admin-guide-configure-secure-ldap.md).

## <a name="configure-an-organizational-unit-and-reverse-dns"></a>Configurer une unité d’organisation et le DNS inversé
Dans cette section, vous ajoutez une machine virtuelle au réseau virtuel Azure AD et installez vos outils administratifs sur cette machine virtuelle de manière à pouvoir configurer une unité d’organisation et le DNS inversé. La recherche de DNS inversé est requise pour l’authentification Kerberos.

**Pour créer une machine virtuelle dans le réseau virtuel**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Nouveau** > **Compute** > **Machine virtuelle** > **Depuis la galerie**.
2. Sélectionnez une image, puis cliquez sur **Suivant**.  Si vous ne savez pas laquelle utiliser, sélectionnez l’élément par défaut, **Windows Server 2012 R2 Datacenter**.
3. Entrez ou sélectionnez les valeurs suivantes :
   
   * Nom de la machine virtuelle : **contosoaadadmin**
   * Niveau : **De base**
   * Nouveau nom d’utilisateur : (saisissez un nom d’utilisateur)
   * Mot de passe : (saisissez un mot de passe)
     
     Notez que le nom d’utilisateur et le mot de passe sont associés à l’administrateur local.
4. Cliquez sur **Suivant**
5. Dans **Région/Réseau virtuel**, sélectionnez le nouveau réseau virtuel créé lors de l’étape précédente (contosoaadvnet), puis cliquez sur **Suivant**.
6. Cliquez sur **Terminé**.

**Pour établir une connexion RDP vers la machine virtuelle**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), cliquez sur **Machines virtuelles** > **contosoaadadmin**.
2. Cliquez sur **Tableau de bord** dans le menu du haut.
3. Cliquez sur **Se connecter** dans la partie inférieure de la page.
4. Suivez les instructions et utilisez le nom d’utilisateur et le mot de passe de l’administrateur local pour vous connecter.

**Pour joindre la machine virtuelle au domaine Azure AD**

1. Dans la session RDP, cliquez sur **Démarrer**, puis cliquez sur **Gestionnaire de serveur**.
2. Dans le volet gauche, cliquez sur **Serveur local**.
3. Dans le groupe de travail, cliquez sur **Groupe de travail**.
4. Cliquez sur **Modifier**.
5. Cliquez sur **Domaine**, saisissez **contoso.onmicrosoft.com**, puis cliquez sur **OK**.
6. Saisissez les informations d’identification de l’utilisateur du domaine, puis cliquez sur **OK**.
7. Cliquez sur **OK**.
8. Cliquez sur **OK** afin de confirmer le redémarrage de l’ordinateur.
9. Cliquez sur **Fermer**.
10. Cliquez sur **Redémarrer maintenant**.

Pour plus d’informations, consultez [Joindre une machine virtuelle Windows Server à un domaine géré](../active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm.md).

**Pour installer les outils d’administration Active Directory et les outils DNS**

1. Établissez une connexion RDP vers **contosoaadadmin** à l’aide du compte utilisateur Azure AD.
2. Cliquez sur **Démarrer**, puis sur **Gestionnaire de serveur**.
3. Cliquez sur **Tableau de bord** dans le menu de gauche.
4. Cliquez sur **Gérer**, puis sur **Ajoutez des rôles et des fonctionnalités**.
5. Cliquez sur **Next**.
6. Sélectionnez **Installation basée sur un rôle ou une fonctionnalité**, puis cliquez sur **Suivant**.
7. Sélectionnez la machine virtuelle actuelle dans le pool de serveurs, puis cliquez sur **Suivant**.
8. Cliquez sur **Suivant** pour ignorer les rôles.
9. Développez **Outils d’administration de serveur distant**, développez **Outils d’administration de rôles**, sélectionnez **Outils AD DS et AD LDS** et **Outils du serveur DNS**, puis cliquez sur **Suivant**. 
10. Cliquez sur **Suivant**
11. Cliquez sur **Installer**.

Pour plus d’informations, consultez la section [Installer les Outils d’administration Azure Directory sur la machine virtuelle](../active-directory-domain-services/active-directory-ds-admin-guide-administer-domain.md#task-2---install-active-directory-administration-tools-on-the-virtual-machine).

**Pour configurer le DNS inversé**

1. Établissez une connexion RDP vers contosoaadadmin à l’aide du compte utilisateur Azure AD.
2. Cliquez sur **Démarrer**, sur **Outils d’administration**, puis sur **DNS**. 
3. Cliquez sur **Non** pour ignorer l’ajout de ContosoAADAdmin.
4. Sélectionnez **L’ordinateur suivant**, saisissez l’adresse IP du premier serveur DNS configuré précédemment, puis cliquez sur **OK**.  Vous devriez voir que le contrôleur de domaine/DNS est ajouté au volet de gauche.
5. Développez le serveur contrôleur de domaine/DNS, cliquez avec le bouton droit sur **Zones de recherche inversée**, puis cliquez sur **Nouvelle zone**. L’Assistant Nouvelle Zone s’ouvre.
6. Cliquez sur **Suivant**.
7. Sélectionnez **Zone principale**, puis cliquez sur **Suivant**.
8. Sélectionnez **To all DNS servers running on domain controllers in this domain (À tous les serveurs DNS exécutés sur les contrôleurs de domaine de ce domaine)**, puis cliquez sur **Suivant**.
9. Sélectionnez **Zone de recherche inversée IPv4**, puis cliquez sur **Suivant**.
10. Dans **ID réseau**, saisissez le préfixe de la plage de réseau virtuel HDInsight, puis cliquez sur **Suivant**. La section suivante vous explique la création du réseau virtuel HDInsight.
11. Cliquez sur **Suivant**.
12. Cliquez sur **Suivant**.
13. Cliquez sur **Terminer**.

L’unité d’organisation créée par la suite sera utilisée lors de la création du cluster HDInsight. Les utilisateurs et les comptes d’ordinateurs du système Hadoop seront placés dans cette unité d’organisation.

**Créer une UO sur un domaine géré par les services de domaine Azure Active Directory**

1. Établissez une connexion RDP à **contosoaadadmin** à l’aide du compte de domaine du groupe **AAD DC Administrators**.
2. Cliquez sur **Démarrer**, sur **Outils d’administration**, puis sur **Centre d’administration Active Directory**.
3. Cliquez sur le nom de domaine du volet de gauche. Par exemple, contoso.
4. Cliquez sur **Nouveau** sous le nom de domaine du volet **Tâche**, puis sur **Unité d’organisation**.
5. Saisissez un nom, par exemple **HDInsightOU**, puis cliquez sur **OK**. 

Pour plus d’informations, consultez [Créer une UO sur un domaine géré par les services de domaine Azure Active Directory](../active-directory-domain-services/active-directory-ds-admin-guide-create-ou.md).

## <a name="create-a-resource-manager-vnet-for-hdinsight-cluster"></a>Créer un réseau virtuel Resource Manager pour le cluster HDInsight
Dans cette section, vous allez créer un réseau virtuel Azure Resource Manager qui sera utilisé pour le cluster HDInsight. Pour plus d’informations sur la création d’un réseau virtuel Azure à l’aide d’autres méthodes, consultez la section [Créez un réseau virtuel](../virtual-network/virtual-networks-create-vnet-arm-pportal.md).

Une fois que vous avez créé le réseau virtuel Resource Manager, vous le configurez pour l’utilisation des serveurs DNS associés au réseau virtuel Azure AD. Si vous avez suivi les étapes de ce didacticiel relatives à la création du réseau virtuel classique et de l’instance Azure AD, les serveurs DNS sont 10.1.0.4 et 10.1.0.5.

**Pour créer un réseau virtuel Resource Manager**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Nouveau**, **Mise en réseau**, puis sur **Réseau virtuel**. 
3. Sous **Sélectionnez un modèle de déploiement**, sélectionnez **Resource Manager**, puis cliquez sur **Créer**.
4. Tapez ou sélectionnez les valeurs suivantes :
   
   * **Nom** : contosohdivnet
   * **Espace d’adressage** : 10.2.0.0/16. Assurez-vous que la plage d’adresses ne peut pas se chevaucher avec la plage d’adresses IP du réseau virtuel classique.
   * **Nom du sous-réseau** : Subnet1
   * **Plage d’adresses du sous-réseau** : 10.2.0.0/24
   * **Abonnement** : (sélectionnez votre abonnement Azure.)
   * **Groupe de ressources** : contosohdirg
   * **Emplacement** : (sélectionnez l’emplacement du réseau virtuel Azure AD, par exemple contosoaadvnet.)
5. Cliquez sur **Create**.

**Pour configurer DNS pour le réseau virtuel Resource Manager**

1. À partir du [portail Azure](https://portal.azure.com), cliquez sur **Plus de services** -> **Réseaux virtuels**. Assurez-vous de ne pas cliquer sur **Réseaux virtuels (classiques)**.
2. Cliquez sur **contosohdivnet**.
3. Cliquez sur **Serveurs DNS** sur le côté gauche du nouveau panneau.
4. Cliquez sur **Personnaliser**, puis saisissez les valeurs suivantes :
   
   * 10.1.0.4
   * 10.1.0.5
     
     Ces adresses IP de serveurs DNS doivent correspondre à celles des serveurs DNS du réseau virtuel Azure AD (réseau virtuel classique).
5. Cliquez sur **Save**.

## <a name="peer-the-azure-ad-vnet-and-the-hdinsight-vnet"></a>Appairez le réseau virtuel Azure AD et le réseau virtuel HDInsight
**Pour appairer les deux réseaux virtuels**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Plus de services** dans le menu de gauche.
3. Cliquez sur **Réseaux virtuels**. Ne cliquez pas sur **Réseaux virtuels (classiques)**.
4. Cliquez sur **contosohdivnet**.  Il s’agit du réseau virtuel HDInsight.
5. Cliquez sur **Homologations** dans le menu gauche du panneau.
6. Cliquez sur **Ajouter** dans le menu supérieur. Le panneau **Ajouter l’homologation** s’ouvre.
7. Sur le panneau **Ajouter l’homologation**, définissez ou sélectionnez les valeurs suivantes :
   
   * **Nom** : ContosoAADHDIVNetPeering
   * **Modèle de déploiement de réseau virtuel** : classique
   * **Abonnement** : sélectionnez l’abonnement utilisé pour le réseau virtuel classique (Azure AD).
   * **Réseau virtuel** : contosoaadvnet.
   * **Autoriser l’accès au réseau virtuel** : (vérification)
   * **Autoriser le trafic transféré** : (vérification). Laissez les deux autres cases non sélectionnées.
8. Cliquez sur **OK**.

## <a name="create-hdinsight-cluster"></a>Créer un cluster HDInsight
Dans cette section, vous créez un cluster Hadoop Linux dans HDInsight à l’aide du portail Azure ou du [modèle Azure Resource Manager](../azure-resource-manager/resource-group-template-deploy.md). Pour d'autres méthodes de création de cluster et comprendre les paramètres, consultez [Création de clusters HDInsight](hdinsight-hadoop-provision-linux-clusters.md). Pour plus d’informations sur l’utilisation d’un modèle Resource Manager pour créer des clusters Hadoop dans HDInsight, consultez [Création de clusters Hadoop dans HDInsight à l’aide de modèles Azure Resource Manager](hdinsight-hadoop-create-windows-clusters-arm-templates.md).

**Pour créer un cluster HDInsight joint à un domaine à l’aide du portail Azure**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Nouveau**, **Intelligence et analyse**, puis sur **HDInsight**.
3. Dans le panneau **Nouveau cluster HDInsight**, entrez ou sélectionnez les valeurs suivantes :
   
   * **Nom du cluster** : saisissez un nom pour le cluster HDInsight joint au domaine.
   * **Abonnement** : sélectionnez l’abonnement Azure utilisé pour créer ce cluster.
   * **Configuration du cluster** :
     
     * **Type de cluster** : Hadoop Actuellement, le cluster HDInsight joint au domaine est uniquement pris en charge sur les clusters Hadoop.
     * **Système d’exploitation** : Linux  Le cluster HDInsight joint au domaine est pris en charge uniquement sur les clusters HDInsight Linux.
     * **Version** : Hadoop 2.7.3 (HDI 3.5). Le cluster HDInsight joint au domaine est pris en charge uniquement sur le cluster HDInsight version 3.5.
     * **Type de cluster** : PREMIUM
       
       Pour enregistrer les modifications, cliquez sur **Sélectionner**.
   * **Informations d’identification** : configurez les informations d’identification pour les utilisateurs du cluster et SSH.
   * **Source de données** : créez un compte de stockage ou utilisez un compte de stockage existant en tant que compte de stockage par défaut pour le cluster HDInsight. L’emplacement doit être celui des deux réseaux virtuels.  Il s’agit également de l’emplacement du cluster HDInsight.
   * **Tarification** : sélectionnez le nombre de nœuds de travail de votre cluster.
   * **Configurations avancées** : 
     
     * **Jonction de domaine et réseau virtuel/sous-réseau** : 
       
       * **Paramètres du domaine** : 
         
         * **Nom du domaine** : contoso.onmicrosoft.com
         * **Mot de passe utilisateur du domaine** :saisissez un nom d’utilisateur du domaine. Ce domaine doit présenter les privilèges suivants : joignez les machines aux domaines et placez-les dans l’unité d’organisation configurée précédemment ; créez les principaux du service au sein de l’unité d’organisation configurée précédemment ; créez les entrées du DNS inversé. Cet utilisateur de domaine devient l’administrateur de ce cluster HDInsight joint à un domaine.
         * **Mot de passe du domaine** : entrez le mot de passe de l’utilisateur du domaine.
         * **Unité d’organisation** : saisissez le nom unique de l’unité d’organisation configurée précédemment. Par exemple : UO = HDInsightOU, CD = contoso, CD = onmicrosoft, CD = com
         * **URL LDAPS** : ldaps://contoso.onmicrosoft.com:636
         * **Access user group (Accéder au groupe d’utilisateurs)** : spécifiez le groupe de sécurité dont vous souhaitez synchroniser les utilisateurs au cluster. Par exemple, HiveUsers.
           
           Pour enregistrer les modifications, cliquez sur **Sélectionner**.
           
           ![Configurer les paramètres du domaine du portail HDInsight joint](./media/hdinsight-domain-joined-configure/hdinsight-domain-joined-portal-domain-setting.png)
       * **Réseau virtuel** : contosohdivnet
       * **Sous-réseau** : Subnet1
         
         Pour enregistrer les modifications, cliquez sur **Sélectionner**.        
         Pour enregistrer les modifications, cliquez sur **Sélectionner**.
   * **Groupe de ressources** : sélectionnez le groupe de ressources utilisé pour le réseau virtuel HDInsight (contosohdirg).
4. Cliquez sur **Create**.  

Pour créer le cluster HDInsight joint au domaine, vous pouvez également utiliser le modèle de gestion des ressources Azure. Appliquez la procédure suivante :

**Pour créer un cluster joint à un domaine à l’aide du modèle de gestion des ressources**

1. Cliquez sur l’image suivante pour ouvrir un modèle Resource Manager dans le portail Azure. Le modèle Resource Manager est situé dans un conteneur blob public. 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-domain-joined-hdinsight-cluster.json" target="_blank"><img src="./media/hdinsight-domain-joined-configure/deploy-to-azure.png" alt="Deploy to Azure"></a>
2. À partir du panneau **Paramètres**, saisissez les informations suivantes :
   
   * **Abonnement** : (sélectionnez votre abonnement Azure.)
   * **Groupe de ressources** : cliquez sur **Use existing (Utiliser l’existant)**, et spécifiez le groupe de ressources utilisé.  Par exemple, contosohdirg. 
   * **Emplacement** : spécifiez un emplacement de groupe de ressources.
   * **Nom du cluster** : saisissez un nom pour le cluster Hadoop créé. Par exemple, contosohdicluster.
   * **Type de cluster** : sélectionnez un type de cluster.  La valeur par défaut est **hadoop**.
   * **Emplacement** : sélectionnez un emplacement pour le cluster.  Le compte de stockage par défaut utilise le même emplacement.
   * **Cluster Worker Node count (Nombre de nœuds de travail du cluster)** : sélectionnez le nombre de nœuds de travail.
   * **Nom d’utilisateur et mot de passe de cluster** : le nom de connexion par défaut est **admin**.
   * **Nom d’utilisateur SSH et mot de passe** : le nom d’utilisateur par défaut est **sshuser**.  Vous pouvez le renommer. 
   * **ID réseau virtuel** : /subscriptions/&lt;SubscriptionID>/resourceGroups/&lt;ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/&lt;VNetName>
   * **Sous-réseau du réseau virtuel** : /subscriptions/&lt;SubscriptionID>/resourceGroups/&lt;ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/&lt;VNetName>/subnets/Subnet1
   * **Nom de domaine** : contoso.onmicrosoft.com
   * **Organization Unit DN (Nom de domaine de l’unité d’organisation)** : UO = HDInsightOU,CD = contoso, CD = onmicrosoft, CD = com
   * **Cluster Users Group D Ns**: "\"NC = HiveUsers, UO = AADDC Users,CD =<DomainName>, CD = onmicrosoft,CD = com\""
   * **LDAPUrls** : ["ldaps://contoso.onmicrosoft.com:636"]
   * **DomainAdminUserName**: (saisissez le nom d’utilisateur de l’administrateur du domaine)
   * **DomainAdminPassword** : (saisissez le mot de passe utilisateur de l’administrateur du domaine)
   * **J’accepte les termes et conditions mentionnés ci-dessus **: (vérification)
   * **Épingler au tableau de bord** : (vérification)
3. Cliquez sur **Achat**. Vous verrez une nouvelle vignette intitulée **Déploiement du déploiement de modèle**. La création d’un cluster prend environ 20 minutes. Une fois le cluster créé, vous pouvez cliquer sur le panneau du cluster dans le portail pour l’ouvrir.

Après avoir terminé ce didacticiel, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées Azure Storage, pour que vous puissiez supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même lorsque vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, économique, mieux vaut supprimer les clusters lorsqu’ils ne sont pas utilisés. Pour obtenir des instructions sur la suppression d’un cluster, consultez [Gestion des clusters Hadoop dans HDInsight au moyen du portail Azure](hdinsight-administer-use-management-portal.md#delete-clusters).

## <a name="next-steps"></a>Étapes suivantes
* Pour configurer un cluster HDInsight joint à un domaine avec Azure PowerShell, consultez [Configuration de clusters HDInsight joints à un domaine avec Azure PowerShell](hdinsight-domain-joined-configure-use-powershell.md).
* Pour configurer des stratégies Hive et exécuter des requêtes Hive, consultez [Configuration de stratégies Hive pour les clusters HDInsight joints à un domaine](hdinsight-domain-joined-run-hive.md).
* Pour utiliser des clusters HDInsight joints à un domaine, consultez [Utilisation de SSH avec Hadoop sous Linux sur HDInsight à partir de Linux, Unix ou OS X](hdinsight-hadoop-linux-use-ssh-unix.md#domainjoined).




<!--HONumber=Nov16_HO5-->


