---
title: Optimización del costo de rendimiento en Azure Cosmos DB
description: En este artículo se explica cómo optimizar los costos de rendimiento de los datos almacenados en Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: e1359fd2a59b49f10bb3b2daa4bcbadae921e188
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012456"
---
# <a name="optimize-provisioned-throughput-cost-in-azure-cosmos-db"></a>Optimización del costo de rendimiento aprovisionado en Azure Cosmos DB

Al ofrecer un modelo de rendimiento aprovisionado, Azure Cosmos DB ofrece un rendimiento predecible a cualquier escala. Reservar o aprovisionar el rendimiento antes de tiempo elimina el "efecto de vecino ruidoso" en el rendimiento. Especifique la cantidad exacta de rendimiento que necesita, y Azure Cosmos DB garantiza el rendimiento configurado, respaldado por un SLA.

Puede comenzar con un rendimiento mínimo de 400 RU/s y escalar verticalmente hasta decenas de millones de solicitudes por segundo o incluso más. Cada solicitud que emite contra el contenedor o la base de datos de Azure Cosmos, como una solicitud de lectura, solicitud de escritura, solicitud de consulta, los procedimientos almacenados tienen un costo correspondiente que se deduce de su rendimiento aprovisionado. Si aprovisiona 400 RU/s y emite una consulta que cuesta 40 RU, podrá emitir 10 de estas consultas por segundo. Cualquier solicitud más allá de eso tendrá una velocidad limitada, por lo que deberá reintentar la solicitud. Si usa controladores de cliente, admiten la lógica de reintento automático.

Puede aprovisionar el rendimiento en bases de datos o contenedores, y cada estrategia puede ayudarle a ahorrar costos según el escenario.

## <a name="optimize-by-provisioning-throughput-at-different-levels"></a>Optimización mediante el aprovisionamiento del rendimiento en distintos niveles

* Si aprovisiona el rendimiento en una base de datos, todos los contenedores, por ejemplo, colecciones, tablas y grafos dentro de esa base de datos, pueden compartir el rendimiento según la carga. El rendimiento reservado en el nivel de base de datos se comparte de forma desigual, según la carga de trabajo en un conjunto específico de contenedores.

* Si aprovisiona el rendimiento en un contenedor, se garantiza el rendimiento para ese contenedor, respaldado por el SLA. La elección de una clave de partición lógica es fundamental para la distribución uniforme de la carga para todas las particiones lógicas de un contenedor. Consulte los artículos de [creación de particiones](partitioning-overview.md) y [escalado horizontal](partition-data.md) para más detalles.

Las siguientes son algunas directrices para decidir una estrategia de rendimiento aprovisionado:

**Considere la posibilidad de aprovisionar el rendimiento en una base de datos de Azure Cosmos (que contiene un conjunto de contenedores) si**:

1. Tiene algunas decenas de contenedores de Azure Cosmos y quiere compartir el rendimiento entre algunos o todos ellos. 

2. Va a migrar desde una base de datos de inquilino único diseñada para ejecutarse en VM hospedadas por IaaS o en entornos locales, por ejemplo, NoSQL o bases de datos relacionales, a Azure Cosmos DB. Y si tiene muchas colecciones, tablas o grafos y no quiere hacer cambios en el modelo de datos. Tenga en cuenta que quizás deba renunciar a algunas de las ventajas que ofrece Azure Cosmos DB si no va a actualizar el modelo de datos al migrar desde una base de datos local. Se recomienda volver a acceder siempre al modelo de datos para sacar el máximo provecho en cuanto a rendimiento y también para optimizar los costos. 

3. Quiere absorber los picos no planeados en cargas de trabajo en virtud de rendimiento agrupado en el nivel de base de datos sometido a un pico inesperado en la carga de trabajo. 

4. En lugar de establecer el rendimiento específico de un contenedor individual, le interesa obtener el rendimiento agregado en un conjunto de contenedores dentro de la base de datos.

**Considere la posibilidad de aprovisionar el rendimiento en un contenedor individual si:**

1. Tiene unos pocos contenedores de Azure Cosmos. Dado que Azure Cosmos DB es independiente del esquema, un contenedor puede contener elementos que tengan esquemas heterogéneos y no requieran que los clientes creen varios tipos de contenedor, uno para cada entidad. Siempre es una opción a tener en cuenta si tiene sentido agrupar, digamos, 10 o 20 contenedores independientes en un único contenedor. Con un mínimo de 400 RU para los contenedores, podría ser más rentable agrupar los 10 o 20 contenedores en uno. 

