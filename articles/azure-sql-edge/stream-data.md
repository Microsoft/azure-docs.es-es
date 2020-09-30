---
title: Streaming de datos en Azure SQL Edge
description: Información sobre el streaming de datos en Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: ca22b3d2c00bfef128455df4ad6b9bb6411f8a13
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900564"
---
# <a name="data-streaming-in-azure-sql-edge"></a>Streaming de datos en Azure SQL Edge

Azure SQL Edge proporciona una implementación nativa de funcionalidades de streaming de datos denominada streaming de T-SQL. Proporciona streaming de datos en tiempo real, análisis y procesamiento de eventos para analizar y procesar grandes volúmenes de datos de streaming rápidos de varios orígenes simultáneamente. El streaming de T-SQL se crea con el mismo motor de streaming de alto rendimiento que potencia [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-introduction) en Microsoft Azure. La característica admite un conjunto similar de funcionalidades ofrecidas por Azure Stream Analytics que se ejecuta en el borde.

Al igual que con Stream Analytics, el streaming de T-SQL reconoce patrones y relaciones en la información extraída de varios orígenes de entrada de IoT, incluidos dispositivos, sensores y aplicaciones. Puede usar estos patrones para desencadenar acciones e iniciar flujos de trabajo. Por ejemplo, puede crear alertas, suministrar información a una solución de informes o visualización, o almacenar los datos para un uso posterior. 

El streaming de T-SQL puede ayudarle a lo siguiente:

* Análisis de los flujos de telemetría en tiempo real desde dispositivos IoT.
* Análisis en tiempo real de los datos generados a partir de vehículos autónomos y sin controlador.
* Uso de supervisión remota y mantenimiento predictivo de recursos industriales o de fabricación de gran valor.
* Uso de detección de anomalías y reconocimiento de patrones de lecturas del sensor de IoT en una granja agrícola o energética.

## <a name="how-does-t-sql-streaming-work"></a>¿Cómo funciona el streaming de T-SQL?

El streaming de T-SQL funciona exactamente de la misma manera que [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-introduction#how-does-stream-analytics-work). Por ejemplo, usa el concepto de *trabajos* de streaming para el procesamiento de streaming de datos en tiempo real. 

Un trabajo de Stream Analytics consta de lo siguiente:

- **Entrada de flujo**: define las conexiones a un origen de datos del que se va a leer el flujo de datos. Actualmente Azure SQL Edge admite los siguientes tipos de entradas de flujo:
    - Centro de Microsoft Edge
    - Kafka (la compatibilidad con las entradas de Kafka solo está disponible actualmente en las versiones Intel/AMD64 de Azure SQL Edge).

- **Salida de flujo**: define las conexiones a un origen de datos en el que se va a escribir el flujo de datos. Azure SQL Edge actualmente admite los siguientes tipos de salidas de flujo:
    - Centro de Microsoft Edge
    - SQL (la salida de SQL puede ser una base de datos local dentro de la instancia de Azure SQL Edge, o una instancia remota de SQL Server o Azure SQL Database). 

- **Consulta de flujo**: define la transformación, las agregaciones, el filtro, la ordenación y las combinaciones que se deben aplicar al flujo de entrada antes de que se escriba en la salida de flujo. La consulta de flujo se basa en el mismo lenguaje de consulta que el utilizado por Stream Analytics. Para más información, consulte [Lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference?).

> [!IMPORTANT]
> El streaming de T-SQL, a diferencia de Stream Analytics, no admite actualmente el [uso de datos de referencia para búsquedas](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-use-reference-data) ni el [uso de UDF y UDA en un trabajo de streaming](https://docs.microsoft.com/azure/stream-analytics/streaming-technologies#you-want-to-write-udfs-udas-and-custom-deserializers-in-a-language-other-than-javascript-or-c).

> [!NOTE]
> El streaming de T-SQL solo admite un subconjunto del área expuesta de lenguaje compatible con Stream Analytics. Para más información, consulte [Lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference?).

## <a name="limitations-and-restrictions"></a>Limitaciones y restricciones

Las limitaciones y restricciones siguientes se aplican al streaming de T-SQL. 

- Solo puede haber un trabajo de streaming activo en un momento específico. Los trabajos que ya se están ejecutando deben detenerse antes de iniciar otro trabajo.
- Cada ejecución de un trabajo de streaming tiene un único subproceso. Si el trabajo de streaming contiene varias consultas, cada consulta se evalúa en orden de serie.
- Al detener un trabajo de streaming en Azure SQL Edge, puede haber algún retraso al iniciar el siguiente. Este retraso se introduce porque el proceso de streaming subyacente debe detenerse en respuesta a la solicitud de detención del trabajo y después reiniciarse en respuesta a la solicitud de inicio del trabajo. 
- Streaming de T-SQL hasta 32 particiones para un flujo de Kafka. Intentar configurar un número de particiones mayor hace que se produzca un error. 

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un trabajo de Stream Analytics en Azure SQL Edge](create-stream-analytics-job.md)
- [Visualización de los metadatos asociados a los trabajos de streaming en Azure SQL Edge](streaming-catalog-views.md)
- [Creación de una transmisión externa](create-external-stream-transact-sql.md)
