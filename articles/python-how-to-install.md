---
title: "Installation de Python et du Kit de développement logiciel (SDK) - Azure"
description: "Découvrez comment installer Python et le Kit de développement logiciel (SDK) à utiliser avec Azure."
services: 
documentationcenter: python
author: lmazuel
manager: wpickett
editor: 
ms.assetid: f36294be-daeb-4caf-9129-fce18130f552
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 09/06/2016
ms.author: lmazuel
translationtype: Human Translation
ms.sourcegitcommit: 63cf1a5476a205da2f804fb2f408f4d35860835f
ms.openlocfilehash: f4c30c4653d8a14c7bf68ec6935c26725c6f623c


---
# <a name="installing-python-and-the-sdk"></a>Installation de Python et du Kit de développement logiciel (SDK)
La configuration de Python sous Windows s’effectue en toute simplicité. La solution est préinstallée sous Mac, Linux et [Bash pour Windows](https://msdn.microsoft.com/commandline/wsl/about). Ce guide décrit les étapes d'installation et de préparation de votre système à l'utilisation d'Azure.

## <a name="whats-in-the-python-azure-sdk"></a>Que contient le Kit de développement logiciel (SDK) Python Azure ?
Le Kit de développement logiciel (SDK) Azure pour Python contient les composants qui vous permettent de développer, déployer et gérer des applications Python pour Azure. Il inclut plus précisément les éléments suivants :

* **Bibliothèques de gestion**. Ces bibliothèques de classes fournissent une interface de gestion des ressources Azure, comme les comptes de stockage ou les machines virtuelles.
* **Bibliothèques Runtime**. Ces bibliothèques de classes fournissent une interface permettant d’accéder aux fonctionnalités Azure, notamment le stockage et Service Bus.

## <a name="which-python-and-which-version-to-use"></a>Quelle solution Python et quelle version utiliser ?
Il existe différentes versions des interpréteurs Python, par exemple :

* CPython : interpréteur Python standard le plus couramment utilisé
* PyPy : implémentation rapide et conforme autre que CPython
* IronPython : interpréteur Python qui s'exécute sur .Net/CLR
* Jython : interpréteur Python qui s’exécute sur la Machine Virtuelle Java

**CPython** v2.7 ou v3.3+ et PyPy 5.4.0 sont testés et pris en charge par le Kit de développement logiciel (SDK) Python Azure.

## <a name="where-to-get-python"></a>Où télécharger Python ?
Il existe plusieurs façons de télécharger CPython :

* Directement depuis [www.python.org][www.python.org]
* Auprès d’un distributeur fiable comme [www.continuum.io][www.continuum.io], [www.enthought.com][www.enthought.com] ou [www.activestate.com][www.activestate.com]
* Génération à partir de la source !

Sauf en cas de nécessité particulière, nous vous recommandons d’opter pour l’une des deux premières options.

## <a name="sdk-installation-on-windows-linux-and-macos-client-libraries-only"></a>Installation du SDK sous Windows, Linux et macOS (bibliothèques client uniquement)
Si vous avez déjà installé Python, vous pouvez utiliser pip pour installer un lot de toutes les bibliothèques client dans votre environnement Python 2.7 ou Python 3.3+ existant. Cela télécharge les packages à partir du [Python Package Index][Python Package Index] (PyPI).

Vous aurez peut-être besoin de droits d’administrateur :

* Linux et macOS, utilisez la commande `sudo` : `sudo pip install azure-mgmt-compute`.
* Windows : ouvrez votre invite de commandes/PowerShell en tant qu’administrateur

Vous pouvez installer chaque bibliothèque individuellement pour chaque service Azure :

```console
   $ pip install azure-batch          # Install the latest Batch runtime library
   $ pip install azure-mgmt-scheduler # Install the latest Storage management library
```

L’aperçu des packages peut être installé à l’aide de l’indicateur `--pre` :

```console
   $ pip install --pre azure-mgmt-compute # will install only the latest Compute Management library
```

Vous pouvez également installer un ensemble de bibliothèques Azure d’une seule ligne à l’aide du méta-package `azure` . Étant donné que tous les packages de ce méta-package ne sont pas encore publiés de façon stable, le méta-package `azure` est encore en version préliminaire.
Toutefois, les packages de base peuvent pour l’instant être considérés comme « stables » du point de vue de la qualité / l’exhaustivité du code

* ils seront officiellement étiquetés comme tels, en phase avec d’autres langages, dès que possible.
  Nous ne prévoyons pas d’autres changements majeurs d’ici-là.

Puisqu’il s’agit d’une version préliminaire, vous devez utiliser l’indicateur `--pre` :

```console
   $ pip install --pre azure
```

ou directement

```console
   $ pip install azure==2.0.0rc6
```

## <a name="getting-more-packages"></a>Téléchargement d'autres packages
[Python Package Index][Python Package Index] (PyPI) comporte une sélection étoffée de bibliothèques Python.  Si vous avez installé une version fournie par un distributeur, vous disposez déjà de la plupart des éléments nécessaires pour différents scénarios, du développement web au calcul technique.

## <a name="python-tools-for-visual-studio"></a>Python Tools for Visual Studio
[Python Tools for Visual Studio][Python Tools pour Visual Studio] (PTVS) est un plug-in gratuit/OSS de Microsoft, qui transforme Visual Studio en un environnement de développement intégré (IDE) Python à part entière :

![how-to-install-python-ptvs](./media/python-how-to-install/how-to-install-python-ptvs.png)

Nous vous recommandons d’utiliser PTVS même s’il est facultatif, car il vous apporte pour les projets/solutions Python et web les fonctionnalités suivantes : support technique, débogage, profilage, fenêtre interactive, modification des modèles et Intellisense.

PTVS simplifie également le déploiement sur Microsoft Azure, avec une prise en charge du déploiement sur [Cloud Services](cloud-services/cloud-services-python-ptvs.md) et [Sites Web](app-service-web/web-sites-python-ptvs-django-mysql.md).

PTVS fonctionne avec votre installation existante de Visual Studio 2013 ou 2015.  Pour accéder à la documentation, aux téléchargements et aux discussions, consultez le site [Python Tools pour Visual Studio].  

## <a name="python-azure-scenarios-for-linux-and-macos"></a>Scénarios Python Azure pour Linux et MacOS
Pour Linux et macOS, les principaux scénarios Azure pris en charge sont les suivants :

1. Consommation des services Azure par le biais des bibliothèques clientes pour Python
2. Exécution de votre application sur une machine virtuelle Linux
3. Développement et publication de sites web Azure à l'aide de Git

Le premier scénario vous permet de créer des applications web enrichies qui tirent parti des fonctionnalités PaaS d’Azure, comme le [Stockage Blob](virtual-machines/virtual-machines-linux-quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json), le [Stockage File d’attente](storage/storage-python-how-to-use-queue-storage.md) ou le [Stockage Table](storage/storage-python-how-to-use-table-storage.md), par le biais de wrappers Python pour les API REST Azure. Le fonctionnement de ces derniers est identique sous Windows, Mac et Linux.  Vous pouvez également utiliser ces bibliothèques clientes à partir de votre ordinateur de développement local ou une machine virtuelle Linux s'exécutant sur Azure.

Pour le scénario nécessitant une machine virtuelle, vous pouvez tout simplement démarrer une machine virtuelle Linux de votre choix (Ubuntu, CentOS, Suse) et exécuter/gérer les logiciels souhaités.  Par exemple, vous pouvez exécuter [IPython][IPython] REPL/Notebook sur votre machine Windows/Mac/Linux et pointer votre navigateur vers une machine virtuelle multiprocesseur Linux ou Windows exécutant le moteur IPython Engine sur Azure. Pour plus d’informations, voir le didacticiel [IPython Notebook sur Azure](virtual-machines/virtual-machines-linux-jupyter-notebook.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) .

Pour plus d’informations sur la configuration d’une machine virtuelle Linux, voir le didacticiel [Création d’une machine virtuelle exécutant Linux](virtual-machines/virtual-machines-linux-quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) .

À l'aide du déploiement Git, vous pouvez développer une application web Python et la publier sur un site web Azure à partir de n'importe quel système d'exploitation.  Quand vous placez votre référentiel sur Azure, il crée automatiquement un environnement virtuel et utilise pip pour installer vos packages requis.

Pour plus d’informations sur le développement et la publication de sites web Azure, consultez les didacticiels [Création de sites web avec Django](app-service-web/web-sites-python-create-deploy-django-app.md), [Création de sites web avec Bottle](app-service-web/web-sites-python-create-deploy-bottle-app.md) et [Création de sites web avec Flask](app-service-web/web-sites-python-create-deploy-flask-app.md). Pour des informations plus générales sur l’utilisation de n’importe quelle infrastructure compatible WSGI, consultez [Configuration de Python avec des Sites Web Azure](app-service-web/web-sites-python-configure.md).

## <a name="additional-software-and-resources"></a>Logiciels et ressources supplémentaires :
* [Kit de développement logiciel (SDK) Azure pour Python Read The Docs](http://azure-sdk-for-python.readthedocs.io/en/latest/)
* [Kit de développement logiciel (SDK) Azure pour Python GitHub](https://github.com/Azure/azure-sdk-for-python)
* [Exemples Azure officiels pour Python](https://azure.microsoft.com/documentation/samples/?platform=python)
* [Distribution de Python par Continuum Analytics][Distribution de Python par Continuum Analytics (en anglais)]
* [Distribution de Python par Enthought][Distribution de Python par Enthought (en anglais)]
* [Distribution de Python par ActiveState][Distribution de Python par ActiveState (en anglais)]
* [SciPy : un vaste panel de bibliothèques Scientific Python][SciPy : un vaste panel de bibliothèques Scientific Python (en anglais)]
* [NumPy : une bibliothèque de valeurs numériques pour Python][NumPy : une bibliothèque de valeurs numériques pour Python (en anglais)]
* [Projet Django : une structure Web/un système de gestion du contenu arrivé à maturité][Projet Django : une infrastructure web/un système de gestion du contenu arrivés à maturité (en anglais)]
* [IPython : une solution REPL/Notebook avancée pour Python][IPython : une solution REPL/Notebook avancée pour Python (en anglais)]
* [IPython Notebook sur Azure](virtual-machines/virtual-machines-linux-jupyter-notebook.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Python Tools pour Visual Studio sur GitHub][Python Tools pour Visual Studio sur GitHub]
* [Centre de développement Python](/develop/python/)

[Distribution de Python par Continuum Analytics (en anglais)]: http://continuum.io
[Distribution de Python par Enthought (en anglais)]: http://www.enthought.com
[Distribution de Python par ActiveState (en anglais)]: http://www.activestate.com
[www.python.org]: http://www.python.org
[www.continuum.io]: http://continuum.io
[www.enthought.com]: http://www.enthought.com
[www.activestate.com]: http://www.activestate.com
[SciPy : un vaste panel de bibliothèques Scientific Python (en anglais)]: http://www.scipy.org
[NumPy : une bibliothèque de valeurs numériques pour Python (en anglais)]: http://www.numpy.org
[Projet Django : une infrastructure web/un système de gestion du contenu arrivés à maturité (en anglais)]: http://www.djangoproject.com
[IPython : une solution REPL/Notebook avancée pour Python (en anglais)]: http://ipython.org
[IPython]: http://ipython.org
[IPython Notebook sur Azure]: virtual-machines-linux-jupyter-notebook.md
[Cloud Services]: cloud-services-python-ptvs.md
[Sites Web]: web-sites-python-ptvs-django-mysql.md
[Python Tools pour Visual Studio]: http://aka.ms/ptvs
[Python Tools pour Visual Studio sur GitHub]: https://github.com/microsoft/ptvs
[Python Package Index]: http://pypi.python.org/pypi
[Kit de développement logiciel (SDK) Microsoft Azure pour Python 2.7]: http://go.microsoft.com/fwlink/?LinkId=254281
[Kit de développement logiciel (SDK) Microsoft Azure pour Python 3.4]: http://go.microsoft.com/fwlink/?LinkID=516990
[Configuration d'une machine virtuelle Linux via le portail Azure]: create-and-configure-opensuse-vm-in-portal.md
[Utilisation des outils en ligne de commande Azure]: crossplat-cmd-tools.md
[Création d’une machine virtuelle exécutant Linux]: virtual-machines-linux-quick-create-cli.md
[Création de sites Web avec Django]: web-sites-python-create-deploy-django-app.md
[Création de sites web avec Bottle]: web-sites-python-create-deploy-bottle-app.md
[Création de sites web avec Flask]: web-sites-python-create-deploy-flask-app.md
[Configuration de Python avec des Sites Web Azure]: web-sites-python-configure.md
[Stockage Table]: storage-python-how-to-use-table-storage.md
[stockage en file d’attente]: storage-python-how-to-use-queue-storage.md
[stockage d’objets blob]: storage-python-how-to-use-blob-storage.md



<!--HONumber=Nov16_HO3-->


