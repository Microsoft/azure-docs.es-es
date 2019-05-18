---
title: archivo de inclusión
description: archivo de inclusión
services: search
author: HeidiSteen
ms.service: search
ms.topic: include
ms.date: 05/06/2019
ms.author: heidist
ms.custom: include file
ms.openlocfilehash: b298c87d802314bd865bd6f38c35e4361eb69f3f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65212041"
---
El almacenamiento está limitado por el espacio en disco o el *número máximo* de índices, documentos u otros recursos de alto nivel, lo que ocurra primero. En la tabla siguiente se documentan los límites de almacenamiento. Para los límites máximos en índices, documentos y otros objetos, consulte [límites por recurso](../articles/search/search-limits-quotas-capacity.md#index-limits).

| Recurso | Gratis | Basic<sup>1</sup> | S1 | S2 | S3 | S3&nbsp;HD<sup>2</sup> | L1 | L2 |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| Nivel acuerdo de servicio (SLA)<sup>3</sup>  |No |Sí |Sí |Sí |Sí |Sí |Sí |Sí |
| Almacenamiento por partición |50 MB |2 GB |25 GB |100 GB |200 GB |200 GB |1 TB |2 TB |
| Particiones por servicio |N/D |1 |12 |12 |12 |3 |12 |12 |
| Tamaño de la partición |N/D |2 GB |25 GB |100 GB |200 GB |200 GB |1 TB |2 TB |
| Réplicas |N/D |3 |12 |12 |12 |12 |12 |12 |

<sup>1</sup> Básico tiene una partición fija. En este nivel, las unidades de búsqueda adicional se usan para asignar más réplicas para cargas de trabajo de consulta.

<sup>2</sup> S3 HD tiene un límite máximo de tres particiones, que es inferior al límite de partición para S3. El límite inferior de la partición se impone porque el número de índice para S3 HD es mucho más alto. Como existen límites de servicio en ambos recursos informáticos (almacenamiento y procesamiento) y el contenido (índices y documentos), el límite de contenido se alcanza primero.

<sup>3</sup> se ofrecen contratos de nivel de servicio para servicios facturables en recursos dedicados. Los servicios gratuitos y las características de versión preliminar no tienen SLA. Para los servicios facturables, los SLA tomarán efecto cuando se aprovisione suficiente redundancia para el servicio. Dos o más réplicas son necesarias para el SLA de consulta (lectura). Son necesarias para la consulta e indexación (lectura y escritura) SLA tres o más réplicas. El número de particiones no es una consideración del SLA. 