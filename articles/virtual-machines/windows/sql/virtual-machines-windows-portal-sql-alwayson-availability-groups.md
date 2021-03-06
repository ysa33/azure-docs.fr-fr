---
title: "Configurer une haute disponibilité pour les machines virtuelles Azure Resource Manager | Microsoft Docs"
description: "Ce didacticiel vous explique comment créer un groupe de disponibilité Always On avec des machines virtuelles Azure en mode Azure Resource Manager."
services: virtual-machines-windows
documentationcenter: na
author: MikeRayMSFT
manager: jhubbard
editor: 
tags: azure-resource-manager
ms.assetid: 64e85527-d5c8-40d9-bbe2-13045d25fc68
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 1/23/2017
ms.author: mikeray
translationtype: Human Translation
ms.sourcegitcommit: d0910bd4e0bf50591ac83991eb2eb679bdc3cadf
ms.openlocfilehash: 4cbe4f189f5d562edbe5d5cbb524581aa4cb66f2


---
# <a name="configure-always-on-availability-group-in-azure-vm-automatically---resource-manager"></a>Configuration automatique d’un groupe de disponibilité Always On dans une machine virtuelle Azure - Resource Manager
> [!div class="op_single_selector"]
> * [Resource Manager : modèle](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)
> * [Resource Manager : mode manuel](virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md)
> * [Classic : interface utilisateur](../sqlclassic/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups.md)
> * [Classic : PowerShell](../sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md)
> 
> 

<br/>

Ce didacticiel complet vous montre comment créer un groupe de disponibilité SQL Server avec des machines virtuelles Azure Resource Manager. Il utilise des panneaux Azure pour configurer un modèle. Au cours de ce didacticiel, vous allez passer en revue les paramètres par défaut, entrer les paramètres exigés et mettre à jour les panneaux du portail.

À la fin du didacticiel, votre solution de groupe de disponibilité SQL Server dans Azure comprendra les éléments suivants :

* un réseau virtuel contenant plusieurs sous-réseaux, notamment un sous-réseau frontal et un sous-réseau principal ;
* deux contrôleurs de domaine avec un domaine Active Directory (AD) ;
* deux machines virtuelles SQL Server déployées dans le sous-réseau principal et jointes au domaine AD ;
* un cluster WSFC à 3 nœuds avec le modèle de quorum Nœud majoritaire ;
* un groupe de disponibilité avec deux réplicas avec validation synchrone d'une base de données de disponibilité.

La figure suivante est une représentation graphique de la solution.

![Architecture du laboratoire de test pour les groupes de disponibilité dans Azure](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/0-EndstateSample.png)

Toutes les ressources de cette solution appartiennent à un groupe de ressources unique.

Ce didacticiel part des principes suivants :

