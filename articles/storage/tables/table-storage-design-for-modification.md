---
title: Diseño de Azure Table Storage para la modificación de datos | Microsoft Docs
description: Diseñe tablas para la modificación de datos en Azure Table Storage. Optimice las operaciones de inserción, actualización y eliminación. Asegúrese la coherencia de las entidades que ha almacenado.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: 25785bc4b945f469e67f2a71eb6676940e091d56
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236767"
---
# <a name="design-for-data-modification"></a>Diseño para la modificación de datos
Este artículo se centra en las consideraciones de diseño para optimizar las inserciones, actualizaciones y eliminaciones. En algunos casos, deberá evaluar el equilibrio entre los diseños que se optimizan para realizar una consulta en diseños que optimizan la modificación de datos como lo hace usted en los diseños de bases de datos relacionales (aunque las técnicas para administrar las ventajas y desventajas de diseño son diferentes en una base de datos relacional). En la sección Patrones de diseño de tablas se describen algunos modelos de diseño detallados para Table service y se destacan algunas de estas ventajas e inconvenientes. En la práctica, encontrará que muchos diseños optimizados para consultar entidades también funcionan bien para la modificación de entidades.  

## <a name="optimize-the-performance-of-insert-update-and-delete-operations"></a>Optimización del rendimiento de las operaciones de inserción, actualización y eliminación
Para actualizar o eliminar una entidad, debe poder identificarla mediante el uso de los valores **PartitionKey** y **RowKey**. En este sentido, la elección de **PartitionKey** y **RowKey** para modificar entidades debería seguir criterios similares a su elección para admitir consultas de punto porque desea identificar las entidades de la forma más eficaz posible. No desea utilizar un examen ineficaz de partición o de tabla para buscar una entidad con el fin de detectar los valores **PartitionKey** y **RowKey** que necesita actualizar o eliminar.  

Los patrones siguientes de la sección Patrones de diseño de tablas abordan la optimización del rendimiento o las operaciones de inserción, actualización y eliminación:  

* [Patrón de eliminación de gran volumen](table-storage-design-patterns.md#high-volume-delete-pattern) : habilitar la eliminación de un gran volumen de entidades mediante el almacenamiento de todas las entidades para su eliminación simultánea en su propia tabla independiente; elimine las entidades mediante la eliminación de la tabla.  
* [Patrón de serie de datos](table-storage-design-patterns.md#data-series-pattern): almacenar una serie de datos completa en una sola entidad para minimizar el número de solicitudes que realice.  
* [Patrón de entidades amplio](table-storage-design-patterns.md#wide-entities-pattern): usar varias entidades físicas para almacenar entidades lógicas con más de 252 propiedades.  
* [Patrón de entidades de gran tamaño](table-storage-design-patterns.md#large-entities-pattern) : use el almacenamiento de blobs para almacenar valores de propiedad de gran tamaño.  

## <a name="ensure-consistency-in-your-stored-entities"></a>Garantía de la coherencia en las entidades almacenadas
El otro factor clave que afecta a su elección de claves para optimizar las modificaciones de datos es cómo garantizar la coherencia mediante el uso de transacciones atómicas. Solo puede utilizar un EGT para operar en las entidades almacenadas en la misma partición.  

Los siguientes patrones del artículo [Patrones de diseño de tablas](table-storage-design-patterns.md) abordan la coherencia en la administración:  

* [Patrón de índice secundario dentro de la partición](table-storage-design-patterns.md#intra-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores **RowKey** (en la misma partición) para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores **RowKey**.  
* [Patrón de índice secundario entre particiones](table-storage-design-patterns.md#inter-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores RowKey en particiones o en tablas independientes para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores **RowKey**.  
* [Patrón final coherente de transacciones](table-storage-design-patterns.md#eventually-consistent-transactions-pattern) : habilitar el comportamiento final coherente a través de límites de partición o los límites del sistema de almacenamiento mediante el uso de las colas de Azure.
* [Patrón de entidades de índice](table-storage-design-patterns.md#index-entities-pattern): mantener las entidades de índice para permitir búsquedas eficaces que devuelvan listas de entidades.  
* [Patrón de desnormalización](table-storage-design-patterns.md#denormalization-pattern): combinar datos relacionados entre sí en una sola entidad para recuperar todos los datos que necesita con una consulta de punto único.  
* [Patrón de serie de datos](table-storage-design-patterns.md#data-series-pattern): almacenar una serie de datos completa en una sola entidad para minimizar el número de solicitudes que realice.  

Para información sobre EGT, consulte la sección [Transacciones de grupos de entidades](table-storage-design.md#entity-group-transactions).  

## <a name="ensure-your-design-for-efficient-modifications-facilitates-efficient-queries"></a>Garantía de modificaciones eficientes en el diseño para facilitar la eficiencia de las consultas
En muchos casos, un diseño para los resultados de consultas eficaces en modificaciones eficaces, pero siempre debe evaluar si este es el caso para su escenario concreto. Algunos de los patrones del artículo [Patrones de diseño de tablas](table-storage-design-patterns.md) evalúan explícitamente las ventajas e inconvenientes de consultar y modificar entidades. Siempre debe tener en cuenta el número de cada tipo de operación.  

Los siguientes patrones del artículo [Patrones de diseño de tablas](table-storage-design-patterns.md) abordan las ventajas e inconvenientes de diseñar para consultas eficaces y diseñar para la modificación eficaz de los datos:  

* [Patrón de clave compuesta](table-storage-design-patterns.md#compound-key-pattern) : utilice valores **RowKey** compuestos para permitir a un cliente buscar datos relacionados con una consulta de punto único.  
* [Patrón final del registro](table-storage-design-patterns.md#log-tail-pattern) : recupere las entidades *n* agregadas recientemente a una partición utilizando un valor **RowKey** que se ordene en orden de fecha y hora inverso.  
