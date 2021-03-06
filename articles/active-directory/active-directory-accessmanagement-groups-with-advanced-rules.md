---
title: "Utilisation d’attributs pour créer des règles avancées | Microsoft Docs"
description: "Procédure de création de règles avancées pour un groupe incluant des paramètres et des opérateurs de règle d’expression pris en charge."
services: active-directory
documentationcenter: 
author: curtand
manager: femila
editor: 
ms.assetid: 04813a42-d40a-48d6-ae96-15b7e5025884
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/13/2017
ms.author: curtand
translationtype: Human Translation
ms.sourcegitcommit: 4bab9f44d1c91f05618ea510b83beb06540429f2
ms.openlocfilehash: 00424292fbc5321a77a4e924530ade97739208d4


---
# <a name="using-attributes-to-create-advanced-rules"></a>Utilisation d’attributs pour créer des règles avancées
Le portail Azure Classic vous permet de créer des règles avancées pour activer des appartenances dynamiques plus complexes basées sur les attributs aux groupes Azure Active Directory (Azure AD).  

Lorsqu’un attribut d’un utilisateur change, le système évalue toutes les règles de groupe dynamique dans un annuaire pour voir si la modification de l’attribut de l’utilisateur déclenche des ajouts ou suppressions de groupe. Si un utilisateur respecte une règle d’un groupe, il est ajouté en tant que membre de ce groupe. S’il ne respecte plus la règle d’un groupe dont il est membre, il est supprimé de ce groupe.

> [!NOTE]
> Vous pouvez définir une règle d’appartenance dynamique sur les groupes de sécurité ou Office 365. Les appartenances à des groupes imbriquées ne sont pas prises en charge pour l’affectation basée sur le groupe à des applications.
>
> L’appartenance dynamique à des groupes nécessite qu’une licence Azure AD Premium soit affectée à
>
> * L’administrateur qui gère la règle sur un groupe
> * Tous les membres du groupe
>

