---
title: "Déploiement d’applications Service Fabric | Microsoft Docs"
description: "Comment déployer et supprimer des applications dans Service Fabric"
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: 
ms.assetid: b120ffbf-f1e3-4b26-a492-347c29f8f66b
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/16/2016
ms.author: ryanwi
translationtype: Human Translation
ms.sourcegitcommit: 6626747c501b01aa5e53657309ec79c75213483c
ms.openlocfilehash: 2e96e978c56ef2b9c901c952a6e96055a6ab91cf


---
# <a name="deploy-and-remove-applications-using-powershell"></a>Déployer et supprimer des applications avec PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [Visual Studio](service-fabric-publish-app-remote-cluster.md)
> 
> 

<br/>

Après avoir [packagé un type d’application][10], celui-ci peut être déployé sur un cluster Azure Service Fabric. Le déploiement implique les trois étapes suivantes :

1. Télécharger le package d'application
2. Enregistrer le type d’application
3. Créer l’instance d’application

> [!NOTE]
> Si vous utilisez Visual Studio pour déployer et déboguer des applications dans votre cluster de développement local, toutes les étapes suivantes sont gérées automatiquement à l’aide d’un script PowerShell.  Ce script se trouve dans le dossier Scripts du projet d’application. Cet article fournit le contexte des actions des scripts afin que vous puissiez effectuer les mêmes opérations en dehors de Visual Studio.
> 
> 

