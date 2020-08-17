---
title: Filtros de tema de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo los suscriptores pueden especificar filtros para definir qué mensajes desean recibir de un tema.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 5df343ff63c01a7cf10315b758e3d6fba8ac5674
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88066753"
---
# <a name="topic-filters-and-actions"></a>Filtros y acciones de temas

Los suscriptores pueden definir los mensajes que quieren recibir de un tema. Estos mensajes se especifican en forma de una o varias reglas de suscripción con nombre. Cada regla está formada por una condición que selecciona mensajes concretos y una acción que anota el mensaje seleccionado. Con cada condición de regla de coincidencia, la suscripción crea una copia del mensaje, que se puede anotar de manera diferente para cada regla coincidente.

Cada suscripción de tema recién creada tiene una regla de suscripción predeterminada inicial. Si no especifica explícitamente una condición de filtro para la regla, el filtro aplicado es el filtro **true** que permite que se seleccionen todos los mensajes de la suscripción. La regla predeterminada no tiene ninguna acción de anotación asociada.

Service Bus admite tres condiciones de filtro:

-   *Filtros booleanos*: el filtro **TrueFilter** y **FalseFilter** hace que se seleccionen todos los mensajes entrantes (**true**) o ninguno de los mensajes entrantes (**false**) de la suscripción.

-   *Filtros SQL*: una propiedad **SqlFilter** contiene una expresión condicional de tipo SQL que se evalúa en el agente contra las propiedades del sistema y las propiedades definidas por el usuario de los mensajes entrantes. Todas las propiedades del sistema deben ir precedidas de `sys.` en la expresión condicional. El [subconjunto de lenguaje SQL para las condiciones de filtro](service-bus-messaging-sql-filter.md) comprueba la existencia de propiedades (`EXISTS`), valores nulos (`IS NULL`), operadores lógicos, relacionales NOT/AND/OR, aritmética numérica simple y coincidencia de patrones de texto simple con `LIKE`.

-   *Filtros de correlación*: una propiedad **CorrelationFilter** contiene un conjunto de condiciones para las que se busca la coincidencia con una o varias de las propiedades del usuario y del sistema de un mensaje entrante. Un uso común es comparar con la propiedad **CorrelationId**, pero la aplicación también puede elegir comparar con las siguientes propiedades:

    - **ContentType**
     - **Label**
     - **MessageId**
     - **ReplyTo**
     - **ReplyToSessionId**
     - **SessionId** 
     - **To**
     - cualquier propiedad definida por el usuario. 
     
     Existe una coincidencia cuando el valor de un mensaje entrante de una propiedad es igual al valor especificado en el filtro de correlación. En las expresiones de cadena, la comparación distingue mayúsculas de minúsculas. Al especificar varias propiedades de coincidencia, el filtro las combina como una condición AND lógica, lo que significa que para que el filtro realice la coincidencia, todas las condiciones deben coincidir.

Todos los filtros evalúan propiedades del mensaje. Los filtros no pueden evaluar el cuerpo del mensaje.

Las reglas de filtro complejas requieren capacidad de procesamiento. En concreto, el uso de reglas de filtro SQL provoca un menor rendimiento general del mensaje en el nivel de suscripción, tema y espacio de nombres. Siempre que sea posible, las aplicaciones deben elegir filtros de correlación por encima de filtros de tipo SQL, ya que son mucho más eficaces en el procesamiento y tienen un impacto menor en el rendimiento.

## <a name="actions"></a>Acciones

Con las condiciones de filtro SQL puede definir una acción que puede anotar el mensaje mediante la adición, eliminación o sustitución de propiedades y sus valores. La acción [usa una expresión de tipo SQL](service-bus-messaging-sql-filter.md) que se basa ligeramente en la sintaxis de la instrucción UPDATE de SQL. La acción se realiza en el mensaje después de que se haya correspondido y antes de que el mensaje se seleccione en la suscripción. Los cambios en las propiedades del mensaje son privados para el mensaje copiado en la suscripción.

## <a name="usage-patterns"></a>Patrones de uso

El escenario de uso más sencillo de un tema es que cada suscripción obtenga una copia de cada mensaje enviado a un tema, lo que permite un patrón de difusión.

Los filtros y las acciones permiten dos grupos adicionales de patrones: creación de particiones y enrutamiento.

En la creación de particiones se usan filtros para distribuir los mensajes entre varias suscripciones a temas existentes de una manera predecible y mutuamente excluyente. El patrón de creación de particiones se usa cuando un sistema se escala horizontalmente para controlar muchos contextos diferentes en compartimientos funcionalmente idénticos que contiene cada uno de ellos un subconjunto de los datos generales; por ejemplo, información del perfil del cliente. Con la creación de particiones, un editor envía el mensaje en un tema sin necesidad de ningún conocimiento del modelo de creación de particiones. A continuación, el mensaje se mueve a la suscripción correcta desde la que el controlador de mensajes de la partición puede recuperarlo.

En el enrutamiento se usan filtros para distribuir los mensajes entre las suscripciones a temas de un modo predecible, pero no necesariamente exclusivo. Junto con la característica de [reenvío automático](service-bus-auto-forwarding.md), pueden usarse filtros de tema para crear gráficos complejos de enrutamiento dentro de un espacio de nombres de Service Bus para la distribución de mensajes dentro de una región de Azure. Si Azure Functions o Azure Logic Apps actúan como un puente entre los espacios de nombres de Azure Service Bus, puede crear topologías globales complejas con integración directa en las aplicaciones de línea de negocio.


> [!NOTE]
> Como Azure Portal ahora admite la funcionalidad de Service Bus Explorer, los filtros de suscripción se pueden crear o editar desde el portal. 

## <a name="next-steps"></a>Pasos siguientes
Consulte los ejemplos siguientes: 

- [.NET: Tutorial básico de envío y recepción con filtros](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/BasicSendReceiveTutorialwithFilters/BasicSendReceiveTutorialWithFilters)
- [.NET: Filtros de tema](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TopicFilters)
- [Plantilla de Azure Resource Manager](/azure/templates/microsoft.servicebus/2017-04-01/namespaces/topics/subscriptions/rules)