---
title: "Déplacer des données depuis Salesforce à l’aide de Data Factory | Microsoft Docs"
description: "Découvrez comment déplacer des données depuis Salesforce à l’aide d’Azure Data Factory."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: dbe3bfd6-fa6a-491a-9638-3a9a10d396d1
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/22/2016
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: 9e70638af1ecdd0bf89244b2a83cd7a51d527037
ms.openlocfilehash: 98b841e300d5b704d134bcfab0968523f3b9c3f0


---
# <a name="move-data-from-salesforce-by-using-azure-data-factory"></a>Déplacer des données depuis Salesforce à l’aide d’Azure Data Factory
Cet article décrit la façon dont vous pouvez utiliser l’activité de copie dans Azure Data Factory pour copier des données depuis Salesforce vers n’importe quel magasin de données répertorié dans la colonne du récepteur du tableau [Sources et récepteurs pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Cet article s’appuie sur l’article des [activités de déplacement des données](data-factory-data-movement-activities.md) qui présente une vue d’ensemble du déplacement des données avec l’activité de copie et les combinaisons de magasins de données prises en charge.

Pour l’instant, Data Factory permet uniquement de déplacer des données de Salesforce vers des [magasins récepteurs pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), mais il ne prend pas en charge le déplacement des données à partir d’autres magasins de données vers Salesforce.

## <a name="supported-versions"></a>Versions prises en charge
Ce connecteur prend en charge les éditions suivantes de Salesforce : Developer Edition, Professional Edition, Enterprise Edition et Unlimited Edition. Et il prend en charge la copie de production Salesforce, bac à sable (sandbox) et domaine personnalisé.

## <a name="prerequisites"></a>Composants requis
* L’autorisation d’API doit être activée. Consultez l’article [How do I enable API access in Salesforce by permission set?](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)
* Pour copier des données depuis Salesforce vers des magasins de données locaux, la passerelle de gestion des données version 2.0 doit être au moins installée dans votre environnement local.

## <a name="salesforce-request-limits"></a>Limites des requêtes Salesforce
Salesforce prend en charge un nombre limité de requêtes d’API totales et de requêtes d’API simultanées. Pour plus de détails, consultez la section « API Request Limits » (Limites de requête d’API) du document [Salesforce Developer Limits](http://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf) (Limites des développeurs Salesforce). Notez que si le nombre de requêtes simultanées dépasse la limite, la limitation est appliquée et des défaillances aléatoires peuvent se produire. Si le nombre total de requêtes dépasse la limite, le compte Salesforce sera bloqué pendant 24 heures. Vous pourriez également recevoir l’erreur « REQUEST_LIMIT_EXCEEDED » dans les deux scénarios.

## <a name="copy-data-wizard"></a>Assistant Copier des données
Le moyen le plus simple de créer un pipeline qui copie les données à partir de Salesforce vers n’importe quel magasin de données récepteur pris en charge consiste à utiliser l’Assistant Copier des données. Consultez la page [Didacticiel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md) pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copier des données.

L’exemple suivant présente des exemples de définitions de JSON que vous pouvez utiliser pour créer un pipeline à l’aide du [portail Azure](data-factory-copy-activity-tutorial-using-azure-portal.md), de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou [d’Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données depuis Salesforce vers Azure Blob Storage. Toutefois, les données peuvent être copiées vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.   

## <a name="sample-copy-data-from-salesforce-to-an-azure-blob"></a>Exemple : copie de données de Salesforce vers un objet blob Azure
Cet exemple copie des données de Salesforce vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples. Vous pouvez copier directement les données vers l’un des récepteurs indiqués dans l’article [Activités de déplacement des données](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , à l’aide de l’activité de copie d’Azure Data Factory.

Voici les artefacts Data Factory dont vous aurez besoin pour implémenter le scénario. Les sections qui suivent la liste fournissent des informations supplémentaires sur ces étapes.

* Un service lié de type [Salesforce](#salesforce-linked-service-properties)
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service)
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [RelationalTable](#salesforce-dataset-properties)
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [RelationalSource](#relationalsource-type-properties) et [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties)

**Service lié Salesforce**

Cet exemple utilise le service lié **Salesforce** . Consultez la section [Service lié Salesforce](#salesforce-linked-service-properties) pour connaître les propriétés prises en charge par ce service lié.  Consultez l’article [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtenir un jeton de sécurité) pour obtenir des instructions sur la réinitialisation et l’obtention du jeton de sécurité.

```JSON
{
    "name": "SalesforceLinkedService",
    "properties":
    {
        "type": "Salesforce",
        "typeProperties":
        {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```
**Service lié Azure Storage**

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
    "type": "AzureStorage",
    "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
    }
}
```
**Jeu de données d’entrée Salesforce**

```JSON
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"  
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

La définition de **external** sur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à la Data Factory et non produit par une activité dans la Data Factory.

> [!IMPORTANT]
> La partie « __c » du nom de l’API est requise pour tout objet personnalisé.
>
>

![Connexion Salesforce - Data Factory - Nom de l’API](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

**Jeu de données de sortie d’objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1).

```JSON
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/alltypes_c"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Pipeline avec activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie ci-dessus, et qui est planifiée pour s’exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **RelationalSource** et le type **sink** est défini sur **BlobSink**.

Pour obtenir la liste des propriétés prises en charge par RelationalSource, voir [Propriétés de type RelationalSource](#relationalsource-type-properties) .

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2016-06-01T18:00:00",
        "end":"2016-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
        {
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "SalesforceInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"                
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }
        ]
    }
}
```
> [!IMPORTANT]
> La partie « __c » du nom de l’API est requise pour tout objet personnalisé.
>

![Connexion Salesforce - Data Factory - Nom de l’API](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## <a name="salesforce-linked-service-properties"></a>Propriétés du service lié Salesforce
Le tableau suivant décrit les éléments JSON spécifiques au service lié Salesforce.

| Propriété | Description | Requis |
| --- | --- | --- |
| type |La propriété de type doit être définie sur **Salesforce**. |Oui |
| environmentUrl | Spécifiez l’URL de l’instance Salesforce. <br><br> - L’URL par défaut est « https://login.salesforce.com ». <br> - Pour copier des données à partir du bac à sable (sandbox), spécifiez « https://test.salesforce.com ». <br> - Pour copier les données du domaine personnalisé, spécifiez, par exemple : « https://[domain].my.salesforce.com ». |Non |
| username |Spécifiez un nom d’utilisateur pour le compte d’utilisateur. |Oui |
| password |Spécifiez le mot de passe du compte d’utilisateur. |Oui |
| securityToken |Spécifiez le jeton de sécurité du compte d’utilisateur. Consultez l’article [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtenir un jeton de sécurité) pour obtenir des instructions sur la réinitialisation et l’obtention d’un jeton de sécurité. Pour en savoir plus sur les jetons de sécurité, consultez l’article [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)(Sécurité et API). |Oui |

## <a name="salesforce-dataset-properties"></a>Propriétés du jeu de données Salesforce
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md) . Les sections comme la structure, la disponibilité et la stratégie d’un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **RelationalTable** comprend les propriétés suivantes :

| Propriété | Description | Requis |
| --- | --- | --- |
| TableName |Nom de la table dans Salesforce. |Non (si une **requête** de type **RelationalSource** est spécifiée) |

> [!IMPORTANT]
> La partie « __c » du nom de l’API est requise pour tout objet personnalisé.
>
>

![Connexion Salesforce - Data Factory - Nom de l’API](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

## <a name="relationalsource-type-properties"></a>Propriétés de type RelationalSource
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés telles que le nom, la description, les tables d’entrée et de sortie, les différentes stratégies, etc. sont disponibles pour tous les types d’activités.

En revanche, les propriétés qui sont disponibles dans la section typeProperties de l’activité varient pour chaque type d’activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Dans l’activité de copie, lorsque la source est de type **RelationalSource** (qui inclut Salesforce), les propriétés suivantes sont disponibles dans la section typeProperties :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Une requête SQL-92 ou une requête [SOQL (Salesforce Object Query Language)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm). Par exemple : `select * from MyTable__c`. |Non (si l’attribut **tableName** de l’élément **dataset** est spécifié) |

> [!IMPORTANT]
> La partie « __c » du nom de l’API est requise pour tout objet personnalisé.
>
>

![Connexion Salesforce - Data Factory - Nom de l’API](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## <a name="query-tips"></a>Conseils pour les requêtes
### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>Récupération de données à l’aide de la clause where sur la colonne DateTime
Lorsque vous spécifiez une requête SOQL ou SQL, faites attention à la différence de format DateTime. Par exemple :

* **Exemple SOQL** : $$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', WindowStart, WindowEnd)
* **Exemple SQL** : $$Text.Format('SELECT * FROM Account  WHERE LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate  < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)`.

### <a name="retrieving-data-from-salesforce-report"></a>Récupération de données à partir d’un rapport Salesforce
Vous pouvez récupérer des données à partir de rapports Salesforce en spécifiant la requête en tant que `{call "<report name>"}`, par exemple, `"query": "{call \"TestReport\"}"`.

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>Récupération d’enregistrements supprimés dans la Corbeille Salesforce
Pour interroger les enregistrements supprimés de manière réversible dans la Corbeille Salesforce, vous pouvez spécifier **« IsDeleted = 1 »** dans votre requête. Par exemple,

* Pour interroger uniquement les enregistrements supprimés, spécifiez « select * from MyTable__c **where IsDeleted= 1** »
* Pour interroger tous les enregistrements, notamment ceux existants et supprimés, spécifiez « select * from MyTable__c **where IsDeleted = 0 or IsDeleted = 1** »

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### <a name="type-mapping-for-salesforce"></a>Mappage de type pour Salesforce
| Type Salesforce | Type basé sur .NET |
| --- | --- |
| Numérotation automatique |String |
| Case à cocher |Boolean |
| Devise |Double |
| Date |DateTime |
| Date/Heure |DateTime |
| Email |String |
| ID |String |
| Relation de recherche |String |
| Liste déroulante à sélection multiple |String |
| Number |Double |
| Pourcentage |Double |
| Téléphone |String |
| Liste déroulante |String |
| Texte |String |
| Zone de texte |String |
| Zone de texte (long) |String |
| Zone de texte (enrichi) |String |
| Texte (chiffré) |String |
| URL |String |

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.



<!--HONumber=Jan17_HO1-->


