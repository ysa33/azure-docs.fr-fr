---
title: Azure IoT Suite et Azure Active Directory | Microsoft Docs
description: "Décrit comment Azure IoT Suite utilise Azure Active Directory pour gérer les autorisations."
services: 
suite: iot-suite
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 246228ba-954a-4d96-b6d6-e53e4590cb4f
ms.service: iot-suite
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/08/2017
ms.author: corywink
translationtype: Human Translation
ms.sourcegitcommit: 14e2fcea9a6afbac640d665d5e44a700f855db4b
ms.openlocfilehash: c6db7f36acf93d319fcbc93244867460a2270979


---
# <a name="permissions-on-the-azureiotsuitecom-site"></a>Autorisations sur le site azureiotsuite.com
## <a name="what-happens-when-you-sign-in"></a>Que se passe-t-il lorsque vous vous connectez ?
Lorsque vous vous connectez pour la première fois à [azureiotsuite.com][lnk-azureiotsuite], le site détermine vos niveaux d’autorisation en fonction du client Azure Active Directory (AAD) et de l’abonnement Azure sélectionné.

1. Tout d’abord, afin de remplir la liste des clients qui apparaît en regard de votre nom d’utilisateur, le site recherche dans Azure les clients ADD auxquels vous appartenez. Actuellement, le site peut obtenir des jetons d’utilisateur pour un seul client. Par conséquent, lorsque vous basculez sur d’autres clients en utilisant la liste déroulante dans le coin supérieur droit, le site vous reconnecte à ce client pour obtenir les jetons pour ce client.
2. Ensuite, le site détecte à partir d’Azure, les abonnements que vous avez associés au client sélectionné. Vous pouvez voir les abonnements disponibles lorsque vous créez une nouvelle solution préconfigurée.
3. Enfin, le site extrait toutes les ressources dans les abonnements et les groupes de ressources marqués en tant que solutions préconfigurées et renseigne les mosaïques de la page d’accueil.

Les sections suivantes décrivent les rôles qui contrôlent l’accès aux solutions préconfigurées.

## <a name="aad-roles"></a>Rôles AAD
Les rôles AAD contrôlent la possibilité d’approvisionnement de solutions préconfigurées et gèrent les utilisateurs dans une solution préconfigurée.

Pour plus d’informations sur les rôles d’administrateur dans ADD, consultez la page [Attribution de rôles d’administrateur dans Azure AD][lnk-aad-admin]. Cet article se concentre sur les rôles **d’administrateur général** et **d’utilisateur/membre de domaine** utilisés par les solutions préconfigurées.

**Administrateur général :** il peut y avoir de nombreux administrateurs généraux par client AAD. Lorsque vous créez un client AAD, vous en devenez par défaut l’administrateur. L’administrateur général peut approvisionner une solution préconfigurée et se voir attribuer le rôle **d’ADMINISTRATEUR** de l’application au sein de leur client AAD. Toutefois, si un autre utilisateur du même client AAD crée une application, le rôle attribué par défaut à l’administrateur général par défaut **LECTURE SEULE IMPLICITE**. Les administrateurs généraux peuvent attribuer des rôles aux applications utilisant le [portail Azure Classic][lnk-classic-portal].

**Utilisateur/membre de domaine :** pour chaque client AAD, il peut y avoir de nombreux utilisateurs/membres. Un utilisateur de domaine peut approvisionner une solution préconfigurée via le site [azureiotsuite.com][lnk-azureiotsuite]. Le rôle par défaut qu’ils se voient attribuer pour l’application qu’ils approvisionnent est celui d’ **ADMINISTRATEUR**. Ils peuvent créer une application à l’aide du script build.cmd dans le référentiel [azure-iot-remote-monitoring][lnk-rm-github-repo] ou [azure-iot-predictive-maintenance][lnk-pm-github-repo]. Toutefois, ils se voient affecter par défaut le rôle **LECTURE SEULE IMPLICITE**, car ils n’ont pas l’autorisation d’attribuer des rôles. Si un autre utilisateur du client AAD crée une application, ils se voient affecter le rôle **LECTURE SEULE IMPLICITE** par défaut pour cette application. Ils ne peuvent pas attribuer des rôles aux applications ; par conséquent, ils ne peuvent pas ajouter des utilisateurs ou des rôles pour les utilisateurs d’une application même s’ils l’ont approvisionnée.

**Utilisateur invité/Invité :** il peut y avoir de nombreux utilisateurs invités/invités par le client AAD. Les utilisateurs invités disposent d’un ensemble limité de droits au sein du client AAD. Par conséquent, les utilisateurs invités ne peuvent pas approvisionner une solution préconfigurée dans le client AAD.

