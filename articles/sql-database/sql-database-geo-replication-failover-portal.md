---
title: "Lancer un basculement planifié ou non planifié pour une base de données SQL Azure avec le portail Azure | Documents Microsoft"
description: "Lancer un basculement planifié ou non planifié pour une base de données SQL Azure avec le portail Azure"
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: a9d184a4-09e0-4f41-b364-40425f68f430
ms.service: sql-database
ms.custom: business continuity
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-management
ms.date: 11/22/2016
ms.author: carlrab
translationtype: Human Translation
ms.sourcegitcommit: 8d988aa55d053d28adcf29aeca749a7b18d56ed4
ms.openlocfilehash: b0180a9f32e1176667fe8e33a4151b2b70956adc


---
# <a name="initiate-a-planned-or-unplanned-failover-for-azure-sql-database-with-the-azure-portal"></a>Lancer un basculement planifié ou non planifié pour une base de données SQL Azure avec le portail Azure

Cet article montre comment lancer le basculement vers une base de données SQL secondaire avec le [portail Azure](http://portal.azure.com). Pour configurer la géoréplication, consultez [Configurer la géoréplication pour Base de données SQL Azure](sql-database-geo-replication-portal.md).

## <a name="initiate-a-failover"></a>Initialisation d’un basculement
La base de données secondaire peut être basculée en base de données primaire.  

1. Dans le [portail Azure](http://portal.azure.com) , accédez à la base de données primaire dans le partenariat de géoréplication.
2. Dans le panneau de la base de données SQL, sélectionnez **Tous les paramètres** > **Géoréplication**.
3. Dans la liste **SECONDAIRES**, sélectionnez la base de données qui doit devenir la nouvelle base primaire et cliquez sur **Basculement**.
   
    ![Basculement][2]
4. Cliquez sur **Oui** pour commencer le basculement.

La commande fait immédiatement basculer la base de données secondaire vers le rôle primaire. 

Il existe une courte période pendant laquelle les deux bases de données ne sont pas disponibles (de l’ordre de 0 à 25 secondes) pendant que les rôles sont activés. Si la base de données primaire comporte plusieurs bases de données secondaires, la commande reconfigure automatiquement les autres bases de données secondaires pour qu’elles se connectent à la nouvelle base de données primaire. Toute l’opération devrait prendre moins d’une minute pour se terminer dans des circonstances normales. 

> [!NOTE]
> Si la base de données primaire est en ligne et valide des transactions lorsque la commande est émise, une perte de données peut se produire.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* Après le basculement, assurez-vous que les exigences d’authentification de votre serveur et de votre base de données sont configurées sur la nouvelle base de données primaire. Pour plus d’informations, consultez [Gestion de la sécurité de la base de données SQL Azure après la récupération d’urgence](sql-database-geo-replication-security-config.md).
* Pour en savoir plus sur la reprise après un sinistre à l’aide de la géoréplication active, notamment les étapes de pré/post-récupération et la simulation d’une récupération d’urgence, consultez [Exécution d’un exercice de récupération d’urgence](sql-database-disaster-recovery.md)
* Consultez le billet de blog publié par Sasha Nosov concernant la géoréplication active : [Coup de projecteur sur les nouvelles fonctionnalités de géoréplication](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication/)
* Pour plus d’informations sur la conception d’applications cloud afin d’utiliser la géoréplication active, consultez [Conception d’applications cloud pour la continuité d’activité à l’aide de la géoréplication](sql-database-designing-cloud-solutions-for-disaster-recovery.md)
* Pour plus d’informations sur la géo-réplication active avec des pools élastiques, consultez [Stratégies de récupération d’urgence de pool élastique](sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool.md).
* Pour une vue d’ensemble de la continuité des activités, consultez [Vue d’ensemble de la continuité des activités](sql-database-business-continuity.md)

<!--Image references-->
[1]: ./media/sql-database-geo-replication-failover-portal/failover.png
[2]: ./media/sql-database-geo-replication-failover-portal/secondaries.png



<!--HONumber=Feb17_HO3-->


