---
title: Servicios de Azure que admiten Azure Data Lake Storage Gen2 | Microsoft Docs
description: Obtenga información sobre qué servicios de Azure se integran con Azure Data Lake Storage Gen2
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: c3a5f3a984c95af400c9e0c1543e3c1883290668
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442963"
---
# <a name="azure-services-that-support-azure-data-lake-storage-gen2"></a>Servicios de Azure que admiten Azure Data Lake Storage Gen2

Puede usar los servicios de Azure para ingerir datos, realizar análisis y crear representaciones visuales. En este artículo se proporciona una lista de los servicios de Azure admitidos, se revela su nivel de compatibilidad y se proporcionan vínculos a artículos que le ayudan a usar estos servicios con Azure Data Lake Storage Gen2.

## <a name="supported-azure-services"></a>Servicios de Azure compatibles

En esta tabla se enumeran los servicios de Azure que puede usar con Azure Data Lake Storage Gen2. Los elementos que aparecen en estas tablas van a cambiar con el tiempo a medida que se siga ampliando la compatibilidad.

> [!NOTE]
> El nivel de compatibilidad solo se refiere al modo en que se admite el servicio con Data Lake Storage Gen 2.

|Servicio de Azure |Nivel de compatibilidad |Azure AD |Clave compartida| Artículos relacionados |
|---------------|-------------------|---|---|---|
|Azure Data Factory|Disponibilidad general|Sí|Sí|[Carga de datos en Azure Data Lake Storage Gen2 con Azure Data Factory](https://docs.microsoft.com/azure/data-factory/load-azure-data-lake-storage-gen2?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Azure Databricks|Disponibilidad general|Sí|Sí|[Uso con Azure Databricks](https://docs.azuredatabricks.net/data/data-sources/azure/azure-datalake-gen2.html) <br> [Inicio rápido: Análisis de datos en Azure Data Lake Storage Gen2 con Azure Databricks](data-lake-storage-quickstart-create-databricks-account.md) <br>[Tutorial: Extracción, transformación y carga de datos mediante Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/databricks-extract-load-sql-data-warehouse) <br>[Tutorial: Acceso a los datos de Azure Data Lake Storage Gen2 con Azure Databricks mediante Spark](data-lake-storage-use-databricks-spark.md)|
|Centro de eventos de Azure|Disponibilidad general|No|Sí|[Captura de eventos a través de Azure Event Hubs en Azure Blob Storage o Azure Data Lake Storage](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)|
|Azure Event Grid|Disponibilidad general|Sí|Sí|[Tutorial: Implementación del patrón de captura de Data Lake para actualizar una tabla de Databricks Delta](data-lake-storage-events.md)|
|Azure Logic Apps|Disponibilidad general|No|Sí|[Introducción: ¿Qué es Azure Logic Apps?](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview)|
|Azure Machine Learning|Disponibilidad general|Sí|Sí|[Acceso a los datos en los servicios de almacenamiento de Azure](https://docs.microsoft.com/azure/machine-learning/how-to-access-data)|
|Azure Stream Analytics|Disponibilidad general|Sí|Sí|[Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-quick-create-portal) <br> [Salida a Azure Data Lake Gen2](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-define-outputs#blob-storage-and-azure-data-lake-gen2)|
|Data Box|Disponibilidad general|No|Sí|[Uso de Azure Data Box para migrar datos de un almacén HDFS local a Azure Storage](data-lake-storage-migrate-on-premises-hdfs-cluster.md)|
|HDInsight |Disponibilidad general|Sí|Sí|[Uso de Data Lake Storage Gen2 con clústeres de Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)<br>[Uso de la CLI de HDFS con Data Lake Storage Gen2](data-lake-storage-use-hdfs-data-lake-storage.md) <br>[Tutorial: Extracción, transformación y carga de datos mediante Apache Hive en Azure HDInsight](data-lake-storage-tutorial-extract-transform-load-hive.md)|
|IoT Hub |Disponibilidad general|No|Sí|[Uso del enrutamiento de mensajes de IoT Hub para enviar mensajes del dispositivo a la nube a distintos puntos de conexión](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c)|
|Power BI|Disponibilidad general|Sí|Sí|[Análisis de datos en Data Lake Storage Gen2 mediante Power BI](https://docs.microsoft.com/power-query/connectors/datalakestorage)|
|Azure Synapse Analytics (anteriormente SQL Data Warehouse)|Disponibilidad general|Sí|Sí|[Uso con Azure Synapse Analytics (antes SQL Data Warehouse)](https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#azure-sql-data-warehouse-polybase)|
|SQL Server Integration Services (SSIS)|Disponibilidad general|Sí|Sí|[Administrador de conexiones de Azure Storage](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-storage-connection-manager?view=sql-server-2017)|
|Explorador de datos de Azure|Disponibilidad general|Sí|Sí|[Consulta de datos en Azure Data Lake con Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/data-lake-query-data)|
|Azure Cognitive Search|Versión preliminar|Sí|Sí|[Indexación y búsqueda de documentos de Azure Data Lake Storage Gen2 (versión preliminar)](https://docs.microsoft.com/azure/search/search-howto-index-azure-data-lake-storage)|
|Azure Content Delivery Network|Todavía no se admite|No aplicable|No aplicable|[Indexación y búsqueda de documentos de Azure Data Lake Storage Gen2 (versión preliminar)](https://docs.microsoft.com/azure/cdn/cdn-overview)|

## <a name="see-also"></a>Consulte también

- [Problemas conocidos con Azure Data Lake Storage Gen2](data-lake-storage-known-issues.md)
- [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md)
- [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md)
- [Acceso multiprotocolo en Azure Data Lake Storage](data-lake-storage-multi-protocol-access.md)