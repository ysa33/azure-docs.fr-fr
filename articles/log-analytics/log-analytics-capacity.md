---
title: "Solution de gestion de la capacité dans Log Analytics | Microsoft Docs"
description: "Vous pouvez utiliser la solution de planification de la capacité dans Log Analytics pour vous aider à comprendre la capacité de vos serveurs Hyper-V gérés par System Center Virtual Machine Manager."
services: log-analytics
documentationcenter: 
author: bandersmsft
manager: carmonm
editor: 
ms.assetid: 51617a6f-ffdd-4ed2-8b74-1257149ce3d4
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/02/2017
ms.author: banders
translationtype: Human Translation
ms.sourcegitcommit: 57e7fbdaa393e078b62a6d6a0b181b67d532523d
ms.openlocfilehash: c34cda0da164c711c8effc78d2af38ad8df581aa


---
# <a name="capacity-management-solution-in-log-analytics"></a>Solution de gestion de la capacité dans Log Analytics
Vous pouvez utiliser la solution de gestion de la capacité dans Log Analytics pour vous aider à comprendre la capacité de vos serveurs Hyper-V. Cette solution nécessite System Center Operations Manager et System Center Virtual Machine Manager. Si vous utilisez des agents connectés directement, la solution de planification de la capacité n’est pas fonctionnelle. La solution lit les compteurs de performances sur le serveur analysé, puis envoie les données d’utilisation au service cloud OMS pour traitement. La logique est appliquée aux données d'utilisation et le service cloud enregistre les données. Au fil du temps, des modèles d'utilisation sont identifiés et la capacité est projetée, en fonction de la consommation actuelle.

Par exemple, une projection peut identifier le moment où de la mémoire supplémentaire ou des cœurs de processeur supplémentaires sont nécessaires pour un serveur. Dans cet exemple, la projection peut indiquer que dans 30 jours, le serveur a besoin de mémoire supplémentaire. Cette projection peut vous aider à planifier une mise à niveau de la mémoire au cours de la fenêtre de maintenance suivante du serveur.

> [!NOTE]
> La solution de gestion de la capacité ne peut pas être ajoutée à des espaces de travail. Les clients qui ont installé la solution de gestion de la capacité peuvent continuer de l’utiliser.  
>
>

Une solution de remplacement pour la capacité et les performances est disponible en préversion privée. Cette solution de remplacement est conçue pour répondre aux défis auxquels les clients disent avoir été confrontés avec la solution de gestion de la capacité d’origine :

* Nécessité d’utiliser Virtual Machine Manager et Operations Manager
* Impossibilité de personnaliser/filtrer des données en fonction de groupes
* Fréquence d’agrégation des données horaires insuffisante
* Absence d’informations au niveau de la machine virtuelle
* Fiabilité des données

Avantages de la nouvelle solution de capacité :

* Prise en charge de la collecte de données granulaires avec plus de précision et de fiabilité
* Prise en charge d’Hyper-V sans l’obligation d’installer VMM
* Informations sur l’utilisation au niveau de la machine virtuelle

La nouvelle solution requiert Hyper-V Server 2012 ou version ultérieure. La solution fournit des informations sur votre environnement Hyper-V et sur l’utilisation globale (processeur, mémoire et disque) des ordinateurs hôtes et des machines virtuelles en cours d’exécution sur ces serveurs Hyper-V. Des mesures sont collectées sur le processeur, la mémoire et le disque sur la totalité des ordinateurs hôtes et des machines virtuelles qui s’exécutent sur ces serveurs.

Le reste de la documentation de cette page fait référence à l’ancienne solution de gestion de la capacité. Cette documentation sera mise à jour quand la nouvelle solution sera disponible en préversion publique.

## <a name="installing-and-configuring-the-solution"></a>Installation et configuration de la solution
Utilisez les informations suivantes pour installer et configurer la solution.

