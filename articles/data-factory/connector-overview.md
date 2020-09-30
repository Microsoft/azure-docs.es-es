---
title: Información general sobre el conector de Azure Data Factory
description: Obtenga información sobre los conectores compatibles en Data Factory.
services: data-factory
author: linda33wj
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: jingwang
ms.reviewer: craigg
ms.openlocfilehash: a729d470cccd4121523c767ada9077a51361c061
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89181990"
---
# <a name="azure-data-factory-connector-overview"></a>Información general sobre el conector de Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Factory admite los siguientes almacenes de datos y formatos a través de las actividades Copiar, Flujo de datos, Buscar, Obtener metadatos y Eliminar. Haga clic en cada almacén de datos para obtener información detallada sobre las funcionalidades admitidas y las configuraciones correspondientes.

## <a name="supported-data-stores"></a>Almacenes de datos compatibles

[!INCLUDE [Connector overview](../../includes/data-factory-v2-connector-overview.md)]

## <a name="integrate-with-more-data-stores"></a>Integración con más almacenes de datos

Azure Data Factory puede llegar a un conjunto más amplio de almacenes de datos que la lista mencionada anteriormente. Si necesita trasladar datos a o desde un almacén de datos que no está en la lista de conectores integrados de Azure Data Factory, estas son algunas opciones extensibles:
- En el caso de bases de datos y almacenamiento de datos, normalmente puede encontrar un controlador ODBC correspondiente, con el que puede usar un [conector ODBC genérico](connector-odbc.md).
- Para las aplicaciones SaaS:
    - Si proporciona API de RESTful, puede usar un [conector de REST genérico](connector-rest.md).
    - Si tiene fuentes de OData, puede usar un [conector OData genérico](connector-odata.md).
    - Si proporciona API de SOAP, puede usar un [conector HTTP genérico](connector-http.md).
    - Si tiene un controlador ODBC, puede usar un [conector ODBC genérico](connector-odbc.md).
- Para otros, compruebe si puede cargar o exponer datos como con cualquier almacén de datos compatible con ADF; por ejemplo, blobs de Azure, archivos, FTP, SFTP, etc. A continuación, deje que el ADF se recoja desde allí. Puede invocar un mecanismo de carga de datos personalizado a través de [funciones de Azure Functions](control-flow-azure-function-activity.md), [actividades personalizadas](transform-data-using-dotnet-custom-activity.md), [Databricks](transform-data-databricks-notebook.md)/[HDInsight](transform-data-using-hadoop-hive.md), [actividades web](control-flow-web-activity.md), etc.

## <a name="supported-file-formats"></a>Formatos de archivos admitidos

Azure Data Factory admite los siguientes formatos de archivo. Consulte los artículos para conocer la configuración basada en el formato.

- [Formato Avro](format-avro.md)
- [Formato binario](format-binary.md)
- [Formato Common Data Model](format-common-data-model.md)
- [Formato de texto delimitado](format-delimited-text.md)
- [Formato Delta](format-delta.md)
- [Formato Excel](format-excel.md)
- [Formato JSON](format-json.md)
- [Formato ORC](format-orc.md)
- [Formato Parquet](format-parquet.md)
- [Formato XML](format-xml.md)

## <a name="next-steps"></a>Pasos siguientes

- [Actividad de copia](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad Lookup](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Eliminar actividad](delete-activity.md)
