---
title: Casos de uso de análisis en tiempo real con Azure Synapse Link para Azure Cosmos DB
description: Obtenga información sobre cómo se usa Azure Synapse Link para Azure Cosmos DB en los análisis de la cadena de suministro, así como sus previsiones, informes, personalización en tiempo real y mantenimiento predictivo de IoT.
author: tknandu
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: ramkris
ms.openlocfilehash: d6039ac3bead535fd9c7de64a8e92a0d7a58f390
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319243"
---
# <a name="azure-synapse-link-for-azure-cosmos-db-near-real-time-analytics-use-cases"></a>Azure Synapse Link para Azure Cosmos DB: Casos de uso del análisis casi en tiempo real

[Azure Synapse Link](synapse-link.md) para Azure Cosmos DB es una capacidad de procesamiento analítico y transaccional híbrido nativo de nube (HTAP) que permite ejecutar análisis casi en tiempo real de datos operativos. Synapse Link crea una integración perfecta y sin contratiempos entre Azure Cosmos DB y Azure Synapse Analytics.

Quizá tenga curiosidad por comprender qué casos de uso del sector pueden aprovechar esta funcionalidad de HTAP nativa de nube para realizar análisis casi en tiempo real de datos operativos. Estos son tres casos de uso comunes de Azure Synapse Link para Azure Cosmos DB:

* Análisis, previsiones e informes de la cadena de suministro
* Personalización en tiempo real
* Mantenimiento predictivo y detección de anomalías en escenarios de IoT

> [!NOTE]
> Azure Synapse Link para Azure Cosmos DB tiene como destino el escenario en el que los equipos de la empresa buscan ejecutar análisis casi en tiempo real. Estos análisis se ejecutan sin ETL de los datos operativos generados en todas las aplicaciones transaccionales creadas en Azure Cosmos DB. Sin embargo, no reemplaza la necesidad de un almacén de datos independiente cuando existen requisitos de almacenamiento de datos tradicionales, como la administración de cargas de trabajo, una alta simultaneidad y la persistencia de agregados en varios orígenes de datos.

## <a name="supply-chain-analytics-forecasting--reporting"></a>Análisis, previsiones e informes de la cadena de suministro

Los estudios de investigación demuestran que integrar análisis de macrodatos en las operaciones de la cadena de suministro conducen a mejoras en los tiempos de entrega del ciclo de pedidos y la eficacia de la cadena de suministro.

Los fabricantes están incorporando tecnologías nativas de la nube para liberarse de las restricciones de los sistemas heredados de planeación de recursos empresariales (ERP) y de administración de cadenas de suministro (SCM). Con cadenas de suministro que generan volúmenes cada vez mayores de datos operativos cada minuto (pedidos, envíos, datos de transacciones), los fabricantes necesitan una base de datos operativa. Esta base de datos operativa se debe escalar para controlar los volúmenes de datos, así como una plataforma analítica para lograr un nivel de inteligencia contextual en tiempo real para mantenerse a la vanguardia.

La siguiente arquitectura muestra la eficacia de usar Azure Cosmos DB como base de datos operativa nativa de nube y Synapse Link en el análisis de la cadena de suministro:

:::image type="content" source="./media/synapse-link-use-cases/supply-chain-analytics.png" alt-text="Azure Synapse Link para Azure Cosmos DB en el análisis de la cadena de suministro" border="false":::

Si se basa en la arquitectura anterior, puede lograr los siguientes casos de uso con Synapse Link para Azure Cosmos DB:

* **Preparación y entrenamiento de una canalización predictiva:** genere conclusiones sobre los datos operativos de toda la cadena de suministro mediante el aprendizaje automático. De esta manera, puede reducir los costos de inventario y operaciones, así como reducir los tiempos de entrega de pedidos para los clientes.

  Synapse Link le permite analizar los datos operativos variables en Azure Cosmos DB sin ningún proceso ETL manual. Le ahorra costos, latencia y complejidad operativa adicionales. Synapse Link permite a los ingenieros y científicos de datos crear canalizaciones predictivas sólidas:

  * Aproveche la integración nativa con los grupos de Apache Spark en Synapse Analytics para consultar los datos operativos del almacén analítico de Azure Cosmos DB. Puede consultar los datos en un cuaderno interactivo o en trabajos remotos programados sin necesidad de ingeniería de datos compleja.

  * Cree modelos de Machine Learning (ML) con los algoritmos de aprendizaje automático de Spark y la integración de Azure Machine Learning en Azure Synapse Analytics.

  * Reescriba los resultados después de la inferencia del modelo en Azure Cosmos DB para calcular una puntuación operativa casi en tiempo real.