2. Quiere controlar el rendimiento en un contenedor específico y obtener el rendimiento garantizado en un contenedor determinado respaldado por el SLA.

**Tenga en cuenta un híbrido de las dos estrategias anteriores:**

1. Como se mencionó anteriormente, Azure Cosmos DB le permite mezclar y combinar las dos estrategias anteriores, por lo que ahora puede tener algunos contenedores dentro de la base de datos de Azure Cosmos, que pueden compartir el rendimiento aprovisionado en la base de datos, así como algunos contenedores dentro de la misma base de datos, que es posible que tengan cantidades dedicadas de rendimiento aprovisionado. 

2. Puede aplicar las estrategias anteriores para proponer una configuración híbrida, donde tenga tanto rendimiento aprovisionado a nivel de la base de datos y algunos contenedores con rendimiento dedicado.

Como se muestra en la tabla siguiente, según la elección de la API, puede aprovisionar el rendimiento con distintas granularidades.

|API|Para rendimiento **compartido**, configure |Para rendimiento **dedicado**, configure |
|----|----|----|
|API DE SQL|Base de datos|Contenedor|
|API de Azure Cosmos DB para MongoDB|Base de datos|Colección|
|Cassandra API|Espacio de claves|Tabla|
|API de Gremlin|Cuenta de base de datos|Grafo|
|Table API|Cuenta de base de datos|Tabla|

Al aprovisionar el rendimiento en diferentes niveles, puede optimizar los costos en función de las características de la carga de trabajo. Como se mencionó anteriormente, puede aumentar o disminuir mediante programación y en cualquier momento el rendimiento aprovisionado para contenedores individuales o de a varios en un conjunto de contenedores. Al escalar elásticamente el rendimiento a medida que cambia la carga de trabajo, solo paga por el rendimiento que haya configurado. Si el contenedor o el conjunto de contenedores se distribuyen entre varias regiones, se garantiza que el rendimiento que configura en el contenedor o el conjunto de contenedores está disponible en todas las regiones.

## <a name="optimize-with-rate-limiting-your-requests"></a>Optimización con limitación de velocidad de las solicitudes

Para cargas de trabajo que no son sensibles a la latencia, puede aprovisionar menos rendimiento y dejar que la aplicación controle la limitación de velocidad cuando el rendimiento real supere el rendimiento aprovisionado. El servidor finalizará de forma preventiva la solicitud con `RequestRateTooLarge` (código de estado HTTP 429) y devolverá el encabezado `x-ms-retry-after-ms` para indicar la cantidad de tiempo, en milisegundos, que el usuario debe esperar antes de reintentar la solicitud. 

```html
HTTP Status 429, 
 Status Line: RequestRateTooLarge 
 x-ms-retry-after-ms :100
```

### <a name="retry-logic-in-sdks"></a>Lógica de reintento en SDK 

Los SDK nativos (.NET/.NET Core, Java, Node.js y Python) capturan implícitamente esta respuesta, respetan el encabezado retry-after especificado por el servidor y reintentan la solicitud. A menos que varios clientes accedan a la cuenta al mismo tiempo, el siguiente reintento se realizará correctamente.

Si tiene más de un cliente de manera acumulativa funcionando constantemente por encima de la tasa de solicitud, el número de reintentos predeterminado que actualmente está establecido en 9 puede no ser suficiente. En estos casos, el cliente genera un `RequestRateTooLargeException` con el estado de código 429 para la aplicación. El número de reintentos predeterminado se puede cambiar estableciendo `RetryOptions` en la instancia ConnectionPolicy. De forma predeterminada, la excepción `RequestRateTooLargeException` con el código de estado 429 se devuelve tras un tiempo de espera acumulativo de 30 segundos si la solicitud sigue funcionando por encima de la tasa de solicitudes. Esto sucede incluso cuando el número de reintentos actual es inferior al número de reintentos máximo de 9, el valor predeterminado, o un valor definido por el usuario. 

