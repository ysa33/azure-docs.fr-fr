---
title: Migrer vers Resource Manager avec PowerShell | Microsoft Docs
description: "Cet article décrit pas à pas la procédure de migration de ressources IaaS prise en charge par la plateforme de l’environnement Classic vers Azure Resource Manager à l’aide de commandes Azure PowerShell"
services: virtual-machines-windows
documentationcenter: 
author: cynthn
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 2b3dff9b-2e99-4556-acc5-d75ef234af9c
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/19/2016
ms.author: cynthn
translationtype: Human Translation
ms.sourcegitcommit: e90036d97451b271451d0ba5845c788ac05d7abf
ms.openlocfilehash: 4253d60a8a12877a3c5dac073bd06d70d020ccdc


---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-powershell"></a>Migration de ressources IaaS d’un environnement Classic vers Azure Resource Manager à l’aide d’Azure PowerShell
Ces étapes vous montrent comment utiliser les commandes Azure PowerShell pour migrer des ressources d’infrastructure en tant que service (IaaS) à partir du modèle de déploiement Classic vers le modèle de déploiement Azure Resource Manager. 

Si vous le souhaitez, vous pouvez également migrer des ressources à l’aide de [l’Interface de ligne de commande Azure (Azure CLI)](virtual-machines-linux-cli-migration-classic-resource-manager.md).

* Pour plus d’informations sur les scénarios de migration pris en charge, consultez [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager.md). 
* Pour un guide détaillé et une procédure pas à pas pour la migration, consultez [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager-deep-dive.md).
* [Passer en revue les erreurs courantes de migration](virtual-machines-migration-errors.md)

## <a name="step-1-plan-for-migration"></a>Étape 1 : Planification de la migration
Voici quelques bonnes pratiques recommandées lorsque vous évaluez la migration de ressources IaaS d’un environnement Classic vers Resource Manager.

* Lisez les [Fonctionnalités et configurations prises en charge et non prises en charge](virtual-machines-windows-migration-classic-resource-manager.md). Si vous avez des machines virtuelles qui utilisent des configurations ou fonctionnalités non prises en charge, nous vous conseillons d’attendre que leur prise en charge soit annoncée. Vous pouvez également supprimer cette fonctionnalité ou modifier cette configuration pour permettre la migration si cela répond à vos besoins.
* Si vous avez déjà des scripts automatisés qui déploient votre infrastructure et vos applications, essayez de créer une configuration de test similaire à l’aide de ces scripts pour la migration. Vous pouvez également configurer des environnements de test à l’aide du portail Azure.

> [!IMPORTANT]
> Les passerelles d’application ne sont actuellement pas prises en charge pour la migration de déploiement classique vers Resource Manager. Pour migrer un réseau virtuel classique avec une passerelle d’application, supprimez la passerelle avant d’exécuter une opération de validation pour déplacer le réseau (vous pouvez exécuter l’étape de préparation sans supprimer la passerelle d’application). Après avoir effectué la migration, reconnectez la passerelle dans Azure Resource Manager. Vous devez contacter le support technique si vous souhaitez migrer des passerelles ExpressRoute dans les cas où la passerelle et le circuit ExpressRoute se trouvent dans le même abonnement. Les passerelles ExpressRoute se connectant à des circuits ExpressRoute dans un autre abonnement ne peuvent pas être migrées. Dans ce cas, supprimez la passerelle ExpressRoute, migrez le réseau virtuel et recréez la passerelle.
> 
> 

## <a name="step-2-install-the-latest-version-of-azure-powershell"></a>Étape 2 : installation de la dernière version d’Azure PowerShell
Il existe deux options principales d’installation d’Azure PowerShell : [PowerShell Gallery](https://www.powershellgallery.com/profiles/azure-sdk/) et [Web Platform Installer (WebPI)](http://aka.ms/webpi-azps). WebPI reçoit des mises à jour mensuelles. PowerShell Gallery reçoit des mises à jour en continu. Cet article est basé sur Azure PowerShell version 2.1.0.

Pour connaître la procédure d’installation, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azureps-cmdlets-docs).

<br>