* **Informes operativos:** los equipos de la cadena de suministro necesitan informes flexibles y personalizados sobre datos operativos precisos en tiempo real. Estos informes son necesarios para obtener una vista de instantánea de la eficacia, rentabilidad y productividad de la cadena de suministro. Permite que los analistas de datos y a otras partes interesadas evalúen constantemente el negocio e identifiquen las áreas que se van a ajustar para reducir los costos operativos. 

  Synapse Link para Azure Cosmos DB habilita escenarios de creación de informes/inteligencia empresarial enriquecidos:

  * Use la integración nativa con Synapse SQL sin servidor y la expresividad completa del lenguaje T-SQL para consultar los datos operativos del almacén analítico de Azure Cosmos DB.

  * Cree modelos y publique paneles de inteligencia empresarial que se actualizan automáticamente a través de Azure Cosmos DB gracias a la compatibilidad de Synapse SQL sin servidor con las herramientas de inteligencia empresarial que ya conoce. Por ejemplo, Azure Analysis Services, Power BI Premium, etc.

A continuación se incluyen algunas instrucciones para la integración de los datos por lotes y la transmisión de datos en Azure Cosmos DB:

* **Orquestación e integración de datos por lotes:** dado que las cadenas de suministro son cada vez más complejas, las plataformas de datos de cadena de suministro deben integrarse con diversos formatos y orígenes de datos. Azure Synapse incluye el mismo motor de integración de datos y experiencias que Azure Data Factory. Esta integración permite a los ingenieros de datos crear canalizaciones de datos enriquecidas sin un motor de orquestación independiente:

  * Migre datos de más de 85 orígenes de datos compatibles a [Azure Cosmos DB con Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).

  * Escriba canalizaciones de ETL sin código en Azure Cosmos DB, incluidas las [asignaciones relacional-jerárquica y jerárquica-jerárquica con flujos de datos de asignación](../data-factory/how-to-sqldb-to-cosmosdb.md).

* **Procesamiento e integración de la transmisión de datos:** dado el crecimiento de IoT industrial (sensores que realizan un seguimiento de los recursos "de la planta a la tienda", flotas de logística conectadas, etc.), se genera una gran cantidad de datos en tiempo real en un flujo que debe integrarse con los datos de movimiento lento tradicionales para generar conclusiones. Azure Stream Analytics es un servicio recomendado para ETL y procesamiento del flujo de datos en Azure [con una amplia variedad de escenarios](../stream-analytics/streaming-technologies.md). Azure Stream Analytics admite [Azure Cosmos DB como receptor de datos nativo](../stream-analytics/stream-analytics-documentdb-output.md).

## <a name="real-time-personalization"></a>Personalización en tiempo real

Actualmente, los distribuidores deben crear soluciones de comercio electrónico seguras y escalables que satisfagan tanto las exigencias de los clientes como las de las empresas. Dichas soluciones de comercio electrónico deben atraer a los clientes con productos y ofertas personalizados, procesar las transacciones de forma rápida y segura, y centrarse en el cumplimiento y el servicio de atención al cliente. Azure Cosmos DB junto con la versión más reciente de Synapse Link para Azure Cosmos DB permite a los distribuidores generar recomendaciones personalizadas para los clientes en tiempo real. Usan la configuración de baja latencia y coherencia ajustable para obtener conclusiones inmediatas, tal como se muestra en la siguiente arquitectura:

:::image type="content" source="./media/synapse-link-use-cases/real-time-personalization.png" alt-text="Azure Synapse Link para Azure Cosmos DB en la personalización en tiempo real" border="false":::

