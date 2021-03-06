---
title: "Bibliothèques d’authentification Azure Active Directory v2.0 | Microsoft Docs"
description: "Bibliothèques clientes et bibliothèques de middleware de serveur compatibles, et liens vers la bibliothèque, la source et des exemples connexes pour le point de terminaison Azure Active Directory v2.0."
services: active-directory
documentationcenter: 
author: skwan
manager: mbaldwin
editor: 
ms.assetid: 19cec615-e51f-4141-9f8c-aaf38ff9f746
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/29/2016
ms.author: skwan;bryanla
translationtype: Human Translation
ms.sourcegitcommit: c579135f798ea0c2a5461fdd7c88244d2d6d78c6
ms.openlocfilehash: f54dbdaa9e1eab9c7c050c2d8163322ee56f003e


---
# <a name="azure-active-directory-v20-authentication-libraries"></a>Bibliothèques d’authentification Azure Active Directory v2.0
Le point de terminaison Azure Active Directory (Azure AD) v2.0 prend en charge les protocoles OAuth 2.0 et OpenID Connect 1.0 standard. Vous pouvez utiliser différentes bibliothèques de Microsoft et d’autres organisations avec le point de terminaison v2.0.

Lorsque vous créez une application qui utilise le point de terminaison v2.0, nous vous recommandons d’utiliser des bibliothèques écrites par des experts du domaine du protocole qui respectent une méthodologie Security Development Lifecycle (SDL) comme [celle que suit Microsoft][Microsoft-SDL]. Si vous décidez de coder à la main la prise en charge des protocoles, nous vous recommandons de suivre la méthodologie SDL et de bien prendre en compte les considérations relatives à la sécurité que vous trouverez dans les spécifications des normes pour chaque protocole.

## <a name="types-of-libraries"></a>Types de bibliothèques
Azure AD v2.0 prend en charge deux types de bibliothèques :

* **Bibliothèques clientes**. Les serveurs et clients natifs utilisent les bibliothèques clientes afin d’obtenir des jetons d’accès pour appeler une ressource, comme Microsoft Graph.
* **Bibliothèques de middleware de serveur**. Les applications web utilisent des bibliothèques de middleware de serveur pour la connexion des utilisateurs. Les API web utilisent des bibliothèques de middleware de serveur pour valider les jetons qui sont envoyés par des clients natifs ou par d’autres serveurs.

## <a name="library-support"></a>Prise en charge de la bibliothèque
Étant donné que vous pouvez choisir n’importe quelle bibliothèque conforme aux normes lors de l’utilisation du point de terminaison v2.0, il est important de savoir où vous adresser pour la prise en charge. Pour tout problème ou demande de fonctionnalité au niveau du code de la bibliothèque, adressez-vous au propriétaire de la bibliothèque. Pour tout problème ou demande de fonctionnalité au niveau de l’implémentation du protocole côté service, adressez-vous à Microsoft.

Les bibliothèques sont classées en deux catégories de prise en charge :

* **Pris en charge par Microsoft**. Microsoft fournit des correctifs de ces bibliothèques, auxquelles il a appliqué la méthodologie SDL standard.
* **Compatible**. Microsoft a testé ces bibliothèques dans des scénarios de base et a confirmé que celles-ci fonctionnent avec le point de terminaison v2.0. Microsoft ne fournit pas de correctifs pour ces bibliothèques et n’a pas vérifié ces bibliothèques. Les problèmes et les demandes de fonctionnalités doivent être orientés vers le projet open source de la bibliothèque.

Pour obtenir la liste des bibliothèques qui fonctionnent avec le point de terminaison v2.0, consultez les sections suivantes de cet article.

## <a name="microsoft-supported-client-libraries"></a>Bibliothèques clientes prises en charge par Microsoft
| Plateforme | Nom de la bibliothèque | Télécharger | Code source | Exemple |
|:---:|:---:|:---:|:---:|:---:|
| .NET, Windows Store, Xamarin |Microsoft Authentication Library (MSAL) pour .NET |[Microsoft.Identity.Client (NuGet)][ClientLib-NET-Lib] |[MSAL pour .NET (GitHub)][ClientLib-NET-Repo] |[Exemple de client natif de bureau Windows][ClientLib-NET-Sample] |
| Node.js |Plug-in Microsoft Azure Active Directory Passport.js |[Passport-Azure-AD (npm)][ClientLib-Node-Lib] |[Passport-Azure-AD (GitHub)][ClientLib-Node-Repo] |Bientôt disponible |

