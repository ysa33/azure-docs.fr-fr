---
title: Forum aux questions sur les machines virtuelles Windows dans Azure | Microsoft Docs
description: "Fournit des réponses à certaines questions courantes sur les machines virtuelles Azure créées avec le modèle de déploiement Resource Manager."
services: virtual-machines-windows
documentationcenter: 
author: cynthn
manager: timlt
editor: 
tags: azure-resource-management
ms.assetid: 757da816-a050-4889-a010-6f75d7978eb7
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 01/17/2017
ms.author: cynthn
translationtype: Human Translation
ms.sourcegitcommit: cfc58b84ccd671b3a34a399bad11d15c9bc3b713
ms.openlocfilehash: f338a124537090894773bb6fce1052fc7f590a33


---
# <a name="frequently-asked-question-about-windows-virtual-machines"></a>Questions fréquentes sur les machines virtuelles Azure
Cet article traite certaines questions courantes concernant les machines virtuelles Windows créées dans Azure avec le modèle de déploiement Resource Manager. Pour la version Linux de cette rubrique, consultez [Forum aux questions sur les machines virtuelles Linux](virtual-machines-linux-faq.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="what-can-i-run-on-an-azure-vm"></a>Qu’est-il possible d’exécuter sur une machine virtuelle Azure ?
Tous les abonnés peuvent exécuter des logiciels serveur sur une machine virtuelle Azure. Pour plus d’informations sur la stratégie de prise en charge des logiciels serveur Microsoft en cours d’exécution dans Azure, consultez [Prise en charge des logiciels serveur Microsoft pour les machines virtuelles Windows Azure](https://support.microsoft.com/kb/2721672)

Certaines versions de Windows 7 et Windows 8.1 sont disponibles pour les abonnés MSDN Azure et les abonnés Développement et test MSDN avec paiement à l’utilisation (pour les tâches de test et de développement). Pour plus d’informations, notamment des instructions et des limitations, voir [Images de client Windows pour les abonnés MSDN](http://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/). 

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>Quelle quantité de stockage puis-je utiliser avec une machine virtuelle ?
Chaque disque de données peut avoir une capacité allant jusqu’à 1 To Le nombre de disques de données que vous pouvez utiliser dépend de la taille de la machine virtuelle. Pour en savoir plus, voir la rubrique [Tailles de machines virtuelles](virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Un compte de stockage Azure fournit le stockage pour le disque du système d’exploitation et tout disque de données. Chaque disque est un fichier .vhd stocké sous la forme d’un objet blob de pages. Pour plus d’informations sur la tarification, voir [Tarification – Stockage](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-can-i-access-my-virtual-machine"></a>Comment puis-je accéder à ma machine virtuelle ?
Établissez une connexion à distance à l’aide du protocole RDP (Remote Desktop Protocol) pour une machine virtuelle Windows. Pour plus d’informations, consultez [Connexion à une machine virtuelle Azure exécutant Windows](virtual-machines-windows-connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Deux connexions simultanées maximum sont prises en charge, sauf si le serveur est configuré en tant qu’hôte de session Services Bureau à distance.  

En cas de problème de connexion, consultez [Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure exécutant Windows](virtual-machines-windows-troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 

Si vous connaissez bien Hyper-V, vous pouvez rechercher un outil similaire à VMConnect. Azure n’offre pas d’outil similaire car l’accès console à une machine virtuelle n’est pas pris en charge.

## <a name="can-i-use-the-temporary-disk-the-d-drive-by-default-to-store-data"></a>Puis-je utiliser le disque temporaire (lecteur D: par défaut) pour stocker des données ?
N’utilisez pas le disque temporaire pour stocker des données. Il ne sert qu’au stockage temporaire. Vous risqueriez donc de perdre des données sans pouvoir les récupérer. Une perte de données peut se produire si la machine virtuelle est déplacée vers un autre hôte. après le redimensionnement d’une machine virtuelle, la mise à jour de l’hôte ou une panne matérielle sur l’hôte, par exemple.

Si vous avez une application qui doit utiliser le lecteur D:, vous pouvez réaffecter les lettres de lecteur afin que le disque temporaire utilise une autre lettre que D. Pour obtenir des instructions, consultez la page [Modification de la lettre de lecteur du disque temporaire Windows](virtual-machines-windows-classic-change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).


## <a name="how-can-i-change-the-drive-letter-of-the-temporary-disk"></a>Comment puis-je modifier la lettre de lecteur d’un disque temporaire ?
Vous pouvez changer la lettre de lecteur en déplaçant le fichier d’échange et en réaffectant les lettres de lecteur. Toutefois, vous devrez veiller à effectuer les étapes dans un ordre spécifique. Pour obtenir des instructions, consultez la page [Modification de la lettre de lecteur du disque temporaire Windows](virtual-machines-windows-classic-change-drive-letter.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="can-i-add-an-existing-vm-to-an-availability-set"></a>Puis-je ajouter une machine virtuelle à un groupe à haute disponibilité ?
Non. Si vous souhaitez que votre machine virtuelle fasse partie d’un groupe à haute disponibilité, vous devez la créer dans le groupe. Il n’existe actuellement aucun moyen d’ajouter une machine virtuelle dans un groupe à haute disponibilité, une fois celle-ci créée.
## <a name="can-i-upload-a-virtual-machine-to-azure"></a>Puis-je télécharger une machine virtuelle dans Azure ?
Oui. Pour obtenir des instructions, consultez [Charger une image de machine virtuelle Windows dans Microsoft Azure pour des déploiements Resource Manager ](virtual-machines-windows-upload-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="can-i-resize-the-os-disk"></a>Puis-je redimensionner le disque du système d’exploitation ?
Oui. Pour obtenir des instructions, consultez [Extension du lecteur de système d’exploitation d’une machine virtuelle dans un groupe de ressources Azure](virtual-machines-windows-expand-os-disk.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>Puis-je copier ou cloner une machine virtuelle Azure ?
Oui. Pour obtenir des instructions, consultez [Création d’une copie d’une machine virtuelle Windows dans le modèle de déploiement Resource Manager](virtual-machines-windows-vhd-copy.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>Pourquoi les régions Centre et Est du Canada ne sont-elles pas visibles dans Azure Resource Manager ?

Les deux régions Centre et Est du Canada ne sont pas enregistrées automatiquement lors de la création de machines virtuelles pour des abonnements Azure existants. Cet enregistrement s’effectue automatiquement lorsqu’une machine virtuelle est déployée par le biais du portail Azure dans n’importe quelle autre région à l’aide d’Azure Resource Manager. Une fois une machine virtuelle déployée dans toute autre région Azure, les nouvelles régions doivent être disponibles pour les machines virtuelles suivantes.

## <a name="does-azure-support-linux-vms"></a>Azure prend-il en charge les machines virtuelles Linux ?
Oui. Pour créer rapidement une machine virtuelle Linux de test, consultez [Création d’une machine virtuelle Linux sur Azure à l’aide du portail](virtual-machines-linux-quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>Puis-je ajouter une carte réseau à ma machine virtuelle après sa création ?
Non. L’ajout d’une carte réseau n’est possible que lors de la création.

## <a name="are-there-any-computer-name-requirements"></a>Existe-t-il des exigences en matière de nom d’ordinateur ?
Oui. Le nom d’ordinateur peut avoir une longueur maximale de 15 caractères. Consultez la rubrique [Instructions de dénomination d’infrastructure](virtual-machines-windows-infrastructure-naming-guidelines.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) pour plus d’informations sur la dénomination de ressources.

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>Quelles sont les exigences en matière de nom d’utilisateur lors de la création d’une machine virtuelle ?

Les noms d’utilisateur peuvent comporter un maximum de 20 caractères et ne doivent pas se terminer par un point («. »). 


Les noms d’utilisateur suivants ne sont pas autorisés :
<table>
    <tr>
        <td style="text-align:center">administrator </td><td style="text-align:center"> admin </td><td style="text-align:center"> user </td><td style="text-align:center"> user1</td>
    </tr>
    <tr>
        <td style="text-align:center">test </td><td style="text-align:center"> user2 </td><td style="text-align:center"> test1 </td><td style="text-align:center"> user3</td>
    </tr>    <tr>
        <td style="text-align:center">admin1 </td><td style="text-align:center"> 1 </td><td style="text-align:center"> 123 </td><td style="text-align:center"> a</td>
    </tr>
    <tr>
        <td style="text-align:center">actuser  </td><td style="text-align:center"> adm </td><td style="text-align:center"> admin2 </td><td style="text-align:center"> aspnet</td>
    </tr>
    <tr>
        <td style="text-align:center">backup </td><td style="text-align:center"> console </td><td style="text-align:center"> david </td><td style="text-align:center"> guest</td>
    </tr>
    <tr>
        <td style="text-align:center">john </td><td style="text-align:center"> propriétaire </td><td style="text-align:center"> root </td><td style="text-align:center"> server</td>
    </tr>
    <tr>
        <td style="text-align:center">sql </td><td style="text-align:center"> support </td><td style="text-align:center"> support_388945a0 </td><td style="text-align:center"> sys</td>
    </tr>
    <tr>
        <td style="text-align:center">test2 </td><td style="text-align:center"> test3 </td><td style="text-align:center"> user4 </td><td style="text-align:center"> user5</td>
    </tr>
</table>

## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>Quelles sont les exigences en matière de mot de passe lors de la création d’une machine virtuelle ?
Les mots de passe doivent comporter de 12 à 123 caractères et répondre à 3 des 4 exigences de complexité suivantes :

* Avoir des minuscules
* Avoir des majuscules
* Avoir un chiffre
* Avoir un caractère spécial (correspondances Regex [\W_])

Les noms mots de passe suivants ne sont pas autorisés :

<table>
    <tr>
        <td>abc@123 </td>
        <td>P@$$w0rd </td>
        <td>P@ssw0rd </td>
        <td>P@ssword123 </td>
        <td>Pa$$word </td>
    </tr>
    <tr>
        <td>pass@word1 </td>
        <td>Password! </td>
        <td>Password1 </td>
        <td>Password22 </td>
        <td>iloveyou! </td>
    </tr>
</table>



<!--HONumber=Feb17_HO3-->