[MaxRetryAttemptsOnThrottledRequests](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.retryoptions.maxretryattemptsonthrottledrequests?view=azure-dotnet) se establece en 3, por lo que, en este caso, si una operación de solicitud tiene una tasa limitada al superar el rendimiento reservado para el contenedor, la operación de solicitud volverá a intentarlo tres veces antes de iniciar la excepción en la aplicación. [MaxRetryWaitTimeInSeconds](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.retryoptions.maxretrywaittimeinseconds?view=azure-dotnet#Microsoft_Azure_Documents_Client_RetryOptions_MaxRetryWaitTimeInSeconds) se establece en 60, por lo que, en este caso, si el tiempo de espera acumulativo de reintento desde la primera solicitud supera los 60 segundos, se inicia la excepción.

```csharp
ConnectionPolicy connectionPolicy = new ConnectionPolicy(); 
connectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 3; 
connectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 60;
```

## <a name="partitioning-strategy-and-provisioned-throughput-costs"></a>Estrategia de creación de particiones y costos de rendimiento aprovisionado

Una buena estrategia de creación de particiones es importante para optimizar los costos en Azure Cosmos DB. Asegúrese de que haya ninguna asimetría de particiones, que se exponen a través de las métricas de almacenamiento. Asegúrese de que haya ninguna asimetría de rendimiento para una partición, que se expone con métricas de rendimiento. Asegúrese de que haya ninguna asimetría hacia claves de partición determinadas. Las claves dominantes en el almacenamiento se exponen a través de las métricas, pero la clave dependerá del patrón de acceso de la aplicación. Lo mejor es pensar en la clave de partición lógica apropiada. Se espera que una buena clave de partición tenga las siguientes características:

* Elija una clave de partición que distribuya la carga de trabajo de manera uniforme entre todas las particiones y también a lo largo del tiempo. En otras palabras, no debería tener algunas claves con la mayoría de los datos y algunas claves con menos datos o ningún dato. 

* Elija una clave de partición que permita patrones de acceso que se distribuyan uniformemente entre las particiones lógicas. La carga de trabajo es razonablemente uniforme entre todas las claves. En otras palabras, la mayoría de la carga de trabajo no debería centrarse en unas pocas claves específicas. 

* Elija una clave de partición que tenga una amplia gama de valores. 

La idea básica es distribuir los datos y la actividad en el contenedor entre el conjunto de particiones lógicas, para que los recursos de rendimiento y almacenamiento de datos se puedan distribuir entre las particiones lógicas. Los candidatos para las claves de partición pueden incluir las propiedades que aparecen con frecuencia como un filtro en las consultas. Las consultas se pueden enrutar de manera eficaz si se incluye la clave de partición en el predicador de filtro. Con una estrategia tal de creación de particiones, la optimización del rendimiento aprovisionado será mucho más fácil. 

### <a name="design-smaller-items-for-higher-throughput"></a>Diseño de los elementos más pequeños para un mayor rendimiento 

El gasto de solicitud, o el costo de procesamiento de solicitudes, de una operación dada está directamente correlacionado con el tamaño del elemento. Las operaciones en elementos grandes costarán más que las operaciones en los elementos más pequeños. 

## <a name="data-access-patterns"></a>Patrones de acceso a datos 

Siempre es una buena práctica para separar lógicamente los datos en categorías lógicas según la frecuencia con la que accede a los datos. Al categorizar los datos según un nivel de acceso frecuente, medio o esporádico, puede ajustar el almacenamiento consumido y el rendimiento requerido. Según la frecuencia de acceso, puede colocar los datos en contenedores diferentes (por ejemplo, tablas, grafos y colecciones) y ajustar el rendimiento aprovisionado en ellos para acomodarse a las necesidades de ese segmento de datos. 

Además, si usa Azure Cosmos DB y sabe que no va a buscar por determinados valores de datos o pocas veces va a acceder a ellos, debería almacenar los valores comprimidos de estos atributos. Con este método, ahorra en espacio de almacenamiento, espacio de índice y rendimiento aprovisionado, lo que reduce los costos.

## <a name="optimize-by-changing-indexing-policy"></a>Optimización al cambiar la directiva de indexación 

De forma predeterminada, Azure Cosmos DB indexa automáticamente todas las propiedades de cada registro. Esto está pensado para facilitar el desarrollo y garantizar un rendimiento excelente en muchos tipos diferentes de consultas ad hoc. Si tiene registros grandes con miles de propiedades, pagar el costo de rendimiento para la indexación de cada propiedad puede no ser útil, en especial si solo una consulta frente a 10 o 20 de esas propiedades. A medida que aprende a manejar la carga de trabajo específica, nuestra sugerencia es ajustar la directiva de índice. Para información detallada sobre la directiva de indexación de Azure Cosmos DB, consulte [aquí](indexing-policies.md). 

## <a name="monitoring-provisioned-and-consumed-throughput"></a>Supervisión del rendimiento aprovisionado y consumido 

Puede supervisar el número total de RU aprovisionadas, el número de solicitudes velocidad limitada, así como el número de RU que ha consumido en Azure Portal. La imagen siguiente muestra un ejemplo de métrica de uso:

:::image type="content" source="./media/optimize-cost-throughput/monitoring.png" alt-text="Supervisión de las unidades de solicitud en Azure Portal":::

También puede establecer alertas para comprobar si el número de solicitudes con velocidad limitada supera un umbral específico. Consulte el artículo [Supervisión de Azure Cosmos DB](use-metrics.md) para más detalles. Estas alertas pueden enviar un correo electrónico a los administradores de cuentas o llamar a un webhook HTTP personalizado o una función de Azure para aumentar automáticamente el rendimiento aprovisionado. 

## <a name="scale-your-throughput-elastically-and-on-demand"></a>Escalado del rendimiento elásticamente y a petición 

Puesto que se le cobrará el rendimiento aprovisionado, asociar el rendimiento aprovisionado con sus necesidades puede ayudarle a evitar los cargos por el rendimiento sin usar. Puede escalar o reducir verticalmente el rendimiento aprovisionado en cualquier momento, según sea necesario. Si sus necesidades de rendimiento son muy predecibles, puede usar Azure Functions y usar un desencadenador de temporizador para [aumentar o disminuir el rendimiento según una programación](scale-on-schedule.md). 

* Supervisar el consumo de las RU y la relación de solicitudes con velocidad limitada puede revelar que no es necesario mantener constante el rendimiento aprovisionado a lo largo del día o de la semana. Puede recibir menos tráfico durante la noche o durante el fin de semana. Mediante los SDK nativos o la API de REST de Azure Portal o Azure Cosmos DB, puede escalar el rendimiento aprovisionado en cualquier momento. La API de REST de Azure Cosmos DB proporciona puntos de conexión para actualizar mediante programación el nivel de rendimiento de los contenedores, lo que simplifica el ajuste del rendimiento desde el código según la hora del día o el día de la semana. La operación se realiza sin ningún tiempo de inactividad y normalmente surte efecto en menos de un minuto. 

* Una de las áreas en las que se debe escalar el rendimiento es cuando ingiere los datos en Azure Cosmos DB, por ejemplo, durante la migración de datos. Una vez que haya completado la migración, puede reducir el rendimiento aprovisionado verticalmente para controlar el estado estable de la solución.  

* Recuerde que la facturación tiene una granularidad de una hora, por lo que no ahorrará dinero si cambia el rendimiento aprovisionado con mayor frecuencia que una hora a la vez.

## <a name="determine-the-throughput-needed-for-a-new-workload"></a>Determinación del rendimiento necesario para una nueva carga de trabajo 

Para determinar el rendimiento aprovisionado para una nueva carga de trabajo, puede seguir los pasos a continuación: 

1. Haga una evaluación inicial, aproximada mediante programador de capacidad y ajuste las estimaciones con la Ayuda del explorador de Azure Cosmos en Azure Portal. 

2. Se recomienda crear los contenedores con un rendimiento superior al esperado y, luego, reducir verticalmente según sea necesario. 

3. Se recomienda usar uno de los SDK nativos de Azure Cosmos DB para aprovechar los reintentos automáticos cuando las solicitudes tienen una velocidad limitada. Si trabaja en una plataforma que no se admite y usa la API de REST de Cosmos DB, implemente su propia directiva de reintentos con el encabezado `x-ms-retry-after-ms`. 

4. Asegúrese de que el código de la aplicación admita correctamente el caso cuando fracasen todos los reintentos. 

5. Puede configurar alertas desde Azure Portal para recibir notificaciones de limitación de velocidad. Puede comenzar con límites conservadores, como 10 solicitudes con velocidad limitada durante los últimos 15 minutos, y cambiar a reglas más ambiciosas una vez que descubra el consumo real. Unos límites de velocidad ocasionales están bien, muestran que juega con los límites que ha establecido y eso es exactamente lo que quiere. 

6. Use la supervisión para entender el patrón de tráfico, para que pueda considerar la necesidad de ajustar dinámicamente el aprovisionamiento de rendimiento durante un día o una semana. 

7. Supervise la relación entre el rendimiento aprovisionado frente al consumido periódicamente para asegurarse de no haber aprovisionado un número de contenedores y bases de datos mayor que el necesario. Tener un poco de rendimiento aprovisionado adicional es una buena comprobación de seguridad.  

### <a name="best-practices-to-optimize-provisioned-throughput"></a>Procedimientos recomendados para optimizar el rendimiento aprovisionado 

Los pasos siguientes le ayudan a que sus soluciones sean altamente escalables y rentables al usar Azure Cosmos DB.  

1. Si tiene mucho rendimiento aprovisionado excesivo a través de contenedores y bases de datos, debe revisar las RU aprovisionadas frente a las RU consumidas y ajustar las cargas de trabajo.  

2. Un método para calcular la cantidad de rendimiento reservado que necesita la aplicación es registrar la carga de unidades de solicitud (RU) asociadas a la ejecución de las operaciones típicas, frente a un contenedor o base de datos representativo de Azure Cosmos que usa la aplicación y, después, calcular el número de operaciones que prevé realizar cada segundo. Asegúrese de medir e incluir las consultas típicas, así como su uso. Para información sobre cómo calcular los costos de RU de consultas mediante programación o con el portal, consulte [Optimización del costo de las consultas](optimize-cost-queries.md). 

3. Otra forma de obtener las operaciones y sus costos en RU es habilitar los registros de Azure Monitor, lo que proporcionará el desglose de operación/duración y el cargo de solicitud. Azure Cosmos DB proporciona el cargo de solicitud para cada operación, por lo que cada cargo de operación se puede almacenar desde la respuesta y luego usarse para el análisis. 

4. Puede escalar o reducir verticalmente y de manera elástica el rendimiento aprovisionado según sea necesario para ajustarse a sus necesidades de carga de trabajo. 

5. Puede agregar y quitar regiones asociadas con su cuenta de Azure Cosmos según lo necesite y controlar los costos. 

6. Asegúrese de tener una distribución uniforme de los datos y las cargas de trabajo entre las particiones lógicas de los contenedores. Si tiene la distribución desigual entre las particiones, puede que se aprovisione una mayor cantidad de rendimiento que el valor necesario. Si descubre que tienen una distribución distorsionada, se recomienda redistribuir la carga de trabajo uniformemente entre las particiones o volver a particionar los datos. 

7. Si tiene muchos contenedores y estos contenedores no requieren SLA, puede usar la oferta basada en base de datos para los casos donde no se apliquen SLA de rendimiento por contenedor. Debe identificar cuál de los contenedores de Azure Cosmos quiere migrar a la oferta rendimiento de nivel de base de datos y después migrarlos mediante una solución basada en la fuente de cambio. 

8. Considere la posibilidad de usar el "nivel Gratis de Cosmos DB" (gratis durante un año), pruebe Cosmos DB (hasta tres regiones) o el emulador de Cosmos DB descargable para escenarios de desarrollo y pruebas. Al usar estas opciones para desarrollo y pruebas, puede reducir considerablemente los costos.  

9. Incluso puede realizar optimizaciones de costos específicos de la carga de trabajo, por ejemplo, al aumentar el tamaño de lote, al equilibrar la carga de lecturas entre varias regiones y al desduplicar los datos, si procede.

10. Con capacidad reservada de Azure Cosmos DB, puede obtener importantes descuentos de hasta un 65 % durante tres años. El modelo de capacidad reservada de Azure Cosmos DB es un compromiso por adelantado en unidades de solicitud necesarias a lo largo del tiempo. Los descuentos están organizados en niveles de modo tal que, cuantas más unidades de solicitud use durante un período más prolongado, mayor será el descuento. Estos descuentos se aplican inmediatamente. Las RU usadas por encima de los valores aprovisionados se cobran según el coste de capacidad no reservada. Consulte [Capacidad reservada de Cosmos DB](cosmos-db-reserved-capacity.md)) para más detalles. Considere la posibilidad de adquirir capacidad reservada para reducir aún más los costos de rendimiento aprovisionado.  

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede seguir obteniendo más información sobre la optimización de costos en Azure Cosmos DB con los siguientes artículos:

* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).

