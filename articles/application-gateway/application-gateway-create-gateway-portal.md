---
title: "Créer une passerelle Application Gateway à l’aide du portail | Microsoft Docs"
description: "Découvrez comment créer une passerelle Application Gateway à l’aide du portail"
services: application-gateway
documentationcenter: na
author: georgewallace
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 54dffe95-d802-4f86-9e2e-293f49bd1e06
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/12/2016
ms.author: gwallace
translationtype: Human Translation
ms.sourcegitcommit: e20f7349f30c309059c2867d7473fa6fdefa9b61
ms.openlocfilehash: f7036e8e629e78c5346688556a5aa5794bde3955


---
# <a name="create-an-application-gateway-by-using-the-portal"></a>Créer une passerelle Application Gateway à l’aide du portail

> [!div class="op_single_selector"]
> * [portail Azure](application-gateway-create-gateway-portal.md)
> * [Commandes PowerShell pour Azure Resource Manager](application-gateway-create-gateway-arm.md)
> * [Azure Classic PowerShell](application-gateway-create-gateway.md)
> * [Modèle Azure Resource Manager](application-gateway-create-gateway-arm-template.md)
> * [Interface de ligne de commande Azure](application-gateway-create-gateway-cli.md)

La passerelle Azure Application Gateway est un équilibreur de charge de couche 7. Elle assure l’exécution des requêtes HTTP de basculement et de routage des performances entre serveurs locaux ou dans le cloud. Application Gateway offre de nombreuses fonctionnalités de contrôleur de livraison d’applications (ADC) : équilibrage de charge HTTP, affinité de session basée sur les cookies, déchargement SSL (Secure Sockets Layer), sondes d’intégrité personnalisées, prise en charge de plusieurs sites, etc. Pour obtenir une liste complète des fonctionnalités prises en charge, consultez [Vue d’ensemble d’Application Gateway](application-gateway-introduction.md)

## <a name="scenario"></a>Scénario

Dans ce scénario, vous allez apprendre à créer une passerelle Application Gateway à l’aide du portail Azure.

Ce scénario va :

* créer une passerelle Application Gateway moyenne avec deux instances ;
* créer un réseau virtuel nommé AdatumAppGatewayVNET avec un bloc CIDR réservé de 10.0.0.0/16 ;
* créer un sous-réseau appelé Appgatewaysubnet qui utilise 10.0.0.0/28 comme bloc CIDR ;
* configurer un certificat pour le déchargement SSL.

![Exemple de scénario][scenario]

> [!IMPORTANT]
> La configuration supplémentaire de la passerelle Application Gateway, y compris les sondes d’intégrité personnalisées, les adresses de pool principal et les règles supplémentaires sont configurées après avoir configuré la passerelle Application Gateway et non lors du déploiement initial.
> 
> 

## <a name="before-you-begin"></a>Avant de commencer

La passerelle Application Gateway Azure requiert son propre sous-réseau. Lorsque vous créez un réseau virtuel, assurez-vous que vous laissez suffisamment d’espace d’adresse pour disposer de plusieurs sous-réseaux. Une fois que vous avez déployé une passerelle Application Gateway sur un sous-réseau, seules les passerelles Application Gateway supplémentaires peuvent être ajoutées au sous-réseau.

## <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

### <a name="step-1"></a>Étape 1

Accédez au portail Azure, cliquez sur **Nouveau** > **Mise en réseau** > **Application Gateway**

![Création d’une passerelle Application Gateway][1]

### <a name="step-2"></a>Étape 2 :

Remplissez ensuite les informations de base relatives à la passerelle Application Gateway. Lorsque vous avez terminé, cliquez sur **OK**

Les informations nécessaires pour les paramètres de base sont les suivantes :

