---
title: Escalado para mejorar el rendimiento
titleSuffix: Azure Cognitive Search
description: Conozca las técnicas y los procedimientos recomendados para optimizar el rendimiento de Azure Cognitive Search y configurar la escala óptima.
manager: nitinme
author: LiamCavanagh
ms.author: liamca
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: 5fd949466978714fe1dc0c4ccc67a3cb8f993314
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934963"
---
# <a name="scale-for-performance-on-azure-cognitive-search"></a>Escalado para mejorar el rendimiento en Azure Cognitive Search

En este artículo se describen las prácticas recomendadas para escenarios avanzados con requisitos sofisticados de escalabilidad y disponibilidad.

## <a name="start-with-baseline-numbers"></a>Inicio con números de base de referencia

Antes de asumir un mayor esfuerzo de implementación, asegúrese de que sabe cómo es una carga de consulta típica. Las instrucciones siguientes pueden ayudarle a llegar a números de consulta de línea de base.

1. Seleccionar una latencia objetivo (o cantidad máxima de tiempo) que una solicitud de búsqueda típica debería tardar en completarse.

1. Cree y pruebe una carga de trabajo real en su servicio de búsqueda con un conjunto de datos realista para medir estas tasas de latencia.

1. Comience con un número reducido de consultas por segundo (QPS) y aumente gradualmente el número que se ejecuta en la prueba hasta que la latencia de las consultas caiga por debajo del destino predefinido. Esta es una prueba comparativa importante que le ayudará a planear el escalado a medida que el uso de la aplicación crece.

1. Siempre que sea posible, reutilice las conexiones HTTP. Si usa el SDK para .NET de Azure Cognitive Search, debe reutilizar una instancia de [SearchIndexClient](/dotnet/api/microsoft.azure.search.searchindexclient), y si usa la API REST, debe reutilizar un única instancia de HttpClient.

1. Varíe el contenido de las solicitudes de consulta para que la búsqueda se realice a través de distintas partes del índice. La variación es importante porque si ejecuta continuamente las mismas solicitudes de búsqueda, el almacenamiento en caché de los datos empezará a ofrecer un rendimiento mejor que el que se obtendría con un conjunto de consultas más dispar.

1. Varíe la estructura de las solicitudes de consulta para obtener diferentes tipos de consultas. No todas las consultas de búsqueda se realizan en el mismo nivel. Por ejemplo, una sugerencia de búsqueda o la búsqueda de documentos normalmente es más rápida que una consulta con un número significativo de facetas y filtros. La composición de prueba debe incluir varias consultas en las mismas proporciones aproximadamente, tal como se esperaría en producción.  

Al crear estas cargas de trabajo de prueba, hay algunas características de Azure Cognitive Search que debe tener en cuenta:

+ Es posible sobrecargar el servicio mediante la inserción de demasiadas consultas de búsqueda al mismo tiempo. Cuando esto sucede, verá códigos de respuesta HTTP 503. Para evitar un código 503 durante las pruebas, empiece con varios intervalos de solicitudes de búsqueda para ver las diferencias en las tasas de latencia a medida que se agregan más solicitudes de búsqueda.

+ Azure Cognitive Search no ejecuta tareas de indexación en segundo plano. Si el servicio controla al mismo tiempo las cargas de trabajo de indexación y consulta, debe tener esto en cuenta mediante la introducción de trabajos de indexación en las pruebas de consulta o la exploración de opciones para ejecutar trabajos de indexación durante las horas de menor actividad.