## <a name="upload-the-application-package"></a>Télécharger le package d'application
Quand vous chargez le package d’application, celui-ci est placé dans un dossier accessible aux composants internes de Service Fabric. Vous pouvez utiliser PowerShell pour effectuer le chargement. Avant d’exécuter des commandes PowerShell dans le cadre de cet article, commencez toujours par vous connecter au cluster Service Fabric à l’aide de la commande [Connect-ServiceFabricCluster](https://docs.microsoft.com/powershell/servicefabric/vlatest/connect-servicefabriccluster).

Supposons que vous ayez un dossier nommé *MonTypeApplication* qui contienne le manifeste de l’application, les manifestes de service et les packages de code/configuration/données. La commande [Copy-ServiceFabricApplicationPackage](https://docs.microsoft.com/powershell/servicefabric/vlatest/copy-servicefabricapplicationpackage) charge le package dans le magasin d’images du cluster. L’applet de commande **Get-ImageStoreConnectionStringFromClusterManifest** , qui fait partie du module PowerShell du SDK de Service Fabric, sert à obtenir la chaîne de connexion au magasin d’images.  Pour importer le module du Kit de développement logiciel (SDK), exécutez :

```
Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
```

Vous pouvez copier un package d’application de *C:\users\ryanwi\Documents\Visual Studio 2015\Projects\MyApplication\myapplication\pkg\debug* vers *c:\temp\MyApplicationType* (renommez « MyApplicationType » le répertoire « debug »). L’exemple suivant charge le package :

```powershell
PS C:\temp> dir

    Directory: c:\temp


Mode                LastWriteTime         Length Name                                                                                   
----                -------------         ------ ----                                                                                   
d-----        8/12/2016  10:23 AM                MyApplicationType                                                                          

PS C:\temp> tree /f .\Stateless1Pkg

C:\TEMP\MyApplicationType
│   ApplicationManifest.xml
|
└───Stateless1Pkg
    |   ServiceManifest.xml
    │
    └───Code
    │   │  Microsoft.ServiceFabric.Data.dll
    │   │  Microsoft.ServiceFabric.Data.Interfaces.dll
    │   │  Microsoft.ServiceFabric.Internal.dll
    │   │  Microsoft.ServiceFabric.Internal.Strings.dll
    │   │  Microsoft.ServiceFabric.Services.dll
    │   │  ServiceFabricServiceModel.dll
    │   │  MyService.exe
    │   │  MyService.exe.config
    │   │  MyService.pdb
    │   │  System.Fabric.dll
    │   │  System.Fabric.Strings.dll
    │   │
    │   └───en-us
    |         Microsoft.ServiceFabric.Internal.Strings.resources.dll
    |         System.Fabric.Strings.resources.dll
    |
    ├───Config
    │     Settings.xml
    │
    └───Data
          init.dat

PS C:\temp> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath MyApplicationType -ImageStoreConnectionString (Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest))
Copy application package succeeded

PS D:\temp>
```

Consultez la page [Comprendre la chaîne de connexion du magasin d’images](service-fabric-image-store-connection-string.md) pour obtenir des informations supplémentaires sur le magasin d’images et sur ImageStoreConnectionString.

## <a name="register-the-application-package"></a>Enregistrer le package d'application
Le type et la version de l’application déclarés dans le manifeste de l’application deviennent utilisables à l’enregistrement du package de l’application. Le système lit le package chargé à l’étape précédente, vérifie le package, traite le contenu du package et copie le package traité dans un emplacement système interne.  Si vous souhaitez vérifier le package de l’application en local, utilisez l’applet de commande [Test-ServiceFabricApplicationPackage](https://docs.microsoft.com/powershell/servicefabric/vlatest/test-servicefabricapplicationpackage).

```powershell
PS D:\temp> Register-ServiceFabricApplicationType MyApplicationType
Register application type succeeded

PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
DefaultParameters      : {}

PS D:\temp>
```

La commande [Register-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/servicefabric/vlatest/register-servicefabricapplicationtype) ne retourne un résultat que lorsque le package d’application a été correctement copié par le système. La durée de l’enregistrement dépend de la taille et du contenu du package de l’application. Si nécessaire, le paramètre **-TimeoutSec** peut être utilisé pour fournir un délai d’expiration plus long (le délai d’expiration par défaut est de 60 secondes).  S’il s’agit d’un package d’application volumineux et que vous rencontrez des problèmes d’expiration, utilisez le paramètre **- Async**.

La commande [Get-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricapplicationtype) répertorie toutes les versions de types d’applications correctement inscrites.

## <a name="create-the-application"></a>Création de l'application
Vous pouvez instancier une application à l’aide de n’importe quelle version de type d’application correctement inscrite via la commande [New-ServiceFabricApplication](https://docs.microsoft.com/powershell/servicefabric/vlatest/new-servicefabricapplication). Le nom de chaque application doit commencer par le schéma *fabric:* et être unique pour chaque instance d'application. Les éventuels services par défaut définis dans le manifeste de l’application du type de l’application cible sont également créés.

```powershell
PS D:\temp> New-ServiceFabricApplication fabric:/MyApp MyApplicationType AppManifestVersion1

ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
ApplicationParameters  : {}

PS D:\temp> Get-ServiceFabricApplication  

ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
ApplicationStatus      : Ready
HealthState            : OK
ApplicationParameters  : {}

PS D:\temp> Get-ServiceFabricApplication | Get-ServiceFabricService

ServiceName            : fabric:/MyApp/MyService
ServiceKind            : Stateless
ServiceTypeName        : MyServiceType
IsServiceGroup         : False
ServiceManifestVersion : SvcManifestVersion1
ServiceStatus          : Active
HealthState            : Ok

PS D:\temp>
```

La commande [Get-ServiceFabricApplication](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricapplication) répertorie toutes les instances d’applications qui ont été correctement créées, ainsi que leur état global.

La commande [Get-ServiceFabricService](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricservice) répertorie toutes les instances de service qui ont été correctement créées dans une instance d’application donnée. Les services par défaut (le cas échéant) sont répertoriés ici.

Plusieurs instances d'application peuvent être créées pour une version donnée d'un type d'application enregistré. Chaque instance de l’application s’exécute en isolement, avec ses propres répertoire de travail et processus.

## <a name="remove-an-application"></a>Supprimer une application
Quand une instance d’application n’est plus utile, elle peut être définitivement supprimée à l’aide de la commande [Remove-ServiceFabricApplication](https://docs.microsoft.com/powershell/servicefabric/vlatest/remove-servicefabricapplication). Cette commande supprime automatiquement tous les services qui appartiennent à l’application, et supprime définitivement tous les états de service. Cette opération ne peut pas être annulée et l’état de l’application ne peut pas être récupéré.

```powershell
PS D:\temp> Remove-ServiceFabricApplication fabric:/MyApp

Confirm
Continue with this operation?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
Remove application instance succeeded

PS D:\temp> Get-ServiceFabricApplication
PS D:\temp>
```

Quand une version donnée d’un type d’application n’est plus utile, vous devez la désinscrire à l’aide de la commande [Unregister-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/servicefabric/vlatest/unregister-servicefabricapplicationtype). La désinscription des types inutilisés libère l’espace de stockage utilisé par le contenu du package d’application de ce type dans le magasin d’images. Vous pouvez désinscrire un type d’application s’il ne contient aucune instance d’application et s’il n’est référencé par aucune mise à niveau d’application en attente.

```powershell
PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v1
DefaultParameters      : {}

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v2
DefaultParameters      : {}

ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
DefaultParameters      : {}

PS D:\temp> Unregister-ServiceFabricApplicationType MyApplicationType AppManifestVersion1
Unregister application type succeeded

PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v1
DefaultParameters      : {}

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v2
DefaultParameters      : {}

PS D:\temp>
```

## <a name="troubleshooting"></a>Résolution de problèmes
### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Copy-ServiceFabricApplicationPackage demande un ImageStoreConnectionString
L'environnement du SDK Service Fabric doit déjà être configuré avec les valeurs par défaut correctes. Toutefois, si besoin, l’ImageStoreConnectionString de toutes les commandes doit correspondre à celui utilisé par le cluster Service Fabric. ImageStoreConnectionString se trouve dans le manifeste de cluster récupéré à l’aide de la commande [Get-ServiceFabricClusterManifest](https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricclustermanifest) :

```powershell
PS D:\temp> Get-ServiceFabricClusterManifest
```

ImageStoreConnectionString se trouve dans le manifeste de cluster :

```xml
<ClusterManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]
```

## <a name="next-steps"></a>Étapes suivantes
[Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)

[Présentation de l’intégrité de Service Fabric](service-fabric-health-introduction.md)

[Diagnostiquer et résoudre les problèmes d'un service Service Fabric](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Modéliser une application dans Service Fabric](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-application-model.md
[11]: service-fabric-application-upgrade.md



<!--HONumber=Feb17_HO2-->