* **Nom** -Nom de la passerelle Application Gateway.
* **Niveau** : il s’agit du niveau de la passerelle d’application. Deux niveaux sont disponibles : **WAF** et **Standard**. WAF active la fonctionnalité de pare-feu d’applications web.
* **Taille de la référence (SKU)** : taille de la passerelle Application Gateway. Les options disponibles sont **Petit**, **Moyen** et **Grand**. L’option Petit n’est pas disponible lorsque le niveau WAF est sélectionné.
* **Nombre d’instances** - Nombre d’instances. Cette valeur doit être un nombre compris entre 2 et 10.
* **Groupe de ressources** - Groupe de ressources destiné à contenir la passerelle Application Gateway. Il peut s’agir d’un groupe de ressources existant ou nouveau.
* **Emplacement** - Région de la passerelle Application Gateway. Il s’agit du même emplacement que celui du groupe de ressources. Cette notion est importante, car le réseau virtuel et l’adresse IP publique doivent se trouver au même emplacement que la passerelle.

![panneau montrant les paramètres de base][2]

> [!NOTE]
> Vous pouvez choisir un nombre d’instances de 1 à des fins de test. Il est important de savoir que n’importe quel nombre d’instances inférieur à&2; n’est pas couvert par le contrat SLA et n’est donc pas recommandé. Les petites passerelles doivent être utilisées pour les tests de développement et non à des fins de production.
> 
> 

### <a name="step-3"></a>Étape 3

Une fois les paramètres de base définis, l’étape suivante consiste à définir le réseau virtuel à utiliser. Le réseau virtuel héberge l’application pour laquelle la passerelle Application Gateway effectue l’équilibrage de charge.

Cliquez sur **Choisir un réseau virtuel** pour configurer le réseau virtuel.

![panneau montrant les paramètres de la passerelle Application Gateway][3]

### <a name="step-4"></a>Étape 4

Dans le panneau **Choisir un réseau virtuel** , cliquez sur **Créer nouveau**

Bien que cela ne soit pas expliqué dans ce scénario, un réseau virtuel peut être sélectionné à ce stade.  Si un réseau virtuel existant est utilisé, il est important de savoir que le réseau virtuel a besoin d’un sous-réseau vide ou d’un sous-réseau contenant uniquement des ressources Application Gateway pour pouvoir être utilisé.

![panneau Choisir un réseau virtuel][4]

### <a name="step-5"></a>Étape 5

