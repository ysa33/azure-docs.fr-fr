---
title: FAQ Azure App Service Web Apps sous Linux | Microsoft Docs
description: FAQ Azure App Service Web Apps sous Linux.
keywords: azure app service, application web, faq, linux, oss
services: app-service
documentationCenter: 
authors: aelnably
manager: erikre
editor: 
ms.assetid: 
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2017
ms.author: aelnably
translationtype: Human Translation
ms.sourcegitcommit: 831ef097027721146531e8d699fe3f67417a57ea
ms.openlocfilehash: b88aa3d0ae89aec81c2b9144fb5de3210a0b8d1e
ms.lasthandoff: 02/18/2017


---

# <a name="azure-app-service-web-apps-on-linux-faq"></a>FAQ Azure App Service Web Apps sous Linux #

Avec la mise en production d’Azure App Service sous Linux (actuellement en version préliminaire), nous travaillons à l’ajout de fonctionnalités et l’amélioration de notre plateforme. Voici quelques-unes des questions que nos clients nous ont fréquemment posées au cours de ces derniers mois.
Si vous avez une question, veuillez commenter l’article ; nous vous répondrons dès que possible.

## <a name="built-in-images"></a>Images prédéfinies ##

**Q :** Je veux répliquer les conteneurs Docker prédéfinis fournis par la plateforme. Où se trouvent ces fichiers ?

**R :** Tous les fichiers Docker se trouvent à cette adresse : https://github.com/azure-app-service. Tous les conteneurs Docker se trouvent ici : https://hub.docker.com/u/appsvc/.

## <a name="management"></a>Gestion ##

**Q :** J’appuie sur le bouton Redémarrer dans le portail, mais mon application web ne redémarre pas, pourquoi ?

**R :** Nous travaillons à l’activation prochaine du bouton de réinitialisation ; pour l’instant, vous avez deux possibilités.
1. Ajoutez ou modifiez un paramètre d’application factice : cela forcera votre application web à redémarrer. 
2. Arrêtez puis démarrez votre application web.

**Q :** Puis-je me connecter à la machine virtuelle avec SSH ?

**R :** Non. Nous fournirons prochainement un moyen de vous connecter au conteneur de l’application avec SSH.

## <a name="continuous-integration--deployment"></a>Intégration continue/déploiement continu ##

**Q :** Mon application web utilise toujours une ancienne image de conteneur Docker après la mise à jour de l’image sur DockerHub. Prenez-vous en charge l’intégration continue / le déploiement continu de conteneurs personnalisés ?

**R :** Vous pouvez actualiser le conteneur soit en arrêtant puis en démarrant votre application web, soit en modifiant/ajoutant un paramètre d’application factice pour en forcer l’actualisation. Nous proposerons prochainement une fonctionnalité d’intégration continue/déploiement continu pour les conteneurs personnalisés.

## <a name="language-support"></a>Prise en charge des langages ##

**Q :** Prenez-vous en charge les applications .NET Core non compilées ?

**R :** Non. Vous devez déployer l’application .NET Core compilée avec toutes les dépendances. Une expérience complète de déploiement et de génération sera prochainement proposée.

## <a name="built-in-images"></a>Images prédéfinies ##

**Q :** Quelles sont les valeurs attendues de la section Fichier de démarrage lorsque je configure la pile d’exécution ?

**R :** Pour Node.Js, vous pouvez spécifier le fichier de configuration PM2 ou votre fichier de script. Pour .Net Core, vous devez spécifier le nom de votre dll compilée. Pour Ruby, vous pouvez spécifier un script Ruby avec lequel initialiser votre application.

## <a name="custom-containers"></a>Conteneurs personnalisés ##

**Q :** J’utilise mon propre conteneur personnalisé. Mon application se trouve dans le répertoire \home\, mais je ne trouve pas mes fichiers lorsque je parcours le contenu avec le site SCM ou un client FTP. Où sont mes fichiers ?

**R :** Nous montons un partage SMB vers le répertoire \home\, ce qui remplace tout son contenu.

**Q :** Je veux exposer plusieurs ports sur l’image de mon conteneur personnalisé. Est-ce possible ?

**R :** Ce n’est pas pris en charge à l’heure actuelle.

**Q :** Puis-je apporter mon propre système de stockage ?

**R :** Ce n’est pas pris en charge à l’heure actuelle.

**Q :** Je n’arrive pas à parcourir le système de fichiers de mon conteneur personnalisé à partir du site SCM. Pourquoi ?

**R :** Le site SCM s’exécute dans un conteneur distinct ; vous ne pouvez pas vérifier le système de fichiers ou les processus en cours d’exécution du conteneur d’application.

**Q :** Mon conteneur personnalisé écoute un autre port que le port 80. Comment puis-je configurer mon application pour acheminer les demandes vers ce port ?

**R :** Vous pouvez spécifier un paramètre d’application appelé **PORT** et lui attribuer la valeur du numéro de port attendu.

## <a name="pricing-and-sla"></a>Tarifs et contrat SLA ##

**Q :** Quelle est la tarification en préversion publique ?

**R :** La moitié du nombre d’heures d’exécution de votre application vous sera facturée, à la tarification Azure App Service normale, ce qui signifie une remise de 50 % sur la tarification normale d’Azure App Service.

## <a name="other"></a>Autres ##

**Q :** Quels sont les caractères pris en charge dans les noms de paramètres d’application ?

**R :** Vous ne pouvez utiliser que les caractères A-Z, a-z, 0-9 et le trait de soulignement pour les paramètres d’application.

**Q :** Où puis-je demander de nouvelles fonctionnalités ?

**R :** Vous pouvez soumettre votre idée à cette adresse : https://aka.ms/webapps-uservoice. Ajoutez [Linux] au titre de votre idée.

## <a name="next-steps"></a>Étapes suivantes
* [Qu’est-ce qu’App Service sur Linux ?](app-service-linux-intro.md)
* [Création d’applications Web dans App Service sur Linux](app-service-linux-how-to-create-a-web-app.md)