* La solution de gestion de la capacité nécessite Operations Manager.
* La solution de gestion de la capacité nécessite Virtual Machine Manager.
* Une connectivité entre Operations Manager et Virtual Machine Manager (VMM) doit être établie. Pour plus d’informations sur la connexion des systèmes, consultez [Connexion de VMM avec Operations Manager](http://technet.microsoft.com/library/hh882396.aspx).
* Operations Manager doit être connecté à Log Analytics.
* Ajoutez la solution d’évaluation de la capacité à votre espace de travail OMS en suivant la procédure décrite dans [Ajouter des solutions Log Analytics à partir de la galerie de solutions](log-analytics-add-solutions.md).  Aucune configuration supplémentaire n’est requise.

## <a name="capacity-management-data-collection-details"></a>Détails de la collecte de données de gestion de la capacité
La gestion de la capacité collecte les données d’état, les métadonnées et les données de performances à l’aide des agents que vous avez activés.

Le tableau suivant présente les méthodes de collecte des données et d’autres informations sur le mode de collecte en vue de la gestion de la capacité.

| plateforme | Agent direct | Agent Operations Manager | Azure Storage | Operations Manager requis ? | Données de l’agent Operations Manager envoyées via un groupe d’administration | fréquence de collecte |
| --- | --- | --- | --- | --- | --- | --- |
| Windows |![Non](./media/log-analytics-capacity/oms-bullet-red.png) |![Oui](./media/log-analytics-capacity/oms-bullet-green.png) |![Non](./media/log-analytics-capacity/oms-bullet-red.png) |![Oui](./media/log-analytics-capacity/oms-bullet-green.png) |![Oui](./media/log-analytics-capacity/oms-bullet-green.png) |Toutes les heures |

Le tableau suivant répertorie des exemples de types de données collectées par la gestion de la capacité :

| **Type de données** | **Champs** |
| --- | --- |
| Metadata |BaseManagedEntityId, ObjectStatus, OrganizationalUnit, ActiveDirectoryObjectSid, PhysicalProcessors, NetworkName, IPAddress, ForestDNSName, NetbiosComputerName, VirtualMachineName, LastInventoryDate, HostServerNameIsVirtualMachine, IP Address, NetbiosDomainName, LogicalProcessors, DNSName, DisplayName, DomainDnsName, ActiveDirectorySite, PrincipalName, OffsetInMinuteFromGreenwichTime |
| Performances |ObjectName, CounterName, PerfmonInstanceName, PerformanceDataId, PerformanceSourceInternalID, SampleValue, TimeSampled, TimeAdded |
| État |StateChangeEventId, StateId, NewHealthState, OldHealthState, Context, TimeGenerated, TimeAdded, StateId2, BaseManagedEntityId, MonitorId, HealthState, LastModified, LastGreenAlertGenerated, DatabaseTimeModified |

## <a name="capacity-management-page"></a>Page Gestion de la capacité
Une fois la solution de gestion de la capacité installée, vous pouvez consulter la capacité de vos serveurs analysés à l’aide de la vignette **Planification de la capacité** dans la page **Vue d’ensemble** d’OMS.

![image de la vignette Planification de la capacité](./media/log-analytics-capacity/oms-capacity01.png)

La vignette ouvre le tableau de bord **Gestion de la capacité** , où vous pouvez afficher un résumé de la capacité de votre serveur. La page présente les vignettes suivantes :

* *Capacité des ordinateurs virtuels*: indique la capacité restante (en jours) des ordinateurs virtuels.
* *Calcul*: affiche les cœurs de processeurs et la mémoire disponible.
* *Stockage*: indique l'espace disque utilisé et la latence.
* *Recherche*: l’explorateur de données que vous pouvez utiliser pour rechercher des données dans le système OMS

![image du tableau de bord Gestion de la capacité](./media/log-analytics-capacity/oms-capacity02.png)

### <a name="to-view-a-capacity-page"></a>Pour afficher une page de capacité
* Dans la page **Vue d’ensemble**, cliquez sur **Gestion de la capacité**, puis sur cliquez sur **Calcul** ou **Stockage**.

## <a name="compute-page"></a>Page Calcul
Vous pouvez utiliser le tableau de bord **Calcul** de Microsoft Azure OMS pour afficher les informations de capacité d’utilisation, de jours de capacité restants et d’efficacité de votre infrastructure. La zone **Utilisation** permet de consulter le taux d'utilisation du cœur de processeur et de la mémoire des hôtes de vos ordinateurs virtuels. Vous pouvez utiliser l'outil de projection pour estimer la capacité disponible pour une période donnée. Vous pouvez utiliser la zone **Efficacité** pour consulter l'efficacité des hôtes de vos ordinateurs virtuels. Vous pouvez afficher les détails des éléments liés en cliquant dessus.

Vous pouvez générer un classeur Excel pour les catégories suivantes :

* Hôtes aux meilleurs taux d'utilisation des cœurs
* Hôtes aux meilleurs taux d'utilisation de mémoire
* Hôtes dont les ordinateurs virtuels sont les moins efficaces
* Hôtes aux meilleurs taux d'utilisation
* Hôtes aux taux d'utilisation les plus faibles

![image de la page Calcul de gestion de la capacité](./media/log-analytics-capacity/oms-capacity03.png)

Le tableau de bord **Calcul** présente les zones suivantes :

**Utilisation**: indique le taux d'utilisation du cœur de processeur et de la mémoire des hôtes de vos ordinateurs virtuels.

* *Cœurs utilisés*: somme de tous les hôtes (% d'utilisation du processeur multiplié par le nombre de cœurs physiques sur l'hôte).
* *Cœurs libres*: nombre total de cœurs moins les cœurs utilisés.
* *Pourcentage de cœurs disponibles*: nombre de cœurs physiques libres divisé par le nombre total de cœurs physiques.
* *Cœurs virtuels par ordinateur virtuel*: nombre total de cœurs dans le système divisé par le nombre total d'ordinateurs virtuels dans le système.
* *Rapport cœurs virtuels/cœurs physiques*: rapport entre le nombre total de cœurs physiques et le nombre de cœurs physiques utilisés par des ordinateurs virtuels du système.
* *Nombre de cœurs virtuels disponibles*: rapport entre le nombre de cœurs virtuels et de cœurs physiques multiplié par le nombre de cœurs physiques disponibles.
* *Mémoire utilisée*: somme de la mémoire utilisée par tous les hôtes.
* *Mémoire disponible*: mémoire physique totale moins la mémoire utilisée.
* *Pourcentage de mémoire disponible*: mémoire physique disponible divisée par la mémoire physique totale.
* *Mémoire virtuelle par ordinateur virtuel*: mémoire virtuelle totale du système divisée par le nombre total d'ordinateurs virtuels dans le système.
* *Rapport mémoire virtuelle/mémoire physique*: mémoire virtuelle totale du système divisée par la mémoire physique totale du système.
* *Mémoire virtuelle disponible*: rapport entre la mémoire virtuelle et la mémoire physique multiplié par la mémoire physique disponible.

**Outil de projection**

L'outil de projection vous permet d'afficher l'historique des tendances d'utilisation de vos ressources. Cela comprend les tendances d'utilisation des machines virtuelles, de la mémoire, des cœurs et de stockage. La fonction de projection utilise un algorithme de projection pour vous aider à planifier l'épuisement de chaque ressource. Cela vous permet de calculer les capacités avec précision et de prévoir l'achat de capacité supplémentaire (mémoire, cœurs ou stockage).

**Efficacité**

* *Ordinateurs virtuels inactifs*: ordinateurs utilisant moins de 10 % du processeur et de 10 % de la mémoire pour une période donnée.
* *Ordinateurs virtuels surexploités*: ordinateurs utilisant plus de 90 % du processeur et de 90 % de la mémoire pour une période donnée.
* *Hôte inactif*: ordinateurs utilisant moins de 10 % du processeur et de 10 % de la mémoire pour une période donnée.
* *Hôte surexploité*: ordinateurs utilisant plus de 90 % du processeur et de 90 % de la mémoire pour une période donnée.

### <a name="to-work-with-items-on-the-compute-page"></a>Utilisation des éléments de la page Calcul
1. Dans la zone **Utilisation** du tableau de bord **Calcul**, vous pouvez consulter les informations de capacité des cœurs de processeurs et de la mémoire utilisés.
2. Cliquez sur un élément pour l'ouvrir dans la page de **recherche** et en afficher les informations détaillées.
3. Dans l'outil de **projection** , déplacez le curseur de date pour afficher une projection de la capacité qui sera utilisée à la date choisie.
4. Dans la zone **Efficacité** , vous pouvez afficher l'efficacité de la capacité des ordinateurs virtuels et des hôtes d'ordinateurs virtuels.

## <a name="direct-attached-storage-page"></a>Page Stockage en attachement direct
Le tableau de bord **Stockage en attachement direct** dans OMS vous permet d’afficher des informations de capacité relatives à l’utilisation du stockage et aux performances des disques, ainsi que la capacité restante des disques (en jours). La zone **Utilisation** permet de consulter le taux d'utilisation de l'espace disque des hôtes de vos ordinateurs virtuels. La zone **Performances du disque** permet de consulter le taux d'utilisation et de latence de l'espace disque des hôtes de vos ordinateurs virtuels. Vous pouvez également utiliser l'outil de projection pour estimer la capacité disponible pour une période donnée. Vous pouvez afficher les détails des éléments liés en cliquant dessus.

Vous pouvez générer un classeur Excel à partir de ces informations de capacité pour les catégories suivantes :

* Utilisation la plus élevée de l'espace disque par hôte
* Latence moyenne la plus élevée par hôte

La page **Stockage** présente les zones suivantes :

* *Utilisation*: permet de consulter le taux d'utilisation de l'espace disque de vos hôtes d'ordinateurs virtuels.
* *Espace disque total*: somme (espace disque logique) de tous les hôtes.
* *Espace disque utilisé*: somme (espace disque logique utilisé) de tous les hôtes.
* *Espace disque disponible*: espace disque total moins l'espace disque utilisé.
* *Pourcentage de disque utilisé*: espace disque utilisé divisé par l'espace disque total.
* *Pourcentage de disque disponible*: espace disque disponible divisé par l'espace disque total.

![image de la page Stockage en attachement direct de gestion de la capacité](./media/log-analytics-capacity/oms-capacity04.png)

**Performances du disque**

OMS vous permet de consulter l’historique des tendances d’utilisation de l’espace disque. La fonction de projection utilise un algorithme pour estimer l'utilisation future. Elle vous permet notamment de prévoir l'épuisement de l'espace disque. Cette projection vous permet de planifier le stockage approprié et de savoir quand acheter plus de stockage.

**Outil de projection**

L'outil de projection vous permet d'afficher l'historique des tendances d'utilisation de l'espace disque. La fonction de projection vous permet également de prévoir l'épuisement de l'espace disque. Cette projection vous permet de planifier la capacité appropriée et de savoir quand acheter plus de capacité.

### <a name="to-work-with-items-on-the-direct-attached-storage-page"></a>Utilisation des éléments de la page Stockage en attachement direct
1. La zone **Utilisation** du tableau de bord **Stockage en attachement direct** vous permet d’afficher les informations relatives à l’utilisation des disques.
2. Cliquez sur un élément lié pour l'ouvrir dans la page de **recherche** et en afficher les informations détaillées.
3. La zone **Performances du disque** permet de consulter le taux d'utilisation et de latence des disques.
4. Dans l' **outil de projection**, déplacez le curseur de date pour afficher une projection de la capacité qui sera utilisée à la date choisie.

## <a name="next-steps"></a>Étapes suivantes
* Utilisez les [Recherches de journaux dans Log Analytics](log-analytics-log-searches.md) pour afficher les données détaillées de la gestion de la capacité.



<!--HONumber=Nov16_HO3-->


