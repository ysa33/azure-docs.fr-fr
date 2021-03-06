---
title: "Débogage de Services cloud Azure | Microsoft Docs"
description: "Débogage d’Azure Cloud Services"
services: visual-studio-online
documentationcenter: n/a
author: TomArcher
manager: douge
editor: 
ms.assetid: 80755da7-8350-4f5c-97ce-2962beabb36d
ms.service: visual-studio-online
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 11/11/2016
ms.author: tarcher
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: f10ede0a6c30c5a8ae593a5dba0f3db77134d50b


---
# <a name="debugging-cloud-services"></a>Débogage de services cloud
Vous pouvez utiliser différentes approches pour déboguer une application Azure à l’aide des outils Azure pour Microsoft Visual Studio et du Kit de développement logiciel (SDK) Azure :

* Vous pouvez déboguer une application Azure à partir de Visual Studio quand vous la développez, de la même façon qu’une application Visual C# ou Visual Basic. Pour plus d’informations, consultez [Déboguer votre service cloud sur votre ordinateur local](vs-azure-tools-debug-cloud-services-virtual-machines.md#debug-your-cloud-service-on-your-local-computer).
* Vous pouvez utiliser les diagnostics Azure pour consigner des informations détaillées à partir du code en cours d’exécution dans des rôles, que ces derniers s’exécutent dans l’environnement de développement ou dans Azure. Pour plus d’informations, consultez [Collecte des données de journalisation avec les diagnostics Azure](http://go.microsoft.com/fwlink/p/?LinkId=400450).
* Si vous utilisez Visual Studio Enterprise pour écrire des rôles ciblés sur .NET Framework 4 ou .NET Framework 4.5, vous pouvez activer IntelliTrace au moment où vous déployez un service cloud Azure depuis Visual Studio. IntelliTrace fournit un journal que vous pouvez utiliser avec Visual Studio pour déboguer votre application comme si elle s’exécutait dans Azure. Pour plus d’informations, consultez [Débogage d’un service cloud publié avec IntelliTrace et Visual Studio](http://go.microsoft.com/fwlink/p/?LinkId=623016).
* Vous pouvez activer le débogage distant sur vos services cloud au moment où vous les déployez depuis Visual Studio. Si vous choisissez d’activer le débogage distant pour un déploiement, les services de débogage distant sont installés sur les machines virtuelles qui exécutent chaque instance de rôle. Ces services, tels que msvsmon.exe, n’affectent pas les performances et n’entraînent pas de coûts supplémentaires. Pour plus d’informations, consultez [Déboguer un service cloud dans Azure](vs-azure-tools-debug-cloud-services-virtual-machines.md#debug-a-cloud-service-in-azure).




<!--HONumber=Nov16_HO3-->