Pour plus d’informations, consultez les ressources suivantes :

* [Création ou modification des utilisateurs dans Azure AD][lnk-create-edit-users]
* [Attribution de rôles d’application dans AAD][lnk-assign-app-roles]

## <a name="azure-subscription-administrator-roles"></a>Rôles de l’administrateur d’abonnement Azure
Les rôles d’administration Azure permettent de contrôler la fonctionnalité de mappage d’un abonnement Azure sur un client AD.

Vous trouverez d’autres informations sur les rôles de coadministrateur, d’administrateur de service et d’administrateur de compte Azure dans l’article [How to add or change Azure Co-Administrator, Service Administrator and Account Administrator][lnk-admin-roles] (Comment ajouter ou modifier un coadministrateur, un administrateur de service et un administrateur de compte Azure).

## <a name="application-roles"></a>Rôles d’application
Les rôles d’application contrôlent l’accès aux périphériques de votre solution préconfigurée.

Dans l’application créée, deux rôles et un rôle implicite sont définis lorsque vous approvisionnez une solution préconfigurée.

* **ADMINISTRATEUR :** contrôle totalement l’ajout, la gestion et la suppression des appareils
* **LECTURE SEULE :** peut afficher les appareils.
* **LECTURE SEULE IMPLICITE :** il s’agit du même rôle que la lecture seule, mais il est accordé à tous les utilisateurs de votre client AAD. Ce rôle a été créé pour des raisons pratiques au cours du développement. Vous pouvez supprimer ce rôle en modifiant le fichier source [RolePermissions.cs][lnk-resource-cs].

### <a name="changing-application-roles-for-a-user"></a>Modification des rôles d’application pour un utilisateur
Vous pouvez utiliser la procédure suivante pour définir un utilisateur dans Active Directory en tant qu’administrateur de votre solution préconfigurée.

Vous devez être un administrateur général AAD pour modifier les rôles d’un utilisateur :

1. Connectez-vous au [Portail Azure Classic][lnk-classic-portal].
2. Sélectionnez **Active Directory**.
3. Cliquez sur le nom de votre client AAD (il s’agit du répertoire que vous avez choisi sur azureiotsuite.com lorsque vous avez approvisionné votre solution).
4. Cliquez sur **Applications**.
5. Cliquez sur le nom de l’application qui correspond au nom de votre solution préconfigurée. Si vous ne voyez pas votre application dans la liste, faites défiler le menu déroulant **Afficher** jusqu’à **Applications que ma société possède**, puis cliquez sur la coche.
6. Cliquez sur **Utilisateurs**.
7. Sélectionnez l’utilisateur dont vous souhaitez permuter les rôles.
8. Cliquez sur **Attribuer**, puis sélectionnez le rôle (par exemple, **Admin**) que vous souhaitez attribuer à l’utilisateur en cochant la case correspondante.

## <a name="faq"></a>Forum Aux Questions
### <a name="im-a-service-administrator-and-id-like-to-change-the-directory-mapping-between-my-subscription-and-a-specific-aad-tenant-how-do-i-complete-this-task"></a>Je suis un administrateur de service et je souhaite modifier le mappage d’annuaire entre mon abonnement et un client AAD spécifique. Comment mener à bien cette tâche ?
1. Accédez au [portail Azure Classic][lnk-classic-portal] et cliquez sur **Paramètres** dans la liste des services sur le côté gauche.
2. Sélectionnez l’abonnement dont vous souhaitez modifier le mappage d’annuaire.
3. Cliquez sur **Modifier l’annuaire**.
4. Sélectionnez l’ **annuaire** que vous souhaitez utiliser dans la liste déroulante. Cliquez sur la flèche Suivant.
5. Vérifiez le mappage d’annuaire et les coadministrateurs affectés. Notez que si vous migrez depuis un autre annuaire, tous les coadministrateurs de l’annuaire d’origine sont supprimés.

### <a name="im-a-domain-usermember-on-the-aad-tenant-and-ive-created-a-preconfigured-solution-how-do-i-get-assigned-a-role-for-my-application"></a>Je suis un utilisateur/membre du domaine sur le client AAD et j’ai créé une solution préconfigurée. Comment puis-je me voir attribuer un rôle pour mon application ?
Demandez à un administrateur général de vous attribuer le rôle d’administrateur général sur le client AAD pour obtenir les autorisations nécessaires pour affecter vous-même les rôles aux utilisateurs, ou demandez à un administrateur général de vous attribuer un rôle. Si vous souhaitez modifier le client AAD sur lequel votre solution préconfigurée a été déployée, passez à la question suivante.

