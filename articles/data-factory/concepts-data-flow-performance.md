---
title: Guía de optimización y rendimiento del flujo de datos de asignación
description: Conozca los factores clave que afectan al rendimiento de los flujos de datos de asignación en Azure Data Factory.
author: kromerm
ms.topic: conceptual
ms.author: makromer
ms.service: data-factory
ms.custom: seo-lt-2019
ms.date: 07/27/2020
ms.openlocfilehash: 55483b93b770687703b381366d48edbc7d48f26e
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475345"
---
# <a name="mapping-data-flows-performance-and-tuning-guide"></a>Guía de optimización y rendimiento de la asignación de instancias de Data Flow

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Los flujos de datos de asignación de Azure Data Factory proporcionan una interfaz sin código para diseñar y ejecutar transformaciones de datos a gran escala. Si no está familiarizado con los flujos de datos de asignación, consulte [Introducción a Mapping Data Flow](concepts-data-flow-overview.md). En este artículo se destacan varias maneras de ajustar y optimizar los flujos de datos para que cumplan con los parámetros deseados de rendimiento.

En el siguiente vídeo se muestran algunos intervalos de ejemplo que transforman datos con flujos de datos.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4rNxM]

## <a name="testing-data-flow-logic"></a>Prueba de la lógica de flujo de datos

Cuando se diseñan y se prueban flujos de datos desde la experiencia del usuario de ADF, el modo de depuración permite realizar pruebas interactivas en un clúster de Spark activo. De este modo se puede obtener una vista previa de los datos y ejecutar los flujos de datos sin esperar a que se prepare un clúster. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

## <a name="monitoring-data-flow-performance"></a>Supervisión del rendimiento de flujo de datos

Después de comprobar la lógica de transformación mediante el modo de depuración, puede ejecutar el flujo de datos de un extremo a otro como una actividad en una canalización. Los flujos de datos se convierten en operaciones de una canalización mediante la [actividad de ejecución de flujo de datos](control-flow-execute-data-flow-activity.md). Esta actividad tiene una experiencia de supervisión única en comparación con otras actividades de Azure Data Factory que muestran un plan de ejecución detallado y un perfil de rendimiento de la lógica de transformación. Para ver información detallada de la supervisión de un flujo de datos, haga clic en el icono de las gafas en la salida de la ejecución de actividad de una canalización. Para más información, consulte [Supervisión de los flujos de datos de asignación](concepts-data-flow-monitoring.md).

![Supervisión de Data Flow](media/data-flow/monitoring-details.png "Supervisión de Data Flow 2")

Cuando se supervisa el rendimiento del flujo de datos, hay cuatro posibles cuellos de botella que se deben considerar:

* Tiempo para el inicio de actividad del clúster
* Lectura desde un origen
* Tiempo de transformación
* Escritura en un receptor 

![Supervisión de Data Flow](media/data-flow/monitoring-performance.png "Supervisión de Data Flow 3")