> [!Tip]
> Puede simular una carga de consulta realista mediante herramientas de prueba de carga. Pruebe la [prueba de carga con Azure DevOps](/azure/devops/test/load-test/get-started-simple-cloud-load-test?view=azure-devops) o use una de estas [alternativas](/azure/devops/test/load-test/overview?view=azure-devops#alternatives).

## <a name="scale-for-high-query-volume"></a>Escalado para lograr un alto volumen de consultas

Un servicio se sobrecarga cuando las consultas tardan demasiado tiempo o cuando el servicio empieza a eliminar solicitudes. Si esto sucede, puede solucionar el problema de una de estas dos formas:

+ **Agregar réplicas**  

  Cada réplica es una copia de los datos, lo que permite que el servicio equilibre la carga de las solicitudes en varias copias.  Azure Cognitive Search administra el equilibrado de la carga y la replicación de los datos, y puede modificar el número de réplicas asignado a su servicio en cualquier momento. Puede asignar hasta 12 réplicas en un servicio de búsqueda estándar y 3 réplicas en un servicio de búsqueda básico. Las réplicas pueden ajustarse desde [Azure Portal](search-create-service-portal.md) o [PowerShell](search-manage-powershell.md).

+ **Crear un nuevo servicio en un nivel superior**  

  Azure Cognitive Search se ofrece con [diferentes niveles](https://azure.microsoft.com/pricing/details/search/), y cada uno de ellos proporciona distintos grados de rendimiento. En algunos casos, puede tener tantas consultas que el nivel en que se encuentra no puede proporcionar una respuesta suficiente, ni siquiera cuando se usa el máximo de réplicas. En este caso, considere la posibilidad de pasar a un nivel de rendimiento superior, como el nivel Estándar S3, que está diseñado para escenarios con un gran número de documentos y cargas de trabajo de consulta extremadamente altas.

## <a name="scale-for-slow-individual-queries"></a>Escalado para consultas individuales lentas

Otro motivo para las tasas de latencia altas es una única consulta que tarda demasiado tiempo en completarse. En este caso, agregar réplicas no ayudará. A continuación, se indican dos opciones posibles que podrían ayudar:

+ **Aumentar particiones**

  Una partición divide los datos entre recursos informáticos adicionales. Dos particiones dividen los datos por la mitad, una tercera partición los divide en tercios, etc. Un efecto secundario positivo es que las consultas más lentas a veces se ejecutan más rápido debido a la computación en paralelo. Se observó la paralelización de consultas de selectividad baja, como las consultas que coinciden con muchos documentos, o facetas que proporcionan recuentos a través de un gran número de documentos. Como se necesita una gran cantidad de cálculo para puntuar la relevancia de los documentos o para contar el número de documentos, agregar particiones adicionales ayuda a que las consultas se completen más rápido.  
   
  En el servicio de búsqueda Estándar puede haber un máximo de 12 particiones, mientras que en el servicio de búsqueda Básico el máximo es 1. Las particiones pueden ajustarse desde [Azure Portal](search-create-service-portal.md) o [PowerShell](search-manage-powershell.md).

+ **Limitar los campos de alta cardinalidad**

  Un campo de alta cardinalidad consta de un campo clasificable o filtrable que tiene un número significativo de valores únicos y, como resultado, consume recursos importantes al procesar los resultados. Por ejemplo, el establecimiento de un campo de identificador de producto o de descripción como filtrable o clasificable contaría como alta cardinalidad ya que la mayoría de los valores de un documento a otro son únicos. Siempre que sea posible, limite el número de campos de alta cardinalidad.

+ **Aumentar el nivel de búsqueda**  

  moverse a un nivel superior de Azure Cognitive Search puede ser otra manera de mejorar el rendimiento de las consultas lentas. Cada nivel superior proporciona CPU más rápidas y más memoria, aspectos que tienen un impacto positivo en el rendimiento de las consultas.

## <a name="scale-for-availability"></a>Escalado para lograr disponibilidad

Las réplicas no solo ayudan a reducir la latencia de las consultas, sino que también permiten una alta disponibilidad. Con una sola réplica, puede esperar que se produzcan tiempos de inactividad periódicamente debido al reinicio del servidor después de las actualizaciones de software o por otros posibles eventos de mantenimiento. Como resultado, es importante tener en cuenta si la aplicación requiere alta disponibilidad de búsquedas (consultas), así como de escrituras (eventos de indexación). Azure Cognitive Search ofrece opciones de Acuerdo de Nivel de Servicio en todas las ofertas de búsqueda de pago con los siguientes atributos:

+ Dos réplicas para alta disponibilidad de cargas de trabajo de solo lectura (consultas)

+ Tres o más réplicas para lograr alta disponibilidad de las cargas de trabajo de lectura-escritura (consultas e indexación)

Para más información al respecto, visite el [Acuerdo de Nivel de Servicio de Azure Cognitive Search](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

Como las réplicas son copias de los datos, tener varias réplicas permite que Azure Cognitive Search realice tareas de reinicio y mantenimiento de la máquina en una réplica, mientras que la ejecución de consultas continúa en otras. Por el contrario, si quita réplicas, provocará la degradación del rendimiento de consulta, suponiendo que esas réplicas eran un recurso infrautilizado.

## <a name="scale-for-geo-distributed-workloads-and-geo-redundancy"></a>Escalado de cargas de trabajo distribuidas geográficamente y de redundancia geográfica

En las cargas de trabajo distribuidas geográficamente, los usuarios que se encuentren lejos del centro de datos de host tendrán mayores tasas de latencia. Una solución consiste en aprovisionar varios servicios de búsqueda en regiones más próximas a estos usuarios.

Azure Cognitive Search no proporciona actualmente un método automatizado de replicación geográfica de índices de Azure Cognitive Search entre regiones, pero existen algunas técnicas que permiten simplificar la implementación y administración de este proceso. Se describen en las siguientes secciones.

El objetivo de un conjunto de servicios de búsqueda distribuido geográficamente es tener dos o más índices disponibles en dos o más regiones, en las que un usuario se enruta al servicio Azure Cognitive Search que proporciona la latencia más baja, como se muestra en este ejemplo:

   ![Tablas de referencias cruzadas de servicios por región][1]

### <a name="keep-data-synchronized-across-multiple-services"></a>Mantenimiento de los datos sincronizados entre varios servicios

Hay dos opciones para mantener sincronizados los servicios de búsqueda distribuidos: usar el [indexador de Azure Cognitive Search](search-indexer-overview.md) o la Push API (también denominada [API REST de Azure Cognitive Search](/rest/api/searchservice/)).  

### <a name="use-indexers-for-updating-content-on-multiple-services"></a>Uso de indexadores para la actualización de contenido en varios servicios

Si ya usa un indexador en un servicio, puede configurar un segundo indexador en un segundo servicio para usar el mismo objeto de origen de datos, extrayendo datos de la misma ubicación. Cada servicio en cada región tiene su propio indexador y un índice de destino (el índice de búsqueda no se comparte, lo que significa que los datos se duplican), pero cada indexador hace referencia al mismo origen de datos.

Este es un objeto visual de alto nivel del aspecto que podría tener esa arquitectura.

   ![Origen de datos único con indexador distribuido y combinaciones de servicios][2]

### <a name="use-rest-apis-for-pushing-content-updates-on-multiple-services"></a>Uso de las API REST para insertar actualizaciones de contenido en varios servicios

Si usa la API REST de Azure Cognitive Search para [insertar contenido en el índice de Azure Cognitive Search](/rest/api/searchservice/update-index) y quiere mantener sincronizados los distintos servicios de búsqueda, inserte los cambios en todos los servicios de búsqueda cada vez que se necesite una actualización. En el código, asegúrese de controlar los casos en los que una actualización en un servicio de búsqueda produce un error, pero se ejecuta correctamente en otros servicios de búsqueda.

## <a name="leverage-azure-traffic-manager"></a>Aprovechamiento de Azure Traffic Manager

[Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) permite enrutar solicitudes a diversos sitios web con ubicación geográfica atendidos por distintos servicios Azure Cognitive Search. Una ventaja de Traffic Manager es que puede sondear Azure Cognitive Search para asegurarse de que está disponible y enrutar los usuarios a servicios de búsqueda alternativos en caso de tiempo de inactividad. Además, si va a enrutar solicitudes de búsqueda a través de Azure Web Sites, Azure Traffic Manager permite equilibrar la carga de aquellos casos en los que el sitio web está activo pero Azure Cognitive Search no. Este es un ejemplo de una arquitectura que utiliza el Administrador de tráfico.

   ![Tablas de referencias cruzadas de servicios por región con Administrador de tráfico central][3]

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los planes de tarifa y los límites de los servicios de cada uno de ellos, consulte [Límites de servicio](search-limits-quotas-capacity.md). Consulte [Planeación de capacidad](search-capacity-planning.md) para obtener información acerca de las combinaciones de particiones y réplicas.

Para obtener una explicación sobre el rendimiento y las demostraciones de las técnicas descritas en este artículo, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 

<!--Image references-->
[1]: ./media/search-performance-optimization/geo-redundancy.png
[2]: ./media/search-performance-optimization/scale-indexers.png
[3]: ./media/search-performance-optimization/geo-search-traffic-mgr.png