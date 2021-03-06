---
title: "Emplacement des ressources Azure dans un modèle | Microsoft Docs"
description: "Explique comment définir un emplacement pour une ressource dans un modèle Azure Resource Manager"
services: azure-resource-manager
documentationcenter: 
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 
ms.service: azure-resource-manager
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/03/2017
ms.author: tomfitz
translationtype: Human Translation
ms.sourcegitcommit: 1f7d7ad6dcec3c7d3b6fec7abcad7c36d2c02b70
ms.openlocfilehash: 6342b2e5f3efa498a911bb82a642fa4672f77180


---
# <a name="set-resource-location-in-azure-resource-manager-templates"></a>Définir l’emplacement des ressources dans des modèles Azure Resource Manager
Lorsque vous déployez un modèle, vous devez fournir un emplacement pour chaque ressource. Cette rubrique vous explique comment déterminer quels emplacements sont disponibles pour votre abonnement pour chaque type de ressource.

## <a name="determine-supported-locations"></a>Déterminer les emplacements pris en charge

Pour obtenir une liste complète des emplacements pris en charge pour chaque type de ressource, consultez [Disponibilité des produits par région](https://azure.microsoft.com/regions/services/). Toutefois, votre abonnement peut ne pas avoir accès à tous les emplacements de cette liste. Pour afficher une liste personnalisée des emplacements disponibles pour votre abonnement, utilisez Azure PowerShell ou l’interface de ligne de commande Azure. 

L’exemple suivant utilise PowerShell pour obtenir les emplacements pour le type de ressource `Microsoft.Web\sites` :

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
```

L’exemple suivant utilise l’interface de ligne de commande Azure 2.0 (version préliminaire) pour obtenir les emplacements pour le type de ressource `Microsoft.Web\sites` :

```azurecli
az provider show -n Microsoft.Web --query "resourceTypes[?resourceType=='sites'].locations"
```

## <a name="set-location-in-template"></a>Définir les emplacements dans un modèle

Après avoir déterminé les emplacements pris en charge pour vos ressources, vous devez définir cet emplacement dans votre modèle. Le moyen le plus simple pour définir cette valeur consiste à créer un groupe de ressources dans un emplacement qui prend en charge les types de ressources, puis de définir chaque emplacement sur `[resourceGroup().location]`. Vous pouvez redéployer le modèle dans des groupes de ressources à des emplacements différents et ne pas modifier les valeurs dans le modèle ou les paramètres. 

L’exemple suivant illustre le déploiement d’un compte de stockage au même emplacement que le groupe de ressources :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
      "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
    },
    "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageName')]",
      "location": "[resourceGroup().location]",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production"
      },
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": { }
    }
    ]
}
```

Si vous devez coder en dur l’emplacement dans votre modèle, indiquez le nom de l’une des régions prises en charge. L’exemple suivant montre un compte de stockage qui est toujours déployé dans la région Nord du centre des États-Unis :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[concat('storageloc', uniqueString(resourceGroup().id))]",
      "location": "North Central US",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production"
      },
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": { }
    }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes
* Pour des recommandations sur la création de modèles, consultez [Bonnes pratiques relatives à la création de modèles Azure Resource Manager](resource-manager-template-best-practices.md).




<!--HONumber=Feb17_HO1-->


