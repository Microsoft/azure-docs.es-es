---
title: Aislamiento multiinquilino y contenido
titleSuffix: Azure Cognitive Search
description: Aprenda sobre patrones de diseño comunes para aplicaciones SaaS multiinquilino mientras usa Azure Cognitive Search.
manager: nitinme
author: LiamCavanagh
ms.author: liamca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: ea0dac74d4f995e41513b3451dd28d177040e672
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935031"
---
# <a name="design-patterns-for-multitenant-saas-applications-and-azure-cognitive-search"></a>Modelos de diseño para aplicaciones SaaS multiinquilino y Azure Cognitive Search

Una aplicación multiinquilino es una que proporciona los mismos servicios y funcionalidades a cualquier número de inquilinos que no pueden ver o compartir los datos de cualquier otro inquilino. En este documento se describen estrategias de aislamiento de inquilinos para aplicaciones miltiinquilino creadas con Azure Cognitive Search.

## <a name="azure-cognitive-search-concepts"></a>Conceptos de Azure Cognitive Search
Como solución de búsqueda como servicio, [Azure Cognitive Search](search-what-is-azure-search.md) permite a los desarrolladores agregar completas experiencias de búsqueda a las aplicaciones sin necesidad de administrar ninguna infraestructura ni de convertirse en expertos en recuperación de información. Los datos se cargan en el servicio y luego se almacenan en la nube. Mediante sencillas solicitudes a la API de Azure Cognitive Search, los datos se pueden modificar y buscar. 

### <a name="search-services-indexes-fields-and-documents"></a>Servicios de búsqueda, índices, campos y documentos

Antes de analizar los patrones de diseño, es importante comprender algunos conceptos básicos.

Al utilizar Azure Cognitive Search, el usuario se suscribe a un *servicio de búsqueda*. A medida que se cargan datos en Azure Cognitive Search, se almacenan en un *índice* dentro del servicio de búsqueda. Puede haber varios índices dentro de un único servicio. Para usar los conceptos de bases de datos que ya conocemos, el servicio de búsqueda se puede vincular a una base de datos, mientras que los índices de un servicio se pueden vincular a tablas dentro de una base de datos.

Cada índice dentro de un servicio de búsqueda tiene su propio esquema, que viene definido por in número de *campos*personalizables. Los datos se agregan a un índice de Azure Cognitive Search en forma de *documentos*individuales. Cada documento se debe cargar en un índice determinado y debe encajar en el esquema de ese índice. Al buscar datos mediante Azure Cognitive Search, las consultas de búsqueda de texto completo se emiten contra un índice determinado.  Para comparar estos conceptos con los de una base de datos, se pueden vincular campos a columnas de una tabla y se pueden vincular documentos a filas.

