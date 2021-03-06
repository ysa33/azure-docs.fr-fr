---
title: "Déplacer des données depuis MongoDB à l’aide de Data Factory | Microsoft Docs"
description: "Découvrez comment déplacer des données depuis une base de données MongoDB à l’aide d’Azure Data Factory."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: 10ca7d9a-7715-4446-bf59-2d2876584550
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/09/2017
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: af15b530dd512873e4534fb61d276c8c8c3a196a
ms.openlocfilehash: 2de70faa090fb3da25fec8f8946e52fcae2677d3


---
# <a name="move-data-from-mongodb-using-azure-data-factory"></a>Déplacer des données depuis MongoDB à l’aide d’Azure Data Factory
Cet article explique comment utiliser l’activité de copie dans une fabrique de données Azure pour déplacer des données entre une base de données MongoDB locale et un autre magasin de données. Cet article s’appuie sur l’article relatif aux [activités de déplacement des données](data-factory-data-movement-activities.md) qui présente une vue d’ensemble du déplacement des données avec l’activité de copie et les combinaisons de banque de données sources/réceptrices prises en charge.

Le service Data Factory prend en charge la connexion à des sources MongoDB locales à l’aide de la passerelle de gestion des données. Consultez l’article [Passerelle de gestion de données](data-factory-data-management-gateway.md) pour en savoir plus sur la passerelle de gestion des données et l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la configuration de la passerelle pour un pipeline de données afin de déplacer des données.

> [!NOTE]
> Vous devez utiliser la passerelle pour vous connecter à MongoDB même si elle est hébergée sur des machines virtuelles IaaS Azure. Si vous essayez de vous connecter à une instance MongoDB hébergée dans le cloud, vous pouvez également installer l’instance de la passerelle dans la machine virtuelle IaaS.
>
>

Actuellement, Data Factory prend uniquement en charge le déplacement de données de MongoDB vers d’autres magasins de données, mais pas l’inverse.

## <a name="supported-versions"></a>Versions prises en charge
Ce connecteur MongoDB prend en charge MongoDB version 2.4, 2.6, 3.0 et 3.2.

## <a name="prerequisites"></a>Composants requis
Pour permettre au service Azure Data Factory de se connecter à votre base de données MongoDB locale, vous devez installer les composants suivants :

* Une passerelle de gestion de données version 2.0 ou ultérieure sur l’ordinateur qui héberge la base de données ou sur un autre ordinateur afin d’éviter toute mise en concurrence avec la base de données pour les ressources. La passerelle de gestion de données est un logiciel qui connecte des sources de données locales à des services cloud de manière gérée et sécurisée. Consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) pour obtenir des informations détaillées sur la passerelle de gestion des données.

    L’installation de la passerelle engendre automatiquement l’installation d’un pilote Microsoft ODBC MongoDB, utilisé pour se connecter à MongoDB.

## <a name="copy-data-wizard"></a>Assistant Copier des données
Le moyen le plus simple de créer un pipeline qui copie des données à partir d’une base de données MongoDB vers n’importe quel magasin de données récepteur pris en charge consiste à utiliser l’Assistant Copier des données. Consultez la page [Didacticiel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Data Factory Copy](data-factory-copy-data-wizard-tutorial.md) pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copier des données.

