---
title: "Basculements régionaux dans Azure DocumentDB | Microsoft Docs"
description: "Découvrez le fonctionnement des basculements manuel et automatiques avec Azure DocumentDB."
services: documentdb
documentationcenter: 
author: arramac
manager: jhubbard
editor: 
ms.assetid: 446e2580-ff49-4485-8e53-ae34e08d997f
ms.service: documentdb
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/09/2017
ms.author: arramac
translationtype: Human Translation
ms.sourcegitcommit: 0921464c10d5ca3d426a535d434eab6cf02013e6
ms.openlocfilehash: c234958f5fc1ba0dbcb727e18e733d13ad0c7e71


---
# <a name="regional-failovers-in-azure-documentdb"></a>Basculements régionaux dans Azure DocumentDB
Azure DocumentDB simplifie la distribution globale de données en gérant complètement les [comptes de bases de données multi-régions](documentdb-distribute-data-globally.md) qui fournissent des compromis clairs entre cohérence, disponibilité et performance, le tout avec les garanties correspondantes. Les comptes DocumentDB offrent des capacités de haute disponibilité, des latences inférieures à&10; millisecondes, des[niveaux de cohérence bien définis](documentdb-consistency-levels.md), un basculement régional transparent avec des API à hébergement multiple et la possibilité de mettre à l’échelle le débit et le stockage dans le monde entier de manière flexible. 

Azure DocumentDB prend en charge des basculements explicites et basés sur des stratégies qui vous permettent de contrôler le comportement du système de bout en bout en cas de défaillance. Dans cet article, nous allons répondre aux questions suivantes :

* Comment fonctionnent les basculements manuels dans DocumentDB ?
* Comment fonctionnent les basculements automatiques dans DocumentDB ?
* Comment utiliser les basculements manuels dans différentes architectures d’application ?

Vous pouvez également découvrir les basculements régionaux dans cette vidéo Azure Friday avec Scott Hanselman et Karthik Raman, responsable principal de l’ingénierie DocumentDB.

>[!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Planet-Scale-NoSQL-with-DocumentDB/player]  

## <a name="a-idconfiguremultiregionapplicationsaconfiguring-multi-region-applications"></a><a id="ConfigureMultiRegionApplications"></a>Configuration d'applications multi-régions
Avant de nous plonger dans les modes de basculement, nous allons examiner comment vous pouvez configurer une application pour tirer parti de plusieurs régions disponibles et être résilient face aux basculements régionaux.