### <a name="scalability"></a>Escalabilidad
Cualquier servicio de Azure Cognitive Search con el [plan de tarifa](https://azure.microsoft.com/pricing/details/search/) Estándar se puede escalar en dos dimensiones: almacenamiento y disponibilidad.

* *particiones* para aumentar el almacenamiento de un servicio de búsqueda.
* *réplicas* a un servicio para aumentar el rendimiento de las solicitudes que puede administrar un servicio de búsqueda.

Agregar y quitar particiones y réplicas permitirá que crezca la capacidad del servicio de búsqueda con la cantidad de datos y el tráfico que demanda la aplicación. Para que un servicio de búsqueda consiga un [SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/)de lectura, se requieren dos réplicas. Para que un servicio de búsqueda consiga un [SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/)de lectura y escritura, se requieren tres réplicas.

### <a name="service-and-index-limits-in-azure-cognitive-search"></a>Límites de servicio e índice en Azure Cognitive Search
Hay algunos [planes de tarifas](https://azure.microsoft.com/pricing/details/search/) en Azure Cognitive Search, y cada uno de ellos tiene diferentes [límites y cuotas](search-limits-quotas-capacity.md). Algunos de estos límites están en el nivel de servicio, otros en el nivel de índice y otros en el nivel de partición.

|  | Básico | Standard1 | Standard2 | Standard3 | Standard3 HD |
| --- | --- | --- | --- | --- | --- |
| **Número máximo de réplicas por servicio** |3 |12 |12 |12 |12 |
| **Número máximo de particiones por servicio** |1 |12 |12 |12 |3 |
| **Número máximo de unidades de búsqueda (réplicas * particiones) por servicio** |3 |36 |36 |36 |36 (3 particiones como máximo) |
| **Almacenamiento máximo por servicio** |2 GB |300 GB |1,2 TB |2,4 TB |600 GB |
| **Almacenamiento máximo por partición** |2 GB |25 GB |100 GB |200 GB |200 GB |
| **Número máximo de índices por servicio** |5 |50 |200 |200 |3000 (1000 índices/partición como máximo) |

#### <a name="s3-high-density"></a>S3 alta densidad'
En el plan de tarifa S3 de Azure Cognitive Search, hay una opción para el modo de alta densidad (HD) diseñada específicamente para los escenarios de varios inquilinos. En muchos casos, es necesario admitir un gran número de inquilinos más pequeños en un único servicio para lograr las ventajas de la simplicidad y la rentabilidad.

S3 HD permite que los muchos índices pequeños se empaqueten bajo la administración de un solo servicio de búsqueda único mediante el intercambio de la capacidad de escalar horizontalmente índices usando particiones para la capacidad de hospedar más índices en un único servicio.

Un servicio S3 está diseñado para hospedar un número fijo de índices (200 máximo) y permite que cada índice se reduzca horizontalmente a medida que se agregan nuevas particiones al servicio. Al agregar particiones a servicios de S3 HD, se aumenta el número máximo de índices que puede hospedar el servicio. El tamaño máximo ideal para un índice de S3 HD individual es de aproximadamente 50 a 80 GB, aunque no hay ningún límite de tamaño estricto en cada índice impuesto por el sistema.

## <a name="considerations-for-multitenant-applications"></a>Consideraciones sobre las aplicaciones multiinquilino
Las aplicaciones multiinquilino deben distribuir de forma eficaz los recursos entre los inquilinos y conservar al mismo tiempo cierto nivel de privacidad entre los distintos inquilinos. Existen algunas consideraciones que se deben tener en cuenta al diseñar la arquitectura para aplicaciones de este tipo:

* *Aislamiento de inquilinos*: los desarrolladores de aplicaciones necesitan tomar las medidas adecuadas para garantizar que ningún inquilino tenga acceso no autorizado o no deseado a los datos de otros inquilinos. Más allá de la perspectiva de privacidad de los datos, las estrategias de aislamiento de inquilinos requieren la administración eficaz de los recursos compartidos y la protección frente a vecinos ruidosos.
* *Costo de los recursos en la nube*: al igual que sucede con cualquier otra aplicación, las soluciones de software deben tener un costo competitivo en cuanto componente de una aplicación multiinquilino.
* *Facilidad de operaciones:* al desarrollar una arquitectura multiinquilino, es importante tener en cuenta el impacto en las operaciones y la complejidad de la aplicación. Azure Cognitive Search tiene un [Acuerdo de Nivel de Servicio de 99,9 %](https://azure.microsoft.com/support/legal/sla/search/v1_0/).
* *Superficie global:* las aplicaciones multiinquilino pueden tener que atender de forma eficaz a inquilinos que se encuentran distribuidos por todo el planeta.
* *Escalabilidad:* los desarrolladores de aplicaciones deben tener en cuenta el equilibrio entre mantener un nivel de complejidad de la aplicación lo suficientemente bajo y diseñar la aplicación para escalarse con el número de inquilinos y el tamaño de los datos y la carga de trabajo de los inquilinos.

Azure Cognitive Search ofrece algunos límites que se pueden usar para aislar los datos y la carga de trabajo de los inquilinos.

## <a name="modeling-multitenancy-with-azure-cognitive-search"></a>Modelado multiinquilino con Azure Cognitive Search
En el caso de un escenario de varios inquilinos, el desarrollador de la aplicación consume uno o varios servicios de búsqueda y divide sus inquilinos entre servicios, índices o ambos. Azure Cognitive Search presenta algunos patrones comunes al modelar un escenario multiinquilino:

1. *Índice por inquilino:* cada inquilino tiene su propio índice dentro de un servicio de búsqueda que comparte con otros inquilinos.
2. *Servicio por inquilino:* Cada inquilino tiene su propio servicio de Azure Cognitive Search dedicado, que ofrece el nivel más alto de separación de datos y carga de trabajo.
3. *Mezcla de ambos:* los inquilinos más grandes y activos se asignan a servicios dedicados, mientras que los inquilinos más pequeños se asignan a índices individuales dentro de servicios compartidos.

## <a name="1-index-per-tenant"></a>1. Índice por inquilino
![Una representación del modelo de índice por inquilino](./media/search-modeling-multitenant-saas-applications/azure-search-index-per-tenant.png)

En un modelo de índice por inquilino, varios inquilinos ocupan un único servicio de Azure Cognitive Search donde cada inquilino tiene su propio índice.

Los inquilinos consiguen el aislamiento de los datos porque todas las solicitudes de búsqueda y las operaciones de documentos se emiten en un nivel de índice en Azure Cognitive Search. En el nivel de aplicación, existe el conocimiento necesario para dirigir el tráfico de los diversos inquilinos a los índices adecuados, al tiempo también que se administran los recursos en el nivel de servicio entre todos los inquilinos.

Un atributo clave del modelo de índice por inquilino es la posibilidad para el desarrollador de la aplicación de saturar la capacidad del número de suscripciones de un servicio de búsqueda entre los inquilinos de la aplicación. Si los inquilinos tienen una distribución desigual de la carga de trabajo, la combinación óptima de inquilinos se puede distribuir entre los índices de un servicio de búsqueda para dar cabida a un número de inquilinos muy activos, que consumen muchos recursos mientras se atiende simultáneamente una minoría de inquilinos menos activos. El inconveniente es la imposibilidad de que el modelo controle situaciones en las que cada inquilino está muy activo al mismo tiempo.

El modelo de índice por inquilino proporciona la base para un modelo de costo variable, donde un servicio entero de Azure Cognitive Search se compra por adelantado y posteriormente se rellena de inquilinos. Esto permite designar la capacidad sin usar para pruebas y cuentas gratuitas.

Es posible que para las aplicaciones con una superficie global, el modelo de índice por inquilino no sea el más adecuado. Si los inquilinos de una aplicación se distribuyen por todo el planeta, puede que sea necesario un servicio aparte para cada región, lo que podría duplicar los costos en cada una de ellas.

Azure Cognitive Search permite escalar tanto los índices individuales como aumentar el número total de índices. Si se elige un plan de tarifa adecuado, se pueden agregar particiones y réplicas al servicio de búsqueda entero cuando un índice individual del servicio crezca demasiado en términos de almacenamiento o tráfico.

Si el número total de índices crece demasiado para un único servicio, se debe aprovisionar otro servicio para acomodar a los nuevos inquilinos. Si se tienen que mover índices entre servicios de búsqueda a medida que se agregan nuevos servicios, los datos del índice se tienen que copiar manualmente de un índice a otro porque Azure Cognitive Search no permite que se mueva un índice.

## <a name="2-service-per-tenant"></a>2. Servicio por inquilino
![Una representación del modelo de servicio por inquilino](./media/search-modeling-multitenant-saas-applications/azure-search-service-per-tenant.png)

En una arquitectura de servicio por inquilino, cada inquilino tiene su propio servicio de búsqueda.

En este modelo, la aplicación obtiene el nivel máximo de aislamiento para sus inquilinos. Cada servicio tiene almacenamiento y capacidad de proceso dedicados para administrar las solicitudes de búsqueda, así como claves de API independientes.

En aplicaciones donde cada inquilino tiene una superficie grande o la carga de trabajo tiene poca variabilidad de un inquilino a otro, el modelo de servicio por inquilino es una opción eficaz porque no se comparten recursos entre las cargas de trabajo de los diversos inquilinos.

Un modelo de servicio por inquilino también ofrece la ventaja de un modelo predecible de costo fijo. No hay ninguna inversión anticipada en un servicio de búsqueda entero hasta que haya un inquilino para llenarlo; sin embargo, el costo por inquilino es más alto que el de un modelo de índice por inquilino.

El modelo de servicio por inquilino es una opción eficaz para aplicaciones con una superficie global. Cuando los inquilinos están distribuidos geográficamente, es fácil tener el servicio de cada inquilino en la región adecuada.

Las dificultades para escalar este patrón surgen cuando inquilinos individuales crecen por encima de la capacidad de su servicio. Azure Cognitive Search no admite actualmente la actualización del plan de tarifa de un servicio de búsqueda, así que todos los datos se tendrían que copiar manualmente en un nuevo servicio.

## <a name="3-mixing-both-models"></a>3. Mezcla de ambos modelos
Otro patrón para modelar la arquitectura multiinquilino es mezclar las estrategias de índice por inquilino y servicio por inquilino.

Al mezclar ambos patrones, los inquilinos más grandes de la aplicación pueden ocupar servicios dedicados, mientras que la cola más larga de inquilinos más pequeños y menos activos pueden ocupar índices de un servicio compartido. Este modelo garantiza que los inquilinos más grandes tienen sistemáticamente un mayor rendimiento desde el servicio, al mismo tiempo que ayuda a proteger los inquilinos más pequeños frente a cualquier vecino ruidoso.

Sin embargo, la implementación de esta estrategia depende de la previsión de qué inquilinos necesitarán un servicio dedicado, por oposición a un índice en un servicio compartido. La complejidad de la aplicación aumenta con la necesidad de administrar ambos modelos multiinquilino.

## <a name="achieving-even-finer-granularity"></a>Obtención incluso de una granularidad más fina
En los patrones de diseño anteriores para modelar escenarios muliinquilino en Azure Cognitive Search se asume un ámbito uniforme donde cada inquilino es una instancia entera de una aplicación. Sin embargo, en ocasiones las aplicaciones pueden controlar ámbitos mucho más pequeños.

Si los modelos de servicio por inquilino o índice por inquilino no son ámbitos lo suficientemente pequeños, es posible modelar un índice para lograr un grado de granularidad más fino si cabe.

Para que un único índice se comporte de forma diferente para diferentes puntos de conexión de cliente, se puede agregar un campo a un índice para designar un determinado valor para cada posible cliente. Cada vez que un cliente llama a Azure Cognitive Search para consultar o modificar un índice, el código de la aplicación cliente especifica el valor adecuado para ese campo mediante la funcionalidad de [filtro](./query-odata-filter-orderby-syntax.md) de Azure Cognitive Search durante la consulta.

Este método puede usarse para lograr la funcionalidad de cuentas de usuario independientes, niveles de permisos independientes e incluso aplicaciones completamente independientes.

> [!NOTE]
> El enfoque descrito anteriormente para configurar un solo índice único y dar servicio a varios inquilinos afecta a la relevancia de los resultados de la búsqueda. Las puntuaciones de importancia de la búsqueda se calculan en un ámbito de nivel del índice, no en el del inquilino, por lo que los datos de todos los inquilinos se incorporan en las estadísticas subyacentes de las puntuaciones de relevancia como frecuencia de término.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
Azure Cognitive Search es una opción atractiva para muchas aplicaciones. Al evaluar los diversos patrones de diseño para aplicaciones multiinquilino, tenga en cuenta los [distintos planes de tarifa](https://azure.microsoft.com/pricing/details/search/) y sus respectivos [límites de servicio](search-limits-quotas-capacity.md) para adaptar mejor Azure Cognitive Search de forma que encaje en cargas de trabajo de aplicaciones y arquitecturas de todos los tamaños.

Las preguntas sobre Azure Cognitive Search y los escenarios multiinquilino se pueden dirigir a azuresearch_contact@microsoft.com.