## <a name="step-3-set-your-subscription-and-sign-up-for-migration"></a>Étape 3 : définir votre abonnement et s’inscrire pour la migration
Commencez par démarrer une invite de commandes PowerShell. Pour la migration, vous devez configurer votre environnement à la fois pour l’environnement Classic et pour Resource Manager.

Connectez-vous à votre compte pour le modèle Resource Manager.

```powershell
    Login-AzureRmAccount
```

Obtenez tous les abonnements disponibles à l’aide de la commande suivante :

```powershell
    Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName
```

Définissez votre abonnement Azure pour la session active. Cet exemple définit le nom d’abonnement par défaut sur **My Azure Subscription**. Remplacez l’exemple de nom d’abonnement par le vôtre. 

```powershell
    Select-AzureRmSubscription –SubscriptionName "My Azure Subscription"
```

> [!NOTE]
> L’inscription constitue une étape unique, mais vous devez le faire une seule fois avant de tenter la migration. Sans vous inscrire, vous verrez le message d’erreur suivant : 
> 
> *Requête invalide : l’abonnement n’est pas inscrit pour la migration.* 
> 
> 

Inscrivez-vous auprès du fournisseur de ressources de migration à l’aide de la commande suivante :

```powershell
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Veuillez patienter cinq minutes le temps que l’inscription se termine. Vous pouvez vérifier l’état de l’approbation à l’aide de la commande suivante :

```powershell
    Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Assurez-vous que RegistrationState est `Registered` avant de continuer. 

Connectez-vous à votre compte pour le modèle Classic.

```powershell
    Add-AzureAccount
```

Obtenez tous les abonnements disponibles à l’aide de la commande suivante :

```powershell
    Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

Définissez votre abonnement Azure pour la session active. Cet exemple définit l’abonnement par défaut sur **My Azure Subscription**. Remplacez l’exemple de nom d’abonnement par le vôtre. 

```powershell
    Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```

<br>

## <a name="step-4-make-sure-you-have-enough-azure-resource-manager-virtual-machine-cores-in-the-azure-region-of-your-current-deployment-or-vnet"></a>Étape 4 : vérification du nombre de cœurs de machines virtuelles Azure Resource Manager dans la région Azure de votre déploiement ou réseau virtuel actuel
Vous pouvez utiliser la commande PowerShell suivante pour vérifier la quantité de cœurs dont vous disposez actuellement dans Azure Resource Manager. Pour en savoir plus sur les quotas de cœurs, consultez [Limites et Azure Resource Manager](../azure-subscription-service-limits.md#limits-and-the-azure-resource-manager). 

Cet exemple vérifie la disponibilité dans la région **ouest des États-Unis**. Remplacez l’exemple de nom de région par le vôtre. 

```powershell
Get-AzureRmVMUsage -Location "West US"
```

## <a name="step-5-run-commands-to-migrate-your-iaas-resources"></a>Étape 5 : exécution de commandes pour effectuer la migration de vos ressources IaaS
> [!NOTE]
> Toutes les opérations décrites ici sont idempotentes. Si vous rencontrez un problème autre qu’une fonctionnalité non prise en charge ou qu’une erreur de configuration, nous vous recommandons de réexécuter la procédure de préparation, d’abandon ou de validation. La plateforme réessaie alors d’exécuter l’action.
> 
> 

### <a name="migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>Migration de machines virtuelles dans un service cloud (ne figurant pas dans un réseau virtuel)
Obtenez la liste des services cloud à l’aide de la commande suivante, puis choisissez le service cloud que vous voulez migrer. La commande renvoie un message d’erreur si les machines virtuelles du service cloud sont dans un réseau virtuel ou si elles ont des rôles web/de travail.

```powershell
    Get-AzureService | ft Servicename
```

Obtenez le nom du déploiement du service cloud. Dans cet exemple, le nom du service est **Mon service**. Remplacez l’exemple de nom de service avec votre propre nom de service. 

```powershell
    $serviceName = "My Service"
    $deployment = Get-AzureDeployment -ServiceName $serviceName
    $deploymentName = $deployment.DeploymentName
