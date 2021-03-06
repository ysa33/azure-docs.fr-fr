---
title: "Gérer l’expiration du contenu web dans Azure CDN | Microsoft Docs"
description: "Découvrez comment gérer l’expiration des contenus d’Azure Web Apps/Services cloud, d’ASP.NET ou d’IIS dans le réseau de distribution de contenu (CDN) Azure."
services: cdn
documentationcenter: .NET
author: zhangmanling
manager: erikre
editor: 
ms.assetid: bef53fcc-bb13-4002-9324-9edee9da8288
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
translationtype: Human Translation
ms.sourcegitcommit: 57d00f2192fed7a2e89ac94e110ebb7e84c83b72
ms.openlocfilehash: 02d0270c5763eb9dd2190bc24b793022ea536746


---
# <a name="manage-expiration-of-azure-web-appscloud-services-aspnet-or-iis-content-in-azure-cdn"></a>Gérer l’expiration des contenus d’Azure Web Apps/Services cloud, d’ASP.NET ou d’IIS dans le réseau de distribution de contenu (CDN) Azure
> [!div class="op_single_selector"]
> * [Azure Web Apps/Cloud Services, ASP.NET ou IIS](cdn-manage-expiration-of-cloud-service-content.md)
> * [Service Azure Storage Blob](cdn-manage-expiration-of-blob-content.md)
> 
> 

Les fichiers d’un serveur web d’origine publiquement accessible peuvent être mis en cache dans le réseau de distribution de contenu (CDN) Azure jusqu’à l’expiration de leur durée de vie (TTL).  La durée de vie est déterminée par [l’en-tête *Cache-Control*](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) dans la réponse HTTP du serveur d’origine.  Cet article décrit comment définir des en-têtes `Cache-Control` pour Azure Web Apps, Services cloud Azure, les applications ASP.NET et les sites Internet Information Services, qui sont tous configurés de façon similaire.

> [!TIP]
> Vous pouvez choisir de ne pas définir de durée de vie sur un fichier.  Dans ce cas, Azure CDN applique automatiquement une durée de vie de sept jours par défaut.
> 
> Pour plus d’informations sur la façon dont le CDN Azure accélère l’accès aux fichiers et à d’autres ressources, consultez [Vue d’ensemble du réseau de distribution de contenu (CDN) Azure](cdn-overview.md).
> 
> 

## <a name="setting-cache-control-headers-in-configuration"></a>Définition d’en-têtes Cache-Control dans la configuration
Pour le contenu statique, tel que les images et les feuilles de style, vous pouvez contrôler la fréquence de mise à jour en modifiant les fichiers **applicationHost.config** ou **web.config** de votre application web.  L’élément **system.webServer\staticContent\clientCache** du fichier de configuration définit l’en-tête `Cache-Control` pour votre contenu. Pour **web.config**, les paramètres de configuration affectent tous les éléments du dossier et de tous les sous-dossiers, sauf s’ils sont remplacés au niveau sous-dossier.  Par exemple, vous pouvez définir une durée de vie par défaut à la racine pour que tout le contenu statique soit mis en cache pendant 3 jours, tout en définissant une durée de mise en cache de 6 heures pour un sous-dossier dont le contenu varie davantage.  Pour **applicationHost.config**, toutes les applications du site sont affectées, mais peuvent être remplacées dans les fichiers **web.config** des applications.

Le code XML suivant montre comment configurer **clientCache** pour spécifier un âge maximal de 3 jours :  

```xml
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

Le fait de spécifier **UseMaxAge** ajoute un en-tête `Cache-Control: max-age=<nnn>` à la réponse en fonction de la valeur spécifiée dans l’attribut **CacheControlMaxAge**. Le format de la période pour l’attribut **cacheControlMaxAge** est <days>.<hours>:<min>:<sec>. Pour plus d’informations sur le nœud **clientCache**, consultez [Cache client <clientCache>](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache).  

## <a name="setting-cache-control-headers-in-code"></a>Définition d’en-têtes Cache-Control dans le code
Pour les applications ASP.NET, vous pouvez définir le comportement de mise en cache dans le CDN par programme en définissant la propriété **HttpResponse.Cache** . Pour plus d’informations sur la propriété **HttpResponse.Cache**, consultez les pages [HttpResponse.Cache, propriété](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx) et [HttpCachePolicy, classe](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx).  

Si vous souhaitez mettre en cache du contenu d’application par programme dans ASP.NET, assurez-vous que le contenu est marqué comme pouvant être mis en cache en définissant HttpCacheability sur *Public*. En outre, assurez-vous qu'un validateur de cache est défini. Le validateur de cache peut être un horodatage de dernière modification défini en appelant SetLastModified, ou une valeur etag définie en appelant SetETag. Si vous le souhaitez, vous pouvez également spécifier un délai d'expiration du cache en appelant SetExpires, ou vous pouvez vous en remettre à la méthode heuristique de mise en cache par défaut décrite précédemment dans ce document.  

Par exemple, pour mettre en cache du contenu pendant une heure, ajoutez les éléments suivants :  

```csharp
// Set the caching parameters.
Response.Cache.SetExpires(DateTime.Now.AddHours(1));
Response.Cache.SetCacheability(HttpCacheability.Public);
Response.Cache.SetLastModified(DateTime.Now);
```

## <a name="next-steps"></a>Étapes suivantes
* [Découvrir les détails de l’élément **clientCache**](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)
* [Consulter la documentation de la propriété **HttpResponse.Cache**](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx) 
* [Lire la documentation concernant la **classe HttpCachePolicy**](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx).  




<!--HONumber=Jan17_HO4-->