Remplissez les informations de réseau dans le panneau **Créer un réseau virtuel** comme décrit dans la description du précédent [scénario](#scenario) .

![panneau Créer un réseau virtuel avec informations saisies][5]

### <a name="step-6"></a>Étape 6

Une fois le réseau virtuel créé, l’étape suivante consiste à définir l’adresse IP frontale de la passerelle Application Gateway. À ce stade, le choix se résume à une adresse IP privée ou publique pour le serveur frontal. Le choix dépend si l’application est accessible sur Internet ou en interne uniquement. Ce scénario suppose l’utilisation d’une adresse IP publique. Pour choisir une adresse IP privée, il est possible de cliquer sur le bouton **Privé** . Une adresse IP affectée automatiquement est choisie, ou vous pouvez cocher la case **Choisir une adresse IP privée spécifique** pour en saisir une manuellement.

### <a name="step-7"></a>Étape 7

Cliquez sur **Choisir une adresse IP publique**. Si une adresse IP publique existante est disponible, elle peut être choisie à ce stade Dans ce scénario, vous créez une nouvelle adresse IP publique. Cliquez sur **Create new**

![Panneau Choisir une adresse IP publique][6]

### <a name="step-8"></a>Étape 8

Attribuez ensuite un nom convivial à l’adresse IP publique et cliquez sur **OK**

![Panneau Créer une adresse IP publique][7]

### <a name="step-9"></a>Étape 9

Le dernier paramètre à configurer lors de la création d’une passerelle Application Gateway est la configuration de l’écouteur.  Si **http** est utilisé, il n’y a rien d’autre à configurer. Vous pouvez alors cliquer sur **OK**. Une configuration supplémentaire est requise pour utiliser **https**.

Un certificat est requis pour utiliser **https**. La clé privée du certificat est nécessaire. Un export .pfx du certificat doit donc être fourni, ainsi que le mot de passe.

### <a name="step-10"></a>Étape 10

Cliquez sur **HTTPS**, puis sur l’icône de **dossier** situé à côté de la zone de texte **Charger un certificat PFX**.
Accédez au fichier de certificat .pfx sur votre système de fichiers. Une fois sélectionné, donnez un nom convivial au certificat et tapez le mot de passe du fichier .pfx.

Une fois terminé, cliquez sur **OK** pour passer en revue les paramètres de la passerelle Application Gateway.

![section Configuration de l’écouteur du panneau Paramètres][9]

### <a name="step-11"></a>Étape 11

Consultez la page Résumé et cliquez sur **OK**.  La passerelle Application Gateway est maintenant mise en file d’attente et créée.

### <a name="step-12"></a>Étape 12

Une fois la passerelle Application Gateway créée, accédez à celle-ci dans le portail pour poursuivre sa configuration.

![Vue des ressources de la passerelle Application Gateway][10]

Ces étapes permettent de créer une passerelle Application Gateway de base avec les paramètres par défaut pour l’écouteur, le pool principal, les paramètres http principaux et les règles. Vous pouvez modifier ces paramètres en fonction de votre déploiement une fois l’approvisionnement réussi.

## <a name="add-servers-to-backend-pools"></a>Ajouter des serveurs aux pools principaux

Une fois la passerelle Application Gateway créée, il reste à y ajouter les systèmes qui hébergent l’application dont la charge doit être équilibrée. Les adresses IP ou les valeurs FQDN de ces serveurs sont ajoutées aux pools d’adresses principaux.

### <a name="step-1"></a>Étape 1 :

Cliquez sur la passerelle Application Gateway que vous avez créée, cliquez sur **Pools principaux**, puis sélectionnez le pool principal actuel.

![Pools principaux Application Gateway][11]

### <a name="step-2"></a>Étape 2 :

Ajoutez les adresses IP ou les valeurs FQDN dans les zones de texte et cliquez sur **Enregistrer**.

![Ajouter des valeurs aux pools principaux Application Gateway][12]

Cette opération enregistre les valeurs dans le pool principal. Une fois la passerelle Application Gateway mise à jour, le trafic qui y accède est acheminé vers les adresses principales ajoutées à cette étape.

## <a name="next-steps"></a>Étapes suivantes

Ce scénario crée une passerelle d’application par défaut. Les étapes suivantes consistent à configurer la passerelle Application Gateway en modifiant les paramètres et en ajustant les règles de la passerelle. Vous trouverez ces étapes dans les articles suivants :

Apprenez à créer des sondes d’intégrité personnalisées en vous rendant sur [Créer une sonde d’intégrité personnalisée](application-gateway-create-probe-portal.md)

Découvrez comment configurer le déchargement SSL et éviter à vos serveurs web le déchiffrement SSL coûteux en vous rendant sur [Configurer le déchargement SSL](application-gateway-ssl-portal.md)

Découvrez comment protéger vos applications grâce au [Pare-feu d’applications Web](application-gateway-webapplicationfirewall-overview.md), une fonctionnalité de passerelle Application Gateway.

<!--Image references-->
[1]: ./media/application-gateway-create-gateway-portal/figure1.png
[2]: ./media/application-gateway-create-gateway-portal/figure2.png
[3]: ./media/application-gateway-create-gateway-portal/figure3.png
[4]: ./media/application-gateway-create-gateway-portal/figure4.png
[5]: ./media/application-gateway-create-gateway-portal/figure5.png
[6]: ./media/application-gateway-create-gateway-portal/figure6.png
[7]: ./media/application-gateway-create-gateway-portal/figure7.png
[8]: ./media/application-gateway-create-gateway-portal/figure8.png
[9]: ./media/application-gateway-create-gateway-portal/figure9.png
[10]: ./media/application-gateway-create-gateway-portal/figure10.png
[11]: ./media/application-gateway-create-gateway-portal/figure11.png
[12]: ./media/application-gateway-create-gateway-portal/figure12.png
[scenario]: ./media/application-gateway-create-gateway-portal/scenario.png



<!--HONumber=Dec16_HO2-->