<!--- COMMENTING OUT UNTIL THEY ARE READY
| iOS, Mac | Microsoft Authentication Library (MSAL) for ObjC | In development | In development | In development |
| Android | Microsoft Authentication Library (MSAL) for Android | In development | In development | In development |
| JavaScript | Microsoft Authentication Library (MSAL) for JavaScript | In development | In development | In development |
 -->

## <a name="microsoft-supported-server-middleware-libraries"></a>Bibliothèques de middleware de serveur prises en charge par Microsoft
| Plateforme | Nom de la bibliothèque | Télécharger | Code source | Exemple |
|:---:|:---:|:---:|:---:|:---:|
| .NET 4.x |OWIN OpenID Connect Middleware pour ASP.NET |[Microsoft.Owin.Security.OpenIdConnect (NuGet)][ServerLib-Net4-Owin-Oidc-Lib] |[Projet Katana (CodePlex)][ServerLib-Net4-Owin-Oidc-Repo] |[Exemple d’application web][ServerLib-Net4-Owin-Oidc-Sample] |
| .NET 4.x |OWIN OAuth Bearer Middleware pour ASP.NET |[Microsoft.Owin.Security.OAuth (NuGet)][ServerLib-Net4-Owin-Oauth-Lib] |[Projet Katana (CodePlex)][ServerLib-Net4-Owin-Oauth-Repo] |[Exemple d’API Web][ServerLib-Net4-Owin-Oauth-Sample] |
| .NET Core |OWIN OpenID Connect Middleware pour .NET Core |[Microsoft.AspNetCore.Authentication.OpenIdConnect (NuGet)][ServerLib-NetCore-Owin-Oidc-Lib] |[Sécurité ASP.NET (GitHub)][ServerLib-NetCore-Owin-Oidc-Repo] |[Exemple d’application web][ServerLib-NetCore-Owin-Oidc-Sample] |
| .NET Core |OWIN OAuth Bearer Middleware pour .NET Core |[Microsoft.AspNetCore.Authentication.OAuth (NuGet)][ServerLib-NetCore-Owin-Oauth-Lib] |[Sécurité ASP.NET (GitHub)][ServerLib-NetCore-Owin-Oauth-Repo] |Bientôt disponible |
| Node.js |Plug-in Microsoft Azure Active Directory Passport.js |[Passport-Azure-AD (npm)][ServerLib-Node-Lib] |[Passport-Azure-AD (GitHub)][ServerLib-Node-Repo] |[Exemple d’application web][ServerLib-Node-Sample] |

