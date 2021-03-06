---
title: "Didacticiel : Intégration d’Azure Active Directory avec Kintone | Microsoft Docs"
description: "Apprenez à utiliser Kintone avec Azure Active Directory pour activer l’authentification unique, l’approvisionnement automatique et bien plus encore."
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: c2b947dc-e1a8-4f5f-b40e-2c5180648e4f
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/18/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7a25e030e8ade95db2ac9cd58fe4d94bbb7775ba


---
# <a name="tutorial-azure-active-directory-integration-with-kintone"></a>Didacticiel : Intégration d’Azure Active Directory à Kintone
L’objectif de ce didacticiel est de montrer comment intégrer Azure et Kintone.  
Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un abonnement Azure valide
* Un abonnement Kintone pour lequel l’authentification unique est activée

À l’issue de ce didacticiel, les utilisateurs Azure AD que vous avez affectés à Kintone pourront s’authentifier de manière unique dans l’application sur votre site d’entreprise Kintone (connexion initiée par le fournisseur du service) ou en s’aidant de la [Présentation du volet d’accès](active-directory-saas-access-panel-introduction.md)

Le scénario décrit dans ce didacticiel se compose des blocs de construction suivants :

1. Activation de l’intégration d’application pour Kintone
2. Configuration de l'authentification unique
3. Configuration de l'approvisionnement des utilisateurs
4. Affectation d’utilisateurs

![Scénario](./media/active-directory-saas-kintone-tutorial/IC785859.png "Scenario")

## <a name="enabling-the-application-integration-for-kintone"></a>Activation de l’intégration d’application pour Kintone
Cette section décrit l’activation de l’intégration d’application pour Kintone.

### <a name="to-enable-the-application-integration-for-kintone-perform-the-following-steps"></a>Pour activer l’intégration d’application pour Kintone, procédez comme suit :
1. Dans le volet de navigation gauche du portail Azure Classic, cliquez sur **Active Directory**.
   
    ![Active Directory](./media/active-directory-saas-kintone-tutorial/IC700993.png "Active Directory")

2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.

3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
    ![Applications](./media/active-directory-saas-kintone-tutorial/IC700994.png "Applications")

4. Cliquez sur **Ajouter** en bas de la page.
   
    ![Ajouter une application](./media/active-directory-saas-kintone-tutorial/IC749321.png "Add application")

5. Dans la boîte de dialogue **Que voulez-vous faire ?**, cliquez sur **Ajouter une application à partir de la galerie**.
   
    ![Ajouter une application à partir de la galerie](./media/active-directory-saas-kintone-tutorial/IC749322.png "Add an application from gallerry")

6. Dans la **zone de recherche**, entrez **Kintone**.
   
    ![Galerie d’applications](./media/active-directory-saas-kintone-tutorial/IC785867.png "Application Gallery")

7. Dans le volet de résultats, sélectionnez **Kintone**, puis cliquez sur **Terminer** pour ajouter l’application.
   
    ![Kintone](./media/active-directory-saas-kintone-tutorial/IC785871.png "Kintone")
   
## <a name="configuring-single-sign-on"></a>Configuration de l'authentification unique

Cette section explique comment permettre aux utilisateurs de s’authentifier sur Kintone avec leur compte Azure AD en utilisant la fédération basée sur le protocole SAML.

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :
1. Dans la page d’intégration d’applications **Kintone** du portail Azure Classic, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kintone-tutorial/IC785872.png "Configure Single Sign-On")

2. Dans la page **Comment voulez-vous que les utilisateurs se connectent à Kintone**, sélectionnez **Authentification unique avec Microsoft Azure AD**, puis cliquez sur **Suivant**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kintone-tutorial/IC785873.png "Configure Single Sign-On")

3. Dans la page **Configurer l’URL de l’application**, dans la zone de texte **URL de connexion à Kintone**, tapez votre URL au format « *https://company.kintone.com* », puis cliquez sur **Suivant***.
   
    ![Configurer l’URL de l’application](./media/active-directory-saas-kintone-tutorial/IC785875.png "Configure App URL")

4. Dans la page **Configurer l’authentification unique sur Kintone**, cliquez sur **Télécharger le certificat**, puis enregistrez le fichier de certificat sur votre ordinateur.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kintone-tutorial/IC785878.png "Configure Single Sign-On")

5. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise **Kintone** en tant qu’administrateur.

6. Cliquez sur **Settings**.
   
    ![Settings](./media/active-directory-saas-kintone-tutorial/IC785879.png "Settings")

