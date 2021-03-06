---
title: Cycle de vie des applications dans Service Fabric | Microsoft Docs
description: "Décrit le développement, le déploiement, le test, la mise à niveau, la maintenance et la suppression d’applications Service Fabric."
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: 
ms.assetid: 08837cca-5aa7-40da-b087-2b657224a097
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/15/2016
ms.author: ryanwi
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 87a303fc04b2a528928eed5ce8f65a19700e0bc0


---
# <a name="service-fabric-application-lifecycle"></a>Cycle de vie des applications Service Fabric
Comme pour les autres plateformes, une application sur Azure Service Fabric passe généralement par les phases suivantes : conception, développement, test, déploiement, mise à niveau, maintenance et suppression. Service Fabric offre une excellente prise en charge du cycle de vie complet des applications cloud : du développement au retrait éventuel, en passant par le déploiement, la gestion quotidienne et la maintenance. Le modèle de service permet à différents rôles de participer indépendamment au cycle de vie des applications. Cet article fournit une vue d'ensemble des API et de la façon dont elles sont utilisées par les différents rôles pendant les phases du cycle de vie des applications Service Fabric.

La vidéo suivante de la Microsoft Virtual Academy décrit comment gérer le cycle de vie de votre application :<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=My3Ka56yC_6106218965">
<img src="./media/service-fabric-application-lifecycle/AppLifecycleVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="service-model-roles"></a>Rôles de modèle de service
Les rôles de modèle de service sont les suivants :

* **Développeur de service**: développe des services génériques et modulaires qui peuvent être ajustés et utilisés dans plusieurs applications du même type ou de différents types. Par exemple, un service de file d'attente peut être utilisé pour la création d'une application de gestion de tickets (support technique) ou d'une application de commerce électronique (panier).
* **Développeur d’application**: crée des applications en intégrant un ensemble de services pour répondre à des scénarios ou exigences spécifiques. Par exemple, un site web de commerce électronique peut intégrer un service frontal sans état JSON, un service sans état d’enchères et un service avec état de file d’attente pour créer une solution de vente aux enchères.
* **Administrateur d’application**: prend des décisions sur la configuration de l’application (indication des paramètres de modèle de configuration), le déploiement (mappage aux ressources disponibles) et la qualité de service. Par exemple, un administrateur d’application détermine la langue locale de l’application (anglais pour les États-Unis ou japonais pour le Japon, par exemple). Une application déployée différente peut avoir différents paramètres.
* **Opérateur**: déploie des applications basées sur la configuration et les spécifications spécifiquement définies par l’administrateur d’application. Par exemple, un opérateur met en service et déploie l'application et s'assure qu'elle s'exécute dans Azure. Les opérateurs surveillent les informations d'intégrité et de performances des applications et gèrent l'infrastructure physique en fonction des besoins.

## <a name="develop"></a>Développement
1. Un *développeur de service* développe différents types de services à l’aide du modèle de programmation [Reliable Actors](service-fabric-reliable-actors-introduction.md) ou [Reliable Services](service-fabric-reliable-services-introduction.md).
2. Un *développeur de service* décrit de manière déclarative les types de service développés dans un fichier de manifeste de service comprenant au moins un package de code, de configuration et de données.
3. Un *développeur d'application* crée ensuite une application à l'aide de différents types de service.
4. Un *développeur d'application* décrit de manière déclarative le type d'application dans un manifeste d'application en référençant les manifestes de service des services constitutifs et en remplaçant et paramétrant correctement les différents paramètres de configuration et de déploiement des services constitutifs.

Pour obtenir des exemples, consultez [Prise en main de Reliable Actors](service-fabric-reliable-actors-get-started.md) et [Prise en main de Reliable Services](service-fabric-reliable-services-quick-start.md).

