---
title: Salida de Azure Cosmos DB desde Azure Stream Analytics
description: En este artículo se describe cómo generar datos desde Azure Stream Analytics a Azure Cosmos DB.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: e322135cfdb7aaff331367e84c603e8344436528
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906262"
---
# <a name="azure-cosmos-db-output-from-azure-stream-analytics"></a>Salida de Azure Cosmos DB desde Azure Stream Analytics

[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) es un servicio de base de datos distribuido globalmente que ofrece un escalado ilimitado en todo el mundo, consultas avanzadas y una funcionalidad de indexación automática a partir de modelos de datos independientes del esquema. Para conocer las opciones de contenedor de Azure Cosmos DB para Stream Analytics, consulte el artículo [Stream Analytics con Azure Cosmos DB como salida](stream-analytics-documentdb-output.md).

La salida de Azure Cosmos DB desde Stream Analytics no está disponible actualmente en las regiones de Azure China (21Vianet) y Azure Alemania (T-Systems International).

> [!Note]
> Azure Stream Analytics solo admite la conexión a Azure Cosmos DB mediante la API de SQL.
> Aún no se admiten otras API de Azure Cosmos DB. Si apunta Azure Stream Analytics a las cuentas de Azure Cosmos DB creadas con otras API, puede que los datos no se almacenen correctamente.

En la tabla siguiente se describen las propiedades para crear una salida de Azure Cosmos DB.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida | Un alias para hacer referencia a esta salida en la consulta de Stream Analytics. |
| Receptor | Azure Cosmos DB. |
| Opción de importación | Elija **Select Cosmos DB from your subscription** (Seleccionar Cosmos DB desde la suscripción) o **Proporcionar la configuración de Cosmos DB manualmente**.
| Identificador de cuenta | El nombre o el URI del punto de conexión de la cuenta de Azure Cosmos DB. |
| Clave de cuenta | La clave de acceso compartido para la cuenta de Azure Cosmos DB. |
| Base de datos | El nombre de la base de datos de Azure Cosmos DB. |
| Nombre del contenedor | Nombre del contenedor que va a usarse, el cual debe existir en Cosmos DB. Ejemplo:  <br /><ul><li> _MyContainer_: debe existir un contenedor llamado "MyContainer".</li>|
| Id. de documento |Opcional. Nombre del campo de los eventos de salida utilizado para especificar la clave principal en la que se basan las operaciones de inserción o actualización.

## <a name="partitioning"></a>Creación de particiones

La clave de partición se basa en la cláusula PARTITION BY de la consulta. El número de escritores de salida sigue las particiones de entrada para [consultas totalmente paralelizadas](stream-analytics-scale-jobs.md). Stream Analytics convierte la clave de partición de salida de Cosmos DB en una cadena. Por ejemplo, si tiene una clave de partición con un valor de 1 de tipo bigint, se convierte en "1" de tipo cadena.

## <a name="output-batch-size"></a>Tamaño de lote de salida

Para obtener el tamaño máximo del mensaje, consulte los [límites de Azure Cosmos DB](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cosmos-db-limits). El tamaño de lote y la frecuencia de escritura se ajustan dinámicamente según las respuestas de Azure Cosmos DB. No existen limitaciones predeterminadas para Stream Analytics.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