```

Préparez les machines virtuelles dans le service cloud pour la migration. Vous disposez de deux options.

* **Option 1. Migrer les machines virtuelles vers un réseau virtuel créé à partir d’une plateforme**
  
    La première étape consiste à valider si vous pouvez migrer le service cloud à l’aide des commandes suivantes :
  
    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    $validate.ValidationMessages
    ```
  
    La commande ci-dessus affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, passez à l’étape de **préparation** :
  
    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    ```
* **Option 2. Migrer vers un réseau virtuel existant dans le modèle de déploiement Resource Manager**
  
    Cet exemple définit le nom de groupe de ressources sur **myResourceGroup**, le nom de réseau virtuel sur **myVirtualNetwork** et le nom du sous-réseau sur **mySubNet**. Remplacez les noms dans l’exemple par les noms de vos propres ressources.
  
    ```powershell
    $existingVnetRGName = "myResourceGroup"
    $vnetName = "myVirtualNetwork"
    $subnetName = "mySubNet"
    ```
  
    La première étape consiste à valider si vous pouvez migrer le service cloud à l’aide de la commande suivante :
  
    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
    $validate.ValidationMessages
    ```
  
    La commande ci-dessus affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez passer à l’étape de la préparation :
  
    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        -VirtualNetworkName $vnetName -SubnetName $subnetName
    ```

Une fois l’opération de préparation effectuée avec une des options précédentes, interrogez l’état de la migration des machines virtuelles. Assurez-vous qu’elles ont l’état `Prepared` .

Cet exemple définit le nom de la machine virtuelle sur **myVM**. Remplacez l’exemple de nom avec votre propre nom de machine virtuelle.

    ```powershell
    $vmName = "myVM"
    $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
    $vm.VM.MigrationState
    ```

Vérifiez la configuration pour les ressources préparées à l’aide de PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

```powershell
    Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

```powershell
    Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName
```

### <a name="migrate-virtual-machines-in-a-virtual-network"></a>Migration de machines virtuelles dans un réseau virtuel
Pour migrer des machines virtuelles dans un service cloud, vous migrez le réseau. Les machines virtuelles migrent automatiquement avec le réseau. Sélectionnez le réseau virtuel dont vous souhaitez effectuer la migration. 

Cet exemple définit le nom de réseau virtuel sur **myVnet**. Remplacez l’exemple de nom de réseau virtuel par le vôtre. 

```powershell
    $vnetName = "myVnet"
```

> [!NOTE]
> Vous obtenez un message d’erreur pour la validation si le réseau virtuel contient des rôles web/de travail ou des machines virtuelles avec des configurations non prises en charge.
> 
> 

La première étape consiste à valider si vous pouvez migrer le réseau virtuel à l’aide de la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

La commande ci-dessus affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez passer à l’étape de la préparation :

```powershell
    Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

Vérifiez la configuration pour les machines virtuelles préparées à l’aide d’Azure PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```

### <a name="migrate-a-storage-account"></a>Migrer un compte de stockage
Une fois que vous avez terminé la migration des machines virtuelles, nous vous recommandons de migrer les comptes de stockage.

Préparez chaque compte de stockage pour la migration à l’aide de la commande suivante. Dans cet exemple, le nom de compte de stockage est **myStorageAccount**. Remplacez le nom d’exemple par celui de votre propre compte de stockage. 

```powershell
    $storageAccountName = "myStorageAccount"
    Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName
```

Vérifiez la configuration pour le compte de stockage préparé à l’aide d’Azure PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

```powershell
    Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

```powershell
    Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName
```

## <a name="next-steps"></a>Étapes suivantes
* Pour plus d’informations sur la migration, consultez [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager.md).
* Pour migrer des ressources réseau supplémentaires vers Resource Manager à l’aide d’Azure PowerShell, utilisez les mêmes étapes avec [Move-AzureNetworkSecurityGroup](https://msdn.microsoft.com/library/mt786729.aspx), [Move-AzureReservedIP](https://msdn.microsoft.com/library/mt786752.aspx), et [Move-AzureRouteTable](https://msdn.microsoft.com/library/mt786718.aspx).
* Pour les scripts open source que vous pouvez utiliser pour migrer les ressources Azure d’un déploiement classique vers Resource Manager, consultez [Outils de la communauté pour la migration vers Azure Resource Manager](virtual-machines-windows-migration-scripts.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)




<!--HONumber=Feb17_HO2-->


