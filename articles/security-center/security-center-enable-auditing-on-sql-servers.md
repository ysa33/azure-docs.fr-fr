---
title: "Activation de l’audit sur les serveurs SQL dans Azure Security Center | Microsoft Docs"
description: "Ce document vous montre comment implémenter la recommandation de l’Azure Security Center **Activer l’audit sur les serveurs SQL**."
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: 
ms.assetid: 042fca4d-7dab-4172-8614-e8c21ccb4960
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/01/2016
ms.author: terrylan
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 2a15377af6f6dee3b9d2b9043f9803923ab3f29b


---
# <a name="enable-auditing-on-sql-servers-in-azure-security-center"></a>Activation de l’audit sur les serveurs SQL dans le Centre de sécurité Azure
Le Centre de sécurité Azure vous recommande d’activer l’audit sur toutes les bases de données de vos serveurs SQL Azure, si ce n’est déjà fait. L’audit peut vous aider à respecter une conformité réglementaire, à comprendre l’activité de la base de données et à découvrir des discordances et anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité.

Une fois que vous avez activé l’audit, vous pouvez configurer les paramètres Threat Detection et les adresses électroniques pour recevoir des alertes de sécurité. Threat Detection permet de détecter les activités base de données anormales indiquant la présence potentielle de menaces de sécurité pour la base de données. Cela vous permet de détecter et de répondre aux menaces potentielles à mesure qu’elles surviennent.

Cette recommandation s’applique uniquement au service SQL Azure, elle ne concerne pas SQL Server en cours d’exécution sur vos machines virtuelles dans Azure Infrastructure Services (Azure IaaS).

> [!NOTE]
> Ce document présente le service à l’aide d’un exemple de déploiement.  Il ne s’agit pas d’un guide pas à pas.
> 
> 

## <a name="implement-the-recommendation"></a>Implémenter la recommandation
1. Dans le panneau **Recommandations**, sélectionnez **Activer l’audit sur les serveurs SQL**.  Cette opération ouvre le panneau **Activer l’audit sur les serveurs SQL** .
   ![Enable auditing on SQL servers][1]
2. Sélectionnez un serveur SQL sur laquel activer l’audit. Cette opération ouvre le panneau **Paramètres d’audit** .
   ![Auditing settings][2]
3. Sur le panneau **Paramètres d’audit**, sélectionnez **ON** sous **Auditing**.
   ![Activer l’audit des paramètres][3]
4. Suivez les étapes de la rubrique [Prise en main de l’audit de base de données SQL](../sql-database/sql-database-auditing-get-started.md) pour configurer l’emplacement de stockage de vos journaux d’audit. Le compte de stockage de l’abonnement pour la collecte de données est le compte de stockage par défaut.
5. Suivez les étapes de la rubrique [Prise en main de Threat Detection pour la base de données SQL](../sql-database/sql-database-threat-detection-get-started.md) pour activer et configurer la détection des menaces (Threat Detection) et configurer la liste des adresses électroniques qui recevront les alertes de sécurité lors de la détection d’activités anormales.

## <a name="see-also"></a>Voir aussi
Cet article vous a montré comment implémenter la recommandation de Security Center « Activer l’audit sur les serveurs SQL ». Pour en savoir plus sur la sécurisation de votre base de données SQL, consultez les rubriques suivantes :

* [Sécurisation de votre base de données SQL](../sql-database/sql-database-security.md)

Pour plus d’informations sur le Centre de sécurité, consultez les rubriques suivantes :

* [Définition des stratégies de sécurité dans Azure Security Center](security-center-policies.md) : découvrez comment configurer des stratégies de sécurité pour vos groupes de ressources et abonnements Azure.
* [Gestion des recommandations de sécurité dans Azure Security Center](security-center-recommendations.md) : découvrez comment les recommandations peuvent vous aider à protéger vos ressources Azure.
* [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md) : découvrez comment surveiller l’intégrité de vos ressources Azure.
* [Gestion et résolution des alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md) : découvrez comment gérer et résoudre les alertes de sécurité.
* [Surveillance des solutions partenaires avec Azure Security Center](security-center-partner-solutions.md) : découvrez comment surveiller l’état d’intégrité de vos solutions partenaires.
* [FAQ sur Azure Security Center](security-center-faq.md) : forum aux questions concernant l’utilisation de ce service.
* [Blog sur la sécurité Azure](http://blogs.msdn.com/b/azuresecurity/) : découvrez les dernières nouvelles et informations sur la sécurité Azure.

<!--Image references-->
[1]: ./media/security-center-enable-auditing-on-sql-server/enable-auditing-on-sql-servers.png
[2]:./media/security-center-enable-auditing-on-sql-server/enable-auditing.png
[3]: ./media/security-center-enable-auditing-on-sql-server/auditing-settings-blade.png



<!--HONumber=Dec16_HO2-->