Caso de uso de Synapse Link para Azure Cosmos DB:

* **Preparación y entrenamiento de una canalización predictiva:** puede generar conclusiones sobre los datos operativos en sus unidades de negocio o en los segmentos de clientes mediante Spark en Synapse y modelos de aprendizaje automático. Esto genera una entrega personalizada a segmentos de clientes, experiencias de usuario final predictivas y marketing dirigido para satisfacer las necesidades de los usuarios finales.

## <a name="iot-predictive-maintenance"></a>Mantenimiento predictivo de IoT

Las innovaciones en IoT industrial han reducido drásticamente los tiempos de inactividad de la maquinaria y han aumentado la eficacia general en todos los campos del sector. Una de estas innovaciones es el análisis de mantenimiento predictivo para maquinaria en el perímetro de la nube.

La siguiente es una arquitectura que aprovecha las funcionalidades de HTAP nativas de nube de Azure Synapse Link para Azure Cosmos DB en el mantenimiento predictivo de IoT:

:::image type="content" source="./media/synapse-link-use-cases/iot-predictive-maintenance.png" alt-text="Azure Synapse Link para Azure Cosmos DB en el mantenimiento predictivo de IoT" border="false" :::

Casos de uso de Synapse Link para Azure Cosmos DB:

* **Preparación y entrenamiento de una canalización predictiva:** los datos operativos históricos de los sensores de dispositivos IoT se pueden usar para entrenar modelos predictivos, como los detectores de anomalías. Estos detectores de anomalías se vuelven a implementar en el perímetro para la supervisión en tiempo real. Este círculo virtuoso permite volver a entrenar de forma continua los modelos predictivos.

* **Informes operativos:** gracias al crecimiento de las iniciativas de Digital Twins, las empresas recopilan grandes cantidades de datos operativos de un gran número de sensores para crear una copia digital de cada equipo. Estos datos alimentan a la inteligencia empresarial, que debe comprender las tendencias de los datos históricos, además de las aplicaciones en tiempo real de los datos de acceso frecuente.

## <a name="sample-scenario-htap-for-azure-cosmos-db"></a>Escenario de ejemplo: HTAP para Azure Cosmos DB

Durante casi una década, miles de clientes han usado Azure Cosmos DB para aplicaciones críticas que requieren un escalado flexible, distribución global inmediata y replicación de la arquitectura multimaestro para ofrecer baja latencia y alta disponibilidad en las operaciones de lectura y escritura de las cargas transaccionales.
 
En la lista siguiente se muestra información general de los distintos patrones de carga de trabajo que son compatibles con los datos operativos al usar Azure Cosmos DB:

* Aplicaciones y servicios en tiempo real
* Procesamiento de flujo de eventos
* Paneles de inteligencia empresarial
* Análisis de macrodatos
* Machine Learning

Gracias a Azure Synapse Link, Azure Cosmos DB no solo puede alimentar las cargas de trabajo transaccionales, sino que también puede realizar cargas de trabajo analíticas casi en tiempo real de datos operativos históricos. Esto es posible sin requisitos de ETL y aislamiento de rendimiento garantizado de las cargas de trabajo transaccionales.

En la imagen siguiente se muestran los patrones de carga de trabajo mediante Azure Cosmos DB: :::image type="content" source="./media/synapse-link-use-cases/synapse-link-workload-patterns.png" alt-text="Patrones de carga de trabajo de Azure Synapse Link para Azure Cosmos DB" border="false":::

Usaremos el ejemplo de la empresa de comercio electrónico CompanyXYZ con operaciones globales en 20 países o regiones para ilustrar las ventajas de elegir Azure Cosmos DB como base de datos única en tiempo real, que cumple tanto los requisitos transaccionales como los de análisis de una plataforma de administración de inventario.

* Las operaciones principales de CompanyXYZ dependen del sistema de administración de inventario; por lo tanto, la disponibilidad y confiabilidad son los requisitos fundamentales. Ventajas de usar Azure Cosmos DB:

  * En virtud de la integración profunda con la infraestructura de Azure y la replicación transparente de la arquitectura multimaestro de forma global, Azure Cosmos DB proporciona una [alta disponibilidad del 99,999 %](high-availability.md) líder en el sector frente a interrupciones regionales.

