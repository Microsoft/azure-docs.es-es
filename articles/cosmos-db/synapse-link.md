---
title: Azure Synapse Link para Azure Cosmos DB, ventajas y cuándo usarlo
description: Más información sobre Azure Synapse Link para Azure Cosmos DB. Synapse Link permite ejecutar análisis casi en tiempo real con Azure Synapse Analytics de datos operativos en Azure Cosmos DB.
author: Rodrigossz
ms.author: rosouz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.reviewer: sngun
ms.openlocfilehash: 9e8dd4b7d7e6f0de1f3ce19238e4ab1e3cf958e1
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88034185"
---
# <a name="what-is-azure-synapse-link-for-azure-cosmos-db-preview"></a>¿Qué es Azure Synapse Link para Azure Cosmos DB (versión preliminar)?

> [!IMPORTANT]
> Azure Synapse Link para Azure Cosmos DB está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Para obtener más información, consulte [Términos de uso complementarios de las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Synapse Link para Azure Cosmos DB es una funcionalidad de procesamiento analítico y transaccional híbrido nativo de nube que permite ejecutar análisis casi en tiempo real de datos operativos en Azure Cosmos DB. Azure Synapse Link crea una integración perfecta y sin contratiempos entre Azure Cosmos DB y Azure Synapse Analytics.

Con el [almacén analítico de Azure Cosmos DB](analytical-store-introduction.md), un almacén de columnas totalmente aislado, Azure Synapse Link no permite ningún análisis de extracción, transformación y carga (ETL) en [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) en los datos operativos a escala. Los analistas de negocios, ingenieros de datos y científicos de datos ahora pueden usar Spark en Synapse o Synapse SQL de forma indistinta para ejecutar canalizaciones de inteligencia empresarial, análisis y aprendizaje automático casi en tiempo real. Puede conseguirlo sin afectar al rendimiento de las cargas de trabajo transaccionales en Azure Cosmos DB. 

En la imagen siguiente se muestra la integración de Azure Synapse Link con Azure Cosmos DB y Azure Synapse Analytics: 

:::image type="content" source="./media/synapse-link/synapse-analytics-cosmos-db-architecture.png" alt-text="Diagrama de la arquitectura de la integración de Azure Synapse Analytics con Azure Cosmos DB" border="false":::

## <a name="benefits"></a><a id="synapse-link-benefits"></a> Ventajas

Para analizar grandes conjuntos de datos operativos a la vez que se reduce el impacto en el rendimiento de las cargas de trabajo transaccionales críticas, normalmente, los datos operativos de Azure Cosmos DB se extraen y procesan mediante las canalizaciones de extracción, transformación y carga (ETL). Las canalizaciones de ETL requieren muchos niveles de movimiento de datos, resulta en una gran complejidad operativa y un impacto en el rendimiento de las cargas de trabajo transaccionales. También aumenta la latencia para analizar los datos operativos desde que se generan.

Cuando se copara con las soluciones tradicionales basadas en ETL, Azure Synapse Link para Azure Cosmos DB ofrece varias ventajas, como:  

### <a name="reduced-complexity-with-no-etl-jobs-to-manage"></a>Complejidad reducida en los trabajos sin ETL que se administrarán

Azure Synapse Link permite acceder directamente al almacén analítico de Azure Cosmos DB con Azure Synapse Analytics sin realizar movimiento de datos complejo. Todas las actualizaciones realizadas en los datos operativos se pueden ver en el almacén analítico casi en tiempo real sin ETL ni trabajos de fuentes de cambios. Puede ejecutar análisis de gran escala en el almacén analítico, desde Synapse Analytics, sin una transformación de datos adicional.

### <a name="near-real-time-insights-into-your-operational-data"></a>Conclusiones casi en tiempo real sobre los datos operativos

Ahora puede obtener conclusiones completas sobre los datos operativos casi en tiempo real, mediante Azure Synapse Link. Los sistemas basados en ETL suelen tener una mayor latencia al analizar los datos operativos, debido a las muchas capas necesarias para extraer, transformar y cargar los datos operativos. Con la integración nativa del almacén analítico de Azure Cosmos DB con Azure Synapse Analytics, puede analizar los datos operativos casi en tiempo real al habilitar nuevos escenarios empresariales. 


### <a name="no-impact-on-operational-workloads"></a>Sin ningún impacto en las cargas de trabajo operativas

Con Azure Synapse Link, puede ejecutar consultas analíticas en un almacén analítico de Azure Cosmos DB (un almacén de columnas independiente) mientras las operaciones transaccionales se procesan con el rendimiento aprovisionado para la carga de trabajo transaccional (un almacén transaccional basado en filas).  La carga de trabajo analítica se proporciona independientemente del tráfico de la carga de trabajo transaccional sin consumir el rendimiento aprovisionado para los datos operativos.

### <a name="optimized-for-large-scale-analytics-workloads"></a>Optimización para las cargas de trabajo de análisis de gran escala

El almacén analítico de Azure Cosmos DB está optimizado para proporcionar escalabilidad, elasticidad y rendimiento para las cargas de trabajo analíticas sin depender de los tiempos de ejecución de proceso. La tecnología de almacenamiento se administra automáticamente para optimizar las cargas de trabajo de análisis. Gracias a la compatibilidad integrada con Azure Synapse Analytics, el acceso a esta capa de almacenamiento ofrece simplicidad y alto rendimiento.

### <a name="cost-effective"></a>Rentable

Con Azure Synapse Link, puede obtener una solución totalmente administrada y optimizada para costos para el análisis operativo. Elimina las capas adicionales de almacenamiento y proceso necesarias en las canalizaciones de ETL tradicionales para analizar los datos operativos. 

El almacén analítico de Azure Cosmos DB sigue un modelo de precios basado en el consumo, en función de las consultas y operaciones de escritura/lectura de análisis y almacenamiento de datos ejecutadas. No requiere que aprovisione ninguna capacidad de proceso, de la forma en que hace actualmente para las cargas de trabajo transaccionales. El acceso a los datos con motores de proceso altamente elásticos desde Azure Synapse Analytics hace que el costo general de ejecutar el almacenamiento y el proceso sea muy eficaz.


### <a name="analytics-for-locally-available-globally-distributed-multi-master-data"></a>Análisis para datos con arquitectura multimaestro, distribuidos globalmente y disponibles de forma local

Puede ejecutar consultas analíticas de forma eficaz en la copia regional más cercana de los datos en Azure Cosmos DB. Azure Cosmos DB proporciona la capacidad de última generación para ejecutar las cargas de trabajo analíticas distribuidas globalmente junto con las cargas de trabajo transaccionales de manera activa /activa.

## <a name="enable-htap-scenarios-for-your-operational-data"></a>Habilitación de escenarios de HTAP para los datos operativos

Synapse Link reúne el almacén analítico de Azure Cosmos DB con la compatibilidad con el tiempo de ejecución de Azure Synapse Analytics. Esta integración le permite crear soluciones HTAP nativas de la nube (procesamiento transaccional y analítico híbrido) que generan conclusiones basadas en actualizaciones en tiempo real de los datos operativos en conjuntos de datos de gran tamaño. Desbloquea nuevos escenarios empresariales para generar alertas basadas en tendencias en vivo, compilar paneles casi en tiempo real y experiencias empresariales basadas en el comportamiento del usuario.

### <a name="azure-cosmos-db-analytical-store"></a>Almacén analítico de Azure Cosmos DB

El almacén analítico de Azure Cosmos DB es una representación orientada a columnas de los datos operativos en Azure Cosmos DB. Este almacén analítico es adecuado para consultas rápidas y rentables en conjuntos de datos operativos de gran tamaño, sin necesidad de copiar los datos ni de afectar al rendimiento de las cargas de trabajo transaccionales.

El almacén analítico selecciona automáticamente las inserciones, actualizaciones y eliminaciones más frecuentes en las cargas de trabajo transaccionales casi en tiempo real, como una funcionalidad totalmente administrada ("sincronización automática") de Azure Cosmos DB. No se requiere ninguna fuente de cambios ni ETL. 

Si tiene una cuenta de Azure Cosmos DB distribuida de forma global, después de habilitar el almacén analítico para un contenedor, estará disponible en todas las regiones de esa cuenta. Para obtener más información sobre el almacén analítico, consulte el artículo [Información general sobre el almacén analítico de Azure Cosmos DB](analytical-store-introduction.md).

### <a name="integration-with-azure-synapse-analytics"></a><a id="synapse-link-integration"></a>Integración con Azure Synapse Analytics

Con Synapse Link, ahora puede conectarse directamente a los contenedores de Azure Cosmos DB desde Azure Synapse Analytics y acceder al almacén analítico sin conectores independientes. Azure Synapse Analytics admite actualmente Synapse Link con [Apache Spark en Synapse](../synapse-analytics/spark/apache-spark-concepts.md) y [Synapse SQL sin servidor](../synapse-analytics/sql/on-demand-workspace-overview.md).

Puede consultar los datos del almacén analítico de Azure Cosmos DB simultáneamente, con interoperabilidad entre los diferentes tiempos de ejecución de análisis que admite Azure Synapse Analytics. No se requieren transformaciones de datos adicionales para analizar los datos operativos. Puede consultar y analizar los datos del almacén analítico mediante:

* Apache Spark en Synapse con compatibilidad completa con Scala, Python, SparkSQL y C#. Spark en Synapse es fundamental para los escenarios de ciencia de datos e ingeniería de datos.

* SQL sin servidor con lenguaje T-SQL y compatibilidad con herramientas de inteligencia empresarial conocidas (por ejemplo, Power BI Premium, etc.).

> [!NOTE]
> Desde Azure Synapse Analytics, puede acceder a almacenes de análisis y transacciones en el contenedor de Azure Cosmos DB. Sin embargo, si quiere ejecutar un análisis o examen a gran escala en los datos operativos, se recomienda que use el almacén analítico para evitar el impacto en el rendimiento de las cargas de trabajo transaccionales.

> [!NOTE]
> Puede ejecutar análisis con baja latencia en una región de Azure al conectar el contenedor de Azure Cosmos DB con el entorno de ejecución de Synapse en esa región.

Esta integración habilita los siguientes escenarios de HTAP para distintos usuarios:

* Un ingeniero de inteligencia empresarial que quiere crear un modelo y publicar un informe para acceder a los datos operativos en Azure Cosmos DB directamente a través de Synapse SQL.

* Un analista de datos que quiere obtener información sobre los datos operativos de un contenedor de Azure Cosmos DB mediante una consulta con Synapse SQL, leer los datos a gran escala y combinar dichos resultados con otros orígenes de datos.

* Un científico de datos que quiere usar Spark en Synapse para buscar una característica para mejorar su modelo y entrenarlo sin realizar ingeniería de datos complejos. También pueden escribir los resultados del modelo después de la inferencia en Azure Cosmos DB para la puntuación en tiempo real de los datos a través de Spark en Synapse.

* Un ingeniero de datos que quiere que los consumidores puedan acceder a los datos mediante la creación de tablas de SQL o Spark en contenedores de Azure Cosmos DB sin realizar procesos de ETL manuales.

Para obtener más información sobre la compatibilidad del entorno de ejecución de Azure Synapse Analytics con Azure Cosmos DB, consulte [Compatibilidad con Azure Synapse Analytics para Cosmos DB](../synapse-analytics/synapse-link/concept-synapse-link-cosmos-db-support.md).

## <a name="when-to-use-azure-synapse-link-for-azure-cosmos-db"></a>Cuándo usar Azure Synapse Link para Azure Cosmos DB

Se recomienda usar Synapse Link en los siguientes casos:

* Si es un cliente de Azure Cosmos DB y quiere ejecutar análisis, inteligencia empresarial y aprendizaje automático con sus datos operativos. En tales casos, Synapse Link proporciona una experiencia de análisis más integrada sin afectar al rendimiento aprovisionado del almacén transaccional. Por ejemplo:

  * Si actualmente está ejecutando análisis o inteligencia empresarial en sus datos operativos de Azure Cosmos DB directamente mediante conectores independientes, o

  * Si está ejecutando procesos de ETL para extraer datos operativos para un sistema de análisis independiente.
 
En tales casos, Synapse Link proporciona una experiencia de análisis más integrada sin afectar al rendimiento aprovisionado del almacén transaccional.

No se recomienda el uso de Synapse Link si busca requisitos de almacenamiento de datos tradicionales, como alta simultaneidad, administración de cargas de trabajo y persistencia de agregados en varios orígenes de datos. Para obtener más información, consulte los [escenarios comunes que pueden aprovechar Azure Synapse Link para Azure Cosmos DB](synapse-link-use-cases.md).


## <a name="limitations"></a>Limitaciones

* Durante la versión preliminar pública, Azure Synapse Link solo es compatible con SQL (Core) API de Azure Cosmos DB. La compatibilidad con la API de Azure Cosmos DB para MongoDB y Cassandra API se encuentra actualmente en versión preliminar validada. Para solicitar acceso a la versión preliminar validada, envíe un correo electrónico al [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

* En este momento, solo se puede habilitar el almacén analítico para los contenedores nuevos (tanto en cuentas nuevas como ya existentes de Azure Cosmos DB).

* En la versión preliminar, para las cuentas de bases de datos habilitadas para Synapse Link, no se admite la copia de seguridad y restauración de contenedores. Si tiene cargas de trabajo de producción que requieren la funcionalidad de copia de seguridad y restauración, se recomienda no habilitar Synapse Link en esas cuentas de bases de datos. 

* El acceso al almacén analítico de Azure Cosmos DB con Synapse SQL sin servidor se encuentra actualmente en versión preliminar validada. Para solicitar acceso, envíe un correo electrónico al [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

* Actualmente no está disponible el acceso al almacén de análisis de Azure Cosmos DB con Synapse SQL aprovisionado.

## <a name="pricing"></a>Precios

El modelo de facturación de Azure Synapse Link incluye los costos en los que se incurre al usar el almacén analítico de Azure Cosmos DB y el entorno de ejecución de Synapse. Para obtener más información, consulte los artículos sobre los [precios del almacén analítico de Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing) y los [precios de Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/synapse-analytics/).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente documentación:

* [Introducción al almacén analítico de Azure Cosmos DB](analytical-store-introduction.md)

* [Introducción a Azure Synapse Link para Azure Cosmos DB](configure-synapse-link.md)
 
* [Características compatibles con el entorno de ejecución de Azure Synapse Analytics](../synapse-analytics/synapse-link/concept-synapse-link-cosmos-db-support.md)

* [Preguntas frecuentes sobre Azure Synapse Link para Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Casos de uso de Azure Synapse Link para Azure Cosmos DB](synapse-link-use-cases.md)
