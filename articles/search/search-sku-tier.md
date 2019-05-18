---
title: 'Selección de SKU o plan de tarifa del servicio Azure Search: Azure Search'
description: 'Azure Search puede aprovisionarse en estas SKU: Gratis, básico y, estándar y están disponible en diversas configuraciones de recursos y niveles de capacidad.'
services: search
author: HeidiSteen
manager: cgronlun
tags: azure-portal
ms.service: search
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 00422209302bbcc2139be4f6b490f0bb2816c051
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539266"
---
# <a name="choose-a-pricing-tier-for-azure-search"></a>Selección de un plan de tarifa de Azure Search

Cuando se crea un servicio Azure Search, un [se crea el recurso](search-create-service-portal.md) en SKU que se fija para la duración del servicio o el plan de tarifa. Los niveles incluyen gratis, básico, estándar y optimizadas para almacenamiento. Estándar y optimizada de almacenamiento están disponibles con varias configuraciones y capacidades.

La mayoría de los clientes iniciar con el nivel gratis para que pueden evaluar el servicio. A continuación actualice a uno de los niveles más altos para las implementaciones de desarrollo y producción. Puede completar todos los inicios rápidos y tutoriales mediante el nivel gratis, incluidos los de búsqueda cognitiva intensivo de recursos.

> [!NOTE]
> Actualmente, Microsoft proporciona los niveles de servicio de almacenamiento optimizado en una vista previa a un precio con descuento para las pruebas y experimentación, con el fin de recabar comentarios. El precio final se anunciará más adelante cuando estos niveles están disponibles con carácter general. Aconsejamos que no se usen estos niveles para las aplicaciones de producción.

Los niveles reflejan las características del hardware que hospeda el servicio (en lugar de las características) y se diferencian por:

+ El número de índices que puede crear.
+ El tamaño y velocidad de particiones (almacenamiento físico).

Aunque todos los niveles, incluido el nivel gratis, por lo general ofrecen una paridad de características, cargas de trabajo mayores pueden dictar la necesidad de niveles más altos. Por ejemplo, [AI indexación con Cognitive Services](cognitive-search-concept-intro.md) tiene habilidades de larga ejecución ese tiempo de espera en un servicio gratuito, a menos que el conjunto de datos es pequeño.

> [!NOTE] 
> La excepción a la paridad de características es [indizadores](search-indexer-overview.md), que no están disponibles en S3 HD.
>

Dentro de un nivel, también puede [ajustar los recursos de réplica y partición](search-capacity-planning.md) para aumentar o disminuir la escala. Puede empezar con uno o dos de cada uno y, a continuación, elevar temporalmente su eficacia sin igual para una gran carga de trabajo de indexación. La posibilidad de ajustar los niveles de recursos dentro de un nivel agrega flexibilidad, pero también complica ligeramente su análisis. Es posible que deba experimentar para ver si un nivel inferior con más recursos y réplicas ofrece un mejor valor y rendimiento que un nivel superior con menos recursos. Para obtener más información acerca de cuándo y por qué debería ajustar la capacidad, consulte [Consideraciones de rendimiento y optimización](search-performance-optimization.md).

## <a name="tiers-for-azure-search"></a>Niveles para Azure Search