* Quizá los asociados de la cadena de suministro de CompanyXYZ se encuentren en ubicaciones geográficas distintas, pero necesitan obtener una sola vista del inventario de productos en todo el mundo para respaldar sus operaciones locales. Esto incluye la necesidad de leer las actualizaciones realizadas por otros asociados de la cadena de suministro en tiempo real. Además de realizar actualizaciones sin preocuparse por los conflictos con otros asociados durante momentos de alto rendimiento. Ventajas de usar Azure Cosmos DB:

  * Con el protocolo único de replicación de arquitectura multimaestro y un almacén transaccional optimizado para escritura y sin bloqueos, Azure Cosmos DB garantiza latencias inferiores a 10 ms para las operaciones de lectura y escritura indexadas en el percentil 99 a nivel mundial.

  * Ingesta de alto rendimiento para las fuentes de datos por lotes y de transmisión con [indexación en tiempo real](index-policy.md) en el almacén transaccional.

  * El almacén transaccional de Azure Cosmos DB proporciona tres opciones adicionales a los dos extremos de nivel de coherencia alta y ocasional para lograr las [compensaciones de disponibilidad frente a rendimiento](consistency-levels-tradeoffs.md) más cercanas a las necesidades de la empresa.

* Los asociados de la cadena de suministro de CompanyXYZ tienen patrones de tráfico que oscilan constantemente entre cientos y millones de solicitudes por segundo y, por lo tanto, la plataforma de administración de inventario debe tratar con picos inesperados de tráfico.  Ventajas de usar Azure Cosmos DB:

  * El almacén transaccional de Azure Cosmos DB admite la escalabilidad flexible del almacenamiento y el rendimiento mediante la creación de partición horizontal. Los contenedores y las bases de datos configurados en el modo Autopilot pueden escalar de forma automática e instantánea la capacidad de proceso aprovisionada en función de las necesidades de la aplicación sin que ello afecte a la disponibilidad, la latencia, la capacidad de proceso o el rendimiento de la carga de manera global.

* CompanyXYZ debe establecer una plataforma de análisis seguro para hospedar los datos de inventario históricos para todo el sistema, con el fin de habilitar los análisis y las conclusiones para todos los asociados de la cadena de suministro, las unidades de negocio y las funciones. La plataforma de análisis necesita habilitar la colaboración en todo el sistema, los casos de uso de informes/inteligencia empresarial tradicionales, los casos de uso de análisis avanzado y las soluciones de inteligencia predictiva en función de los datos de inventario operativo. Ventajas de usar Synapse Link para Azure Cosmos DB:

  * Mediante el [almacén analítico de Azure Cosmos DB](analytical-store-introduction.md), un almacén de columnas totalmente aislado, Synapse Link no permite ningún análisis de extracción, transformación y carga (ETL) en [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) en los datos operativos a escala distribuidos de forma global.  Los analistas de negocios, los ingenieros de datos y los científicos de datos ahora pueden usar Spark en Synapse o Synapse SQL de forma interoperable para ejecutar canalizaciones de inteligencia empresarial, análisis y aprendizaje automático casi en tiempo real sin afectar al rendimiento de sus cargas de trabajo transaccionales en Azure Cosmos DB. Consulte las [ventajas de Synapse Link en Azure Cosmos DB](synapse-link.md) para obtener más información.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los documentos siguientes:

* [Azure Synapse Link para Azure Cosmos DB](synapse-link.md) 

* [Almacén analítico de Azure Cosmos DB](analytical-store-introduction.md)

* [Trabajo con Azure Synapse Link para Azure Cosmos DB](configure-synapse-link.md)

* [Preguntas frecuentes sobre Azure Synapse Link para Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Apache Spark en Azure Synapse Analytics](../synapse-analytics/spark/apache-spark-concepts.md)

* [Compatibilidad del entorno de ejecución sin servidor de SQL con Azure Synapse Analytics](../synapse-analytics/sql/on-demand-workspace-overview.md)