El tiempo para el inicio de actividad del clúster es el tiempo que se tarda en poner en marcha un clúster de Apache Spark. Este valor se encuentra en la esquina superior derecha de la pantalla de supervisión. Los flujos de datos se ejecutan en un modelo Just-in-Time en el que cada trabajo usa un clúster aislado. Por lo general, el inicio de actividad tarda de tres a cinco minutos. En el caso de los trabajos secuenciales, este tiempo se puede reducir si se habilita un valor de período de vida. Para más información, consulte [Optimización de Azure Integration Runtime](#ir).

Los flujos de datos usan un optimizador de Spark que reordena y ejecuta la lógica de negocios en "fases" para actuar lo más rápido posible. La salida de la supervisión muestra la duración de cada fase de transformación de cada uno de los receptores en los que escribe el flujo de datos, junto con el tiempo que se tarda en escribir los datos en el receptor. El mayor valor de tiempo es probablemente el cuello de botella del flujo de datos. Si la fase de transformación que más tarda contiene un origen, puede que le interese estudiar cómo mejorar la optimización del tiempo de lectura. Si una transformación tarda mucho tiempo, puede que tenga que volver a crear particiones o aumentar el tamaño del entorno de ejecución de integración. Si el tiempo de procesamiento del receptor es elevado, puede que necesite escalar verticalmente la base de datos o comprobar que no se está generando la salida en un único archivo.

Cuando haya identificado el cuello de botella del flujo de datos, use las siguientes estrategias de optimización para mejorar el rendimiento.

## <a name="optimize-tab"></a>Pestaña Optimizar

La pestaña **Optimize** (Optimizar) contiene valores para configurar el esquema de partición del clúster de Spark. Esta pestaña existe en cada transformación de flujo de datos y especifica si se desea volver a crear particiones de los datos **después** de que se haya completado la transformación. Ajustar las particiones permite controlar la distribución de los datos entre los nodos de proceso y las optimizaciones de la localidad de los datos, lo que puede tener efectos tanto positivos como negativos en el rendimiento general de los flujos de datos.

![Optimize](media/data-flow/optimize.png "Optimización") (Optimizar)

De forma predeterminada, se selecciona *Use current partitioning* (Usar particiones actuales), que indica a Azure Data Factory que mantenga las particiones de salida actuales de la transformación. Dado que una nueva partición de datos lleva su tiempo, en la mayoría de los escenarios se recomienda la opción *Use current partitioning* (Usar particiones actuales). Entre los escenarios en los que es posible que desee volver a crear particiones de los datos se incluyen los que surgen tras agregados y combinaciones que sesgan significativamente los datos o cuando se usa la creación de particiones de origen en una base de datos SQL.

Para cambiar las particiones de cualquier transformación, seleccione la pestaña **Optimizar** y elija el botón de radio **Set Partitioning** (Establecer particiones). Aparecen una serie de opciones relacionadas con la creación de particiones. El mejor método para crear particiones depende de los volúmenes de datos, las claves candidatas, los valores nulos y la cardinalidad. 

> [!IMPORTANT]
> La opción Single partition (Partición única) combina todos los datos distribuidos en una sola partición. Se trata de una operación muy lenta que también afecta significativamente a todas las operaciones de transformación y escritura de nivel inferior. Azure Data Factory recomienda no usar esta opción a menos que haya una razón empresarial explícita para hacerlo.

Las siguientes opciones de creación de particiones están disponibles en todas las transformaciones:

### <a name="round-robin"></a>Round robin 

Round robin distribuye los datos equitativamente entre las particiones. Use round robin cuando los candidatos principales no sean lo suficientemente buenos como para implementar una estrategia de creación de particiones sólida e inteligente. Puede establecer el número de particiones físicas.

### <a name="hash"></a>Hash

Azure Data Factory genera un valor hash de las columnas para generar particiones uniformes, por ejemplo, filas con valores similares en la misma partición. Si usa la opción Hash, pruebe si hay una posible distorsión de las particiones. Puede establecer el número de particiones físicas.

### <a name="dynamic-range"></a>Intervalo dinámico

El intervalo dinámico usa los intervalos dinámicos de Spark en función de las columnas o expresiones que proporcione. Puede establecer el número de particiones físicas. 

### <a name="fixed-range"></a>Intervalo fijo

Cree una expresión que proporcione un intervalo de valores fijo en las columnas de datos con particiones. Para evitar la distorsión de la partición, es preciso que conozca perfectamente los datos antes de usar esta opción. Los valores que especifique para la expresión se usan como parte de una función de partición. Puede establecer el número de particiones físicas.

### <a name="key"></a>Clave

Si conoce bien la cardinalidad de los datos, la creación de particiones clave puede ser una buena estrategia. La creación de particiones clave crea particiones para cada valor único de la columna. No puede establecer el número de particiones porque dicho número depende de valores únicos de los datos.

> [!TIP]
> Al establecer manualmente el esquema de partición, los datos se reordenan y se pueden perder las ventajas del optimizador de Spark. Como procedimiento recomendado, conviene no establecer manualmente las particiones a menos que sea necesario hacerlo.

## <a name="optimizing-the-azure-integration-runtime"></a><a name="ir"></a> Optimización de Azure Integration Runtime

Los flujos de datos se ejecutan en clústeres de Spark que se activan en tiempo de ejecución. La configuración del clúster que se usa se define en el entorno de ejecución de integración (IR) de la actividad. Hay tres consideraciones de rendimiento que se deben tener en cuenta a la hora de definir el entorno de ejecución de integración: el tipo de clúster, el tamaño del clúster y el período de vida.

Para más información sobre cómo crear un entorno de Integration Runtime, consulte [Integration Runtime en Azure Data Factory](concepts-integration-runtime.md).

### <a name="cluster-type"></a>Tipo de clúster

Hay tres opciones disponibles para la activación del tipo de clúster de Spark: uso general, optimizado para memoria y optimizado para proceso.

Los clústeres de **uso general** son la selección predeterminada y resultan la opción idónea en la mayoría de las cargas de trabajo de flujo de datos. Suelen ofrecer el mejor equilibrio entre rendimiento y costo.

Si el flujo de datos tiene muchas combinaciones y búsquedas, puede que desee usar un clúster **optimizado para memoria**. Los clústeres optimizados para memoria pueden almacenar más datos en memoria y reducirán los errores de memoria insuficiente que puedan aparecer. Son los más caros por núcleo, pero también tienden a generar canalizaciones más correctas. Si aparecen errores de memoria insuficiente al ejecutar los flujos de datos, cambie a una configuración de Azure IR optimizada para memoria. 

Los clústeres **optimizados para proceso** no son idóneos para flujos de trabajo de ETL y el equipo de Azure Data Factory no los recomienda para la mayoría de las cargas de trabajo de producción. En el caso de transformaciones de datos más sencillas, sin un uso intensivo de memoria, como el filtrado de datos o la incorporación de columnas derivadas, se pueden usar los clústeres optimizados para proceso, con un precio más barato por núcleo.

### <a name="cluster-size"></a>Tamaño del clúster

Los flujos de datos distribuyen el procesamiento de datos en diferentes nodos de un clúster de Spark para realizar operaciones en paralelo. Un clúster de Spark con más núcleos aumenta el número de nodos en el entorno de proceso. Más nodos aumentan la capacidad de procesamiento del flujo de datos. El aumento del tamaño del clúster suele ser una forma sencilla de reducir el tiempo de procesamiento.

El tamaño de clúster predeterminado es de cuatro nodos de controlador y cuatro nodos de trabajo.  A medida que se procesan más datos, se recomiendan clústeres más grandes. A continuación se muestran las opciones de tamaño posibles:

| Núcleos de trabajo | Núcleos de controlador | Núcleos totales | Notas |
| ------------ | ------------ | ----------- | ----- |
| 4 | 4 | 8 | No disponible en la opción optimizada para proceso |
| 8 | 8 | 16 | |
| 16 | 16 | 32 | |
| 32 | 16 | 48 | |
| 64 | 16 | 80 | |
| 128 | 16 | 144 | |
| 256 | 16 | 272 | |

El precio de los flujos de datos se calcula en horas de núcleo virtual, por lo que se tiene en cuenta tanto el tamaño del clúster como el tiempo de ejecución. A medida que escale verticalmente, aumentará el costo por minuto del clúster, pero se reducirá el tiempo total.

> [!TIP]
> Existe un punto máximo en el que el tamaño de un clúster deja de afectar al rendimiento de un flujo de datos. En función del tamaño de los datos, hay un punto en el que el aumento del tamaño de un clúster dejará de mejorar el rendimiento. Por ejemplo, si tiene más nodos que particiones de datos, agregar más nodos no ayudará más. Un procedimiento recomendado es comenzar con un tamaño pequeño y escalar verticalmente hasta satisfacer las necesidades de rendimiento. 

### <a name="time-to-live"></a>Período de vida

De forma predeterminada, cada actividad de flujo de datos pone en marcha un nuevo clúster en función de la configuración de IR. El clúster tarda unos minutos en iniciar su actividad y el procesamiento de datos no puede iniciarse hasta que se complete. Si las canalizaciones contienen varios **flujos de datos** secuenciales, puede habilitar un valor de período de vida (TTL). Si se especifica un valor de período de vida, el clúster se mantiene activo durante un determinado período de tiempo después de que se complete la ejecución. Si un nuevo trabajo comienza a usar el IR durante el tiempo de TTL, volverá a usar el clúster existente y el tiempo de inicio de actividad será de segundos en lugar de minutos. Una vez completado el segundo trabajo, el clúster permanecerá activo de nuevo durante el tiempo de TTL.

Solo se puede ejecutar un trabajo en un único clúster cada vez. Si hay un clúster disponible, pero se inician dos flujos de datos, solo uno usará el clúster activo. El segundo trabajo pondrá en marcha su propio clúster aislado.

Si la mayoría de los flujos de datos se ejecutan en paralelo, no se recomienda habilitar el TTL. 

> [!NOTE]
> El período de vida no está disponible cuando se usa el entorno de ejecución de integración de resolución automática.

## <a name="optimizing-sources"></a>Optimización de orígenes

En todos los orígenes, excepto Azure SQL Database, se recomienda mantener **Use current partitioning**  (Usar particiones actuales) como valor seleccionado. Cuando leen desde el resto de los sistemas de origen, los flujos de datos crean automáticamente particiones de los datos en función de su tamaño. Se crea una nueva partición aproximadamente por cada 128 MB de datos. A medida que aumenta el tamaño de los datos, aumenta el número de particiones.

Las particiones personalizadas tienen lugar *después* de que Spark lee los datos y afectan negativamente al rendimiento del flujo de datos. Puesto que los datos se dividen en particiones uniformes durante la lectura, esta opción no se recomienda. 

> [!NOTE]
> Las velocidades de lectura pueden verse limitadas por la capacidad de proceso del sistema de origen.

### <a name="azure-sql-database-sources"></a>Orígenes de Azure SQL Database

Azure SQL Database tiene una opción única de creación de particiones denominada creación de particiones "de origen". Si se habilita esta opción, pueden mejorar los tiempos de lectura de Azure SQL DB, ya que se habilitan conexiones paralelas en el sistema de origen. Se especifica el número de particiones y cómo se crean particiones de los datos. Se usa una columna de partición con alta cardinalidad. También se puede escribir una consulta que coincida con el esquema de partición de la tabla de origen.

> [!TIP]
> En la creación de particiones de origen, la E/S de SQL Server es el cuello de botella. Si se agregan demasiadas particiones, se puede saturar la base de datos de origen. Normalmente, cuando se usa esta opción el número ideal de particiones es de cuatro o cinco.

![Creación de particiones de origen](media/data-flow/sourcepart3.png "Creación de particiones de origen")

#### <a name="isolation-level"></a>Nivel de aislamiento

El nivel de aislamiento de la lectura en un sistema de origen de Azure SQL afecta al rendimiento. La opción "Read uncommitted" (Lectura no confirmada) proporcionará el rendimiento más rápido y evitará bloqueos de base de datos. Para obtener más información acerca de los niveles de aislamiento de SQL, consulte [Descripción de los niveles de aislamiento](https://docs.microsoft.com/sql/connect/jdbc/understanding-isolation-levels?view=sql-server-ver15).

#### <a name="read-using-query"></a>Lectura mediante consulta

Puede leer de Azure SQL Database mediante una tabla o una consulta SQL. Si está ejecutando una consulta SQL, esta debe finalizar antes de que se pueda iniciar la transformación. Las consultas SQL pueden resultar útiles para insertar operaciones que pueden ejecutarse más rápido y reducir la cantidad de datos que se leen de una instancia de SQL Server como las instrucciones SELECT, WHERE y JOIN. Cuando se insertan las operaciones, se pierde la capacidad de realizar un seguimiento del linaje y el rendimiento de las transformaciones antes de que los datos lleguen al flujo de datos.

### <a name="azure-synapse-analytics-sources"></a>Orígenes de Azure Synapse Analytics

Cuando se usa Azure Synapse Analytics, las opciones de origen incluyen un valor denominado **Enable staging** (Habilitar almacenamiento provisional). Este valor permite que ADF lea de Synapse mediante [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide?view=sql-server-ver15), lo que mejora en gran medida el rendimiento de lectura. La habilitación de PolyBase requiere que especifique una ubicación de almacenamiento provisional en Azure Blob Storage o Azure Data Lake Storage gen2 en la configuración de la actividad de flujo de datos.

![Enable staging](media/data-flow/enable-staging.png "Enable staging (Permitir almacenamiento provisional)") (Habilitar almacenamiento provisional)

### <a name="file-based-sources"></a>Orígenes basados en archivos

Aunque los flujos de datos admiten una variedad de tipos de archivo, el equipo de Azure Data Factory recomienda usar el formato Parquet nativo de Spark para conseguir tiempos óptimos de lectura y escritura.

Si está ejecutando el mismo flujo de datos en un conjunto de archivos, se recomienda leer de una carpeta, usar rutas de acceso con caracteres comodín o leer de una lista de archivos. Una sola ejecución de actividad de flujo de datos puede procesar todos los archivos por lotes. Puede encontrar más información sobre cómo establecer esta configuración en la documentación del conector, como [Azure Blob Storage](connector-azure-blob-storage.md#source-transformation).

Si es posible, evite usar la actividad For-Eeach para ejecutar flujos de datos en un conjunto de archivos. Esto haría que cada iteración de For-Each activase su propio clúster de Spark, lo que a menudo no suele ser necesario y puede resultar caro. 

## <a name="optimizing-sinks"></a>Optimización de receptores

Cuando los flujos de datos escriben en los receptores, la creación de particiones personalizada se producirá inmediatamente antes de la escritura. Al igual que en el origen, en la mayoría de los casos se recomienda mantener **Use current partitioning** (Usar particiones actuales) como opción de partición seleccionada. Los datos con particiones se escribirán mucho más rápido que los datos sin particiones, incluso si el destino no tiene particiones. A continuación se muestran las cuestiones específicas que se aplican a varios tipos de receptores. 

### <a name="azure-sql-database-sinks"></a>Receptores de Azure SQL Database

En el caso de Azure SQL Database, la creación de particiones predeterminada debería funcionar en la mayoría de los casos. Existe la posibilidad de que el receptor pueda tener demasiadas particiones para que la base de datos SQL pueda administrarlas. Si ve que este puede ser el caso, reduzca el número de particiones que salen del receptor de SQL Database.

#### <a name="disabling-indexes-using-a-sql-script"></a>Deshabilitación de índices mediante un script SQL

Deshabilitar los índices antes de una carga en una base de datos SQL puede mejorar considerablemente el rendimiento de la escritura en la tabla. Ejecute el siguiente comando antes de escribir en el receptor de SQL.

`ALTER INDEX ALL ON dbo.[Table Name] DISABLE`

Una vez finalizada la escritura, vuelva a generar los índices con el siguiente comando:

`ALTER INDEX ALL ON dbo.[Table Name] REBUILD`

En los flujos de datos de asignación, ambos se pueden ejecutar de forma nativa mediante scripts anteriores y posteriores a SQL en una base de datos SQL de Azure o un receptor de Synapse.

![Deshabilitación de índices](media/data-flow/disable-indexes-sql.png "Deshabilitación de índices")

> [!WARNING]
> Cuando se deshabilitan los índices, el flujo de datos toma de hecho el control sobre una base de datos y es poco probable que las consultas se realicen correctamente en este caso. Como resultado, se desencadenan muchos trabajos ETL en medio de la noche para evitar este conflicto. Para obtener más información, consulte las [restricciones que conlleva deshabilitar los índices](https://docs.microsoft.com/sql/relational-databases/indexes/disable-indexes-and-constraints?view=sql-server-ver15).

#### <a name="scaling-up-your-database"></a>Escalado vertical de la base de datos

Programe un cambio de tamaño del origen y el receptor de Azure SQL DB y Azure SQL DataWarehouse antes de que la canalización se ejecute para aumentar el rendimiento y minimizar la limitación de Azure una vez que se alcancen los límites de DTU. Una vez completada la ejecución de la canalización, cambie el tamaño de las bases de datos a la velocidad de ejecución normal.

### <a name="azure-synapse-analytics-sinks"></a>Receptores de Azure Synapse Analytics

Al escribir en Azure Synapse Analytics, asegúrese de que la opción **Enable staging**  (Habilitar almacenamiento provisional) esté establecida en true. Esto permite que ADF escriba mediante [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide), que carga de forma eficaz los datos en bloque. Tendrá que hacer referencia a una cuenta de Azure Data Lake Storage gen2 o de Azure Blob Storage para el almacenamiento provisional de los datos al usar PolyBase.

Además de PolyBase, en Azure Synapse Analytics se aplican los mismos procedimientos recomendados que en Azure SQL Database.

### <a name="file-based-sinks"></a>Receptores basados en archivos 

Aunque los flujos de datos admiten una variedad de tipos de archivo, el equipo de Azure Data Factory recomienda usar el formato Parquet nativo de Spark para conseguir tiempos óptimos de lectura y escritura.

Si los datos se distribuyen uniformemente, **Use current partitioning** (Usar particiones actuales) será la opción de creación de particiones más rápida para escribir archivos.

#### <a name="file-name-options"></a>Opciones de nombre de archivo

Al escribir archivos, tiene una selección de opciones de nomenclatura que afectan de diferente forma al rendimiento.

![Opciones del receptor](media/data-flow/file-sink-settings.png "opciones de receptor")

La opción **Default** (Predeterminada) ofrece la escritura más rápida. Cada partición equivaldrá a un archivo con el nombre predeterminado de Spark. Esto resulta útil si solo está leyendo de la carpeta de datos.

La opción **Pattern** (Patrón) establece un patrón de nomenclatura que cambiará el nombre de cada archivo de partición a un nombre más descriptivo. Esta operación se produce después de la escritura y es ligeramente más lenta que elegir la opción predeterminada. Per partition (Por partición) permite asignar un nombre a cada partición individual manualmente.

Si una columna corresponde a la forma en la que desea generar los datos, puede seleccionar **As data in column** (Como datos de columna). Esta opción reordena los datos y puede afectar al rendimiento si las columnas no están distribuidas uniformemente.

**Output to single file** (Salida a un solo archivo) combina todos los datos en una sola partición. Esto supone tiempos de escritura largos, especialmente en grandes conjuntos de datos. El equipo de Azure Data Factory recomienda **no** elegir esta opción a menos que haya una razón empresarial explícita para hacerlo.

### <a name="cosmosdb-sinks"></a>Receptores de CosmosDB

Al escribir en CosmosDB, la modificación de la capacidad de proceso y del tamaño del lote durante la ejecución del flujo de datos pueden mejorar el rendimiento. Estos cambios solo surten efecto durante la ejecución de la actividad de flujo de datos y volverán a la configuración original de la colección tras la conclusión. 

**Batch size** (Tamaño de lote): calcule el tamaño de fila aproximado de los datos y asegúrese de que el tamaño de fila * tamaño de lote es inferior a dos millones. Si lo es, aumente el tamaño del lote para obtener un mejor rendimiento

**Throughput** (Capacidad de proceso): establezca aquí un valor de rendimiento mayor aquí para que los documentos escriban más rápidamente en CosmosDB. Tenga en cuenta que los costos de RU son mayores cuando aumenta el valor de capacidad de proceso.

**Write throughput budget** (Presupuesto de capacidad de proceso de escritura): use un valor que sea menor que el total de RU por minuto. Si tiene un flujo de datos con un número elevado de particiones de Spark y establece un presupuesto de capacidad de proceso, habrá más equilibrio entre las particiones.


## <a name="optimizing-transformations"></a>Optimización de transformaciones

### <a name="optimizing-joins-exists-and-lookups"></a>Optimización de transformaciones de tipo Combinación, Existe y Búsqueda

#### <a name="broadcasting"></a>Difusión

En las transformaciones de tipo Combinación, Búsqueda y Existe, cuando uno o ambos flujos de datos son lo suficientemente pequeños como para caber en la memoria del nodo de trabajo, puede optimizar el rendimiento si habilita la opción **Broadcast** (Difusión). La difusión consiste en enviar tramas de datos pequeñas a todos los nodos del clúster. Esto permite que el motor de Spark realice una combinación sin reordenar los datos del flujo grande. De forma predeterminada, el motor de Spark decidirá automáticamente si difundir o no un lado de una combinación. Si está familiarizado con los datos de entrada y sabe que un flujo será significativamente menor que el otro, puede seleccionar la opción de difusión **Fixed** (Fija). La difusión fija fuerza a Spark a difundir el flujo seleccionado. 

Si el tamaño de los datos difundidos es demasiado grande para el nodo de Spark, puede aparecer un error de memoria insuficiente. Para evitar estos errores, utilice los clústeres **optimizados para memoria**. Si hay tiempos de espera de difusión durante las ejecuciones de flujo de datos, puede desactivar la optimización de difusión. Sin embargo, esto dará lugar a flujos de datos con un rendimiento más lento.

![Optimización de la transformación de combinación](media/data-flow/joinoptimize.png "Optimización de la combinación")

#### <a name="cross-joins"></a>Combinaciones cruzadas

Si usa valores literales en las condiciones de combinación o tiene varias coincidencias en ambos lados de una combinación, Spark ejecutará la combinación como una combinación cruzada. Una combinación cruzada es un producto cartesiano completo que, a continuación, filtra los valores combinados. Es significativamente más lenta que otros tipos de combinación. Asegúrese de que tiene referencias de columna en ambos lados de las condiciones de combinación para evitar que afecte al rendimiento.

#### <a name="sorting-before-joins"></a>Ordenación antes de las combinaciones

A diferencia de la unión de combinación en herramientas como SSIS, la transformación Combinación no es una operación de unión de combinación obligatoria. Las claves de combinación no requieren que se ordene antes de la transformación. El equipo de Azure Data Factory no recomienda el uso de transformaciones de tipo Ordenar en los flujos de datos de asignación.

### <a name="repartitioning-skewed-data"></a>Creación de nuevas particiones de datos asimétricos

Ciertas transformaciones, como las combinaciones y los agregados, reordenan las particiones de datos y en ocasiones pueden provocar datos asimétricos. Los datos asimétricos son aquellos que no se distribuyen uniformemente entre las particiones. Los datos muy asimétricos pueden dar lugar a transformaciones y escrituras de receptor de nivel inferior más lentas. Puede comprobar la asimetría de los datos en cualquier momento de una ejecución de flujo de datos haciendo clic en la transformación en la pantalla de supervisión.

![Asimetría y curtosis](media/data-flow/skewness-kurtosis.png "Asimetría y curtosis")

La pantalla de supervisión mostrará cómo se distribuyen los datos en cada partición, junto con dos métricas, asimetría y curtosis. El valor **Skewness** (Asimetría) es una medida del grado de asimetría que tienen los datos; puede tener un valor positivo, cero, negativo o indefinido. Una asimetría negativa significa que la cola izquierda es más larga que la derecha. El valor **Kurtosis** (Curtosis) es la medida que indica si los datos son de cola pesada o de cola ligera. No son deseables valores altos de curtosis. Los intervalos ideales de asimetría se encuentran entre -3 y 3 y los de curtosis son menores que 10. Una manera fácil de interpretar estos números es examinar el gráfico de particiones y ver si una barra es significativamente mayor que el resto.

Si los datos no quedan distribuidos uniformemente en las particiones después de una transformación, puede usar la [pestaña Optimize](#optimize-tab) (Optimizar) para volver a crear las particiones. La reordenación de los datos tarda tiempo y puede que no mejore el rendimiento del flujo de datos.

> [!TIP]
> Si vuelve a crear particiones de los datos, pero tiene transformaciones de nivel inferior que reordenan los datos, utilice la creación de particiones por hash en una columna utilizada como clave de combinación.

## <a name="next-steps"></a>Pasos siguientes

Vea el resto de artículos sobre Data Flow relacionados con el rendimiento:

- [Actividad Data Flow](control-flow-execute-data-flow-activity.md)
- [Supervisión del rendimiento de flujo de datos](concepts-data-flow-monitoring.md)
