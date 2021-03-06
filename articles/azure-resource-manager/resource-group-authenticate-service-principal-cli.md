---
title: "Créer une identité pour une application Azure avec l’interface de ligne de commande Azure | Microsoft Docs"
description: "Explique comment utiliser l’interface de ligne de commande (CLI) Azure pour créer une application et un principal du service Active Directory, et comment accorder à l’application l’accès aux ressources par le biais du contrôle d’accès en fonction du rôle. Cet article montre comment authentifier l’application avec un mot de passe ou un certificat."
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: c224a189-dd28-4801-b3e3-26991b0eb24d
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 01/17/2017
ms.author: tomfitz
translationtype: Human Translation
ms.sourcegitcommit: 2a9075f4c9f10d05df3b275a39b3629d4ffd095f
ms.openlocfilehash: 76c5bdeb2a27b733d8566c7a19f9457feebdc273


---
# <a name="use-azure-cli-to-create-a-service-principal-to-access-resources"></a>Créer un principal du service pour accéder aux ressources à l’aide de l’interface de ligne de commande (CLI) Azure
> [!div class="op_single_selector"]
> * [PowerShell](resource-group-authenticate-service-principal.md)
> * [Interface de ligne de commande Azure](resource-group-authenticate-service-principal-cli.md)
> * [Portail](resource-group-create-service-principal-portal.md)
> 
> 

Lorsque vous avez une application ou un script qui doit accéder à des ressources, vous pouvez configurer une identité pour l’application et l’authentifier avec ses propres informations d’identification. Cette approche est préférable à l’exécution de l’application avec vos propres informations d’identification, car :

* Vous pouvez affecter à l’identité de l’application des autorisations différentes de vos propres autorisations. En règle générale, ces autorisations sont strictement limitées à ce que l’application doit faire.
* Il est inutile de modifier les informations d’identification de l’application si vos responsabilités évoluent. 
* Vous pouvez utiliser un certificat pour automatiser l’authentification lors de l’exécution d’un script sans assistance.

Cette rubrique vous montre comment utiliser l’[interface de ligne de commande Azure pour Mac, Linux et Windows](../xplat-cli-install.md) pour configurer une application qui s’exécute sous ses propres informations d’identification et sous sa propre identité.

L’interface de ligne de commande Azure vous propose deux options pour authentifier votre application AD :

* password
* certificat