## <a name="to-create-the-advanced-rule"></a>Pour créer une règle avancée
1. Dans le [portail Azure Classic](https://manage.windowsazure.com), sélectionnez **Active Directory**, puis ouvrez le répertoire de votre organisation.
2. Sélectionnez l’onglet **Groupes** , puis ouvrez le groupe que vous souhaitez modifier.
3. Sélectionnez l’onglet **Configurer**, sélectionnez l’option **Règle avancée** et entrez la règle avancée dans la zone de texte.

## <a name="constructing-the-body-of-an-advanced-rule"></a>Construction du corps d’une règle avancée
La règle avancée que vous pouvez créer pour l’appartenance dynamique à des groupes est essentiellement une expression binaire qui se compose de trois parties et qui génère un résultat true ou false. Les trois parties sont les suivantes :

* Paramètre de gauche (leftParameter)
* Opérateur binaire (binaryOperator)
* Constante de droite (rightConstant)

Une règle avancée complète ressemble à ceci : (leftParameter binaryOperator «RightConstant»). Elle nécessite des parenthèses ouvrantes et fermantes pour l’ensemble de l’expression binaire, des guillemets doubles pour la constante de droite et l’utilisation de la syntaxe user.property pour le paramètre de gauche. Une règle avancée peut se composer de plusieurs expressions binaires séparées par les opérateurs logiques -and, -or et -not.
Voici des exemples de règles avancées correctement construites :

* (user.department -eq "Sales") -or (user.department -eq "Marketing")
* (user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")

Pour obtenir la liste complète des paramètres et des opérateurs de règle d’expression pris en charge, consultez les sections ci-dessous.

Notez que la propriété doit être préfixée par le type d’objet approprié : utilisateur ou appareil.
La règle ci-dessous échoue à la validation : mail –ne null

La règle correcte serait :

user.mail –ne null

La longueur totale du corps de votre règle avancée ne peut pas dépasser 2 048 caractères.

> [!NOTE]
> Les opérations de chaîne et regex (expressions régulières) ne prennent pas en compte la casse.
> Les chaînes contenant des guillemets doubles doivent être placées dans une séquence d’échappement à l’aide du caractère « ' ». Par exemple : `"\`Sales".
> Utilisez uniquement des guillemets pour les valeurs de type chaîne ; utilisez uniquement des guillemets anglais.
>
>

## <a name="supported-expression-rule-operators"></a>Opérateurs de règle d’expression pris en charge
Le tableau suivant répertorie tous les opérateurs de règle d’expression pris en charge et leur syntaxe à utiliser dans le corps de la règle avancée :

| Opérateur | Syntaxe |
| --- | --- |
| Non égal à |-ne |
| Égal à |-eq |
| Ne commence pas par |-notStartsWith |
| Commence par |-startsWith |
| Ne contient pas |-notContains |
| Contient |-contains |
| Ne correspond pas |-notMatch |
| Correspond |-match |

## <a name="operator-precedence"></a>Précédence des opérateurs

Tous les opérateurs sont répertoriés ci-dessous par précédence, de la plus basse à la plus élevée, les opérateurs dans la même ligne sont de même précédence -any -all -or -and -not -eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch

Tous les opérateurs peuvent être utilisés avec ou sans le préfixe de trait d’union.

Notez que les parenthèses ne sont pas toujours nécessaires, vous devez uniquement ajouter des parenthèses lorsque la précédence ne satisfait pas vos exigences Par exemple :

   user.department –eq "Marketing" –and user.country –eq "US"

équivaut à :

   (user.department –eq "Marketing") –and (user.country –eq "US")

## <a name="query-error-remediation"></a>Correction d’erreur de requête
Le tableau suivant répertorie les erreurs potentielles et la méthode pour les corriger si elles se produisent

| Erreur d’analyse de requête | Utilisation incorrecte | Utilisation corrigée |
| --- | --- | --- |
| Erreur : attribut non pris en charge. |(user.invalidProperty -eq "Value") |(user.department -eq "value")<br/>La propriété doit correspondre à l’une de celles figurant dans la [liste des propriétés prises en charge](#supported-properties). |
| Erreur : l’opérateur n’est pas pris en charge sur l’attribut. |(user.accountEnabled -contains true) |(user.accountEnabled - eq true)<br/>La propriété est de type booléen. Utilisez les opérateurs pris en charge (-eq ou -ne) sur un type booléen dans la liste ci-dessus. |
| Erreur : erreur de compilation de la requête. |(user.department -eq "Sales") -and (user.department -eq "Marketing")(user.userPrincipalName -match "*@domain.ext") |(user.department -eq "Sales") -and (user.department -eq "Marketing")<br/>L’opérateur logique doit correspondre à une des valeurs de la liste des propriétés prises en charge ci-dessus. (user.userPrincipalName -match ".*@domain.ext")or(user.userPrincipalName -match "@domain.ext$")Error dans l’expression régulière. |
| Erreur : l’expression binaire n’est pas au format correct. |(user.department –eq “Sales”) (user.department -eq "Sales")(user.department-eq"Sales") |(user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain")<br/>La requête comporte plusieurs erreurs. Une parenthèse n’est pas au bon endroit. |
| Erreur : une erreur inconnue s’est produite lors de la configuration des appartenances dynamiques. |(user.accountEnabled -eq "True" AND user.userPrincipalName -contains "alias@domain") |(user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain")<br/>La requête comporte plusieurs erreurs. Une parenthèse n’est pas au bon endroit. |

## <a name="supported-properties"></a>Propriétés prises en charge
Voici toutes les propriétés d’utilisateur que vous pouvez utiliser dans vos règles avancées :

### <a name="properties-of-type-boolean"></a>Propriétés de type booléen
Opérateurs autorisés

* -eq
* -ne

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true) |
| dirSyncEnabled |true false null |(user.dirSyncEnabled -eq true) |

### <a name="properties-of-type-string"></a>Propriétés de type chaîne
Opérateurs autorisés

* -eq
* -ne
* -notStartsWith
* -startsWith
* -contains
* -notContains
* -match
* -notMatch

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| city |Toute valeur de chaîne ou $null |(user.city -eq "value") |
| country |Toute valeur de chaîne ou $null |(user.country -eq "value") |
| department |Toute valeur de chaîne ou $null |(user.department -eq "value") |
| displayName |Toute valeur de chaîne. |(user.displayName -eq "value") |
| facsimileTelephoneNumber |Toute valeur de chaîne ou $null |(user.facsimileTelephoneNumber -eq "value") |
| givenName |Toute valeur de chaîne ou $null |(user.givenName -eq "value") |
| jobTitle |Toute valeur de chaîne ou $null |(user.jobTitle -eq "value") |
| mail |Toute valeur de chaîne ou $null (adresse SMTP de l’utilisateur) |(user.mail -eq "value") |
| mailNickName |Toute valeur de chaîne (alias de messagerie de l’utilisateur) |(user.mailNickName -eq "value") |
| mobile |Toute valeur de chaîne ou $null |(user.mobile -eq "value") |
| objectId |GUID de l’objet utilisateur |(user.objectId -eq "1111111-1111-1111-1111-111111111111") |
| passwordPolicies |Aucune DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |Toute valeur de chaîne ou $null |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |Toute valeur de chaîne ou $null |(user.postalCode -eq "value") |
| preferredLanguage |Code ISO 639-1 |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |Toute valeur de chaîne ou $null |(user.sipProxyAddress -eq "value") |
| state |Toute valeur de chaîne ou $null |(user.state -eq "value") |
| streetAddress |Toute valeur de chaîne ou $null |(user.streetAddress -eq "value") |
| surname |Toute valeur de chaîne ou $null |(user.surname -eq "value") |
| telephoneNumber |Toute valeur de chaîne ou $null |(user.telephoneNumber -eq "value") |
| usageLocation |Paramètre régional à deux lettres |(user.usageLocation -eq "US") |
| userPrincipalName |Toute valeur de chaîne. |(user.userPrincipalName -eq "alias@domain") |
| userType |member guest $null |(user.userType -eq "Member") |

### <a name="properties-of-type-string-collection"></a>Propriétés de type collection de chaînes
Opérateurs autorisés

* -contains
* -notContains

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| otherMails |Toute valeur de chaîne. |(user.otherMails -contains "alias@domain") |
| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

## <a name="use-of-null-values"></a>Utiliser des valeurs Null

Pour spécifier une valeur null dans une règle, vous pouvez utiliser « null » ou $null. Exemple :

   user.mail –ne null équivaut à user.mail –ne $null

## <a name="extension-attributes-and-custom-attributes"></a>Attributs d’extension et attributs personnalisés
Les attributs d’extension et les attributs personnalisés sont pris en charge dans les règles d’appartenance dynamique.

Les attributs d’extension sont synchronisés à partir d’un Windows Server AD et prennent le format « ExtensionAttributeX », où X est égal à 1-15.
Voici en exemple de règle utilisant un attribut d’extension :

(user.extensionAttribute15 -eq "Marketing")

Les attributs personnalisés sont synchronisés à partir d’un système AD Windows Server local ou d’une application SaaS connectée et du format de « user.extension[GUID]\__[Attribut] », où [GUID] est l’identificateur unique, dans AAD, de l’application qui a créé l’attribut dans AAD, et [Attribut] est le nom de l’attribut tel qu’il a été créé.
Voici un exemple de règle utilisant un attribut personnalisé :

user.extension_c272a57b722d4eb29bfe327874ae79cb__OfficeNumber  

Vous pouvez accéder au nom de l’attribut personnalisé dans le répertoire en lançant une requête sur un attribut d’utilisateur, à l’aide de l’explorateur graphique, et en recherchant le nom d’attribut.

## <a name="support-for-multi-value-properties"></a>Prise en charge des propriétés à valeurs multiples

Pour inclure une propriété à valeurs multiples dans une règle, utilisez l’opérateur "-any" comme dans

  user.assignedPlans -any assignedPlan.service -startsWith "SCO"

## <a name="direct-reports-rule"></a>Règle de collaborateurs
Vous pouvez remplir les membres d’un groupe en fonction de l’attribut de responsable hiérarchique d’un utilisateur.

**Pour configurer un groupe en tant que groupe « Responsable »**

1. Dans le portail Azure Classic, cliquez sur **Active Directory**, puis sur le nom de l’annuaire de votre organisation.
2. Sélectionnez l’onglet **Groupes** , puis ouvrez le groupe que vous souhaitez modifier.
3. Sélectionnez l’onglet **Configurer** puis **RÈGLE AVANCÉE**.
4. Entrez la règle avec la syntaxe suivante :

    Rapports directs pour *Rapports directs pour {ID objet_du_responsable}*. Voici un exemple e règle valable pour Collaborateurs :

                    Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863”

    où « 62e19b97-8b3d-4d4a-a106-4ce66896a863 » est l’ID objet du responsable. L’ID d’objet se trouve dans Azure AD, dans l’ **onglet Profil** de la page Utilisateur de l’utilisateur qui est responsable.
5. Une fois cette règle enregistrée, tous les utilisateurs qui satisfont à la règle seront joints en tant que membres du groupe. Le remplissage initial du groupe peut prendre quelques minutes.

## <a name="using-attributes-to-create-rules-for-device-objects"></a>Utilisation d’attributs pour créer des règles pour les objets d’appareil
Vous pouvez également créer une règle qui sélectionne des objets d’appareil pour l’appartenance à un groupe. Les attributs d’appareil suivants peuvent être utilisés :

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| displayName |Toute valeur de chaîne. |(device.displayName -eq "Rob Iphone”) |
| deviceOSType |Toute valeur de chaîne. |(device.deviceOSType -eq "IOS") |
| deviceOSVersion |Toute valeur de chaîne. |(device.OSVersion -eq "9.1") |
| isDirSynced |true false null |(device.isDirSynced -eq true) |
| isManaged |true false null |(device.isManaged -eq false) |
| isCompliant |true false null |(device.isCompliant -eq true) |
| deviceCategory |Toute valeur de chaîne. |(device.deviceCategory -eq "") |
| deviceManufacturer |Toute valeur de chaîne. |(device.deviceManufacturer -eq "Microsoft") |
| deviceModel |Toute valeur de chaîne. |(device.deviceModel -eq "IPhone 7+") |
| deviceOwnership |Toute valeur de chaîne. |(device.deviceOwnership -eq "") |
| domainName |Toute valeur de chaîne. |(device.domainName -eq "contoso.com") |
| enrollmentProfileName |Toute valeur de chaîne. |(device.enrollmentProfileName -eq "") |
| isRooted |true false null |(device.isRooted -eq true) |
| managementType |Toute valeur de chaîne. |(device.managementType -eq "") |
| organizationalUnit |Toute valeur de chaîne. |(device.organizationalUnit -eq "") |
| deviceId |un deviceId valide |(device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d" |

> [!NOTE]
> Ces règles d’appareil ne peuvent pas être créées à l’aide de la liste déroulante « règle simple » dans le portail Azure Classic.
>
>

## <a name="next-steps"></a>Étapes suivantes
Ces articles fournissent des informations supplémentaires sur Azure Active Directory.

* [Résolution des problèmes liés à l’appartenance dynamique à des groupes](active-directory-accessmanagement-troubleshooting.md)
* [Gestion de l’accès aux ressources avec les groupes Azure Active Directory](active-directory-manage-groups.md)
* [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](active-directory-accessmanagement-groups-settings-cmdlets.md)
* [Index d’articles pour la gestion des applications dans Azure Active Directory](active-directory-apps-index.md)
* [Intégration de vos identités locales avec Azure Active Directory](active-directory-aadconnect.md)



<!--HONumber=Feb17_HO2-->


