---
title: ¿Qué es el almacén analítico de Azure Cosmos DB (versión preliminar)?
description: Obtenga información sobre el almacén transaccional (basado en filas) y analítico (basado en columnas) de Azure Cosmos DB. Ventajas del almacén analítico, el impacto en el rendimiento de las cargas de trabajo a gran escala y la sincronización automática de datos desde el almacén transaccional al almacén analítico
author: Rodrigossz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: rosouz
ms.openlocfilehash: 3b210ea558f857d017504d07e571e94e34c0d4f6
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037106"
---
# <a name="what-is-azure-cosmos-db-analytical-store-preview"></a>¿Qué es el almacén analítico de Azure Cosmos DB (versión preliminar)?

> [!IMPORTANT]
> El almacén analítico de Azure Cosmos DB se encuentra actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Para obtener más información, consulte [Términos de uso complementarios de las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El almacén analítico de Azure Cosmos DB es un almacén de columnas completamente aislado para habilitar el análisis a gran escala en los datos operativos de su instancia de Azure Cosmos DB, sin que ello afecte a las cargas de trabajo transaccionales.  

## <a name="challenges-with-large-scale-analytics-on-operational-data"></a>Desafíos del análisis a gran escala de los datos operativos

Los datos operativos de varios modelos de un contenedor de Azure Cosmos DB se almacenan internamente en un "almacén transaccional" basado en filas indexado. El formato del almacén de filas está diseñado para permitir lecturas y escrituras transaccionales rápidas, con tiempos de respuesta en el orden de milisegundos, y consultas operativas. Si el conjunto de datos crece en gran medida, las consultas analíticas complejas con los datos almacenados en este formato pueden ser costosas en cuanto al rendimiento aprovisionado. A su vez, el alto consumo del rendimiento aprovisionado afecta al rendimiento de las cargas de trabajo transaccionales que utilizan sus servicios y aplicaciones en tiempo real.

Tradicionalmente, para analizar grandes cantidades de datos, los datos operativos se extraen del almacén transaccional de Azure Cosmos DB y se almacenan en una capa de datos independiente. Por ejemplo, los datos se almacenan en un almacenamiento de datos o en un lago de datos con un formato adecuado. Más tarde, estos datos se usan para análisis a gran escala, y se analizan mediante un motor de proceso, como los clústeres de Apache Spark. Esta separación entre las capas de almacenamiento analítico y de proceso y los datos operativos produce latencia adicional, ya que las canalizaciones ETL (extracción, transformación y carga) se ejecutan con menos frecuencia para minimizar el posible impacto en las cargas de trabajo transaccionales.

Las canalizaciones ETL también se vuelven complejas al administrar las actualizaciones de los datos operativos en comparación con cuando se administran únicamente los datos operativos recién ingeridos. 

## <a name="column-oriented-analytical-store"></a>Almacén analítico orientado a columnas

El almacén analítico de Azure Cosmos DB aborda los desafíos de complejidad y latencia que se presentan con las canalizaciones ETL tradicionales. El almacén analítico de Azure Cosmos DB puede sincronizar automáticamente los datos operativos en un almacén de columnas independiente. El formato del almacén de columnas es adecuado para las consultas analíticas a gran escala que se van a realizar de forma optimizada, lo que aumenta la latencia de dichas consultas.

Con Azure Synapse Link, ahora puede compilar soluciones de HTAP sin ETL mediante la vinculación directa al almacén analítico de Azure Cosmos DB desde Synapse Analytics. Esto le permite ejecutar análisis a gran escala casi en tiempo real con los datos operativos.

## <a name="analytical-store-details"></a>Detalles del almacén analítico

Cuando se habilita el almacén analítico en un contenedor de Azure Cosmos DB, se crea internamente un nuevo almacén de columnas en función de los datos operativos del contenedor. Este almacén de columnas se conserva de forma independiente del almacén transaccional orientado a filas de ese contenedor. Las inserciones, actualizaciones y eliminaciones de los datos operativos se sincronizan automáticamente con el almacén analítico. No necesita la fuente de cambios ni ETL para sincronizar los datos.

### <a name="column-store-for-analytical-workloads-on-operational-data"></a>Almacén de columnas para cargas de trabajo analíticas en datos operativos

En general, las cargas de trabajo analíticas implican agregaciones y exámenes secuenciales de los campos seleccionados. Al almacenar los datos en un orden de columna principal, el almacén analítico permite serializar en conjunto un grupo de valores para cada campo. Este formato reduce las IOPS necesarias para examinar o procesar las estadísticas de los campos específicos. Esto mejora drásticamente los tiempos de respuesta de las consultas para los exámenes en grandes conjuntos de datos. 

Por ejemplo, si las tablas operativas se encuentran en el formato siguiente:

:::image type="content" source="./media/analytical-store-introduction/sample-operational-data-table.png" alt-text="Tabla operativa de ejemplo" border="false":::

El almacén de filas conserva los datos anteriores en un formato serializado, por fila, en el disco. Este formato permite lecturas y escrituras transaccionales y consultas operativas más rápidas, como "Devolver información sobre Product1". Sin embargo, a medida que el conjunto de datos crece en gran medida y si desea ejecutar consultas analíticas complejas con los datos, esto puede ser caro. Por ejemplo, si desea obtener "las tendencias de ventas de un producto en la categoría denominada 'Equipment' en diferentes unidades de negocio y meses", debe ejecutar una consulta compleja. Los análisis de gran tamaño de este conjunto de datos pueden resultar caros en cuanto al rendimiento aprovisionado, además de poder afectar al rendimiento de las cargas de trabajo transaccionales que potencian sus aplicaciones y servicios en tiempo real.

El almacén analítico, que es un almacén de columnas, es más adecuado para estas consultas, ya que serializa en conjunto los campos de datos similares y reduce las IOPS del disco.

En la imagen siguiente se muestra el almacén de filas transaccional frente el almacén de columnas analítico en Azure Cosmos DB:

:::image type="content" source="./media/analytical-store-introduction/transactional-analytical-data-stores.png" alt-text="Almacén de filas transaccional frente al almacén de columnas analítico en Azure Cosmos DB" border="false":::

### <a name="decoupled-performance-for-analytical-workloads"></a>Rendimiento desacoplado para cargas de trabajo analíticas

No hay ningún impacto en el rendimiento de las cargas de trabajo transaccionales debido a las consultas analíticas, ya que el almacén analítico es independiente del almacén transaccional.  El almacén analítico no necesita que se asignen Unidades de solicitud (RU) independientes.

### <a name="auto-sync"></a>Sincronización automática

La sincronización automática hace referencia a la funcionalidad totalmente administrada de Azure Cosmos DB donde las inserciones, actualizaciones y eliminaciones de datos operativos se sincronizan automáticamente del almacén transaccional al almacén analítico casi en tiempo real en un plazo de 5 minutos.

La funcionalidad de sincronización automática, junto con el almacén analítico, proporciona las siguientes ventajas clave:

#### <a name="scalability--elasticity"></a>Escalabilidad y elasticidad

Mediante la creación de partición horizontal, el almacén transaccional de Azure Cosmos DB puede escalar elásticamente el almacenamiento y el rendimiento sin tiempo de inactividad. La creación de partición horizontal en el almacén transaccional proporciona escalabilidad y elasticidad en la sincronización automática para garantizar que los datos se sincronicen con el almacén analítico casi en tiempo real. La sincronización de datos se produce independientemente del rendimiento del tráfico transaccional, ya sean 1000 operaciones/s o 1 millón de operaciones/s, y no afecta al rendimiento aprovisionado en el almacén transaccional. 

#### <a name="automatically-handle-schema-updates"></a><a id="analytical-schema"></a>Control automático de las actualizaciones de esquema

El almacén transaccional de Azure Cosmos DB es independiente del esquema y le permite iterar las aplicaciones transaccionales sin tener que encargarse de la administración de esquemas o índices. A diferencia de esto, el almacén analítico de Azure Cosmos DB está esquematizado para optimizar el rendimiento de las consultas analíticas. Con la funcionalidad de sincronización automática, Azure Cosmos DB administra la inferencia del esquema en las actualizaciones más recientes del almacén transaccional.  También administra la representación del esquema en el almacén analítico de manera integrada, lo que incluye el control de los tipos de datos anidados.

Al haber una evolución del esquema, donde se agregan nuevas propiedades a lo largo del tiempo, el almacén analítico presenta automáticamente un esquema unificado en todos los esquemas históricos del almacén transaccional.

Si todos los datos operativos de Azure Cosmos DB siguen un esquema bien definido para el análisis, el esquema se inferirá automáticamente y se representará correctamente en el almacén analítico. Si ciertos elementos infringen el esquema bien definido para análisis, como se define a continuación, no se incluirán en el almacén analítico. Si tiene escenarios bloqueados debido un esquema bien definido para la definición del análisis, envíe un correo electrónico al [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

Un esquema bien definido para el análisis se define teniendo en cuenta las siguientes consideraciones:

* Una propiedad siempre tiene el mismo tipo entre los varios elementos

  * Por ejemplo, `{"a":123} {"a": "str"}` no tiene un esquema bien definido porque `"a"` a veces es una cadena y, a veces, un número. 
  
    En este caso, el almacén analítico registra el tipo de datos de `“a”` como el tipo de datos de `“a”` en el primer elemento durante la vigencia del contenedor. Los elementos en los que el tipo de datos de `“a”` difiere no se incluirán en el almacén analítico.
  
    Esta condición no se aplica a las propiedades NULL. Por ejemplo, `{"a":123} {"a":null}` sigue estando bien definido.

* Los tipos de matriz deben contener un único tipo repetido.

  * Por ejemplo, `{"a": ["str",12]}` no es un esquema bien definido porque la matriz contiene una mezcla de tipos enteros y de cadena.

* Hay un máximo de 200 propiedades en cualquier nivel de anidamiento de un esquema, y una profundidad de anidamiento máxima de 5.

  * Un elemento con propiedades 201 en el nivel superior no tiene un esquema bien definido.

  * Un elemento con más de cinco niveles anidados en el esquema tampoco tiene un esquema bien definido. Por ejemplo: `{"level1": {"level2":{"level3":{"level4":{"level5":{"too many":12}}}}}}`

* Los nombres de propiedad son únicos cuando se comparan sin distinción de mayúsculas y minúsculas.

  * Por ejemplo, los siguientes elementos no tienen un esquema bien definido `{"Name": "fred"} {"name": "john"}`: `"Name"` y `"name"` son iguales cuando se comparan sin distinguir entre mayúsculas y minúsculas.

### <a name="cost-effective-archival-of-historical-data"></a>Archivado rentable de datos históricos

La organización en capas de datos hace referencia a la separación de los datos entre las infraestructuras de almacenamiento optimizadas para los distintos escenarios. Con lo que se mejora el rendimiento general y la rentabilidad de la pila de datos de un extremo a otro. Con el almacén analítico, Azure Cosmos DB ahora admite la organización automática en capas de datos desde el almacén transaccional hacia el almacén analítico con diferentes diseños de datos. Con el almacén analítico optimizado en cuanto al costo de almacenamiento en comparación con el almacén transaccional, usted puede conservar un horizonte mucho más largo de datos operativos para el análisis histórico.

Una vez que el almacén analítico esté habilitado, en función de las necesidades de retención de datos de las cargas de trabajo transaccionales, puede configurar la propiedad "Período de vida del almacén transaccional (TTL transaccional)" para que los registros se eliminen automáticamente del almacén transaccional después de un período de tiempo determinado. Del mismo modo, el "Período de vida del almacén analítico (TTL analítico)" le permite administrar el ciclo de vida de los datos retenidos en el almacén analítico de manera independiente del almacén transaccional. Al habilitar el almacén analítico y configurar las propiedades TTL, puede establecer un nivel y definir sin problemas el período de retención de los datos para los dos almacenes.

### <a name="global-distribution"></a>Distribución global

Si tiene una cuenta de Azure Cosmos DB distribuida globalmente, después de habilitar el almacén analítico para un contenedor, estará disponible en todas las regiones de dicha cuenta.  Los cambios en los datos operativos se replican globalmente en todas las regiones. Puede ejecutar consultas analíticas de forma eficaz en la copia regional más cercana de los datos en Azure Cosmos DB.

### <a name="security"></a>Seguridad

La autenticación con el almacén analítico es igual que en un almacén transaccional para una base de datos determinada. Puede usar claves maestras o de solo lectura para la autenticación. Puede aprovechar el servicio vinculado en Synapse Studio para evitar pegar las claves de Azure Cosmos DB en los cuadernos de Spark. El acceso a este servicio vinculado está disponible para todos los usuarios que tengan acceso al área de trabajo.

### <a name="support-for-multiple-azure-synapse-analytics-runtimes"></a>Compatibilidad con varios runtimes de Azure Synapse Analytics

El almacén analítico está optimizado para proporcionar escalabilidad, elasticidad y rendimiento para las cargas de trabajo analíticas sin depender de los runtimes de proceso. La tecnología de almacenamiento se administra automáticamente para optimizar las cargas de trabajo analíticas sin esfuerzo manual.

Al desacoplar el sistema de almacenamiento analítico del sistema de proceso analítico, los datos en el almacén analítico de Azure Cosmos DB se pueden consultar simultáneamente desde los distintos runtimes analíticos admitidos por Azure Synapse Analytics. En la actualidad, Synapse Analytics admite Apache Spark y SQL sin servidor con el almacén analítico de Azure Cosmos DB.

> [!NOTE]
> Solo puede leer desde el almacén analítico mediante el runtime de Synapse Analytics. Puede volver a escribir los datos en el almacén transaccional como capa de servicio.

## <a name="pricing"></a><a id="analytical-store-pricing"></a> Precios

El almacén analítico sigue un modelo de precios basado en el consumo, donde se le cobra del modo siguiente:

* Almacenamiento: El volumen de los datos retenidos en el almacén analítico cada mes, incluidos los datos históricos, tal como se define en el TTL analítico.

* Operaciones de escritura analíticas: Sincronización totalmente administrada de las actualizaciones de datos operativos en el almacén analítico desde el almacén transaccional (sincronización automática).

* Operaciones de lectura analíticas: Operaciones de lectura realizadas en el almacén analítico desde Synapse Analytics Spark y los runtimes de SQL sin servidor.

> [!NOTE]
> El almacén analítico de Azure Cosmos DB está disponible en versión preliminar pública sin cargos hasta el 30 de agosto de 2020.

Los precios del almacén analítico son independientes del modelo de precios del almacén transaccional. No hay ningún concepto de RU aprovisionadas en el almacén analítico. Consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener información completa sobre el modelo de precios del almacén analítico.

Para obtener una estimación general del costo de habilitación del almacén analítico en un contenedor de Azure Cosmos DB, puede usar el [planificador de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) y obtener una estimación de los costos de almacenamiento analítico y de las operaciones de escritura. Los costos de las operaciones de lectura analíticas dependen de las características de la carga de trabajo analítica, pero como estimación general, el análisis de 1 TB de datos en el almacén analítico suele generar 130 000 operaciones de lectura analíticas, con un costo de 0,065 USD.

## <a name="analytical-time-to-live-ttl"></a><a id="analytical-ttl"></a> Período de vida (TTL) analítico

El TTL analítico indica cuánto tiempo deben conservarse los datos en el almacén analítico para un contenedor dado. 

Las inserciones, actualizaciones y eliminaciones de datos operativos se sincronizan automáticamente desde el almacén transaccional hacia el almacén analítico, independientemente de la configuración del TTL transaccional. La retención de estos datos operativos en el almacén analítico puede controlarse mediante el valor de TTL analítico en el nivel de contenedor, tal como se especifica a continuación:

El TTL analítico en un contenedor se establece mediante la propiedad `AnalyticalStoreTimeToLiveInSeconds`:

* Si el valor se establece en "0", falta (o se establece en NULL): el almacén analítico se deshabilita y no se replica ningún dato del almacén transaccional al almacén analítico.

* Si está presente y el valor se establece en "-1": el almacén analítico conserva todos los datos históricos, independientemente de la retención de los datos en el almacén transaccional. Este valor indica que el almacén analítico tiene una retención infinita de los datos operativos.

* Si está presente y el valor se establece en algún número positivo "n": los elementos expirarán del almacén analítico "n" segundos después de la hora de la última modificación en el almacén transaccional. Este valor se puede aprovechar si desea conservar los datos operativos durante un período de tiempo limitado en el almacén analítico, independientemente de la retención de los datos en el almacén transaccional.

Algunos puntos que se deben tener en cuenta:
*   Después de habilitar el almacén analítico con un valor de TTL analítico, puede actualizarse a un valor válido diferente más tarde. 
*   Aunque el TTL transaccional se puede establecer a nivel de contenedor o de elemento, actualmente el TTL analítico solo se puede establecer a nivel de contenedor.
*   Puede lograr una retención más prolongada de los datos operativos en el almacén analítico si establece un TTL analítico >= TTL transaccional a nivel de contenedor.
*   Se puede hacer que el almacén analítico refleje el almacén transaccional si se establece un TTL analítico = TTL transaccional.

Para obtener más información, consulte [Configuración del TTL analítico en un contenedor](configure-synapse-link.md#create-analytical-ttl).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los documentos siguientes:

* [Azure Synapse Link para Azure Cosmos DB](synapse-link.md)

* [Introducción a Azure Synapse Link para Azure Cosmos DB](configure-synapse-link.md)

* [Preguntas frecuentes sobre Synapse Link para Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Casos de uso de Azure Synapse Link para Azure Cosmos DB](synapse-link-use-cases.md)