Cette rubrique explique comment utiliser ces deux options dans l’interface de ligne de commande Azure. Si vous souhaitez vous connecter à Azure à partir d’une infrastructure de programmation (par exemple Python, Ruby ou Node.js), l’authentification par mot de passe peut se révéler la meilleure option. Avant de déterminer s’il est préférable d’utiliser un mot de passe ou un certificat, consultez la section [Exemples d’applications](#sample-applications) pour obtenir des exemples d’authentification dans les différentes infrastructures.

## <a name="active-directory-concepts"></a>Concepts Active Directory
Dans cet article, vous créez deux objets : l’application Active Directory (AD) et le principal du service. L’application AD est la représentation globale de votre application. Elle contient les informations d’identification (c’est-à-dire un ID d’application et un mot de passe ou un certificat). Le principal du service est la représentation locale de votre application dans un annuaire Active Directory. Il contient l’affectation de rôle. Cette rubrique se concentre sur une application à client unique conçue pour s’exécuter au sein d’une seule organisation. Les applications à locataire unique sont généralement utilisées pour les applications métier exécutées au sein de votre organisation. Dans une application à locataire unique, vous avez une application AD et un principal du service.

Vous vous demandez peut-être pourquoi vous avez besoin des deux objets. Cette approche paraît plus logique lorsqu’on envisage des applications mutualisées. Les applications mutualisées sont généralement utilisées pour les applications SaaS (Software-as-a-Service), où votre application s’exécute dans un grand nombre d’abonnements différents. Pour les applications mutualisées, vous avez une application Active Directory et plusieurs principaux du service (un dans chaque annuaire Active Directory qui octroie un accès à l’application). Pour configurer une application mutualisée, consultez le [Guide du développeur pour l’authentification avec l’API Azure Resource Manager](resource-manager-api-authentication.md).

## <a name="required-permissions"></a>Autorisations requises
Pour effectuer cette rubrique, vous devez disposer des autorisations suffisantes dans votre annuaire Azure Active Directory et votre abonnement Azure. Plus précisément, vous devez être en mesure de créer une application dans l’annuaire Active Directory et d’affecter un rôle au principal du service. 

Le moyen le plus simple pour vérifier que votre compte dispose des autorisations adéquates est d’utiliser le portail. Consultez [Vérifier l’autorisation requise dans le portail](resource-group-create-service-principal-portal.md#required-permissions).

Passez maintenant à la section appropriée pour vous authentifier à l’aide d’un [mot de passe](#create-service-principal-with-password) ou d’un [certificat](#create-service-principal-with-certificate).

## <a name="create-service-principal-with-password"></a>Créer un principal du service avec un mot de passe
Dans cette section, vous allez suivre la procédure qui vous permettra de créer l’application AD à l’aide d’un mot de passe et d’affecter le rôle Lecteur au principal du service.

Examinons ces étapes.

1. Connectez-vous à votre compte.
   
   ```azurecli
   azure login
   ```
2. Il existe deux façons de créer l’application AD. Vous pouvez créer l’application AD et le principal de service en une étape, ou les créer séparément. Créez-les en une étape si vous n’avez pas besoin de spécifier une page d’accueil et des URI d’identificateur pour votre application. Créez-les séparément si vous devez définir ces valeurs pour une application web. Les deux options sont présentées dans cette étape.
   
   * Pour créer l’application Active Directory et le principal du service en une seule étape, fournissez le nom de l’application et un mot de passe, comme indiqué dans la commande suivante :
     
     ```azurecli
     azure ad sp create -n exampleapp -p {your-password}
     ```
   * Pour créer l’application AD séparément, fournissez :

      * le nom de l’application ;
      * l’URL de la page d’accueil de l’application ;
      * une liste séparée par des virgules des URI qui identifient l’application ;
      * Mot de passe

      Comme illustré dans la commande suivante :
     
     ```azurecli
     azure ad app create -n exampleapp --home-page http://www.contoso.org --identifier-uris https://www.contoso.org/example -p {Your_Password}
     ```

       La commande précédente renvoie une valeur AppId. Pour créer un principal du service, fournissez cette valeur en tant que paramètre dans la commande suivante :
     
     ```azurecli
     azure ad sp create -a {AppId}
     ```
     
     Si votre compte ne dispose pas des [autorisations requises](#required-permissions) dans l’annuaire Active Directory, un message d’erreur indiquant « Authentication_Unauthorized » (Authentification non autorisée) ou « No subscription found in the context » (Aucun abonnement trouvé dans le contexte) s’affiche.
     
     Pour les deux options, le nouveau principal du service est renvoyé. L’élément `Object Id` est nécessaire lorsque vous accordez des autorisations. Le GUID répertorié avec l’élément `Service Principal Names` est nécessaire pour se connecter. Ce guid est la même valeur que l’id de l’application. Dans les exemples d’applications, cette valeur est désignée sous le nom `Client ID`. 
     
     ```azurecli
     info:    Executing command ad sp create
     
     Creating application exampleapp
       / Creating service principal for application 7132aca4-1bdb-4238-ad81-996ff91d8db+
       data:    Object Id:               ff863613-e5e2-4a6b-af07-fff6f2de3f4e
       data:    Display Name:            exampleapp
       data:    Service Principal Names:
       data:                             7132aca4-1bdb-4238-ad81-996ff91d8db4
       data:                             https://www.contoso.org/example
       info:    ad sp create command OK
      ```

3. Accordez des autorisations sur votre abonnement au principal du service. Dans cet exemple, vous ajoutez le principal du service au rôle Lecteur, ce qui l’autorise à lire toutes les ressources de l’abonnement. Pour les autres rôles, voir [RBAC : rôles intégrés](../active-directory/role-based-access-built-in-roles.md). Pour le paramètre `objectid`, indiquez la propriété `Object Id` que vous avez utilisée lors de la création de l’application. Avant d’exécuter cette commande, vous devez donner le temps au nouveau principal du service de se propager dans Active Directory. Lorsque vous exécutez ces commandes manuellement, en général il s’est écoulé suffisamment de temps entre les tâches. Dans un script, vous devez ajouter une étape de veille entre les commandes (telle que `sleep 15`). Si une erreur s’affiche indiquant que le principal n’existe pas dans le répertoire, réexécutez la commande.
   
   ```azurecli
   azure role assignment create --objectId ff863613-e5e2-4a6b-af07-fff6f2de3f4e -o Reader -c /subscriptions/{subscriptionId}/
   ```
   
     Si votre compte ne dispose pas des autorisations suffisantes pour affecter un rôle, un message d’erreur s’affiche. Le message indique que votre compte « n’est pas autorisé à effectuer l’action ’Microsoft.Authorization/roleAssignments/write’ sur l’étendue ’/subscriptions/{guid}’ ».

Et voilà ! Votre application Active Directory et votre principal du service sont maintenant configurés. La section suivante vous montre comment vous connecter avec les informations d’identification via l’interface de ligne de commande Azure. Si vous souhaitez utiliser les informations d’identification dans votre application de code, vous pouvez ignorer cette rubrique. Vous pouvez passer directement aux [exemples d’applications](#sample-applications) pour obtenir des exemples de connexion à l’aide d’un ID d’application et d’un mot de passe. 

### <a name="provide-credentials-through-azure-cli"></a>Fournir des informations d’identification via Azure CLI
Maintenant, vous devez vous connecter en tant qu’application pour effectuer des opérations.

1. Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Active Directory. Pour récupérer l’ID de locataire pour l’abonnement actuellement authentifié, utilisez :
   
   ```azurecli
   azure account show
   ```
   
     Résultat :
   
   ```azurecli
   info:    Executing command account show
   data:    Name                        : Windows Azure MSDN - Visual Studio Ultimate
   data:    ID                          : {guid}
   data:    State                       : Enabled
   data:    Tenant ID                   : {guid}
   data:    Is Default                  : true
   ...
   ```
   
     Si vous avez besoin d’obtenir l’ID de locataire d’un autre abonnement, utilisez la commande suivante :
   
   ```azurecli
   azure account show -s {subscription-id}
   ```
2. Si vous avez besoin de récupérer l’id de client à utiliser pour la connexion, utilisez :
   
   ```azurecli
   azure ad sp show -c exampleapp --json
   ```
   
     La valeur à utiliser pour la connexion est le guid répertorié dans les noms de principal du service.
   
   ```azurecli
   [
     {
       "objectId": "ff863613-e5e2-4a6b-af07-fff6f2de3f4e",
       "objectType": "ServicePrincipal",
       "displayName": "exampleapp",
       "appId": "7132aca4-1bdb-4238-ad81-996ff91d8db4",
       "servicePrincipalNames": [
         "https://www.contoso.org/example",
         "7132aca4-1bdb-4238-ad81-996ff91d8db4"
       ]
     }
   ]
   ```
3. Connectez-vous en tant que principal du service.
   
   ```azurecli
   azure login -u 7132aca4-1bdb-4238-ad81-996ff91d8db4 --service-principal --tenant {tenant-id}
   ```
   
    Vous êtes invité à entrer le mot de passe. Entrez le mot de passe que vous avez indiqué lors de la création de l’application AD.
   
   ```azurecli
   info:    Executing command login
   Password: ********
   ```

Vous êtes maintenant authentifié en tant que principal du service pour le principal du service que vous avez créé.

Vous pouvez également appeler des opérations REST à partir de la ligne de commande pour vous connecter. Dans la réponse d’authentification, vous pouvez récupérer le jeton d’accès en vue de l’utiliser avec d’autres opérations. Pour un exemple de récupération du jeton d’accès en appelant des opérations REST, consultez [Génération d’un jeton d’accès](resource-manager-rest-api.md#generating-an-access-token).

## <a name="create-service-principal-with-certificate"></a>Créer un principal du service avec un certificat
Cette section explique comment :

* créer un certificat auto-signé ;
* créer l’application Active Directory avec le certificat, ainsi que le principal du service ;
* affecter le rôle Lecteur au principal du service.

Pour ce faire, [OpenSSL](http://www.openssl.org/) doit être installé.

1. Vous pouvez créer un certificat auto-signé.
   
   ```
   openssl req -x509 -days 3650 -newkey rsa:2048 -out cert.pem -nodes -subj '/CN=exampleapp'
   ```
2. Combinez les clés publiques et privées.
   
   ```
   cat privkey.pem cert.pem > examplecert.pem
   ```
3. Ouvrez le fichier **examplecert.pem** et recherchez la longue séquence de caractères entre **-----BEGIN CERTIFICATE-----** et **-----END CERTIFICATE-----**. Copiez les données du certificat. Vous transmettrez ces données en tant que paramètre lors de la création du principal du service.
4. Connectez-vous à votre compte.
   
   ```azurecli
   azure login
   ```
5. Il existe deux façons de créer l’application AD. Vous pouvez créer l’application AD et le principal de service en une étape, ou les créer séparément. Créez-les en une étape si vous n’avez pas besoin de spécifier une page d’accueil et des URI d’identificateur pour votre application. Créez-les séparément si vous devez définir ces valeurs pour une application web. Les deux options sont présentées dans cette étape.
   
   * Pour créer l’application Active Directory et le principal du service en une seule étape, fournissez le nom de l’application et les données de certificat, comme indiqué dans la commande suivante :
     
     ```azurecli
     azure ad sp create -n exampleapp --cert-value {certificate data}
     ```
   * Pour créer l’application AD séparément, fournissez :
      
      * le nom de l’application ;
      * l’URL de la page d’accueil de l’application ;
      * une liste séparée par des virgules des URI qui identifient l’application ;
      * les données du certificat.

      Comme illustré dans la commande suivante :

     ```azurecli
     azure ad app create -n exampleapp --home-page http://www.contoso.org --identifier-uris https://www.contoso.org/example --cert-value {certificate data}
     ```
     
       La commande précédente renvoie une valeur AppId. Pour créer un principal du service, fournissez cette valeur en tant que paramètre dans la commande suivante :
     
     ```azurecli
     azure ad sp create -a {AppId}
     ```
     
     Si votre compte ne dispose pas des [autorisations requises](#required-permissions) dans l’annuaire Active Directory, un message d’erreur indiquant « Authentication_Unauthorized » (Authentification non autorisée) ou « No subscription found in the context » (Aucun abonnement trouvé dans le contexte) s’affiche.
     
     Pour les deux options, le nouveau principal du service est renvoyé. L’ID d’objet est nécessaire lorsque vous accordez des autorisations. Le GUID répertorié avec l’élément `Service Principal Names` est nécessaire pour se connecter. Ce guid est la même valeur que l’id de l’application. Dans les exemples d’applications, cette valeur est désignée sous le nom `Client ID`. 
     
     ```azurecli
     info:    Executing command ad sp create
     
     Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
       data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
       data:    Display Name:     exampleapp
       data:    Service Principal Names:
       data:                      4fd39843-c338-417d-b549-a545f584a745
       data:                      https://www.contoso.org/example
       info:    ad sp create command OK
     ```
6. Accordez des autorisations sur votre abonnement au principal du service. Dans cet exemple, vous ajoutez le principal du service au rôle Lecteur, ce qui l’autorise à lire toutes les ressources de l’abonnement. Pour les autres rôles, voir [RBAC : rôles intégrés](../active-directory/role-based-access-built-in-roles.md). Pour le paramètre `objectid`, indiquez la propriété `Object Id` que vous avez utilisée lors de la création de l’application. Avant d’exécuter cette commande, vous devez donner le temps au nouveau principal du service de se propager dans Active Directory. Lorsque vous exécutez ces commandes manuellement, en général il s’est écoulé suffisamment de temps entre les tâches. Dans un script, vous devez ajouter une étape de veille entre les commandes (telle que `sleep 15`). Si une erreur s’affiche indiquant que le principal n’existe pas dans le répertoire, réexécutez la commande.
   
   ```azurecli
   azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/
   ```
   
     Si votre compte ne dispose pas des autorisations suffisantes pour affecter un rôle, un message d’erreur s’affiche. Le message indique que votre compte « n’est pas autorisé à effectuer l’action ’Microsoft.Authorization/roleAssignments/write’ sur l’étendue ’/subscriptions/{guid}’ ».

### <a name="provide-certificate-through-automated-azure-cli-script"></a>Fournir un certificat via un script Azure CLI automatisé
Maintenant, vous devez vous connecter en tant qu’application pour effectuer des opérations.

1. Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Active Directory. Pour récupérer l’ID de locataire pour l’abonnement actuellement authentifié, utilisez :
   
   ```azurecli
   azure account show
   ```
   
     Résultat :
   
   ```azurecli
   info:    Executing command account show
   data:    Name                        : Windows Azure MSDN - Visual Studio Ultimate
   data:    ID                          : {guid}
   data:    State                       : Enabled
   data:    Tenant ID                   : {guid}
   data:    Is Default                  : true
   ...
   ```
   
     Si vous avez besoin d’obtenir l’ID de locataire d’un autre abonnement, utilisez la commande suivante :
   
   ```azurecli
   azure account show -s {subscription-id}
   ```
2. Pour récupérer l’empreinte de certificat et supprimer les caractères inutiles, utilisez :
   
   ```
   openssl x509 -in "C:\certificates\examplecert.pem" -fingerprint -noout | sed 's/SHA1 Fingerprint=//g'  | sed 's/://g'
   ```
   
     Vous obtenez alors une valeur d’empreinte semblable à ce qui suit :
   
   ```
   30996D9CE48A0B6E0CD49DBB9A48059BF9355851
   ```
3. Si vous avez besoin de récupérer l’id de client à utiliser pour la connexion, utilisez :
   
   ```azurecli
   azure ad sp show -c exampleapp
   ```
   
     La valeur à utiliser pour la connexion est le guid répertorié dans les noms de principal du service.
     
   ```azurecli
   [
     {
       "objectId": "7dbc8265-51ed-4038-8e13-31948c7f4ce7",
       "objectType": "ServicePrincipal",
       "displayName": "exampleapp",
       "appId": "4fd39843-c338-417d-b549-a545f584a745",
       "servicePrincipalNames": [
         "https://www.contoso.org/example",
         "4fd39843-c338-417d-b549-a545f584a745"
       ]
     }
   ]
   ```
4. Connectez-vous en tant que principal du service.
   
   ```azurecli
   azure login --service-principal --tenant {tenant-id} -u 4fd39843-c338-417d-b549-a545f584a745 --certificate-file C:\certificates\examplecert.pem --thumbprint {thumbprint}
   ```

Vous êtes maintenant authentifié en tant que principal du service pour l’application Active Directory que vous avez créée.

## <a name="change-credentials"></a>Modifier les informations d’identification

Pour modifier les informations d’identification pour une application Active Directory, en raison d’une faille de sécurité ou de l’expiration des informations d’identification, utilisez `azure ad app set`.

Pour modifier un mot de passe, utilisez :

```azurecli
azure ad app set --applicationId 4fd39843-c338-417d-b549-a545f584a745 --password p@ssword
```

Pour modifier une valeur de certificat, utilisez :

```azurecli
azure ad app set --applicationId 4fd39843-c338-417d-b549-a545f584a745 --cert-value {certificate data}
```


## <a name="sample-applications"></a>Exemples d'applications
Les exemples d’applications suivants montrent comment ouvrir une session en tant que principal du service.

**.NET**

* [Deploy an SSH Enabled VM with a Template with .NET (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)
* [Manage Azure resources and resource groups with .NET (Gérer des ressources et des groupes de ressources Azure avec .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)

**Java**

* [Getting Started with Resources - Deploy Using Azure Resource Manager Template - in Java (Découverte des ressources - Déployer à l’aide du modèle Azure Resource Manager dans Java)](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)
* [Getting Started with Resources - Manage Resource Group - in Java (Découverte des ressources - Gérer un groupe de ressources dans Java)](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group//)

**Python**

* [Deploy an SSH Enabled VM with a Template in Python (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)
* [Managing Azure Resource and Resource Groups with Python (Gérer des ressources et des groupes de ressources Azure avec Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)

**Node.JS**

* [Deploy an SSH Enabled VM with a Template in Node.js (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)
* [Manage Azure resources and resource groups with Node.js (Gérer des ressources et des groupes de ressources Azure avec Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)

**Ruby**

* [Deploy an SSH Enabled VM with a Template in Ruby (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)
* [Managing Azure Resource and Resource Groups with Ruby (Gérer des ressources et des groupes de ressources Azure avec Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)

## <a name="next-steps"></a>Étapes suivantes
* Pour obtenir des instructions détaillées sur l’intégration d’une application à Azure pour la gestion des ressources, consultez [Guide du développeur pour l’authentification avec l’API Azure Resource Manager](resource-manager-api-authentication.md).
* Pour obtenir des informations supplémentaires sur l’utilisation de certificats et de l’interface de ligne de commande Azure, consultez l’article [Certificate-based authentication with Azure Service Principals from Linux command line](http://blogs.msdn.com/b/arsen/archive/2015/09/18/certificate-based-auth-with-azure-service-principals-from-linux-command-line.aspx)(Authentification par certificat à l’aide de principaux du service Azure à partir de la ligne de commande Linux). 




<!--HONumber=Jan17_HO4-->