* Tout d’abord, déployez votre application multi-régions
* Pour garantir un accès à faible latence à partir de chaque région de votre application, configurez la [liste des régions préférées](https://msdn.microsoft.com/library/microsoft.azure.documents.client.connectionpolicy.preferredlocations.aspx#P:Microsoft.Azure.Documents.Client.ConnectionPolicy.PreferredLocations) pour chaque région par le biais de l'un des kits de développement logiciel (SDK) pris en charge.

L’extrait de code suivant montre comment initialiser une application multi-régions. Ici, le compte DocumentDB `contoso.documents.azure.com` est configuré avec deux zones (États-Unis de l'Ouest et Europe du Nord). 

* L’application est déployée dans la région États-Unis de l'Ouest (par exemple, à l’aide d'Azure App Service) 
* Configuré avec `West US` en tant que première région préférée pour des lectures à faible latence
* Configuré avec `North Europe` comme deuxième région préférée (pour une haute disponibilité durant les défaillances régionales)

Dans .NET, cette configuration ressemble à l’extrait de code suivant :

    ConnectionPolicy usConnectionPolicy = new ConnectionPolicy 
    { 
        ConnectionMode = ConnectionMode.Direct,
        ConnectionProtocol = Protocol.Tcp
    };

    usConnectionPolicy.PreferredLocations.Add(LocationNames.WestUS);
    usConnectionPolicy.PreferredLocations.Add(LocationNames.NorthEurope);

    DocumentClient usClient = new DocumentClient(
        new Uri("https://contosodb.documents.azure.com"),
        "memf7qfF89n6KL9vcb7rIQl6tfgZsRt5gY5dh3BIjesarJanYIcg2Edn9uPOUIVwgkAugOb2zUdCR2h0PTtMrA==",
        usConnectionPolicy);

L’application est également déployée dans la région Europe du Nord, dans l’ordre inversé des régions préférées. Autrement dit, la région Europe du Nord est spécifiée en premier pour les lectures à faible latence. La région États-Unis de l'Ouest est ensuite spécifiée comme deuxième région préférée pour une haute disponibilité durant les défaillances régionales.

Le diagramme d’architecture suivant montre un déploiement d’application multi-régions où DocumentDB et l'application sont configurés de manière à être disponibles dans quatre régions géographiques Azure.  

![Déploiement d’une application distribuée dans le monde entier avec Azure DocumentDB](./media/documentdb-regional-failovers/app-deployment.png)

À présent, nous allons voir comment le service DocumentDB gère les défaillances régionales par le biais de basculements automatiques. 

## <a name="a-idautomaticfailoversaautomatic-failovers"></a><a id="AutomaticFailovers"></a>Basculements automatiques
Dans des cas rares de panne régionale Azure, DocumentDB déclenche automatiquement le basculement de tous les comptes DocumentDB présents dans la région affectée. 

**Que se passe-t-il si une région de lecture connaît une panne ?**

Les comptes DocumentDB avec une région de lecture dans l’une des régions affectées sont automatiquement déconnectés de leur région d’écriture et marqués comme hors connexion. Les kits de développement logiciel (SDK) DocumentDB implémentent un protocole de découverte régionale qui leur permet de détecter automatiquement si une région est disponible et de rediriger les appels en lecture vers la prochaine région disponible dans la liste des régions préférées. Si aucune des régions dans la liste des régions préférées n'est disponible, les appels sont automatiquement acheminés vers la zone d’écriture en cours. Aucune modification n’est nécessaire dans votre code d’application pour gérer les basculements régionaux. Pendant tout le processus, des garanties de cohérence continuent à être respectées par DocumentDB.

![Défaillances de la région de lecture dans Azure DocumentDB](./media/documentdb-regional-failovers/read-region-failures.png)

Une fois que la région affectée récupère après une panne, tous les comptes DocumentDB affectés dans la région sont récupérés automatiquement par le service. Les comptes DocumentDB ayant une région de lecture dans la région affectée sont automatiquement synchronisés sur la région d’écriture en cours et mis en ligne. Les kits de développement logiciel (SDK) DocumentDB détectent la disponibilité de la nouvelle région et évalue si la région doit être sélectionnée en tant que région de lecture en cours sur la base de la liste des régions préférées configurée par l’application. Les lectures suivantes sont redirigées vers la région récupérée sans modification nécessaire de votre code d’application.

**Que se passe-t-il si une région d'écriture connaît une panne ?**

Si la région affectée est la zone d’écriture en cours pour un compte Azure DocumentDB donné, la région est automatiquement marquée comme hors connexion. Ensuite, une autre région alternative est promue comme région d'écriture de chaque compte DocumentDB affecté. Vous pouvez contrôler totalement l’ordre de sélection des régions pour vos comptes DocumentDB par le biais du portail Azure ou [par programme](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_FailoverPriorityChange). 

![Priorités de basculement pour Azure DocumentDB](./media/documentdb-regional-failovers/failover-priorities.png)

Pendant les basculements automatiques, DocumentDB choisit automatiquement la prochaine région d'écriture d’un compte Azure DocumentDB donné selon l’ordre de priorité spécifié. 

![Défaillances de la région d'écriture dans Azure DocumentDB](./media/documentdb-regional-failovers/write-region-failures.png)

Une fois que la région affectée récupère après une panne, tous les comptes DocumentDB affectés dans la région sont récupérés automatiquement par le service. 

* Des comptes DocumentDB avec leur région d'écriture précédente dans la région affectée restent en mode hors connexion avec une disponibilité de lecture même après la récupération de la région. 
* Vous pouvez interroger cette région pour calculer toutes les opérations d'écriture non répliquées pendant une panne en les comparant avec les données disponibles dans la région d'écriture en cours. En fonction des besoins de votre application, vous pouvez effectuer une fusion et/ou une résolution de conflit et écrire l’ensemble final des modifications apportées à la zone d’écriture en cours. 
* Une fois que vous avez effectué la fusion des modifications, vous pouvez remettre la région affectée en ligne en la supprimant et en la réajoutant à votre compte DocumentDB. Une fois la région à nouveau ajoutée, vous pouvez la configurer de nouveau comme zone d’écriture en effectuant un basculement manuel par le biais du portail Azure ou [par programme](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_CreateOrUpdate).

## <a name="a-idmanualfailoversamanual-failovers"></a><a id="ManualFailovers"></a>Basculements manuels

Outre les basculements automatiques, la région d’écriture en cours d’un compte DocumentDB donné peut être modifiée manuellement de manière dynamique en l'une des régions de lecture existantes. Les basculements manuels peuvent être lancés par le biais du portail Azure ou [par programme](https://docs.microsoft.com/rest/api/documentdbresourceprovider/databaseaccounts#DatabaseAccounts_CreateOrUpdate). 

Les basculements manuels assurent **qu'aucune perte de données** ni **aucune diminution de la disponibilité** ne se produit et que l'état d'écriture est transféré correctement de l’ancienne région d'écriture vers la nouvelle pour le compte DocumentDB spécifié. Comme dans les basculements automatiques, le Kit de développement logiciel (SDK) Azure DocumentDB gère automatiquement les modifications de région d’écriture pendant les basculements manuels et garantit que les appels sont automatiquement redirigés vers la nouvelle zone d’écriture. Aucune modification de code ou de configuration n’est nécessaire dans votre application pour gérer les basculements. 

![Basculements manuels dans Azure DocumentDB](./media/documentdb-regional-failovers/manual-failovers.png)

Voici certains des scénarios courants où le basculement manuel peut être utile :

**Suivre le modèle d’horloge** : si vos applications ont des schémas de trafic prévisible basés sur l’heure, vous pouvez modifier régulièrement l’état de l’écriture sur la région géographique plus active selon l’heure.

**Mise à jour du service** : certains déploiements d’applications distribués globalement peuvent impliquer la redirection du trafic vers une région différente par le biais du Traffic Manager lors de la mise à jour de service planifiée. De tels déploiements d’applications peuvent utiliser le basculement manuel pour conserver l’état d’écriture de la région où il y aura un trafic actif pendant la mise à jour du service.

**Simulations de continuité d’activité et de récupération d’urgence (Business Continuity and Disaster Recovery, BCDR)** : la plupart des applications d’entreprise incluent des tests de continuité d’activité dans le cadre de leurs processus de développement et de mise sur le marché. Les tests de continuité d’activité et de récupération d’urgence sont souvent une étape importante de certification de conformité et de garantie de disponibilité en cas de pannes régionales. Vous pouvez tester la disponibilité de BCDR dans applications qui utilisent DocumentDB pour le stockage en déclenchant un basculement manuel de votre compte DocumentDB et/ou en ajoutant et en supprimant une région dynamiquement.

Dans cet article, nous avons examiné le fonctionnement des basculements manuels et automatiques dans Azure DocumentDB et comment vous pouvez configurer vos comptes DocumentDB et les applications de manière à ce qu'ils soient disponibles au niveau mondial. Avec la prise en charge de la réplication au niveau mondial d’Azure DocumentDB, vous pouvez améliorer la latence de bout en bout et garantir une haute disponibilité même en cas de défaillance de la région. 

## <a name="a-idnextstepsanext-steps"></a><a id="NextSteps"></a>Étapes suivantes
* Découvrez-en plus sur la manière dont DocumentDB rend en charge la [distribution mondiale](documentdb-distribute-data-globally.md)
* Découvrez-en plus sur [la cohérence globale avec DocumentDB](documentdb-consistency-levels.md)
* Développez en mode multi-régions à l’aide du [Kit de développement logiciel (SDK) Azure DocumentDB](documentdb-developing-with-multiple-regions.md)
* Découvrez comment créer [des architectures d'écriture multi-régions](documentdb-multi-region-writers.md) avec Azure DocumentDB




<!--HONumber=Feb17_HO2-->


