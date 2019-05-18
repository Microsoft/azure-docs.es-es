---
title: Preguntas frecuentes sobre Azure SQL Data Warehouse | Microsoft Docs
description: En este artículo se muestran las preguntas más frecuentes sobre Azure SQL Data Warehouse para clientes y desarrolladores
services: sql-data-warehouse
author: happynicolle
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: design
ms.date: 04/17/2018
ms.author: nicw
ms.reviewer: igorstan
ms.openlocfilehash: c16d95ea15fc358cb81b17b42570cb35f2e8c52d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795566"
---
# <a name="sql-data-warehouse-frequently-asked-questions"></a>Preguntas más frecuentes de SQL Data Warehouse

## <a name="general"></a>General

P: ¿Qué ofrece SQL Data Warehouse en cuanto a seguridad de datos?

A. SQL Data Warehouse ofrece varias soluciones para proteger datos, como TDE y la auditoría. Para obtener más información, consulte [Seguridad].

P: ¿Dónde puedo encontrar los estándares empresariales o legales que cumple SQL Data Warehouse?

A. Visite la página [Cumplimiento de Microsoft] para consultar diversas ofertas de cumplimiento por producto, como SOC e ISO. Primero, elija por el título Cumplimiento, luego, expanda Azure en la sección de servicios en la nube de Microsoft del lado derecho de la página para ver qué servicios son compatibles con Azure.

P: ¿Puedo conectar Power BI?

A. ¡Sí! Aunque Power BI admite la consulta directa con SQL Data Warehouse, no se ha diseñado para administrar gran cantidad de usuarios o datos en tiempo real. Para usar Power BI con fines de producción, se recomienda usar Power BI sobre IaaS de servicio de análisis o Azure Analysis Services. 

P: ¿Qué son los límites de capacidad de SQL Data Warehouse?

A. Consulte nuestra página de [límites de capacidad] actuales. 

P: ¿Por qué tardan tanto tiempo en completarse las operaciones de escalado, pausar y reanudación?

A. Hay varios factores que puede influir en el tiempo que tardan en completarse las operaciones de administración de procesos. Un caso habitual de operaciones de ejecución prolongada es la reversión de transacciones. Cuando se inicia una operación de escalado o pausa, se bloquean todas las sesiones entrantes y se purgan las consultas. Para dejar el sistema en un estado estable, las transacciones se deben revertir antes de que pueda comenzar una operación. Cuanto mayor sea su número y el tamaño del registro de las transacciones, más tiempo estará detenida la operación al restaurar el sistema a un estado estable.

## <a name="user-support"></a>Asistencia técnica de usuario

P: Tengo una solicitud de característica, ¿dónde puede enviarla?

A. Si tiene una solicitud de característica, envíala a nuestra página [UserVoice].

P: ¿Cómo puedo hacer una determinada acción?

A. Para obtener ayuda con las tareas de desarrollo con SQL Data Warehouse, puede hacer preguntas en nuestra [Stack Overflow]. 

P: ¿Cómo puedo enviar una vale de asistencia técnica?

A. Los [vales de asistencia técnica] puede presentarse a través de Azure Portal.

## <a name="sql-languagefeature-support"></a>Compatibilidad con características o lenguajes de SQL 

P: ¿Qué tipos de datos admite SQL Data Warehouse?

A. Vea los [tipos de datos] de SQL Data Warehouse.

P: ¿Qué características de tablas se admiten?

A. Aunque SQL Data Warehouse es compatible con muchas características, algunas no se admiten y se documentan en la página sobre [Características de tablas no compatibles].

## <a name="tooling-and-administration"></a>Administración y herramientas

P: ¿Se admiten proyectos de bases de datos en Visual Studio?

A. En estos momentos, no admitimos proyectos de bases de datos en Visual Studio para SQL Data Warehouse. Si desea enviar un voto para habilitar esta característica, visite nuestra página de [Solicitud de características de proyectos de bases de datos] de User Voice.

P: ¿SQL Data Warehouse admite las API de REST?

A. Sí. La mayoría de las funciones REST que se pueden utilizar con SQL Database también están disponibles con SQL Data Warehouse. Puede encontrar información sobre las API en las páginas de documentación de REST o [MSDN].


## <a name="loading"></a>Carga

P: ¿Qué controladores de cliente se admiten?

A. La compatibilidad de los controladores con SQL Data Warehouse puede encontrarse en la página [Cadenas de conexión].

P: ¿Qué formatos de archivo admite PolyBase con SQL Data Warehouse?

R: Orc, RC, Parquet y texto delimitado sin formato.

P: ¿Qué puedo conectar desde SQL Data Warehouse con PolyBase? 

R: [Azure Data Lake Store] e instancias de [Blobs de Azure Storage].

P: ¿Se puede usar el cálculo de aplicación al conectarse a instancias de Azure Storage Blob o ADLS? 

R: No, PolyBase de SQL Data Warehouse interactúa solo con los componentes de almacenamiento. 

P: ¿Puedo conectarlo a HDI?

R: HDI puede usar ADLS o WASB como capa HDFS. Si tiene uno de los dos como la capa HDFS, puede cargar datos en SQL Data Warehouse. Sin embargo, no se puede generar el cálculo de aplicación en la instancia HDI. 

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre SQL Data Warehouse, vea nuestra página [Información general].


<!-- Article references -->
[UserVoice]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Cadenas de conexión]: ./sql-data-warehouse-connection-strings.md
[Stack Overflow]: https://stackoverflow.com/questions/tagged/azure-sqldw
[Vales de asistencia técnica]: ./sql-data-warehouse-get-started-create-support-ticket.md
[Seguridad]: ./sql-data-warehouse-overview-manage-security.md
[Cumplimiento de Microsoft]: https://www.microsoft.com/en-us/trustcenter/compliance/complianceofferings
[Límites de capacidad]: ./sql-data-warehouse-service-capacity-limits.md
[tipos de datos]: ./sql-data-warehouse-tables-data-types.md
[Características de tablas no compatibles]: ./sql-data-warehouse-tables-overview.md#unsupported-table-features
[Azure Data Lake Store]: ./sql-data-warehouse-load-from-azure-data-lake-store.md
[Blobs de Azure Storage]: ./sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md
[Solicitud de características de proyectos de bases de datos]: https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/13313247-database-project-from-visual-studio-to-support-azu
[MSDN]: https://msdn.microsoft.com/library/azure/mt163685.aspx
[Información general]: ./sql-data-warehouse-overview-faq.md