7. Cliquez sur **Users & System Administration**.
   
    ![Users & System Administration](./media/active-directory-saas-kintone-tutorial/IC785880.png "Users & System Administration")

8. Sous **System Administration \> Security**, cliquez sur **Login**.
   
    ![Connexion](./media/active-directory-saas-kintone-tutorial/IC785881.png "Login")

9. Cliquez sur **Enable SAML authentication**.
   
    ![Authentification SAML](./media/active-directory-saas-kintone-tutorial/IC785882.png "SAML Authentication")

10. Dans la section SAML Authentication, procédez comme suit :
    
    ![SAML Authentication](./media/active-directory-saas-kintone-tutorial/IC785883.png "SAML Authentication")
    
    1. Dans la page de boîte de dialogue **Configurer l’authentification unique sur Kintone** du portail Azure Classic, copiez la valeur **URL de connexion distante**, puis collez-la dans la zone de texte **URL de connexion**.
   
    2. Dans la page de boîte de dialogue **Configurer l’authentification unique sur Kintone** du portail Azure Classic, copiez la valeur **URL de déconnexion distante**, puis collez-la dans la zone de texte **URL de déconnexion**.
    
    3. Cliquez sur **Parcourir** pour charger le certificat téléchargé.
    
    4. Cliquez sur **Save**.

11. Dans le portail Azure Classic, sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer l’authentification unique**.
    
    ![Configurer l’authentification unique](./media/active-directory-saas-kintone-tutorial/IC785884.png "Configure Single Sign-On")
    
## <a name="configuring-user-provisioning"></a>Configuration de l'approvisionnement des utilisateurs

Pour se connecter à Kintone, les utilisateurs d’Azure AD doivent être approvisionnés dans Kintone.  
Dans le cas de Kintone, l’approvisionnement est une tâche manuelle.

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>Pour approvisionner un compte d’utilisateur, procédez comme suit :
1. Connectez-vous à votre site d’entreprise **Kintone** en tant qu’administrateur.

2. Cliquez sur **Setting**.
   
    ![Settings](./media/active-directory-saas-kintone-tutorial/IC785879.png "Settings")

3. Cliquez sur **Users & System Administration**.
   
    ![Administration système et utilisateur](./media/active-directory-saas-kintone-tutorial/IC785880.png "User & System Administration")

4. Sous **User Administration**, cliquez sur **Departments & Users**.
   
    ![Département et utilisateurs](./media/active-directory-saas-kintone-tutorial/IC785888.png "Department & Users")

5. Cliquez sur **New User**.
   
    ![Nouveaux utilisateurs](./media/active-directory-saas-kintone-tutorial/IC785889.png "New Users")

6. Dans la section **New User** , procédez comme suit :
   
    ![Nouveaux utilisateurs](./media/active-directory-saas-kintone-tutorial/IC785890.png "New Users")
   
    1. Tapez le nom complet, le nom de connexion, le nouveau mot de passe et sa confirmation et l’adresse de messagerie d’un compte AAD valide que vous souhaitez approvisionner dans les zones de texte correspondantes, à savoir, **Display Name**, **Login Name**, **New Password**, **Confirm Password** et **E-mail Address**.
 
    2. Cliquez sur **Save**.

> [!NOTE]
> Vous pouvez utiliser n’importe quel autre outil ou API de création de compte d’utilisateur Kintone fourni par ce service pour approvisionner des comptes d’utilisateurs Azure Active Directory.
> 
> 

## <a name="assigning-users"></a>Affectation d’utilisateurs
Pour tester votre configuration, vous devez autoriser les utilisateurs d’Azure AD concernés à accéder à votre application.

### <a name="to-assign-users-to-kintone-perform-the-following-steps"></a>Pour affecter des utilisateurs à Kintone, procédez comme suit :
1. Dans le portail Azure Classic, créez un compte de test.

2. Dans la page d’intégration d’applications **Kintone**, cliquez sur **Affecter des utilisateurs**.
   
    ![Affecter des utilisateurs](./media/active-directory-saas-kintone-tutorial/IC785891.png "Assign Users")

3. Sélectionnez votre utilisateur de test, cliquez sur **Affecter**, puis sur **Oui** pour confirmer votre affectation.
   
    ![Oui](./media/active-directory-saas-kintone-tutorial/IC767830.png "Yes")

Si vous souhaitez tester vos paramètres d’authentification unique, ouvrez le volet d’accès. Pour plus d'informations sur le panneau d'accès, consultez [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).




<!--HONumber=Nov16_HO3-->


