---
title: Limites de Stretch Database | Microsoft Docs
description: En savoir plus sur les limites de Stretch Database.
services: sql-server-stretch-database
documentationcenter: 
author: douglaslMS
manager: jhubbard
editor: 
ms.assetid: 0a865872-7d42-4873-99b9-cbae45691e54
ms.service: sql-server-stretch-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/26/2016
ms.author: anvang
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 3c1599dfc6aee8adf7ec8714c392a7d8d4427d78


---
# <a name="limitations-for-stretch-database"></a>Limites de Stretch Database
Découvrez les limites des tables Stretch ainsi que les restrictions qui vous empêchent actuellement d’activer Stretch pour une table.

## <a name="a-namecaveatsa-limitations-for-stretch-enabled-tables"></a><a name="Caveats"></a> Limites des tables Stretch
Les tables Stretch présentent les limites suivantes.

### <a name="constraints"></a>Contraintes
* L’unicité n’est pas appliquée pour les contraintes UNIQUES et de contraintes de CLÉ PRIMAIRE dans une table Azure qui contient les données migrées.

### <a name="dml-operations"></a>Opérations DML
* Il est impossible de METTRE À JOUR ou SUPPRIMER des lignes ayant été migrées ou pouvant être migrées dans une table Stretch ou dans une vue comprenant des tables Stretch.
* Vous ne pouvez pas effectuer d’opérations INSERT sur des lignes dans une table Stretch Database sur un serveur lié.

### <a name="indexes"></a>Index
* Il est impossible de créer un index pour une vue comprenant des tables Stretch.
* Les filtres sur les index SQL Server ne sont pas propagés à la table distante.

## <a name="a-namelimitationsa-limitations-that-currently-prevent-you-from-enabling-stretch-for-a-table"></a><a name="Limitations"></a> Limites empêchant l’activation de Stretch pour une table
Les limites suivantes vous empêchent actuellement d’activer Stretch Database pour une table.

### <a name="table-properties"></a>Propriétés des tables
* Tables qui comportent plus de 1 023 colonnes ou plus de 998 index
* FileTables ou tables qui contiennent des données FILESTREAM
* Tables répliquées ou qui utilisent activement le suivi des modifications ou la capture de données modifiées
* Tables optimisées en mémoire

### <a name="data-types"></a>Types de données
* text, ntext et image
* timestamp
* sql\_variant
* XML
* Types de données CLR, y compris géométrie, géographie, hierarchyid et types CLR définis par l’utilisateur

### <a name="column-types"></a>Types de colonnes
* COLUMN\_SET
* Colonnes calculées

### <a name="constraints"></a>Contraintes
* Contraintes par défaut et contraintes de vérification
* Contraintes de clés étrangères qui référencent la table Dans une relation parent\-enfant \(par exemple, Order et Order\_Detail\), vous pouvez activer Stretch pour la table enfant \(Order\_Detail\), mais pas pour la table parente \(Order\).

### <a name="indexes"></a>Index
* Index en texte intégral
* Index XML
* Index spatiaux
* Vues indexées qui référencent la table

## <a name="see-also"></a>Voir aussi
[Identifier des bases de données et des tables pour Stretch Database en exécutant Stretch Database Advisor](sql-server-stretch-database-identify-databases.md)

[Activer Stretch Database pour une base de données](sql-server-stretch-database-enable-database.md)

[Activer Stretch Database pour une table](sql-server-stretch-database-enable-table.md)




<!--HONumber=Nov16_HO3-->


