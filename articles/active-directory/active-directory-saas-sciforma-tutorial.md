---
title: "Didacticiel : intégration d’Azure Active Directory à Sciforma | Microsoft Docs"
description: "Découvrez comment utiliser Sciforma avec Azure Active Directory pour activer l’authentification unique, l’approvisionnement automatique et bien plus encore."
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: abbfb5ac-7687-4153-b263-8090102dae37
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/03/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 9a653ac435198e89a527070a0174a1adaf830dc3
ms.openlocfilehash: 166524147c050512a84c1c0340de93960d8b7632


---
# <a name="tutorial-azure-ad-integration-with-sciforma"></a>Didacticiel : Intégration d’Azure AD à Sciforma
L’objectif de ce didacticiel est de montrer comment intégrer Azure et Sciforma.  
Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un abonnement Azure valide
* Un locataire Sciforma

À l’issue de ce didacticiel, les utilisateurs d’Azure AD que vous avez affectés à Sciforma pourront s’authentifier de manière unique dans l’application sur votre site d’entreprise Sciforma (connexion initiée par le fournisseur du service) ou en s’aidant de la [Présentation du volet d’accès](active-directory-saas-access-panel-introduction.md).

Le scénario décrit dans ce didacticiel se compose des blocs de construction suivants :

1. Activation de l’intégration d’applications pour Sciforma
2. Configuration de l'authentification unique
3. Configuration de l'approvisionnement des utilisateurs
4. Affectation d’utilisateurs

![Scénario](./media/active-directory-saas-sciforma-tutorial/IC777369.png "Scénario")

## <a name="enabling-the-application-integration-for-sciforma"></a>Activation de l’intégration d’applications pour Sciforma
Cette section décrit l’activation de l’intégration d’applications pour Sciforma.

### <a name="to-enable-the-application-integration-for-sciforma-perform-the-following-steps"></a>Pour activer l’intégration d’applications pour Sciforma, procédez comme suit :
1. Dans le volet de navigation gauche du portail Azure Classic, cliquez sur **Active Directory**.
   
    ![Active Directory](./media/active-directory-saas-sciforma-tutorial/IC700993.png "Active Directory")

2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.

3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
    ![Applications](./media/active-directory-saas-sciforma-tutorial/IC700994.png "Applications")

4. Cliquez sur **Ajouter** en bas de la page.
   
    ![Ajouter une application](./media/active-directory-saas-sciforma-tutorial/IC749321.png "Ajouter une application")

5. Dans la boîte de dialogue **Que voulez-vous faire ?**, cliquez sur **Ajouter une application à partir de la galerie**.
   
    ![Ajouter une application à partir de la galerie](./media/active-directory-saas-sciforma-tutorial/IC749322.png "Ajouter une application à partir de la galerie")

6. Dans la **zone de recherche**, entrez **Sciforma**.
   
    ![Galerie d’applications](./media/active-directory-saas-sciforma-tutorial/IC777370.png "Galerie d’applications")

7. Dans le volet des résultats, sélectionnez **Sciforma**, puis cliquez sur **Terminer** pour ajouter l’application.
   
    ![Sciforma](./media/active-directory-saas-sciforma-tutorial/IC777371.png "Sciforma")
   
## <a name="configuring-single-sign-on"></a>Configuration de l'authentification unique

Cette section explique comment permettre aux utilisateurs de s’authentifier sur Sciforma avec leur compte Azure AD en utilisant la fédération basée sur le protocole SAML.

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :
1. Sur la page d’intégration d’applications **Sciforma** du portail Azure Classic, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-sciforma-tutorial/IC777372.png "Configurer l’authentification unique")

2. Dans la page **Comment voulez-vous que les utilisateurs se connectent à Sciforma**, sélectionnez **Authentification unique avec Microsoft Azure AD**, puis cliquez sur **Suivant**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-sciforma-tutorial/IC777373.png "Configurer l’authentification unique")

3. Dans la zone de texte **URL de connexion à Sciforma** de la page **Configurer l’URL de l’application**, tapez votre URL selon le modèle suivant « *https://\<nom-locataire\>.Sciforma.com* », puis cliquez sur **Suivant**.
   
    ![Configurer l’URL de l’application](./media/active-directory-saas-sciforma-tutorial/IC777374.png "Configurer l’URL de l’application")

4. Dans la page **Configurer l’authentification unique sur Sciforma**, pour télécharger vos métadonnées, cliquez sur **Télécharger les métadonnées**, puis enregistrez le fichier de données en local sous le nom **c:\\\SciformaMetaData.xml**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-sciforma-tutorial/IC777375.png "Configurer l’authentification unique")

5. Transmettez ce fichier de métadonnées à l’équipe de support de Sciforma. L’équipe de support configure l’authentification unique pour vous.

6. Sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer l’authentification unique**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-sciforma-tutorial/IC777376.png "Configurer l’authentification unique")
   
## <a name="configuring-user-provisioning"></a>Configuration de l'approvisionnement des utilisateurs

Vous n’avez rien à faire pour configurer l’approvisionnement des utilisateurs dans Sciforma.  
Lorsqu’un utilisateur tente de se connecter à Sciforma à l’aide du panneau d’accès, Sciforma vérifie si cet utilisateur existe.  
Si aucun compte d’utilisateur n’est disponible, Sciforma le crée automatiquement.

## <a name="assigning-users"></a>Affectation d’utilisateurs
Pour tester votre configuration, vous devez autoriser les utilisateurs d’Azure AD concernés à accéder à votre application.

### <a name="to-assign-users-to-sciforma-perform-the-following-steps"></a>Pour affecter des utilisateurs à Sciforma, procédez comme suit :
1. Dans le portail Azure Classic, créez un compte de test.

2. Dans la page d’intégration d’applications **Sciforma**, cliquez sur **Affecter des utilisateurs**.
   
    ![Affecter des utilisateurs](./media/active-directory-saas-sciforma-tutorial/IC777377.png "Affecter des utilisateurs")

3. Sélectionnez votre utilisateur de test, cliquez sur **Affecter**, puis sur **Oui** pour confirmer votre affectation.
   
    ![Oui](./media/active-directory-saas-sciforma-tutorial/IC767830.png "Oui")

Si vous souhaitez tester vos paramètres d’authentification unique, ouvrez le volet d’accès. Pour plus d'informations sur le panneau d'accès, consultez [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).




<!--HONumber=Jan17_HO1-->