* Vous disposez déjà d’un compte Azure. Si vous n’en avez pas, [inscrivez-vous pour obtenir un compte d’évaluation](http://azure.microsoft.com/pricing/free-trial/).
* Vous savez déjà comment configurer une machine virtuelle SQL Server dans la galerie de machines virtuelles avec l’interface graphique utilisateur. Pour plus d'informations, consultez [Approvisionnement d’une machine virtuelle SQL Server dans le portail Azure](virtual-machines-windows-portal-sql-server-provision.md)
* Vous disposez déjà d’une connaissance approfondie des groupes de disponibilité. Pour plus d’informations, voir [Groupes de disponibilité AlwaysOn (SQL Server)](http://msdn.microsoft.com/library/hh510230.aspx).

> [!NOTE]
> Si l’utilisation des groupes de disponibilité avec SharePoint vous intéresse, consultez également [Configurer des groupes de disponibilité AlwaysOn SQL Server 2012 pour SharePoint 2013](http://technet.microsoft.com/library/jj715261.aspx).
> 
> 

Dans ce didacticiel, nous allons utiliser le portail Azure pour :

* sélectionner le modèle Toujours actif à partir du portail ;
* passer en revue les paramètres du modèle et mettre à jour quelques paramètres de configuration pour votre environnement ;
* surveiller Azure pendant la création de tout l’environnement ;
* nous connecter à l’un des contrôleurs de domaine, puis à l’un des serveurs SQL Server ;

[!INCLUDE [availability-group-template](../../../../includes/virtual-machines-windows-portal-sql-alwayson-ag-template.md)]

## <a name="provision-the-cluster-from-the-gallery"></a>Approvisionnement du cluster à partir de la galerie
Azure fournit une image de galerie pour l’ensemble de la solution. Pour localiser le modèle :

1. Connectez-vous au portail Azure avec votre compte.
2. Dans le portail Azure, cliquez sur **+Nouveau**. Le portail ouvre le panneau Nouveau.
3. Dans le panneau Nouveau, recherchez **AlwaysOn**.
   ![Rechercher le modèle AlwaysOn](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/16-findalwayson.png)
4. Dans les résultats de la recherche, recherchez **Cluster AlwaysOn SQL Server**.
   ![Modèle AlwaysOn](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/17-alwaysontemplate.png)
5. Dans **Sélectionner un modèle de déploiement**, choisissez **Resource Manager**.

### <a name="basics"></a>Concepts de base
Cliquez sur les **concepts de base** et configurez les paramètres suivants :

* **Nom d’utilisateur de l’administrateur** correspond à un compte d’utilisateur doté d’autorisations d’administrateur de domaine et est membre du rôle serveur fixe sysadmin SQL Server sur les deux instances de SQL Server. Pour ce didacticiel, utilisez **DomainAdmin**.
* **Mot de passe** correspond au mot de passe du compte d’administrateur de domaine. Utilisez un mot de passe complexe. Confirmez le mot de passe.
* **Abonnement** correspond à l’abonnement facturé par Azure pour exécuter toutes les ressources déployées pour le groupe de disponibilité. Vous pouvez spécifier un autre abonnement si votre compte en dispose de plusieurs.
* **Groupe de ressources** correspond au nom du groupe auquel appartiennent toutes les ressources Azure créées dans le cadre de ce didacticiel. Pour ce didacticiel, utilisez **SQL-HA-RG**. Pour plus d’informations, consultez (Vue d’ensemble d’Azure Resource Manager)[resource-group-overview.md/#resource-groups].
* **Emplacement** correspond à la région Azure dans laquelle les ressources sont créées pour ce didacticiel. Sélectionnez une région Azure pour héberger l’infrastructure.

Voici ce à quoi le panneau des **concepts de base** va ressembler :

![Concepts de base](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/1-basics.png)

* Cliquez sur **OK**.

### <a name="domain-and-network-settings"></a>Paramètres de domaine et réseau
Ce modèle de galerie Azure crée un domaine avec de nouveaux contrôleurs de domaine. Il crée également un réseau et deux sous-réseaux. Le modèle ne permet pas la création de serveurs dans un domaine ou un réseau virtuel existant. L’étape suivante consiste à configurer les paramètres de domaine et réseau.

Dans le panneau des **paramètres de domaine et réseau** , examinez les valeurs prédéfinies des paramètres de domaine et de réseau :

* **Nom de domaine racine de la forêt** correspond au nom de domaine utilisé pour le domaine Active Directory qui héberge le cluster. Pour ce didacticiel, utilisez **contoso.com**.
* **Nom du réseau virtuel** correspond au nom du réseau virtuel Azure. Pour ce didacticiel, utilisez **autohaVNET**.
* **Nom du sous-réseau de contrôleur de domaine** correspond au nom d’une partie du réseau virtuel qui héberge le contrôleur de domaine. Pour ce didacticiel, utilisez **subnet-1**. Ce sous-réseau utilise le préfixe d’adresse **10.0.0.0/24**.
* **Nom du sous-réseau SQL Server** correspond au nom d’une partie du réseau virtuel qui héberge les serveurs SQL et le témoin de partage de fichiers. Pour ce didacticiel, utilisez **subnet-2**. Ce sous-réseau utilise le préfixe d’adresse **10.0.1.0/26**.

Pour plus d’informations sur les réseaux virtuels dans Azure, voir [Présentation du réseau virtuel](../../../virtual-network/virtual-networks-overview.md).  

Les **paramètres de domaine et de réseau** doivent ressembler à ceci :

![Paramètres de domaine et réseau](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/2-domain.png)

Si nécessaire, vous pouvez modifier ces valeurs. Pour ce didacticiel, nous utilisons les valeurs prédéfinies.

* Vérifiez les paramètres, puis cliquez sur **OK**.

### <a name="availability-group-settings"></a>paramètres de groupe de disponibilité
Dans les **paramètres de groupe de disponibilité** , passez en revue les valeurs prédéfinies du groupe de disponibilité et de l’écouteur.

* **Nom du groupe de disponibilité** correspond au nom de ressource en cluster du groupe de disponibilité. Pour ce didacticiel, utilisez **Contoso-ag**.
* **Nom de l’écouteur de groupe de disponibilité** est utilisé par le cluster et l’équilibreur de charge interne. Les clients qui se connectent à SQL Server peuvent utiliser ce nom pour se connecter au réplica approprié de la base de données. Pour ce didacticiel, utilisez **Contoso-listener**.
* **Port de l’écouteur de groupe de disponibilité** spécifie le port TCP que l’écouteur SQL Server utilise. Pour ce didacticiel, utilisez le port par défaut, **1433**.

Si nécessaire, vous pouvez modifier ces valeurs. Pour ce didacticiel, utilisez les valeurs prédéfinies.  

![paramètres de groupe de disponibilité](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/3-availabilitygroup.png)

* Cliquez sur **OK**.

### <a name="vm-size-storage-settings"></a>Taille de la machine virtuelle, paramètres de stockage
Pour **Taille de la machine virtuelle, paramètres de stockage** , choisissez une taille de machine virtuelle SQL Server et vérifiez les autres paramètres.

* **Taille de la machine virtuelle SQL Server** correspond à la taille de la machine virtuelle Azure pour les deux serveurs SQL. Choisissez une taille de machine virtuelle appropriée à votre charge de travail. Si vous générez cet environnement pour le didacticiel, utilisez **DS2**. Pour des charges de travail de production, choisissez une taille de machine virtuelle pouvant gérer la charge de travail. Dans le cas d’importants volumes de charges de travail de production, **DS4** ou plus est nécessaire. Le modèle génère deux machines virtuelles de cette taille et installe SQL Server sur chacune d’elles. Pour plus d’informations, consultez [Tailles des machines virtuelles](../../virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

> [!NOTE]
> Azure installe SQL Server Enterprise Edition. Le coût dépend de l’édition et de la taille de la machine virtuelle. Pour plus d’informations sur les coûts actuels, voir la [tarification des machines virtuelles](http://azure.microsoft.com/pricing/details/virtual-machines/#Sql).
> 
> 

* **Taille de la machine virtuelle de contrôleur de domaine** correspond à la taille de la machine virtuelle des contrôleurs de domaine. Pour ce didacticiel, utilisez **D2**.
* **Taille de la machine virtuelle du témoin de partage de fichiers** correspond à la taille de la machine virtuelle du témoin de partage de fichiers. Pour ce didacticiel, utilisez **A1**.
* **Compte de stockage SQL** correspond au nom du compte de stockage qui contient les données SQL Server et les disques du système d’exploitation. Pour ce didacticiel, utilisez **alwaysonsql01**.
* **Compte de stockage des contrôleurs de domaine** correspond au nom du compte de stockage des contrôleurs de domaine. Pour ce didacticiel, utilisez **alwaysondc01**.
* **Taille du disque de données SQL Server** correspond à la taille du disque de données SQL Server en To. Spécifiez un nombre compris entre 1 et 4 Il s’agit de la taille du disque de données qui est attaché à chaque serveur SQL Server. Pour ce didacticiel, utilisez **1**.
* **Optimisation du stockage** définit les paramètres de configuration de stockage spécifiques des machines virtuelles SQL Server en fonction du type de charge de travail. Tous les serveurs SQL de ce scénario utilisent Premium Storage avec un cache hôte de disque Azure défini en lecture seule. De plus, vous pouvez optimiser les paramètres SQL Server pour la charge de travail en choisissant l’un de ces trois paramètres :
  
  * **Charge de travail générale** ne définit aucun paramètre de configuration spécifique
  * **Traitement transactionnel** définit les indicateurs de suivi 1117 et 1118
  * **Data warehousing** définit les indicateurs de suivi 1117 et 610

Pour ce didacticiel, utilisez le paramètre **Charge de travail générale**.

![Taille de la machine virtuelle, paramètres de stockage](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/4-vm.png)

* Vérifiez les paramètres, puis cliquez sur **OK**.

#### <a name="a-note-about-storage"></a>Remarque sur le stockage
D’autres optimisations dépendent de la taille des disques de données SQL Server. Pour chaque téraoctet de disque de données, Azure ajoute un stockage premium de 1 To supplémentaire (SSD). Si un serveur exige 2 To ou plus, le modèle crée un pool de stockage sur chaque serveur SQL Server. Un pool de stockage est une forme de virtualisation du stockage où plusieurs disques sont configurés pour fournir une capacité, une résilience et des performances plus élevées.  Le modèle crée alors un espace de stockage dans le pool de stockage et le présente sous la forme de données uniques au système d’exploitation. Le modèle désigne ce disque en tant que disque de données pour SQL Server. Il règle le pool de stockage pour SQL Server avec les paramètres suivants :

* La taille d’entrelacement est le paramètre d’entrelacement du disque virtuel. Ce paramètre a pour valeur 64 Ko pour les charges de travail transactionnelles. Pour les charges de travail d’entreposage de données, le paramètre s’élève à 256 Ko.
* La résilience est simple (aucune résilience).

> [!NOTE]
> Azure Premium Storage est localement redondant et conserve trois copies des données au sein d’une seule région, si bien qu’aucune résilience supplémentaire n’est exigée dans le pool de stockage.
> 
> 

* Le nombre de colonnes est égal au nombre de disques dans le pool de stockage.

Pour plus d’informations sur l’espace de stockage et les pools de stockage, consultez :

* [Vue d’ensemble des espaces de stockage](http://technet.microsoft.com/library/hh831739.aspx).
* [Sauvegarde et pools de stockage Windows Server](http://technet.microsoft.com/library/dn390929.aspx)

Pour plus d’informations sur les bonnes pratiques de configuration SQL Server, voir [Meilleures pratiques relatives aux performances de SQL Server dans Azure Virtual Machines](virtual-machines-windows-sql-performance.md)

### <a name="sql-server-settings"></a>Paramètres de SQL Server
Dans **Paramètres SQL Server** , passez en revue et modifiez le préfixe du nom de la machine virtuelle SQL Server, la version de SQL Server, le compte de service et le mot de passe SQL Server, ainsi que la planification de maintenance de la mise à jour corrective automatique SQL.

* **Préfixe du nom SQL Server** est utilisé pour créer un nom pour chaque serveur SQL Server. Pour ce didacticiel, utilisez **Contoso-ag**. Les noms SQL Server sont *Contoso-ag-0* et *Contoso-ag-1*.
* **Version de SQL Server** correspond à la version de SQL Server. Pour ce didacticiel, utilisez **SQL Server 2014**. Vous pouvez également choisir **SQL Server 2012** ou **SQL Server 2016**.
* **Nom d’utilisateur du compte de service SQL Server** correspond au nom du compte de domaine pour le service SQL Server. Pour ce didacticiel, utilisez **sqlservice**.
* **Mot de passe** correspond au mot de passe du compte de service SQL Server.  Utilisez un mot de passe complexe. Confirmez le mot de passe.
* **Planification de maintenance de la mise à jour corrective automatique SQL** identifie le jour de la semaine pendant lequel Azure effectue automatiquement les mises à jour correctives des serveurs SQL Server. Pour ce didacticiel, tapez **Sunday**(Dimanche).
* **Heure de début de maintenance de la mise à jour corrective automatique SQL** correspond à l’heure à laquelle la mise à jour corrective automatique commence pour la région Azure.

> [!NOTE]
> La fenêtre de mise à jour corrective de chaque machine virtuelle est décalée d’une heure. Une seule machine virtuelle est corrigée à la fois afin d’éviter toute interruption des services.
> 
> 

![Paramètres de SQL Server](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/5-sql.png)

Vérifiez les paramètres, puis cliquez sur **OK**.

### <a name="summary"></a>Résumé
Dans la page Résumé, Azure valide les paramètres. Vous pouvez également télécharger le modèle. Passez en revue le résumé. Cliquez sur **OK**.

### <a name="buy"></a>Acheter
Ce panneau final contient les **conditions d’utilisation** et la **politique de confidentialité**. Prenez connaissance de ces informations. Quand vous êtes prêt pour qu’Azure commence à créer les machines virtuelles et toutes les autres ressources nécessaires pour le groupe de disponibilité, cliquez sur **Créer**.

Le portail Azure crée le groupe de ressources et toutes les ressources.

## <a name="monitor-deployment"></a>Surveiller le déploiement
Surveillez la progression du déploiement depuis le portail Azure. Une icône représentant le déploiement est automatiquement épinglée au tableau de bord du portail Azure.

![Tableau de bord Azure](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/11-deploydashboard.png)

## <a name="connect-to-sql-server"></a>Se connecter à SQL Server
Les nouvelles instances de SQL Server sont en cours d’exécution sur les machines virtuelles qui n’ont pas de connexions à Internet. En revanche, les contrôleurs de domaine ont une connexion Internet. Pour vous connecter aux serveurs SQL avec le Bureau à distance, commencez par établir une connexion RDP à l’un des contrôleurs de domaine. À partir du contrôleur de domaine, ouvrez une deuxième connexion RDP à SQL Server.

Pour établir une connexion RDP au contrôleur de domaine principal, procédez comme suit :

1. À partir du tableau de bord du portail Azure, vérifiez que le déploiement a réussi.
2. Cliquez sur **Ressources**.
3. Dans le panneau **Ressources**, cliquez sur **ad-primary-dc**, qui correspond au nom de la machine virtuelle pour le contrôleur de domaine principal.
4. Dans le panneau pour **ad-primary-dc** click **Se connecter**. Votre navigateur vous demande si vous voulez ouvrir ou enregistrer l’objet de connexion à distance. Cliquez sur **Ouvrir**.
   ![Se connecter au contrôleur de domaine](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups/13-ad-primary-dc-connect.png)
5. **connexion Bureau à distance** peut vous informer que le serveur de publication de cette connexion à distance n’est pas identifiable. Cliquez sur **Connecter**.
6. La sécurité de Windows vous invite à entrer vos informations d’identification pour vous connecter à l’adresse IP du contrôleur de domaine principal. Cliquez sur **Utiliser un autre compte**. Pour **Nom d’utilisateur**, tapez **contoso\DomainAdmin**. Il s’agit du compte que vous avez choisi pour le nom d’utilisateur de l’administrateur. Utilisez le mot de passe complexe que vous avez choisi quand vous avez configuré le modèle.
7. **Bureau à distance** peut vous avertir que l’ordinateur distant n’a pas pu être authentifié en raison de problèmes liés à son certificat de sécurité. Le nom du certificat de sécurité vous est indiqué. Si vous avez suivi le didacticiel, ce nom est **ad-primary-dc.contoso.com**. Cliquez sur **Oui**.

Vous êtes maintenant connecté au contrôleur de domaine principal. Pour établir une connexion RDP au serveur SQL Server, procédez comme suit :

1. Sur le contrôleur de domaine, ouvrez **Connexion Bureau à distance**.
2. Pour **Ordinateur**, tapez le nom de l’un des serveurs SQL. Pour ce didacticiel, tapez **sqlserver-0**.
3. Utilisez le même compte d’utilisateur et le même mot de passe que ceux que vous avez utilisés pour établir la connexion RDP au contrôleur de domaine.

Vous êtes maintenant connecté avec RDP au serveur SQL Server. Vous pouvez ouvrir SQL Server Management Studio, vous connecter à l’instance par défaut de SQL Server et vérifier que le groupe de disponibilité est configuré.




<!--HONumber=Jan17_HO4-->