<!--- COMMENTING UNTIL SAMPLE IS AVAILABLE
| .NET 4.x, .NET Core | JSON Web Token Handler for .NET | [System.IdentityModel.Tokens.Jwt (NuGet)][ServerLib-Net-Jwt-Lib] | [Azure AD identity model extensions for .NET (GitHub)][ServerLib-Net-Jwt-Repo] | Coming soon |
--->
## <a name="compatible-client-libraries"></a>Bibliothèques clientes compatibles
| Plateforme | Nom de la bibliothèque | Version testée | Code source | Exemple |
|:---:|:---:|:---:|:---:|:---:|
| Android |[OIDCAndroidLib](https://github.com/kalemontes/OIDCAndroidLib/wiki) |0.2.1 |[OIDCAndroidLib](https://github.com/kalemontes/OIDCAndroidLib) |[Exemple d’application native](active-directory-v2-devquickstarts-android.md) |
| iOS |[NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) |1.2.8 |[NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) |[Exemple d’application native](active-directory-v2-devquickstarts-ios.md) |
| Java | [Scribe Java scribejava](https://github.com/scribejava/scribejava) | [Version 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [Scribe Java](https://github.com/scribejava/scribejava/archive/scribejava-3.2.0.zip) | Bientôt disponible |
| JavaScript |[Hello.js](https://adodson.com/hello.js/) |1.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |Bientôt disponible |
| PHP | [The PHP League oauth2-client](https://github.com/thephpleague/oauth2-client) | [Version 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/archive/1.4.2.zip) | Bientôt disponible |
| Python-Flask |[Flask-OAuthlib](https://github.com/lepture/flask-oauthlib) |0.9.3 |[Flask-OAuthlib](https://github.com/lepture/flask-oauthlib) |Bientôt disponible |
| Ruby |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |omniauth:1.3.1</br>omniauth-oauth2:1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)</br>[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |Bientôt disponible |

<!--- REMOVING BRANDON'S FOR NOW
|  |  |  |  |  |
| Android | [OAuth2 Client](https://github.com/wuman/android-oauth-client) |   | [OAuth2 Client](https://github.com/wuman/android-oauth-client)  | Coming soon  |
| Java | [WSO2 Identity Server](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server) | [Version 5.2.0](http://wso2.com/products/identity-server/) | [Source](https://docs.wso2.com/display/IS500/Building+from+Source) | [Samples index](https://docs.wso2.com/display/IS500/Samples)  |
| Java | [Java Gluu Server](https://gluu.org/docs/) |   | [oxAuth](https://github.com/GluuFederation/oxAuth)  | Coming soon |
| Node.js | [NPM passport-openidconnect](https://www.npmjs.com/package/passport-openidconnect) | 0.0.1  | [Passport-OpenID Connect](https://github.com/jaredhanson/passport-openidconnect) | Coming soon  |
| PHP | [OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP) |   | [OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP)  | Coming soon  |
-->

## <a name="compatible-server-middleware-libraries"></a>Bibliothèques de middleware de serveur compatibles
Bientôt disponible

## <a name="related-content"></a>Contenu connexe
Pour plus d’informations sur le point de terminaison Azure AD v2.0, consultez [Vue d’ensemble du modèle d’application Azure AD v2.0][AAD-App-Model-V2-Overview].

Utilisez la fonctionnalité de commentaire Disqus disponible à la fin de cet article pour fournir des commentaires et nous aider à affiner et à mettre en forme notre contenu.

<!--Image references-->

<!--Reference style links -->
[AAD-App-Model-V2-Overview]: ../active-directory-appmodel-v2-overview.md
[ClientLib-NET-Lib]: http://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-NET-Sample]: active-directory-v2-devquickstarts-wpf.md
[ClientLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ClientLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad
[ClientLib-Node-Sample]:/
[ClientLib-Iosmac-Lib]:/
[ClientLib-Iosmac-Repo]:/
[ClientLib-Iosmac-Sample]:/
[ClientLib-Android-Lib]:/
[ClientLib-Android-Repo]:/
[ClientLib-Android-Sample]:/
[ClientLib-Js-Lib]:/
[ClientLib-Js-Repo]:/
[ClientLib-Js-Sample]:/

[Microsoft-SDL]: http://www.microsoft.com/sdl/default.aspx
[ServerLib-Net4-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/
[ServerLib-Net4-Owin-Oidc-Repo]: http://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oidc-Sample]: active-directory-v2-devquickstarts-dotnet-web.md
[ServerLib-Net4-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth/
[ServerLib-Net4-Owin-Oauth-Repo]: http://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oauth-Sample]: https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-devquickstarts-dotnet-api/
[ServerLib-Net-Jwt-Lib]: https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt
[ServerLib-Net-Jwt-Repo]: https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
[ServerLib-Net-Jwt-Sample]:/
[ServerLib-NetCore-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OpenIdConnect/
[ServerLib-NetCore-Owin-Oidc-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oidc-Sample]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore-v2
[ServerLib-NetCore-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OAuth/
[ServerLib-NetCore-Owin-Oauth-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oauth-Sample]:/
[ServerLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ServerLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad/
[ServerLib-Node-Sample]: https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-devquickstarts-node-web/



<!--HONumber=Jan17_HO3-->


