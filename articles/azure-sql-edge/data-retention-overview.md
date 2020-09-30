---
title: 'Información general sobre la Directiva de retención de datos: Azure SQL Edge'
description: Más información sobre la Directiva de retención de datos en Azure SQL Edge
keywords: SQL Edge, retención de datos
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: bb059a946c03f41e5b65944eec67070f84ee6b08
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90976344"
---
# <a name="data-retention-overview"></a>Introducción a la retención de datos

La recopilación y el almacenamiento de datos de los dispositivos IoT conectados es importante para impulsar y obtener información empresarial y operativa. Sin embargo, dado el volumen de datos que se originan en estos dispositivos, es importante que las organizaciones planeen cuidadosamente la cantidad de datos que desean conservar y con qué granularidad. Aunque es conveniente conservar todos los datos con toda la granularidad, no siempre es práctico. Además, el volumen de datos que se puede conservar está restringido por la cantidad de espacio de almacenamiento disponible en los dispositivos IoT o Edge. 

En la base de datos de Azure SQL Edge, los administradores pueden definir la Directiva de retención de datos en una base de datos de SQL Edge y sus tablas subyacentes. Una vez definida la Directiva de retención de datos, se ejecutará una tarea de sistema en segundo plano para purgar los datos obsoletos (antiguos) de las tablas del usuario. 

> [!Note]
> Los datos que se purgan de la tabla no se pueden recuperar. La única manera posible de recuperar los datos purgados es restaurar la base de datos a partir de una copia de seguridad anterior.

Guías de inicio rápido:

- [Habilitación y deshabilitación de directivas de retención de datos](data-retention-enable-disable.md)
- [Administración de datos históricos con directivas de retención](data-retention-cleanup.md)

## <a name="how-data-retention-works"></a>Cómo funciona la retención de datos

Para configurar la retención de datos, puede usar instrucciones DDL. Para obtener más información, vea [Habilitación y deshabilitación de las directivas de retención de datos](data-retention-enable-disable.md). Para la eliminación automática de los registros obsoletos, primero se debe habilitar la retención de datos para la base de datos y las tablas que desea purgar dentro de esa base de datos. 

Una vez configurada la retención de datos para una tabla, se ejecuta una tarea en segundo plano para identificar los registros obsoletos de una tabla y eliminar esos registros. Si, por algún motivo, la limpieza automática de las tareas no se está ejecutando o no puede seguir con las eliminaciones, se puede llevar a cabo una operación de limpieza manual en estas tablas. Para obtener más información sobre las limpiezas automáticas y manuales, consulte [Limpieza automática y manual](data-retention-cleanup.md).

## <a name="limitations-and-restrictions"></a>Limitaciones y restricciones

- La retención de datos, si está habilitada, se deshabilita automáticamente cuando la base de datos se restaura a partir de una copia de seguridad completa o se vuelve a adjuntar. 
- No se puede habilitar la retención de los datos para una tabla de historial temporal
- No se puede modificar el filtro de retención de datos colomn. Para modificar la columna, deshabilite la retención de datos en la tabla.  

## <a name="next-steps"></a>Pasos siguientes

- [Aprendizaje automático e inteligencia artificial con ONNX en SQL Edge](onnx-overview.md).
- [Creación de una solución de IoT de un extremo a otro con SQL Edge mediante IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de datos en Azure SQL Edge](stream-data.md)
