---
title: 'Troubleshooting: ''Active Directory'' item is missing or not available | Microsoft Docs'
description: "Que faire lorsque l’élément de menu Active Directory n’apparaît pas dans le portail de gestion Azure."
services: active-directory
documentationcenter: na
author: bryanla
manager: mbaldwin
editor: 
ms.assetid: 3383020d-6397-43ea-b7aa-c6a9d6a1e3df
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/07/2017
ms.author: mbaldwin
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 9c52e9257272c5b0c55c488d2c7ca6fef500260b


---
# <a name="troubleshooting-active-directory-item-is-missing-or-not-available"></a>Dépannage : l’élément « Active Directory » est manquant ou non disponible
De nombreuses instructions relatives à l’utilisation des fonctionnalités et des services d’Azure Active Directory commencent par « Accédez au portail de gestion Azure et cliquez sur **Active Directory**. » Mais que faire si l’élément de menu ou l’extension Active Directory n’apparaît pas ou si elle est marquée comme **non disponible**? Cette rubrique d’aide répond à cette question. Elle décrit les conditions dans lesquelles **Active Directory** n’apparaît pas ou n’est pas disponible et explique comment procéder.

## <a name="active-directory-is-missing"></a>Active Directory est manquant
En règle générale, un élément **Active Directory** apparaît dans le menu de navigation gauche. Les instructions dans les procédures d’Azure Active Directory supposent que cet élément est visible.

![Capture d’écran : Active Directory dans Azure](./media/active-directory-troubleshooting/typical-view.png)

L’élément Active Directory s’affiche dans le menu de navigation gauche lorsqu’une des conditions suivantes est vraie. Sinon, l’élément n’apparaît pas.

* L’utilisateur actuel est connecté avec un compte Microsoft (anciennement appelé identifiant Windows Live ID).
  
    OU
* Le locataire Azure possède un répertoire et le compte actuel est un administrateur d’annuaire.
  
    OU
* Le locataire Azure possède au moins un espace de noms Azure AD Access Control (ACS). Pour plus d’informations, consultez la page [À propos des noms d’espaces de contrôle d’accès](https://msdn.microsoft.com/library/azure/gg185908.aspx).
  
    OU
* Le locataire Azure possède au moins un fournisseur d’authentification multifacteur Azure. Pour plus d’informations, consultez la rubrique [Administration des fournisseurs Azure Multi-Factor Authentication](../multi-factor-authentication/multi-factor-authentication-get-started-cloud.md).

Pour créer un espace de noms de contrôle d’accès ou un fournisseur Multi-Factor Authentication, cliquez sur **+ New** > **App Services** > **Active Directory**.

Pour obtenir des droits d’administration d’un répertoire, demandez à un administrateur d’attribuer un rôle d’administrateur à votre compte. Pour plus de détails, consultez [Attribution de rôles d’administrateur](active-directory-assign-admin-roles.md)

## <a name="active-directory-is-not-available"></a>Active Directory n’est pas disponible
Lorsque vous cliquez sur **+ New** > **App Services**, un élément **Active Directory** s’affiche. Plus précisément, l’élément Active Directory s’affiche lorsqu’une des fonctionnalités Active Directory, comme le répertoire, le contrôle d’accès ou le fournisseur Multi-Factor Auth, est disponible pour l’utilisateur actuel.

Toutefois, pendant le chargement de la page, l’élément est estompé et est marqué comme **non disponible**. Il s’agit d’un état temporaire. Si vous attendez quelques secondes, l’élément devient disponible. Si le délai se prolonge, l’actualisation de la page Web résout souvent le problème.

![Capture d’écran : Active Directory n’est pas disponible](./media/active-directory-troubleshooting/not-available.png)




<!--HONumber=Nov16_HO3-->