## <a name="deploy"></a>Déployer
1. Un *administrateur d’application* personnalise le type d’application en une application spécifique en vue de son déploiement sur un cluster Service Fabric en spécifiant les paramètres appropriés de l’élément **ApplicationType** du manifeste d’application.
2. Un *opérateur* charge le package d’application vers le magasin d’images du cluster à l’aide de la [méthode **CopyApplicationPackage**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_CopyApplicationPackage_System_String_System_String_System_String_) ou de [l’applet de commande **Copy-ServiceFabricApplicationPackage**](https://docs.microsoft.com/powershell/servicefabric/vlatest/copy-servicefabricapplicationpackage). Le package d'application contient le manifeste d'application et la collection de packages de service. Service Fabric déploie les applications à partir du package d’application stocké dans le magasin d’images, qui peut être un magasin d’objets blob Azure ou le service système Service Fabric.
3. Ensuite, *l’opérateur* approvisionne le type d’application dans le cluster cible à partir du package d’application chargé à l’aide de la [méthode **ProvisionApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_ProvisionApplicationAsync_System_String_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Register-ServiceFabricApplicationType**](https://docs.microsoft.com/powershell/servicefabric/vlatest/register-servicefabricapplicationtype) ou de [l’opération REST **d’approvisionnement d’une application**](https://docs.microsoft.com/rest/api/servicefabric/provision-an-application).
4. Une fois l’application approvisionnée, un *opérateur* la démarre avec les paramètres fournis par *l’administrateur d’application* à l’aide de la [méthode **CreateApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_CreateApplicationAsync_System_Fabric_Description_ApplicationDescription_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **New-ServiceFabricApplication**](https://docs.microsoft.com/powershell/servicefabric/vlatest/new-servicefabricapplication) ou de [l’opération REST de **création d’application**](https://docs.microsoft.com/rest/api/servicefabric/create-an-application).
5. Une fois l’application déployée, un *opérateur* utilise la [méthode **CreateServiceAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient#System_Fabric_FabricClient_ServiceManagementClient_CreateServiceAsync_System_Fabric_Description_ServiceDescription_System_TimeSpan_System_Threading_CancellationToken_), [l’applet de commande **New-ServiceFabricService**](https://docs.microsoft.com/powershell/servicefabric/vlatest/new-servicefabricservice) ou [l’opération REST de **création de service**](https://docs.microsoft.com/rest/api/servicefabric/create-a-service) pour créer des instances de service pour l’application en fonction des types de services disponibles.
6. À ce stade, l'application est en cours d'exécution dans le cluster Service Fabric.

Pour obtenir des exemples, consultez [Déployer une application](service-fabric-deploy-remove-applications.md) .

## <a name="test"></a>Test
1. Après le déploiement sur le cluster de développement local ou sur un cluster de test, un *développeur de service* exécute le scénario de test de basculement intégré à l’aide des classes [**FailoverTestScenarioParameters**](https://docs.microsoft.com/dotnet/api/system.fabric.testability.scenario.failovertestscenarioparameters#System_Fabric_Testability_Scenario_FailoverTestScenarioParameters) et [**FailoverTestScenario**](https://docs.microsoft.com/dotnet/api/system.fabric.testability.scenario.failovertestscenario#System_Fabric_Testability_Scenario_FailoverTestScenario) ou de l’applet de commande [**Invoke-ServiceFabricFailoverTestScenario**](https://docs.microsoft.com/powershell/servicefabric/vlatest/invoke-servicefabricfailovertestscenario). Le scénario de test de basculement exécute un service spécifié en procédant à des transitions et à des basculements intensifs pour s’assurer de sa disponibilité et de son bon fonctionnement.
2. Le *développeur de service* exécute ensuite le scénario de test de chaos intégré à l’aide des classes [**ChaosTestScenarioParameters**](https://docs.microsoft.com/dotnet/api/system.fabric.testability.scenario.chaostestscenarioparameters#System_Fabric_Testability_Scenario_ChaosTestScenarioParameters) et [**ChaosTestScenario**](https://docs.microsoft.com/dotnet/api/system.fabric.testability.scenario.chaostestscenario#System_Fabric_Testability_Scenario_ChaosTestScenario) ou de [l’applet de commande **Invoke-ServiceFabricChaosTestScenario**](https://docs.microsoft.com/powershell/servicefabric/vlatest/invoke-servicefabricchaostestscenario). Le scénario de test chaos provoque aléatoirement plusieurs défaillances de nœud, de package de code et de réplica dans le cluster.
3. Le *développeur de service* [teste la communication entre les services](service-fabric-testability-scenarios-service-communication.md) en créant des scénarios de test qui déplacent des réplicas principaux dans le cluster.

Consultez la rubrique [Introduction au service d’analyse des erreurs](service-fabric-testability-overview.md) pour plus d'informations.

## <a name="upgrade"></a>Mise à niveau
1. Un *développeur de service* met à jour les services constitutifs de l’application instanciée et/ou corrige les bogues et fournit une nouvelle version du manifeste de service.
2. Un *développeur d'application* substitue et ajuste les paramètres de configuration et de déploiement des services constitutifs et fournit une nouvelle version du manifeste d'application. Le développeur d'application intègre ensuite les nouvelles versions des manifestes de service à l'application et fournit une nouvelle version du type d'application dans un package d'application mis à jour.
3. Un *administrateur d'application* incorpore la nouvelle version du type d'application dans l'application cible en mettant à jour les paramètres appropriés.
4. Un *opérateur* charge le package d’application mis à jour vers le magasin d’images du cluster à l’aide de la [méthode **CopyApplicationPackage**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_CopyApplicationPackage_System_String_System_String_System_String_) ou de [l’applet de commande **Copy-ServiceFabricApplicationPackage**](https://docs.microsoft.com/powershell/servicefabric/vlatest/copy-servicefabricapplicationpackage). Le package d'application contient le manifeste d'application et la collection de packages de service.
5. Un *opérateur* approvisionne la nouvelle version de l’application dans le cluster cible à l’aide de la [méthode **ProvisionApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_ProvisionApplicationAsync_System_String_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Register-ServiceFabricApplicationType**](https://docs.microsoft.com/powershell/servicefabric/vlatest/register-servicefabricapplicationtype) ou de [l’opération REST **d’approvisionnement d’une application**](https://docs.microsoft.com/rest/api/servicefabric/provision-an-application).
6. Un *opérateur* met à niveau l’application cible vers la nouvelle version à l’aide de la [méthode **UpgradeApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_UpgradeApplicationAsync_System_Fabric_Description_ApplicationUpgradeDescription_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Start-ServiceFabricApplicationUpgrade**](https://docs.microsoft.com/powershell/servicefabric/vlatest/start-servicefabricapplicationupgrade) ou de [l’opération REST de **mise à niveau d’une application**](https://docs.microsoft.com/rest/api/servicefabric/upgrade-an-application).
7. Un *opérateur* vérifie la progression de la mise à niveau à l’aide de la [méthode **GetApplicationUpgradeProgressAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_GetApplicationUpgradeProgressAsync_System_Uri_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Get-ServiceFabricApplicationUpgrade**](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricapplicationupgrade) ou de [l’opération REST **d’obtention de la progression de la mise à niveau d’une application**](https://docs.microsoft.com/rest/api/servicefabric/get-the-progress-of-an-application-upgrade1).
8. Si nécessaire, *l’opérateur* modifie et applique de nouveau les paramètres de la mise à niveau d’application actuelle à l’aide de la [méthode **UpdateApplicationUpgradeAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_UpdateApplicationUpgradeAsync_System_Fabric_Description_ApplicationUpgradeUpdateDescription_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Update-ServiceFabricApplicationUpgrade**](https://docs.microsoft.com/powershell/servicefabric/vlatest/update-servicefabricapplicationupgrade) ou de [l’opération REST de **mise à jour d’une mise à niveau d’application**](https://docs.microsoft.com/rest/api/servicefabric/update-an-application-upgrade).
9. Si nécessaire, *l’opérateur* restaure la mise à niveau d’application actuelle à l’aide de la [méthode **RollbackApplicationUpgradeAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_RollbackApplicationUpgradeAsync_System_Uri_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Start-ServiceFabricApplicationRollback**](https://docs.microsoft.com/powershell/servicefabric/vlatest/start-servicefabricapplicationrollback) ou de [l’opération REST de **restauration d’une mise à niveau d’application**](https://docs.microsoft.com/rest/api/servicefabric/rollback-an-application-upgrade).
10. Service Fabric met à niveau l'application cible en cours d'exécution dans le cluster sans perdre la disponibilité des services qui la composent.

Pour obtenir des exemples, consultez [Didacticiel de mise à niveau d’application](service-fabric-application-upgrade-tutorial.md) .

## <a name="maintain"></a>Maintenance
1. Pour les mises à niveau et les correctifs du système d'exploitation, Service Fabric interagit avec l'infrastructure Azure pour garantir la disponibilité de toutes les applications en cours d'exécution dans le cluster.
2. Pour les mises à niveau et les correctifs de la plateforme Service Fabric, Service Fabric se met à niveau sans perdre la disponibilité des applications en cours d’exécution dans le cluster.
3. Un *administrateur d'application* approuve l'ajout ou la suppression de nœuds d'un cluster après avoir analysé les données historiques relatives à l'utilisation des capacités et estimé la demande future.
4. Un *opérateur* ajoute et supprime des nœuds spécifiés par *l’administrateur d’application*.
5. Quand de nouveaux nœuds sont ajoutés au cluster ou que des nœuds existants sont supprimés de celui-ci, Service Fabric équilibre automatiquement la charge des applications en cours d’exécution sur tous les nœuds du cluster pour obtenir des performances optimales.

## <a name="remove"></a>Supprimer
1. Un *opérateur* peut supprimer une instance spécifique d’un service en cours d’exécution dans le cluster sans supprimer l’ensemble de l’application à l’aide de la [méthode **DeleteServiceAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient#System_Fabric_FabricClient_ServiceManagementClient_DeleteServiceAsync_System_Fabric_Description_DeleteServiceDescription_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Remove-ServiceFabricService**](https://docs.microsoft.com/powershell/servicefabric/vlatest/remove-servicefabricservice) ou de [l’opération REST de **suppression d’un service**](https://docs.microsoft.com/rest/api/servicefabric/delete-a-service).  
2. Un *opérateur* peut également supprimer une instance d’application et tous ses services à l’aide de la [méthode **DeleteApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_DeleteApplicationAsync_System_Fabric_Description_DeleteApplicationDescription_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Remove-ServiceFabricApplication**](https://docs.microsoft.com/powershell/servicefabric/vlatest/remove-servicefabricapplication) ou de [l’opération REST de **suppression d’une application**](https://docs.microsoft.com/rest/api/servicefabric/delete-an-application).
3. Une fois que l’application et les services sont arrêtés, *l’opérateur* peut annuler l’approvisionnement du type d’application à l’aide de la [méthode **UnprovisionApplicationAsync**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_UnprovisionApplicationAsync_System_String_System_String_System_TimeSpan_System_Threading_CancellationToken_), de [l’applet de commande **Unregister-ServiceFabricApplicationType**](https://docs.microsoft.com/powershell/servicefabric/vlatest/unregister-servicefabricapplicationtype) ou de [l’opération REST **d’annulation de l’approvisionnement d’une application**](https://docs.microsoft.com/rest/api/servicefabric/unprovision-an-application). L’annulation de la mise en service du type d’application ne supprime pas le package d’application du magasin d’images. Vous devez donc supprimer le package d’application manuellement.
4. Un *opérateur* supprime le package d’application du magasin d’images à l’aide de la [méthode **RemoveApplicationPackage**](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient#System_Fabric_FabricClient_ApplicationManagementClient_RemoveApplicationPackage_System_String_System_String_) ou de [l’applet de commande **Remove-ServiceFabricApplicationPackage**](https://docs.microsoft.com/powershell/servicefabric/vlatest/remove-servicefabricapplicationpackage).

Pour obtenir des exemples, consultez [Déployer une application](service-fabric-deploy-remove-applications.md) .

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur le développement, le test et la gestion des services et des applications Service Fabric, consultez :

* [Acteurs fiables (Reliable Actors)](service-fabric-reliable-actors-introduction.md)
* [Services fiables (Reliable Services)](service-fabric-reliable-services-introduction.md)
* [Déployer une application](service-fabric-deploy-remove-applications.md)
* [Mise à niveau de l’application](service-fabric-application-upgrade.md)
* [Vue d’ensemble de la testabilité](service-fabric-testability-overview.md)
* [Échantillon de cycle de vie des applications basé sur REST](service-fabric-rest-based-application-lifecycle-sample.md)




<!--HONumber=Dec16_HO2-->