L’exemple suivant présente des exemples de définitions de JSON que vous pouvez utiliser pour créer un pipeline à l’aide du [portail Azure](data-factory-copy-activity-tutorial-using-azure-portal.md), de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou [d’Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données depuis une base de données MongoDB vers Azure Blob Storage. Toutefois, les données peuvent être copiées vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.

## <a name="sample-copy-data-from-mongodb-to-azure-blob"></a>Exemple : copie de données à partir de MongoDB vers Azure Blob
Cet exemple indique comment copier des données à partir d’une base de données MongoDB locale vers Azure Blob Storage. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.  

L’exemple contient les entités de fabrique de données suivantes :

1. Un service lié de type [OnPremisesMongoDb](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [MongoDbCollection](#dataset-type-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [MongoDbSource](#copy-activity-type-properties) et [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).

L’exemple copie toutes les heures les données de résultat d’une requête de base de données MongoDB vers un objet blob. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

Dans un premier temps, configurez la passerelle de gestion des données en suivant les instructions de l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) .

**Service lié MongoDB**

```JSON
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties":
    {
        "type": "OnPremisesMongoDb",
        "typeProperties":
        {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",  
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
           "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<mygateway>"
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

**jeu de données d’entrée MongoDB** informe le service Data Factory que la table est externe à la fabrique de données et non produite par une activité dans la fabrique de données.

```JSON
{
     "name":  "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"    
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Jeu de données de sortie d’objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et l’heure de l’heure de début.

```JSON
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/frommongodb/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "%M"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "%d"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "%H"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Pipeline avec activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie ci-dessus, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **MongoDbSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **query** sélectionne les données de la dernière heure à copier.

```JSON
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "MongoDbSource",
                        "query": "$$Text.Format('select * from  MyTable where LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "MongoDbInputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "MongoDBToAzureBlob"
            }
        ],
        "start": "2016-06-01T18:00:00Z",
        "end": "2016-06-01T19:00:00Z"
    }
}
```

## <a name="linked-service-properties"></a>Propriétés du service lié
La table suivante fournit une description des éléments JSON spécifiques au service lié **OnPremisesMongoDB** .

| Propriété | Description | Requis |
| --- | --- | --- |
| type |Le type de propriété doit être défini sur **OnPremisesMongoDb** |Oui |
| server |Nom d’hôte ou adresse IP du serveur MongoDB. |Oui |
| port |Le port TCP utilisé par le serveur MongoDB pour écouter les connexions clientes. |Facultatif, valeur par défaut : 27017 |
| authenticationType |De base ou anonyme. |Oui |
| username |Compte d’utilisateur pour accéder à MongoDB. |Oui (si l’authentification de base est utilisée). |
| password |Mot de passe pour l’utilisateur. |Oui (si l’authentification de base est utilisée). |
| authSource |Nom de la base de données MongoDB que vous souhaitez utiliser pour vérifier vos informations d’identification pour l’authentification. |Facultatif (si l’authentification de base est utilisée). Par défaut : utilise le compte d’administrateur et la base de données spécifiée à l’aide de la propriété databaseName. |
| databaseName |Nom de la base de données MongoDB à laquelle vous souhaitez accéder. |Oui |
| gatewayName |Nom de la passerelle qui accède au magasin de données. |Oui |
| Encryptedcredential |Informations d’identification chiffrées par la passerelle. |Facultatif |

Consultez [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour plus d’informations sur la définition des informations d’identification pour une source de données MongoDB locale.

## <a name="dataset-type-properties"></a>Propriétés de type du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **MongoDbCollection** a les propriétés suivantes :

| Propriété | Description | Requis |
| --- | --- | --- |
| collectionName |Nom de la collection dans la base de données MongoDB. |Oui |

## <a name="copy-activity-type-properties"></a>Propriétés de type de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

En revanche, les propriétés disponibles dans la section **typeProperties** de l'activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Lorsque la source est de type **MongoDbSource** , les propriétés suivantes sont disponibles dans la section typeProperties :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL-92. Par exemple : select * from MyTable. |Non (si **collectionName** du **jeu de données** est spécifié) |

## <a name="schema-by-data-factory"></a>Schéma par Data Factory
Le service Azure Data Factory déduit le schéma à partir d’une collection MongoDB à l’aide des 100 derniers documents dans la collection. Si ces 100 documents ne contiennent pas de schéma complet, certaines colonnes peuvent être ignorées lors de l’opération de copie.

## <a name="type-mapping-for-mongodb"></a>Mappage de type pour MongoDB
Comme mentionné dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md) , l’activité de copie convertit automatiquement les types source en types récepteur à l’aide de l’approche en 2 étapes suivante :

1. Conversion de types natifs source en types .NET
2. Conversion de types .NET en types récepteur natifs

Lors du déplacement de données vers MongoDB, les mappages suivants sont utilisés pour passer des types MongoDB aux types .NET.

| Type MongoDB | Type de .NET Framework |
| --- | --- |
| Fichier binaire |Byte[] |
| Boolean |Boolean |
| Date |DateTime |
| NumberDouble |Double |
| NumberInt |Int32 |
| NumberLong |Int64 |
| ObjectID |Chaîne |
| Chaîne |Chaîne |
| UUID |Guid |
| Object |Renormalisé dans des colonnes aplaties avec « _ » comme séparateur imbriqué |

> [!NOTE]
> Pour en savoir plus sur la prise en charge des tableaux à l’aide de tables virtuelles, reportez-vous à la section [Prise en charge des types complexes à l’aide de tables virtuelles](#support-for-complex-types-using-virtual-tables) ci-dessous.
>
>

Actuellement, les types de données MongoDB suivants ne sont pas pris en charge : DBPointer, JavaScript, clé max./min., expression régulière, symbole, horodatage, non définie

## <a name="support-for-complex-types-using-virtual-tables"></a>Prise en charge des types complexes à l’aide de tables virtuelles
Azure Data Factory utilise un pilote ODBC intégré pour assurer la connexion à votre base de données MongoDB et copier des données à partir de cette dernière. Pour les types complexes tels que des tableaux ou des objets avec des types différents entre les documents, le pilote normalise de nouveau les données dans les tables virtuelles correspondantes. En particulier, si une table contient de telles colonnes, le pilote génère les tables virtuelles suivantes :

* Une **table de base**, qui contient les mêmes données que la table réelle, à l’exception des colonnes de type complexe. La table de base utilise le même nom que la table réelle qu’elle représente.
* Une **table virtuelle** pour chaque colonne de type complexe, qui étend les données imbriquées. Le nom des tables virtuelles est composé du nom de la table réelle, d’un séparateur « _ » et du nom du tableau ou de l’objet.

Les tables virtuelles font référence aux données présentées dans la table réelle, de manière à permettre au pilote d’accéder aux données dénormalisées. Consultez la section Exemple ci-dessous pour plus d’informations. Vous pouvez accéder au contenu des tableaux MongoDB en interrogeant et en joignant les tables virtuelles.

Vous pouvez utiliser [l’Assistant de copie](data-factory-data-movement-activities.md#create-a-pipeline-with-copy-activity) afin d’afficher de manière intuitive la liste des tables dans la base de données MongoDB, y compris les tables virtuelles, et de prévisualiser les données qui s’y trouvent. Vous pouvez également construire une requête dans l’Assistant de copie et valider pour voir le résultat.

### <a name="example"></a>Exemple
Par exemple, « ExampleTable » ci-dessous est une table MongoDB qui dispose d’une colonne avec un tableau d’objets dans chaque cellule (Factures) et d’une colonne avec un tableau de types scalaires (Évaluations).

| _id | Nom du client | Factures | Niveau de service | Évaluations |
| --- | --- | --- | --- | --- |
| 1111 |ABC |[{invoice_id:”123”, item:”toaster”, price:”456”, discount:”0.2”}, {invoice_id:”124”, item:”oven”, price: ”1235”, discount: ”0.2”}] |Silver |[5,6] |
| 2222 |XYZ |[{invoice_id:”135”, item:”fridge”, price: ”12543”, discount: ”0.0”}] |Gold |[1,2] |

Le pilote génère plusieurs tables virtuelles pour représenter cette table. La première table virtuelle est la table de base ci-dessous nommée « ExampleTable ». La table de base contient toutes les données de la table d’origine, mais les données dans les tableaux ont été omises et sont développées dans les tables virtuelles.

| _id | Nom du client | Niveau de service |
| --- | --- | --- |
| 1111 |ABC |Silver |
| 2222 |XYZ |Gold |

Les tables suivantes montrent les tables virtuelles qui représentent les tableaux d’origine dans l’exemple. Ces tables contiennent les éléments suivants :

* Une référence à la colonne de clé primaire d’origine correspondant à la ligne du tableau d’origine (via la colonne _id)
* Une indication de la position des données dans le tableau d’origine
* Les données développées pour chaque élément du tableau

Table « ExampleTable_Invoices » :

| _id | ExampleTable_Invoices_dim1_idx | invoice_id | item | price | Remise |
| --- | --- | --- | --- | --- | --- |
| 1111 |0 |123 |grille-pain |456 |0.2 |
| 1111 |1 |124 |four |1235 |0.2 |
| 2222 |0 |135 |réfrigérateur |12543 |0.0 |

Table « ExampleTable_Ratings » :

| _id | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings |
| --- | --- | --- |
| 1111 |0 |5 |
| 1111 |1 |6 |
| 2222 |0 |1 |
| 2222 |1 |2 |

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la création d’un pipeline de données qui déplace les données à partir d’un magasin de données local vers un magasin de données Azure.



<!--HONumber=Feb17_HO2-->