En la tabla siguiente se enumera los niveles disponibles. Puede encontrar más información sobre los distintos niveles en la [página de precios](https://azure.microsoft.com/pricing/details/search/), en el [límites de servicio en Azure Search](search-limits-quotas-capacity.md) artículo y en el portal de página cuando Aprovisione un servicio.

|Nivel: | Capacity |
|-----|-------------|
|Gratis | Se comparte con otros suscriptores. No escalable. Limitado a tres índices y 50 MB de almacenamiento. |
|Básica | Recursos informáticos dedicados para cargas de trabajo de producción en una escala menor. Una partición de 2 GB y hasta tres réplicas. |
|1 estándar (S1) | Para S1 y versiones posteriores, dedicado a las máquinas con más capacidad de almacenamiento y procesamiento en todos los niveles. S1, el tamaño de partición es 25 GB por partición (con un máximo de 300 GB por servicio). |
|Estándar 2 (S2) | Similar a S1, pero con las particiones de 100 GB (y un máximo de 1,2 TB por servicio). |
|Estándar 3 (S3) | Particiones de 200 GB (con un máximo de 2,4 TB por servicio). |
|Estándar 3 de alta densidad (HD S3) | Alta densidad es una *modo del alojamiento* para S3. El hardware subyacente está optimizado para un gran número de índices más pequeños y está pensado para escenarios multiinquilino. S3 HD tiene el mismo cargo por unidad como S3, pero el hardware está optimizado para las lecturas de archivos rápidas en un gran número de índices más pequeños.|
|Almacenamiento optimizado 1 (L1) | Particiones de 1 TB (con un máximo de 12 TB por servicio). |
|Almacenamiento optimizado 2 (L2) | Particiones de 2 TB (con un máximo de 24 TB por servicio). |

> [!NOTE] 
> Los niveles de almacenamiento optimizado para ofrecen mayor capacidad de almacenamiento a un precio menor por cada TB que los niveles estándar. El principal inconveniente es mayor latencia de consulta, lo que debe validar para sus requisitos de aplicación específica.  Para obtener más información sobre las consideraciones de rendimiento de este nivel, consulte [consideraciones de rendimiento y optimización](search-performance-optimization.md).
>

## <a name="how-billing-works"></a>Cómo funciona la facturación

Hay tres maneras de incurrir en costes en Azure Search, y existen componentes fijos y variables. En esta sección se describe los tres componentes de facturación: core indexación enriquecida de inteligencia artificial, los cargos de salida de datos y los costos de servicio.

### <a name="core-service-costs-fixed-and-variable"></a>Costos de servicio de núcleo (fijos y variables)

Para el servicio, el cargo mínimo es la primera unidad de búsqueda (partición 1 réplica 1). Este valor mínimo es fijo durante la vigencia del servicio porque el servicio no se puede ejecutar en cualquier cosa menor que esta configuración.

Más allá de la mínima, puede agregar réplicas y particiones por separado. Por ejemplo, puede agregar sólo las réplicas o solo las particiones. Aumento incremental en capacidad a través de las réplicas y particiones conforman el componente de coste de la variable.

La facturación se basa en un [fórmula (réplicas x particiones x velocidad)](#search-units). La tarifa que se aplica depende del plan de tarifa que seleccione.

En la siguiente captura de pantalla, los precios por unidad se indican para gratis, básico y S1. (No se muestran S2, S3, L1 y L2.) Si crea un servicio básico, el costo mensual promedio, el valor que aparece para *precio 1*. Para un servicio estándar, el costo mensual promedio, el valor que aparece para *precio 2*. Porque la capacidad de almacenamiento y potencia computacional es mayor en cada nivel consecutivo, aumentar los costos de unidad para cada nivel. Las tarifas de Azure Search están disponibles en el [página de precios de Azure Search](https://azure.microsoft.com/pricing/details/search/).

![Precio por unidad](./media/search-sku-tier/per-unit-pricing.png "precio por unidad")

Cuando esté realizando el costo de una solución de búsqueda, tenga en cuenta que los precios y capacidad no lineal. (Más de se duplica la capacidad duplica el costo). Para obtener un ejemplo de cómo de la fórmula funciona, consulte [cómo asignar particiones y réplicas](search-capacity-planning.md#how-to-allocate-replicas-and-partitions).

#### <a name="billing-based-on-search-units"></a>Facturación según las unidades de búsqueda

El concepto de facturación más importante para comprender para las operaciones de búsqueda de Azure es la *unidad de búsqueda* (SU). Dado que Azure Search depende de las réplicas y las particiones para la indexación y las consultas, no tiene sentido facturar solo por una o la otra. En su lugar, la facturación se basa en una composición de ambas.

Unidades de búsqueda es el producto de la *réplicas* y *particiones* usada por un servicio: **(R x P = SU)**.

Cada servicio se inicia como mínimo con una SU (una réplica multiplicada por una partición). El máximo para cualquier servicio es de 36 unidades de búsqueda. Este valor máximo se puede conseguir de varias maneras: las réplicas de 6 particiones x 6 o réplicas de 3 particiones x 12, por ejemplo. Es habitual usar menos capacidad total (por ejemplo, un réplica de 3, 3-partición servicio facturado como 9 su). Consulte la [combinaciones de particiones y réplicas](search-capacity-planning.md#chart) gráfico para las combinaciones válidas.

La tasa de facturación es cada hora por unidad de Streaming. Cada nivel tiene una tasa mayor progresivamente. Los niveles superiores vienen con las particiones más grandes y más rápidas, y esto contribuye a un precio por hora total mayor para ese nivel. Puede ver las tarifas de cada nivel en el [los detalles de precios](https://azure.microsoft.com/pricing/details/search/) página.

La mayoría de los clientes solo ponen en línea una parte de la capacidad total, manteniendo el resto en la reserva. Para la facturación, el número de particiones y réplicas que poner en línea, calcula el SU fórmula, determina lo que paga por hora.

### <a name="data-egress-charges-during-indexing"></a>Cargos de salida de datos durante la indización.

Uso de [indexadores de Azure Search](search-indexer-overview.md) podrían afectar a la facturación, dependiendo de la ubicación de los servicios. Puede eliminar los cargos de salida de datos completamente si crea el servicio Azure Search en la misma región que los datos. Aquí es cierta información desde el [página de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/):

+ Microsoft no cobra para los datos entrantes a cualquier servicio de Azure, o para los datos de salida de Azure Search.

+ En soluciones de múltiples servicios, no hay ningún cargo por datos que atraviesan la conexión cuando todos los servicios están en la misma región.

Si hay servicios en regiones diferentes se aplican cargos por los datos salientes. Estos cargos no son realmente parte de la factura de Azure Search. Se menciona aquí porque si usa datos o indexadores enriquecida de inteligencia artificial para extraer datos desde distintas regiones, verá los costos que se refleja en su factura general.

### <a name="ai-enriched-indexing-with-cognitive-services"></a>Indexación con Cognitive Services enriquecida de inteligencia artificial

Para [AI indexación con Cognitive Services](cognitive-search-concept-intro.md), debe planear asociar un recurso de Azure Cognitive Services facturable, en la misma región que Azure Search, en el plan de tarifa S0 para el procesamiento de pago por uso. No hay ningún costo fijo asociado con la asociación de Cognitive Services. Solo paga por los procesos que necesite.

Extracción de la imagen durante la averiguación de documentos es un cargo de Azure Search. Se factura según el número de imágenes que se extraen de los documentos. La extracción de texto es actualmente gratuita.

Enriquecimientos de otros, como el procesamiento de lenguaje natural, se basan en [integrados conocimientos cognitivos](cognitive-search-predefined-skills.md) y factura con respecto a un recurso de Cognitive Services. Factura a la misma velocidad como si la tarea que haya realizado directamente mediante Cognitive Services. Para obtener más información, consulte [adjuntar un recurso de Cognitive Services con un conjunto de habilidades](cognitive-search-attach-cognitive-services.md).

<a name="search-units"></a>

#### <a name="billing-for-image-extraction-in-cognitive-search"></a>Facturación por la extracción de imágenes en Cognitive Search

Si extraer las imágenes de los archivos de una canalización de indización de búsqueda cognitiva, se le cobrará en su factura de Azure Search para esa operación. En un [configuración del indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer#indexer-parameters), **imageAction** es el parámetro que desencadena la extracción de la imagen. Si **imageAction** está establecido en "none" (valor predeterminado), no se le cobrará por extracción de la imagen.

Los precios está sujeta a cambios. Se documenta en el [los detalles de precios](https://azure.microsoft.com/pricing/details/search/) página de búsqueda de Azure.

#### <a name="billing-for-built-in-skills-in-cognitive-search"></a>Facturación de las habilidades integradas en Cognitive Search

Al configurar una canalización de enriquecimiento, cualquier [habilidad integrada](cognitive-search-predefined-skills.md) que se use en ella se basa en modelos de aprendizaje automático. Estos modelos son proporcionados por Cognitive Services. Si utiliza estos modelos durante la indización, le facturará al mismo precio como sería si solicita el recurso directamente.

Por ejemplo, supongamos que tiene una canalización que usa el reconocimiento óptico de caracteres (OCR) con los archivos JPEG digitalizados y el texto resultante se inserta en un índice de Azure Search para consultas de búsqueda de forma libre. La canalización de indexación incluiría un indexador con la [habilidad OCR](cognitive-search-skill-ocr.md), que se [asociaría a un recurso de Cognitive Services](cognitive-search-attach-cognitive-services.md). Al ejecutar el indexador, los cargos para la ejecución de OCR aparecerá en la factura de los recursos de Cognitive.

## <a name="tips-for-reducing-costs"></a>Sugerencias para reducir los costos

No se puede cerrar el servicio para reducir su factura. Recursos dedicados son siempre operativos, asignada para su uso exclusivo durante la vigencia de su servicio. La única manera de reducir su factura consiste en reducir las réplicas y particiones a un nivel que proporciona un rendimiento aceptable y [cumplimiento SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

Es una forma de reducir los costos elegir un nivel con una tarifa por hora inferior. Las tarifas por hora de S1 son inferiores a las tarifas de S2 o S3. Suponiendo que aprovisionar el servicio del extremo inferior de la carga de proyecciones, si superan el servicio, puede crear un segundo servicio mayores niveles, volver a generar los índices en el segundo servicio y, a continuación, elimine la primera de ellas.

Si ha realizado planear la capacidad de los servidores locales, sabrá que es común a "¡Compre" por lo que puede controlar el crecimiento previsto. Con un servicio en la nube, que puede aplicarse a ahorros de costos más agresivamente porque no está bloqueado una compra específico. Siempre puede cambiar a un servicio de nivel superior si actual no es suficiente.

### <a name="capacity"></a>Capacity

En Azure Search, la capacidad se estructura como *réplicas* y *particiones*.

+ Las réplicas son instancias del servicio de búsqueda. Cada réplica hospeda una copia con equilibrio de carga de un índice. Por ejemplo, un servicio con seis réplicas tiene seis copias de todos los índices cargados en el servicio.

+ Las particiones almacenan índices y dividen los datos utilizables en búsquedas de manera automática. Dos particiones dividen su índice en la mitad; tres particiones división en tercios y así sucesivamente. En términos de capacidad, *tamaño de la partición* es la principal característica distintiva entre niveles.

> [!NOTE]
> Todos los niveles estándar y optimizada de almacenamiento admiten [combinaciones flexibles de réplicas y particiones](search-capacity-planning.md#chart) para que pueda [optimizar el sistema de almacenamiento o la velocidad](search-performance-optimization.md) cambiando el saldo. El nivel básico ofrece hasta tres réplicas para alta disponibilidad, pero tiene sólo una partición. Niveles gratuitos no proporcionan recursos dedicados: informática de los recursos compartidos por varios suscriptores.

### <a name="more-about-service-limits"></a>Más información acerca de los límites de servicio

Servicios de recursos de host, como índices e indexadores. Cada nivel impone [límites de servicio](search-limits-quotas-capacity.md) en el número de recursos que puede crear. Por lo tanto, el número máximo de índices (y otros objetos) es la segunda característica distintiva entre niveles. Revisar cada opción en el portal, tenga en cuenta los límites del número de índices. Otros recursos, como los indizadores, orígenes de datos y conjuntos de habilidades, se anexa a los límites de índice.

## <a name="consumption-patterns"></a>Patrones de consumo

La mayoría de los clientes iniciar con el servicio gratuito, que mantienen indefinidamente, y, a continuación, elija uno de los niveles estándar o almacenamiento optimizado para cargas de trabajo de desarrollo o producción graves.

![Planes de tarifa de Azure Search](./media/search-sku-tier/tiers.png "planes de tarifa de Azure Search")

En los extremos superior e inferior, básico y S3 HD son para patrones de consumo importante pero inusual. Basic es para cargas de trabajo de producción pequeño. Ofrece los SLA, los recursos dedicados y alta disponibilidad, pero proporciona almacenamiento modesto, uso llega hasta el total de 2 GB. Este nivel fue diseñado para los clientes que constantemente una infrautilización capacidad disponible. En el extremo superior, S3 HD es para cargas de trabajo típicos de ISV, asociados, [soluciones para varios inquilinos](search-modeling-multitenant-saas-applications.md), o cualquier configuración que requiere un gran número de índices pequeños. A menudo resulta evidente cuando básica o S3 HD es el nivel correcto. Si desea que la confirmación, puede publicar en [StackOverflow](https://stackoverflow.com/questions/tagged/azure-search) o [póngase en contacto con soporte técnico de Azure](https://azure.microsoft.com/support/options/) para obtener instrucciones.

Los niveles estándar más comúnmente usados, S1 a través de S3, constituyen una progresión de niveles crecientes de capacidad. Hay puntos de inflexión en el tamaño de la partición y los límites de número de índices, indizadores e iniciémoslo recursos:

|  | S1 | S2 | S3 |  |  |  |  |
|--|----|----|----|--|--|--|--|
| Tamaño de la partición|  25 GB | 100 GB | 200 GB |  |  |  |  |
| límites de índice e indexador| 50 | 200 | 200 |  |  |  |  |

S1 es una opción común para los clientes que necesitan recursos dedicados y varias particiones. S1 ofrece particiones de 25 GB y hasta 12 particiones, lo que proporciona un límite por servicio de 300 GB si maximizar las particiones a través de las réplicas. (Consulte [asignar particiones y réplicas](search-capacity-planning.md#chart) para obtener más información con equilibrio de las asignaciones.)

Las páginas de precios y portal colocan el foco en el almacenamiento y tamaño de la partición, pero, para cada nivel, todos los proceso capacidades (capacidad de disco, velocidad, CPU) generalmente aumenta linealmente con el precio. Una réplica de S2 es más rápido que S1 y S3 es más rápido que S2. Niveles S3 interrupción el modelo de precios de proceso lineal con E/S desproporcionadamente más rápido. Si espera ser el cuello de botella de E/S, tenga en cuenta que puede hacer mucho más IOPS con S3 que se obtiene con un nivel inferior.

S3 y S3 HD están respaldados por la infraestructura de alta capacidad idéntica, pero que lleguen a sus límites máximos de maneras diferentes. S3 tiene como destino un menor número de índices muy grandes, por lo que su límite máximo es el límite de recursos (2,4 TB para cada servicio). S3 HD se dirige a un gran número de índices muy pequeños. A los 1000 índices, S3 HD alcanza sus límites en forma de restricciones de índice. Si es un cliente de S3 HD y necesita más de 1000 índices, póngase en contacto con Microsoft Support para obtener información sobre cómo continuar.

> [!NOTE]
> Los límites de documento eran una consideración al mismo tiempo, pero ya no son aplicables para los nuevos servicios. Para obtener información acerca de las condiciones en las que aplican todavía los límites de documento, vea [límites de documento](search-limits-quotas-capacity.md#document-limits).
>

Capas de almacenamiento optimizado L1 y L2, son ideales para aplicaciones con requisitos de datos de gran tamaño, un número relativamente reducido de los usuarios finales, pero cuando se minimiza la latencia de consulta no es la principal prioridad.  

|  | L1 | L2 |  |  |  |  |  |
|--|----|----|--|--|--|--|--|
| Tamaño de la partición|  1 TB | 2 TB |  |  |  |  |  |
| límites de índice e indexador| 10 | 10 |  |  |  |  |  |

L2 ofrece dos veces la capacidad de almacenamiento general L1.  Elija su plan según la cantidad máxima de datos que cree que necesita el índice. Las particiones de nivel L1 escalar en incrementos de 1 TB para un máximo de 12 TB. Aumentar las particiones de nivel 2 por 2 TB por partición hasta un máximo de 24 TB.

## <a name="evaluating-capacity"></a>Evaluar la capacidad

Capacidad y los costos de ejecutar el servicio están directamente relacionados. Niveles de imponen límites en dos niveles: almacenamiento y recursos. Debería pensar en ambos porque el límite que llegue el primero es el límite real.

Normalmente, los requisitos empresariales exigen el número de índices que necesitará. Por ejemplo, podría necesitar un índice global para un repositorio de documentos de gran tamaño. O bien, puede que tenga varios índices en función de la región, la aplicación o el nicho de negocio.

Para determinar el tamaño de un índice, tendrá que [crear uno](search-create-index-portal.md). La estructura de datos en Azure Search es principalmente un [índice invertido](https://en.wikipedia.org/wiki/Inverted_index) estructura, que tiene características diferentes de datos de origen. Un índice invertido, tamaño y la complejidad se determinan según el contenido, no necesariamente por la cantidad de datos incorporan a él. Un origen de datos de gran tamaño con alta redundancia podría dar lugar a un índice menor que un conjunto de datos más pequeño que incluye contenido muy variable. Por lo que es poco probable deducir el tamaño del índice en función del tamaño del conjunto de datos original.

> [!NOTE] 
> Aunque calcular las necesidades futuras de almacenamiento y los índices puede parecer un suposiciones, merece la pena hacerlo. Si la capacidad de un nivel resulta demasiado bajo, necesitará aprovisionar un nuevo servicio en un nivel superior y, a continuación, [volver a cargar los índices](search-howto-reindex.md). No hay ninguna actualización en contexto de un servicio de una SKU a otra.
>

### <a name="step-1-develop-rough-estimates-by-using-the-free-tier"></a>Paso 1: Desarrollar cálculos aproximados mediante el nivel gratis

Un enfoque para estimar la capacidad es comenzar con el nivel gratis. Recuerde que el servicio gratuito ofrece hasta tres índices, 50 MB de almacenamiento y 2 minutos de tiempo de indización. Puede resultar complicado para calcular un tamaño de índice proyectado con estas restricciones. Este es un enfoque que puede llevar a cabo:

+ [Crear un servicio gratuito](search-create-service-portal.md).
+ Preparar un conjunto de datos representativo, pequeña (por ejemplo, 5.000 documentos y tamaño de muestra de 10 por ciento).
+ [Generar un índice inicial](search-create-index-portal.md) y tenga en cuenta su tamaño en el portal (por ejemplo, 30 MB).

Si el ejemplo es representante y 10 por ciento del origen de datos completo, un índice de 30 MB se convierte en aproximadamente 300 MB si todos los documentos se indexan. Armado con este número preliminar, podría duplicar esa cantidad de presupuesto para dos índices (desarrollo y producción). Esto le da un total de 600 MB en los requisitos de almacenamiento. Este requisito se cumple con facilidad mediante el nivel básico, por lo que podría empezar por ahí.

### <a name="step-2-develop-refined-estimates-by-using-a-billable-tier"></a>Paso 2: Desarrollar las estimaciones refinadas mediante el uso de un nivel facturable

Algunos clientes prefieren iniciar con recursos dedicados que pueden dar cabida a muestreo más grande y tiempos de procesamiento y, a continuación, desarrolle realistas estimaciones de los volúmenes de consultas, el tamaño y cantidad de índice durante el desarrollo. Inicialmente, se aprovisiona un servicio basado en una estimación de la mejor estimación. A continuación, a medida que crezca el proyecto de desarrollo, los equipos suelen saber si el servicio existente está por encima o debajo de la capacidad para cargas de trabajo de producción previsto.

1. [Revise los límites de servicio en cada nivel](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity#index-limits) para determinar si los niveles más bajos pueden admitir el número de índices que necesita. En todos los niveles básico, S1 y S2, límites de índice son 15, 50 y 200, respectivamente. El nivel optimizado de almacenamiento tiene un límite de 10 índices porque se ha diseñado para admitir un número reducido de índices muy grandes.

1. [Cree un servicio en un nivel facturable](search-create-service-portal.md):

    + Iniciar baja, los niveles básico o S1, si está al principio de la curva de aprendizaje.
    + Iniciar alta, en S2 o S3 incluso, si sabe que va a tener cargas de consulta e indización a gran escala.
    + Comience con almacenamiento optimizado, en L1 o L2, si está indexando una gran cantidad de datos y carga de consultas es relativamente bajo, como con una aplicación empresarial interna.

1. [Genere un índice inicial](search-create-index-portal.md) para determinar cómo se traducen los datos de origen a un índice. Esta es la única manera de calcular el tamaño del índice.

1. [Supervise el almacenamiento, los límites del servicio, el volumen de consultas y la latencia](search-monitor-usage.md) en el portal. El portal muestra las consultas por las consultas en segundo lugar, limitadas y latencia de búsqueda. Todos estos valores pueden ayudarle a decidir si ha seleccionado el nivel correcto. También puede configurar la supervisión profunda de los valores como el análisis de Click-through habilitando [análisis de tráfico de búsqueda](search-traffic-analytics.md).

Número de índice y el tamaño son igualmente importantes para el análisis. Esto es porque se alcanzan los límites máximos a través del uso total de almacenamiento (particiones) o por los límites máximos de recursos (índices, indizadores y así sucesivamente), lo que ocurra primero. El portal le ayuda a realizar un seguimiento de ambos, mostrando el uso actual y los límites máximos en paralelo en la página Información general.

> [!NOTE]
> Los requisitos de almacenamiento pueden estar inflados si los documentos contienen datos extraños. Idealmente, los documentos contienen solo los datos que necesita para la experiencia de búsqueda. Datos binarios no se pueden buscar y deben almacenarse por separado (quizás en un almacenamiento de Azure blobs o tablas). A continuación, se debe agregar un campo en el índice para mantener una referencia de URL a los datos externos. El tamaño máximo de un documento individual es 16 MB (o menos si eres varios documentos en una solicitud de carga en masa). Para más información, consulte [Límites de servicio en Azure Search](search-limits-quotas-capacity.md).
>

**Consideraciones del volumen de consultas**

Consultas por segundo (QPS) es una métrica importante durante la optimización del rendimiento, pero generalmente es sólo una consideración de nivel si espera volumen elevado de consultas desde el principio.

Los niveles estándar pueden proporcionar un equilibrio entre réplicas y particiones. Puede aumentar el tiempo de respuesta de consulta mediante la adición de réplicas para el equilibrio de carga o agregar particiones para el procesamiento paralelo. A continuación, puede optimizar rendimiento después de aprovisionar el servicio.

Si espera elevados sostenida volúmenes de consultas desde el principio, debe considerar los planes estándares superiores, respaldados por un hardware más eficaz. Puede poner a las particiones y réplicas sin conexión o incluso cambiar a un servicio de nivel inferior, si no se producen esos volúmenes de consulta. Para más información sobre cómo calcular el rendimiento de las consultas, consulte [Rendimiento y optimización de Azure Search](search-performance-optimization.md).

Los niveles de almacenamiento optimizado son útiles para cargas de trabajo de datos de gran tamaño, compatibilidad con más almacenamiento del índice global disponible para cuando los requisitos de latencia de consulta son menos importantes. Todavía debe usar réplicas adicionales para las particiones adicionales y equilibrio de carga para el procesamiento paralelo. A continuación, puede optimizar rendimiento después de aprovisionar el servicio.

**Acuerdos de nivel de servicio**

No proporcionan las características gratuitas y preliminares [acuerdos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/search/v1_0/). Para todos los niveles facturables, los SLA tomarán efecto cuando se aprovisione suficiente redundancia para el servicio. Debe tener dos o más réplicas para SLA de consulta (lectura). Debe tener tres o más réplicas para SLA (lectura y escritura) de indexación y consulta. El número de particiones no afecta a los SLA.

## <a name="tips-for-tier-evaluation"></a>Sugerencias para la evaluación de nivel

+ Aprenda a crear índices eficaces y obtenga información sobre qué métodos de actualización tienen menos impacto. Use [análisis de tráfico de búsqueda](search-traffic-analytics.md) para obtener información sobre la actividad de consultas.

+ Permitir que las métricas compilar en torno a las consultas y recopilar datos en torno a patrones de uso (consultas durante el horario laboral, durante las horas de la indización). Use estos datos para informar a las decisiones de aprovisionamiento de servicio. Aunque no resulta práctico en una cadencia de hora o por día, puede ajustar dinámicamente las particiones y recursos para dar cabida a los cambios planeados en volúmenes de consultas. También puede adaptarse a cambios no planeados, pero sostenidos si los niveles de contienen suficiente para garantizar el realizar ninguna acción.

+ Recuerde que la única desventaja de underprovisioning es posible que deba eliminar un servicio si los requisitos reales son mayores que las predicciones. Para evitar la interrupción del servicio, se crea un nuevo servicio en la misma suscripción en un nivel superior y se ejecuta en paralelo hasta que todas las solicitudes y las aplicaciones se dirijan al nuevo punto de conexión.

## <a name="next-steps"></a>Pasos siguientes

Comience con un nivel gratuito y cree un índice inicial mediante el uso de un subconjunto de los datos para comprender sus características. La estructura de datos en Azure Search es una estructura de índice invertido. El tamaño y la complejidad de un índice invertido viene determinada por el contenido. Recuerde que un contenido de gran redundancia tiende a producir un índice más pequeño que el contenido de alta irregularidad. Por lo que las características de contenido en lugar de con el tamaño del conjunto de datos determinan los requisitos de almacenamiento del índice.

Una vez que tenga una estimación inicial del tamaño de los índices, [aprovisionar un servicio facturable](search-create-service-portal.md) en uno de los niveles descritos en este artículo: Básico, estándar u optimizada para almacenamiento. Relajar las restricciones sobre el tamaño de datos artificiales y [a generar el índice](search-howto-reindex.md) para incluir todos los datos que desea que permitan realizar búsquedas.

[Asigne particiones y réplicas](search-capacity-planning.md) según sea necesario para obtener el rendimiento y la escala que necesite.

Si la capacidad y rendimiento están bien, ya ha terminado. En caso contrario, vuelva a crear un servicio de búsqueda en otro nivel que se adapte más estrechamente a sus necesidades.

> [!NOTE]
> Si tiene preguntas, publíquelas en [StackOverflow](https://stackoverflow.com/questions/tagged/azure-search) o [póngase en contacto con soporte técnico de Azure](https://azure.microsoft.com/support/options/).
