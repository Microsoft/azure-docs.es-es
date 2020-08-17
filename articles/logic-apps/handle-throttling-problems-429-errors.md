---
title: Solucionar problemas de limitación, o errores "429 - Demasiadas solicitudes"
description: Solución de problemas de limitación, o errores "429 - Demasiadas solicitudes", en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 04/13/2020
ms.openlocfilehash: 495847d31682aff64fed3c81b1d5d68cf67dfd38
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086447"
---
# <a name="handle-throttling-problems-429---too-many-requests-errors-in-azure-logic-apps"></a>Solución de problemas de limitación (429 - "Demasiadas solicitudes") en Azure Logic Apps

En [Azure Logic Apps](../logic-apps/logic-apps-overview.md), la aplicación lógica devuelve un error ["HTTP 429 Demasiadas solicitudes"](https://developer.mozilla.org/docs/Web/HTTP/Status/429) al experimentar la limitación, lo que sucede cuando el número de solicitudes supera la velocidad a la que el destino puede responder durante un período de tiempo específico. La limitación puede crear problemas como retraso en el procesamiento de datos o reducción en la velocidad de rendimiento y errores como la superación de la directiva de reintentos especificada.

![Limitación en el conector de SQL Server](./media/handle-throttling-problems-429-errors/example-429-too-many-requests-error.png)

Estos son algunos tipos comunes de limitaciones que puede experimentar la aplicación lógica:

* [Aplicación lógica](#logic-app-throttling)
* [Conector](#connector-throttling)
* [Servicio o sistema de destino](#destination-throttling)

<a name="logic-app-throttling"></a>

## <a name="logic-app-throttling"></a>Limitación de aplicaciones lógicas

El servicio Azure Logic Apps tiene sus propios [límites de rendimiento](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Por lo tanto, si la aplicación lógica supera estos límites, el recurso de la aplicación lógica se ve limitado, no solo una instancia o ejecución específica.

Para buscar eventos de limitación en este nivel, consulte el panel **Métricas** de la aplicación lógica en Azure Portal.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Supervisión**, seleccione **Métricas**.

1. En **Título del gráfico**, seleccione **Agregar métrica** para agregar otra métrica a la existente.

1. En la primera barra de métricas, en la lista **MÉTRICA**, seleccione **Eventos limitados de acciones**. En la segunda barra de métricas, en la lista **MÉTRICA**, seleccione **Eventos limitados del desencadenador**.

Para controlar la limitación en este nivel, tiene estas opciones:

* Limite el número de instancias de aplicación lógica que se pueden ejecutar al mismo tiempo.

  De forma predeterminada, si la condición de desencadenador de la aplicación lógica se cumple más de una vez al mismo tiempo, se ejecutan varias instancias del desencadenador para la aplicación lógica de manera simultánea o *en paralelo*. Este comportamiento implica la activación de cada instancia del desencadenador antes de que finalice la ejecución de la instancia del flujo de trabajo anterior.

  Aunque el número predeterminado de instancias de desencadenador que se puede ejecutar simultáneamente es [ilimitado](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits), puede limitar este número [activando la configuración de simultaneidad del desencadenador](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) y, si es necesario, seleccionar un límite distinto del valor predeterminado.

* Habilite el modo de alto rendimiento.

  Una aplicación lógica tiene un [límite predeterminado para el número de acciones que se pueden ejecutar en un intervalo gradual de 5 minutos](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Para aumentar este límite para el número máximo de acciones, active el [modo de alto rendimiento](../logic-apps/logic-apps-workflow-actions-triggers.md#run-high-throughput-mode) en la aplicación lógica.

* Deshabilite el comportamiento de desagrupación de matrices ("división") en los desencadenadores.

  Si un desencadenador devuelve una matriz para las acciones de flujo de trabajo restantes que se van a procesar, el [ajuste de **división**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch) del desencadenador divide los elementos de la matriz e inicia una instancia de flujo de trabajo para cada elemento de la matriz, con lo que se desencadenan de manera eficaz varias ejecuciones simultáneas hasta el [límite de **división**](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits). Para controlar la limitación, desactive el comportamiento de **división** y haga que la aplicación lógica procese toda la matriz con una sola llamada, en lugar de controlar un solo elemento por llamada.

* Refactorice acciones en aplicaciones lógicas más pequeñas.

  Como se mencionó anteriormente, una aplicación lógica está limitada a un [número predeterminado de acciones que pueden ejecutarse durante un período de 5 minutos](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Aunque puede aumentar este límite habilitando el [modo de alto rendimiento](../logic-apps/logic-apps-workflow-actions-triggers.md#run-high-throughput-mode), también puede considerar si desea desglosar las acciones de la aplicación lógica en aplicaciones lógicas más pequeñas para que el número de acciones que se ejecutan en cada aplicación lógica permanezca bajo el límite. De este modo, reducirá la carga en un único recurso de aplicación lógica y distribuirá la carga entre varias aplicaciones lógicas. Esta solución funciona mejor para las acciones que administran grandes conjuntos de datos o establecen tantas acciones en ejecución, iteraciones de bucle o acciones dentro de cada iteración de bucle a la vez que superan el límite de ejecución de la acción.

  Por ejemplo, esta aplicación lógica realiza todo el trabajo para obtener tablas de una base de datos de SQL Server y obtiene las filas de cada tabla. El bucle **For Each** recorre en iteración simultáneamente cada tabla para que la acción **Obtener filas** devuelva las filas de cada tabla. En función de las cantidades de datos de esas tablas, estas acciones pueden superar el límite en las ejecuciones de acciones.

  ![Aplicación lógica "antes" de la refactorización](./media/handle-throttling-problems-429-errors/refactor-logic-app-before-version.png)

  Después de la refactorización, la aplicación lógica se ha convertido en una aplicación lógica primaria y secundaria. El elemento primario obtiene las tablas de SQL Server y, a continuación, llama a una aplicación lógica secundaria para cada tabla para obtener las filas:

  ![Creación de una aplicación lógica para una acción](./media/handle-throttling-problems-429-errors/refactor-logic-app-single-connection-1.png)

  Esta es la aplicación lógica secundaria a la que llama la aplicación lógica primaria para obtener las filas de cada tabla:

  ![Creación de otra aplicación lógica para una segunda acción](./media/handle-throttling-problems-429-errors/refactor-logic-app-single-connection-2.png)

<a name="connector-throttling"></a>

## <a name="connector-throttling"></a>Limitación de conectores

Cada conector tiene sus propios límites de en cuanto a la limitación, que puede encontrar en la página de referencia técnica del conector. Por ejemplo, el [conector de Azure Service Bus](/connectors/servicebus/) tiene un límite para la limitación que permite hasta 6000 llamadas por minuto, mientras que el conector de SQL Server tiene [límites para la limitación que varían en función del tipo de operación](/connectors/sql/).

Algunos desencadenadores y acciones, como HTTP, tienen una ["directiva de reintentos"](../logic-apps/logic-apps-exception-handling.md#retry-policies) que puede personalizar en función de los [límites de la directiva de reintentos ](../logic-apps/logic-apps-limits-and-config.md#retry-policy-limits) para implementar el control de excepciones. Esta directiva especifica si un desencadenador o una acción reintenta una solicitud cuando se produce un error en la solicitud original o se agota el tiempo de espera y se genera una respuesta 408, 429 o 5xx, y con qué frecuencia. Por lo tanto, cuando se inicia la limitación y se devuelve un error 429, Logic Apps sigue la directiva de reintentos si se admite.

Para saber si un desencadenador o una acción admite directivas de reintentos, compruebe la configuración de la acción o el desencadenador. Para ver el número de reintentos de un desencadenador o una acción, vaya al historial de ejecuciones de la aplicación lógica, seleccione la ejecución que desea revisar y expanda el desencadenador o la acción para ver detalles sobre las entradas, las salidas y los reintentos, por ejemplo:

![Visualización del historial de ejecución de la acción, los reintentos, las entradas y las salidas](./media/handle-throttling-problems-429-errors/example-429-too-many-requests-retries.png)

Aunque el historial de reintentos proporciona información de errores, puede que tenga problemas para diferenciar entre la limitación del conector y la [limitación del destino](#destination-throttling). En este caso, es posible que tenga que revisar los detalles de la respuesta o realizar algunos cálculos de intervalo de limitación para identificar el origen.

En el caso de las aplicaciones lógicas del servicio multiinquilino y global de Azure Logic Apps, la limitación se produce en el nivel de *conexión*. Por lo tanto, por ejemplo, en el caso de las aplicaciones lógicas que se ejecutan en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la limitación se sigue produciendo para las conexiones que no son ISE porque se ejecutan en el servicio multiinquilino y global de Azure Logic Apps. Sin embargo, las conexiones ISE, que se crean mediante conectores ISE, no se limitan porque se ejecutan en su ISE.

Para controlar la limitación en este nivel, tiene estas opciones:

* Configure varias conexiones para una única acción para que la aplicación lógica cree particiones de los datos para su procesamiento.

  Para esta opción, considere si puede distribuir la carga de trabajo dividiendo las solicitudes de una acción entre varias conexiones en el mismo destino con las mismas credenciales.

  Por ejemplo, supongamos que la aplicación lógica obtiene las tablas de una base de datos de SQL Server y, a continuación, obtiene las filas de cada tabla. En función del número de filas que se deban procesar, puede usar varias conexiones y varios bucles **For Each** para dividir el número total de filas en conjuntos más pequeños para su procesamiento. En este escenario se utilizan dos bucles **For Each** para dividir el número total de filas en la mitad. El primer bucle **For Each** usa una expresión que obtiene la primera mitad. El otro bucle **For Each** utiliza una expresión diferente que obtiene la segunda mitad, por ejemplo:<p>

    * Expresión 1: La función `take()` obtiene la parte delantera de una colección. Para obtener más información, vea [Función **`take()`** ](workflow-definition-language-functions-reference.md#take).

      `@take(collection-or-array-name, div(length(collection-or-array-name), 2))`

    * Expresión 2: La función `skip()` quita la parte delantera de una colección y devuelve todos los demás elementos. Para obtener más información, vea [Función **`skip()`** ](workflow-definition-language-functions-reference.md#skip).

      `@skip(collection-or-array-name, div(length(collection-or-array-name), 2))`

    Este es un ejemplo visual que muestra cómo puede usar estas expresiones:

    ![Creación y uso de varias conexiones para una única acción](./media/handle-throttling-problems-429-errors/create-multiple-connections-per-action.png)

* Configure una conexión diferente para cada acción.

  Para esta opción, considere si puede distribuir la carga de trabajo mediante la propagación de las solicitudes de cada acción a través de su propia conexión, incluso cuando las acciones se conectan al mismo servicio o sistema y usan las mismas credenciales.

  Por ejemplo, supongamos que la aplicación lógica obtiene las tablas de una base de datos de SQL Server y obtiene cada fila de cada tabla. Puede usar conexiones independientes de manera que la obtención de las tablas use una conexión y la obtención de cada fila use otra conexión.

  ![Creación y uso de conexiones diferentes para cada acción](./media/handle-throttling-problems-429-errors/create-connection-per-action.png)

* Cambie la simultaneidad o el paralelismo en un [bucle "For Each"](../logic-apps/logic-apps-control-flow-loops.md#foreach-loop).

  De forma predeterminada, las iteraciones de bucles "For Each" se ejecutan al mismo tiempo hasta el [límite de simultaneidad](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Si tiene un conector que se está limitando dentro de un bucle "For Each", puede reducir el número de iteraciones de bucle que se ejecutan en paralelo. Para más información, consulte los temas siguientes:
  
  * [Bucles "For Each": cambiar simultaneidad o ejecutar secuencialmente](../logic-apps/logic-apps-control-flow-loops.md#sequential-foreach-loop)

  * [Esquema del lenguaje de definición de flujo de trabajo: bucles For Each](../logic-apps/logic-apps-workflow-actions-triggers.md#foreach-action)

  * [Esquema de lenguaje de definición de flujo de trabajo: cambiar la simultaneidad de bucles "For Each"](../logic-apps/logic-apps-workflow-actions-triggers.md#change-for-each-concurrency)

  * [Esquema del lenguaje de definición de flujo de trabajo: ejecutar bucles "For Each" secuencialmente](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-for-each)

<a name="destination-throttling"></a>

## <a name="destination-service-or-system-throttling"></a>Limitación del sistema o del servicio de destino

Si bien un conector tiene sus propios límites para la limitación, el servicio o sistema de destino al que llama el conector también podría tener límites para la limitación. Por ejemplo, algunas API de Microsoft Exchange Server tienen límites para la limitación más estrictos que el conector de Outlook de Office 365.

De forma predeterminada, las instancias de una aplicación lógica y cualquier bucle o rama dentro de esas instancias se ejecutan *en paralelo*. Este comportamiento significa que varias instancias pueden llamar al mismo punto de conexión al mismo tiempo. Cada instancia no conoce la existencia de la otra, por lo que los intentos de realizar de nuevo acciones con error pueden crear [condiciones de carrera](https://en.wikipedia.org/wiki/Race_condition) en las que varias llamadas intentan ejecutarse al mismo tiempo, pero para que se completen correctamente, dichas llamadas deben llegar al servicio o sistema de destino antes de que se inicie la limitación.

Por ejemplo, supongamos que tiene una matriz con 100 elementos. Use un bucle "For Each" para recorrer en iteración la matriz y activar el control de simultaneidad del bucle para que pueda restringir el número de iteraciones paralelas a 20 o el [límite predeterminado actual](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits). Dentro de ese bucle, una acción inserta un elemento de la matriz en una base de datos de SQL Server, que solo permite 15 llamadas por segundo. Este escenario genera un problema de limitación porque se acumulan los reintentos y nunca se ejecutan.

En esta tabla se describe la escala de tiempo de lo que ocurre en el bucle cuando el intervalo de reintento de la acción es 1 segundo:

| A un momento dado | Número de acciones que se ejecutan | Número de acciones con errores | Número de reintentos en espera |
|---------------|----------------------------|-----------------------------|---------------------------|
| T + 0 segundos | 20 inserciones | 5 errores debido a un límite de SQL | 5 reintentos |
| T + 0,5 segundos | 15 inserciones, porque había 5 reintentos anteriores en espera | Las 15 dan error, porque el límite anterior de SQL sigue vigente durante otros 0,5 segundos | 20 reintentos <br>(5 anteriores + 15 nuevos) |
| T + 1 segundo | 20 inserciones | 5 errores más los 20 reintentos anteriores, debido al límite de SQL | 25 reintentos (20 anteriores + 5 nuevos)
|||||

Para controlar la limitación en este nivel, tiene estas opciones:

* Cree aplicaciones lógicas para que cada una de ellas controle una sola operación.

  * Siguiendo con el escenario de SQL Server de ejemplo de esta sección, puede crear una aplicación lógica que coloque los elementos de la matriz en una cola, como una [cola de Azure Service Bus](../connectors/connectors-create-api-servicebus.md). A continuación, cree otra aplicación lógica que realice solo la operación de inserción para cada elemento de la cola. De este modo, solo se ejecuta una instancia de aplicación lógica en un momento concreto, lo que completa la operación de inserción y pasa al siguiente elemento de la cola, o la instancia obtiene errores 429 pero no intenta realizar reintentos que no sean productivos.

  * Cree una aplicación lógica principal que llame a una aplicación lógica secundaria o anidada para cada acción. Si el elemento primario necesita llamar a diferentes aplicaciones secundarias en función del resultado del elemento primario, puede usar una acción de condición o una acción de conmutador que determine la aplicación secundaria a la que debe llamar. Este patrón puede ayudarle a reducir el número de llamadas u operaciones.

    Por ejemplo, supongamos que tiene dos aplicaciones lógicas, cada una con un desencadenador de sondeo que comprueba su cuenta de correo electrónico cada minuto para un asunto específico, como "correcto" o "error". Esta configuración da como resultado 120 llamadas por hora. En su lugar, si crea una única aplicación lógica primaria que sondea cada minuto, pero llama a una aplicación lógica secundaria que se ejecuta en función de si el asunto es "correcto" o "error", se reduce el número de comprobaciones de sondeo a la mitad, o 60 en este caso.

* Configure el procesamiento por lotes.

  Si el servicio de destino admite operaciones por lotes, puede solucionar el límite mediante el procesamiento de elementos en grupos o lotes, en lugar de hacerlo individualmente. Para implementar la solución de procesamiento por lotes, cree una aplicación lógica "receptora del lote" y una aplicación lógica "remitente del lote". La remitente del lote recopila mensajes o elementos hasta que se cumplan los criterios especificados y, a continuación, envía esos mensajes o elementos en un único grupo. La receptora del lote acepta ese grupo y procesa esos mensajes o elementos. Para obtener más información, consulte [Procesamiento por lotes de mensajes en Azure Logic Apps](../logic-apps/logic-apps-batch-process-send-receive-messages.md).

* Use las versiones de webhook para los desencadenadores y las acciones, en lugar de las versiones de sondeo.

  ¿Por qué? Un desencadenador de sondeo continúa comprobando el servicio o sistema de destino a intervalos específicos. Un intervalo muy frecuente, como una vez cada segundo, puede crear problemas de limitación. Sin embargo, un desencadenador o una acción de webhook, como [webhook de HTTP](../connectors/connectors-native-webhook.md), solo crea una llamada al servicio o sistema de destino, que se produce en el momento de la suscripción y solicita que el destino notifique el desencadenador o la acción solo cuando se produce un evento. De este modo, el desencadenador o la acción no tienen que comprobar continuamente el destino.
  
  Por lo tanto, si el servicio o el sistema de destino admite webhooks o proporciona un conector con una versión de webhook, esta opción es mejor que el uso de la versión de sondeo. Para identificar las acciones y los desencadenadores de webhook, confirme que tienen el tipo de `ApiConnectionWebhook` o que no requieren que especifique una periodicidad. Para obtener más información, vea [Desencadenador de APIConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) y [Acción de APIConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-action).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [límites y configuración de Logic Apps](../logic-apps/logic-apps-limits-and-config.md).
