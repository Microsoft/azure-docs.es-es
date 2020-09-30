---
title: Salida de Table Storage desde Azure Stream Analytics
description: En este artículo se describe Azure Table Storage como salida para Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 3e5b747e55c8f246167bcf8bbde3542146e69706
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891955"
---
# <a name="table-storage-output-from-azure-stream-analytics"></a>Salida de Table Storage desde Azure Stream Analytics

[almacenamiento de tablas de Azure](../storage/common/storage-introduction.md) ofrece un tipo de almacenamiento de alta disponibilidad y escalabilidad masiva, de forma que las aplicaciones pueden escalarse automáticamente para ajustarse a la demanda de los usuarios. Table Storage es un almacén de claves/atributos NoSQL de Microsoft que puede usar para datos estructurados con menos restricciones en el esquema. El almacenamiento de tablas de Azure puede usarse para almacenar datos con de persistencia y recuperación eficaz.

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear una salida de tabla.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida |Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este almacenamiento de tablas. |
| Cuenta de almacenamiento |Nombre de la cuenta de almacenamiento a donde está enviando la salida. |
| Clave de cuenta de almacenamiento |La clave de acceso asociada con la cuenta de almacenamiento. |
| Nombre de la tabla |Nombre de la tabla. Se crea la tabla si no existe. |
| Clave de partición |Nombre de la columna de salida que contiene la clave de partición. La clave de partición es un identificador único de la partición dentro de una tabla que constituye la primera parte de la clave principal de la entidad. Es un valor de cadena que puede tener un tamaño de hasta 1 KB. |
| Clave de fila |Nombre de la columna de salida que contiene la clave de fila. La clave de fila es un identificador único de una entidad dentro de una partición. Forma la segunda parte de la clave principal de la entidad. La clave de fila es un valor de cadena que puede tener un tamaño de hasta 1 KB. |
| Tamaño de lote |El número de registros para una operación por lotes. El valor predeterminado (100) es suficiente para la mayoría de los trabajos. Para obtener más información sobre la modificación de esta configuración, consulte [Especificaciones de la operación por lotes de tablas](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.table.tablebatchoperation). |

## <a name="partitioning"></a>Creación de particiones

La clave de partición es cualquier columna de salida. El número de escritores de salida sigue las particiones de entrada para [consultas totalmente paralelizadas](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Tamaño de lote de salida

Para obtener el tamaño máximo del mensaje, consulte los [límites de Azure Storage](../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits). El valor predeterminado es de 100 entidades por transacción única, pero puede configurarse con un valor menor según sea necesario.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
