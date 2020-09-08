---
title: Directrices de diseño de tablas de Azure Storage | Microsoft Docs
description: Conozca las directrices de diseño del servicio Azure Table Storage para respaldar las operaciones de lectura y escritura de manera eficaz.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: f84707e454a8b1f5d5947478fe65108a142a9757
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236325"
---
# <a name="guidelines-for-table-design"></a>Directrices para el diseño de tablas

Diseñar tablas para su uso con Table service de Azure Storage es muy diferente a las consideraciones de diseño de las bases de datos relacionales. En este artículo se describen las directrices para diseñar la solución de Table service para lecturas y escrituras eficientes.

## <a name="design-your-table-service-solution-to-be-read-efficient"></a>Diseño de una solución de Table service para lecturas eficientes

* ***Diseño para realizar consultas en aplicaciones con muchas lecturas.*** Al diseñar las tablas, piense en las consultas (especialmente las sensibles a la latencia) que ejecutará antes de pensar cómo actualizará las entidades. Normalmente esto produce una solución eficiente y de rendimiento.  
* ***Especificar tanto PartitionKey como RowKey en sus consultas.*** *consultas puntuales* como estas son las consultas más eficaces de servicio de tabla.  
* ***Tenga en cuenta la posibilidad de almacenar copias duplicadas de las entidades.*** El almacenamiento en tablas es barato por lo que puede almacenar la misma entidad varias veces (con claves diferentes) para permitir que se realicen consultas más eficaces.  
* ***Considere la posibilidad de desnormalizar sus datos.*** El almacenamiento en tablas es barato, por tanto, piense en desnormalizar sus datos. Por ejemplo, almacene entidades de resumen para que las consultas para datos agregados solo necesiten acceder a una única entidad.  
* ***Use valores de clave compuestos.*** Las únicas claves de las que dispone son **PartitionKey** y **RowKey**. Por ejemplo, s los valores de clave compuestos para habilitar rutas de acceso con clave alternativas a las entidades.  
* ***Use la proyección de consultas.*** Puede reducir la cantidad de datos que se transfieren a través de la red mediante el uso de consultas que seleccionen solo los campos que necesite.  

## <a name="design-your-table-service-solution-to-be-write-efficient"></a>Diseño de una solución de Table service para escrituras eficientes  

* ***No cree particiones activas.*** Elija claves que le permitan distribuir las solicitudes en varias particiones en cualquier momento.  
* ***Evite picos de tráfico.*** Equilibre el tráfico en un período de tiempo razonable y evite picos de tráfico.
* ***No es preciso crear necesariamente una tabla independiente para cada tipo de entidad.*** Cuando necesite transacciones atómicas en los tipos de entidad, puede almacenar estos distintos tipos de entidad en la misma partición de la misma tabla.
* ***Tenga en cuenta el rendimiento máximo que debe alcanzar.*** Debe tener en cuenta los objetivos de escalabilidad de Table service y asegurarse de que su diseño no los superará.  

A medida que lea esta guía, verá ejemplos en los que se ponen en práctica todos estos principios. 

## <a name="next-steps"></a>Pasos siguientes

- [Patrones de diseño de tablas](table-storage-design-patterns.md)
- [Diseño de consulta](table-storage-design-for-query.md)
- [Cifrado de datos de tabla](table-storage-design-encrypt-data.md)
- [Diseño para la modificación de datos](table-storage-design-for-modification.md)