### <a name="how-do-i-switch-the-aad-tenant-my-remote-monitoring-preconfigured-solution-and-application-are-assigned-to"></a>Comment puis-je activer le client AAD auquel la solution préconfigurée de contrôle à distance et l’application sont affectées ?
Vous pouvez exécuter un déploiement cloud à partir de <https://github.com/Azure/azure-iot-remote-monitoring>, puis redéployer avec un client AAD nouvellement créé. Comme vous êtes par défaut administrateur général, lorsque vous créez un client AAD, vous disposez des autorisations pour ajouter des utilisateurs et leur affecter des rôles.

1. Créez un répertoire ADD dans le [portail Azure Classic][lnk-classic-portal].
2. Accédez à <https://github.com/Azure/azure-iot-remote-monitoring>.
3. Exécutez `build.cmd cloud [debug | release] {name of previously deployed remote monitoring solution}` (par exemple, `build.cmd cloud debug myRMSolution`)
4. Lorsque vous y êtes invité, définissez le **tenantid** du client que vous venez de créer et non celui du client précédent.

### <a name="i-want-to-change-a-service-administrator-or-co-administrator-when-logged-in-with-an-organisational-account"></a>Je souhaite modifier la fonctionnalité administrateur de service ou coadministrateur lors d’une connexion avec un compte de société
Voir l’article de support [Modification de la fonctionnalité Administrateur de service et Coadministrateur lors d’une connexion avec un compte de société][lnk-service-admins].

### <a name="why-am-i-seeing-this-error-your-account-does-not-have-the-proper-permissions-to-create-a-solution-please-check-with-your-account-administrator-or-try-with-a-different-account"></a>Pourquoi est-ce que je reçois cette erreur ? « Votre compte n’a pas les autorisations suffisantes pour créer une solution. Veuillez contacter votre administrateur de compte ou essayer avec un autre compte. »
Examinez le schéma suivant pour obtenir des conseils :

![][img-flowchart]

> [!NOTE]
> Si l’erreur persiste après votre validation en tant qu’administrateur global sur le client AAD et que coadministrateur sur l’abonnement, demandez à votre administrateur de compte de supprimer l’utilisateur et de réattribuer les autorisations nécessaires dans l’ordre suivant : tout d’abord, ajoutez l’utilisateur en tant qu’administrateur global, puis ajoutez un utilisateur en tant que coadministrateur sur l’abonnement Azure. Si les problèmes persistent, contactez le service [Aide et support][lnk-help-support].
> 
> 

**Pourquoi affiche-t-il cette erreur alors que j’ai un abonnement Azure ?** *Un abonnement Azure est nécessaire pour créer des solutions préconfigurées. Vous pouvez créer un compte d’essai gratuit en quelques minutes.*

Si vous êtes sûr de disposer d’un abonnement Azure, validez le mappage de votre abonnement client et assurez-vous que c’est le bon client qui est sélectionné dans la liste déroulante. Si vous avez validé le client souhaité, suivez le schéma ci-dessus et validez le mappage de votre abonnement et de ce client AAD.

## <a name="next-steps"></a>Étapes suivantes
Pour poursuivre votre formation concernant IoT Suite, découvrez comment [personnaliser une solution préconfigurée][lnk-customize].

[img-flowchart]: media/iot-suite-permissions/flowchart.png

[lnk-azureiotsuite]: https://www.azureiotsuite.com/
[lnk-rm-github-repo]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-pm-github-repo]: https://github.com/Azure/azure-iot-predictive-maintenance
[lnk-aad-admin]: https://azure.microsoft.com/documentation/articles/active-directory-assign-admin-roles/
[lnk-classic-portal]: https://manage.windowsazure.com/
[lnk-create-edit-users]: https://azure.microsoft.com/documentation/articles/active-directory-create-users/
[lnk-assign-app-roles]: https://azure.microsoft.com/documentation/articles/active-directory-application-manifest/
[lnk-service-admins]: https://azure.microsoft.com/support/changing-service-admin-and-co-admin/
[lnk-admin-roles]: https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/
[lnk-resource-cs]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/DeviceAdministration/Web/Security/RolePermissions.cs
[lnk-help-support]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[lnk-customize]: iot-suite-guidance-on-customizing-preconfigured-solutions.md



<!--HONumber=Feb17_HO2-->


