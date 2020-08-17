---
title: Modelos de precios y facturación
description: Información general sobre cómo funciona el modelo de precios y facturación para Azure Logic Apps
services: logic-apps
ms.suite: integration
author: jonfancey
ms.author: jonfan
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 06/25/2020
ms.openlocfilehash: a5511d7cd4b5bb0f3fe901a735535f8db9036ee7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078152"
---
# <a name="pricing-model-for-azure-logic-apps"></a>Modelo de precios de Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) ayuda a crear y ejecutar flujos de trabajo de integración automatizados en la nube. En este artículo se detallan los precios y la facturación de Azure Logic Apps. Para ver las tarifas de precios, consulte los [precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

<a name="consumption-pricing"></a>

## <a name="consumption-pricing-model"></a>Modelo de precios de consumo

Con las nuevas aplicaciones lógicas que se ejecutan en el servicio público "global", multiinquilino de Azure Logic Apps, paga solo lo que usa. Estas aplicaciones lógicas usan un plan de consumo y el modelo de precios. En la aplicación lógica, cada paso es una acción y Azure Logic Apps mide todas las acciones que se ejecutan en la aplicación lógica.

Las acciones son, por ejemplo:

* [Desencadenadores](#triggers), que son acciones especiales. Todas las aplicaciones lógicas requieren un desencadenador como primer paso.

* [Acciones nativas o "integradas"](../connectors/apis-list.md#built-in), como HTTP, llamadas a Azure Functions y a API Management, etc.

* Llamadas a [conectores administrados](../connectors/apis-list.md#managed-connectors) como Outlook 365, Dropbox, etc.

* [Acciones del flujo de trabajo de control](../connectors/apis-list.md#control-workflow), como bucles, instrucciones condicionales, etc.

Los [conectores estándar](../connectors/apis-list.md#managed-connectors) se cobran al [precio del conector estándar](https://azure.microsoft.com/pricing/details/logic-apps). Los [conectores empresariales](../connectors/apis-list.md#managed-connectors) de disponibilidad general se cobran al [precio del conector empresarial](https://azure.microsoft.com/pricing/details/logic-apps), mientra que los conectores empresariales de versión preliminar pública se cobran al [precio del conector estándar](https://azure.microsoft.com/pricing/details/logic-apps).

Más información sobre la facturación en los niveles de [desencadenadores](#triggers) y [acciones](#actions). Para obtener más información sobre los límites, consulte [Límites y configuración para Azure Logic Apps](logic-apps-limits-and-config.md).

<a name="fixed-pricing"></a>

## <a name="fixed-pricing-model"></a>Modelo de precios fijos

Un [*entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) proporciona una manera aislada de crear y ejecutar aplicaciones lógicas que pueden acceder a los recursos de una red virtual de Azure. Las aplicaciones lógicas que se ejecutan en un ISE no comportan gastos de retención de datos. Al crear un ISE y solo durante la creación, puede elegir un [nivel de ISE o "SKU"](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level), que tienen [tarifas de precios](https://azure.microsoft.com/pricing/details/logic-apps) diferentes:

* ISE **Premium**: La unidad base de este ISE tiene una capacidad fija, pero si necesita más rendimiento, puede [agregar más unidades de escalado](../logic-apps/ise-manage-integration-service-environment.md#add-capacity), ya sea durante el proceso de creación del ISE o posteriormente. Para conocer los límites del ISE, consulte [Límites y configuración para Azure Logic Apps](logic-apps-limits-and-config.md#integration-service-environment-ise).

* ISE de **Desarrollador**: Esta SKU no tiene ninguna funcionalidad de escalado vertical, ningún acuerdo de nivel de servicio (SLA) y ningún límite publicado. Use esta SKU solo para experimentar, desarrollar y probar, no para pruebas de rendimiento ni en producción.

En el caso de las aplicaciones lógicas que crea y ejecuta dentro de un ISE, paga un [precio fijo](https://azure.microsoft.com/pricing/details/logic-apps) (en lugar de un pago por uso) para estas funcionalidades:

* Acciones y desencadenadores [integrados](../connectors/apis-list.md#built-in)

  En un ISE, los desencadenadores y acciones integrados muestran la etiqueta **Core** y se ejecutan en el mismo ISE que las aplicaciones lógicas.

* Conectores [estándar](../connectors/apis-list.md#managed-connectors) y conectores [empresariales](../connectors/apis-list.md#enterprise-connectors), que le permiten tener tantas conexiones empresariales como quiera.

   Los conectores estándar y empresariales en los que se muestra la etiqueta **ISE** se ejecutan en el mismo ISE que las aplicaciones lógicas. Los conectores en los que no se muestra la etiqueta ISE se ejecutan en el servicio público "global", multiinquilino de Logic Apps. Los precios fijos también se aplican a los conectores que se ejecutan en el servicio multiinquilino cuando se usan con aplicaciones lógicas que se ejecutan en un ISE.

* Uso de la [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) sin costo adicional, en función de la [SKU de ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level):

  * SKU de ISE **Premium**: una cuenta de integración de [nivel estándar](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits) única

  * SKU de ISE de **Desarrollador**: una cuenta de integración de [nivel Gratis](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits) única

  Cada SKU de ISE está limitada a 5 cuentas de integración totales. Para un costo adicional, puede tener más cuentas de integración en función de la SKU de ISE:

  * SKU de ISE **Premium**: hasta cuatro cuentas Estándar más. Sin cuentas Gratis ni Básicas.

  * SKU de ISE de **Desarrollador**: hasta 4 cuentas Estándar más o hasta 5 cuentas Estándar en total. Sin cuentas Básicas.

  Para obtener más información sobre los límites de las cuentas de integración, consulte [Límites y configuración de Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits). Puede obtener más información sobre los [niveles de las cuentas de integración y su modelo de precios](#integration-accounts) más adelante en este tema.

<a name="connectors"></a>

## <a name="connectors"></a>Conectores

Los conectores de Azure Logic Apps ayudan a la aplicación lógica a acceder a las aplicaciones, servicios y sistemas en la nube o locales, proporcionando para ello [desencadenadores](#triggers), [acciones](#actions) o ambos elementos. Los conectores se clasifican como estándar o empresariales. Para obtener información general sobre estos conectores, vea [Conectores para Azure Logic Apps](../connectors/apis-list.md). Si no hay conectores creados previamente disponibles para las API de REST que quiere usar en las aplicaciones lógicas, puede crear [conectores personalizados](/connectors/custom-connectors), que son solo contenedores en torno a esas API de REST. Los conectores personalizados se facturan como conectores estándar. En las secciones siguientes se proporciona más información sobre la facturación de desencadenadores y acciones.

<a name="triggers"></a>

## <a name="triggers"></a>Desencadenadores

Un desencadenador es siempre el primer paso del flujo de trabajo de una aplicación lógica y es una acción especial que crea y ejecuta una instancia de aplicación lógica cuando se cumplen determinados criterios o cuando se produce un evento específico. Los desencadenadores actúan de formas distintas, que afectan al modo en que la aplicación lógica se mide. Estos son los distintos tipos de desencadenadores que hay en Azure Logic Apps:

* **Desencadenador de periodicidad**: puede usar este desencadenador genérico, que no es específico de ningún servicio o sistema, para iniciar un flujo de trabajo de aplicación lógica y crear una instancia de aplicación lógica que se ejecute según el intervalo de periodicidad configurado en el desencadenador. Por ejemplo, puede establecer un desencadenador de periodicidad que se ejecute cada tres días o según una programación más compleja.

* **Desencadenador de sondeo**: puede usar este desencadenador de periodicidad más especializado, que normalmente está asociado al conector administrado de un servicio o sistema específicos, para comprobar si hay eventos o mensajes que cumplan los criterios para crear y ejecutar una instancia de aplicación lógica según el intervalo de periodicidad configurado en el desencadenador. Incluso cuando no se crea ninguna instancia de aplicación lógica, por ejemplo, cuando se omiten los desencadenadores, el servicio Logic Apps mide cada solicitud de sondeo como una ejecución. Para especificar el intervalo de sondeo, configure el desencadenador a través del Diseñador de aplicaciones lógicas.

  [!INCLUDE [logic-apps-polling-trigger-non-standard-metering](../../includes/logic-apps-polling-trigger-non-standard-metering.md)]

* **Desencadenador de webhook**: en lugar de usar un desencadenador de sondeo, puede usar un desencadenador de webhook para esperar a que el cliente envíe una solicitud a la aplicación lógica a una dirección URL de punto de conexión específica. Cada solicitud enviada al punto de conexión de webhook se considera una ejecución de acción. Por ejemplo, el desencadenador de solicitud y de webhook HTTP son ambos desencadenadores de webhook. Algunos conectores de servicios o sistemas también tienen desencadenadores de webhook.

<a name="actions"></a>

## <a name="actions"></a>Acciones

Azure Logic Apps mide las acciones "integradas" (como HTTP) como acciones nativas. Por ejemplo, las acciones integradas engloban las llamadas HTTP, llamadas desde Azure Functions o API Management y pasos del flujo de control como condiciones, bucles e instrucciones switch. Cada acción tiene su propio tipo de acción. Así, las acciones que llaman a [conectores](/connectors) son de tipo "ApiConnection". Estos conectores se clasifican como estándar o empresarial, y se miden según sus [precios](https://azure.microsoft.com/pricing/details/logic-apps) respectivos. Los conectores empresariales en *versión preliminar* se cobran como conectores estándar.

Azure Logic Apps mide todas las acciones ejecutadas correcta e incorrectamente como ejecuciones, pero no mide estas acciones:

* Acciones que se omitan debido a condiciones no satisfechas
* Acciones que no se ejecuten porque la aplicación lógica se detuviera antes de finalizar

En las acciones que se ejecutan dentro de bucles, Azure Logic Apps cuenta las acciones de cada ciclo del bucle. Por ejemplo, supongamos que tiene un bucle "for each" que procesa una lista. Logic Apps mide una acción de ese bucle al multiplicar el número de elementos de lista por el número de acciones del bucle, y agrega la acción que inicia el bucle. Por lo tanto, el cálculo de una lista de 10 elementos es (10x1)+1, lo que da como resultado 11 ejecuciones de acción.

## <a name="disabled-logic-apps"></a>Aplicaciones lógicas deshabilitadas

Las aplicaciones lógicas deshabilitadas no se cobran, ya que al estar deshabilitadas no pueden crear instancias. Después de deshabilitar una aplicación lógica, las instancias que se estén ejecutando pueden tardar un tiempo antes de detenerse por completo.

<a name="integration-accounts"></a>

## <a name="integration-accounts"></a>Cuentas de integración

Un [modelo de precios fijos](https://azure.microsoft.com/pricing/details/logic-apps) se aplica a las [cuentas de integración](logic-apps-enterprise-integration-create-integration-account.md) en las que puede explorar, desarrollar y probar las características [B2B y EDI](logic-apps-enterprise-integration-b2b.md) y las características de [procesamiento XML](logic-apps-enterprise-integration-xml.md) en Azure Logic Apps sin ningún costo adicional. Cada suscripción de Azure puede tener hasta un [límite específico de cuentas de integración](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits). Cada cuenta de integración puede almacenar hasta un [límite específico de artefactos](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits), como entidades, contratos, mapas, esquemas, ensamblados, certificados y configuraciones por lotes, etc.

Azure Logic Apps ofrece cuentas de integración gratuitas, básicas y estándar. Los niveles básico y estándar son compatibles con el acuerdo de nivel de servicio (SLA) de Logic Apps, mientras que el nivel gratuito no es compatible con un SLA y tiene límites en cuanto a disponibilidad regional, así como un rendimiento y un uso limitados. Excepto en las cuentas de integración del nivel Gratis, puede tener más de una cuenta de integración en cada región de Azure. Para ver las tarifas de precios, consulte los [precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

Si tiene un [*entorno de servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), ya sea [Premium o Desarrollador](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level), el ISE puede tener 5 cuentas de integración en total. Para información sobre cómo funciona el modelo de precios fijos para un ISE, consulte la sección anterior sobre el [modelo de precios fijos](#fixed-pricing) en este tema. Para ver las tarifas de precios, consulte los [precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

Para elegir una cuenta de integración Gratis, Básica o Estándar, revise estas descripciones de casos de uso:

* **Gratis**: adecuada si se quieren probar escenarios de exploración, no escenarios de producción. Este nivel solo está disponible para las regiones públicas en Azure, por ejemplo, Oeste de EE. UU. o Sudeste Asiático, pero no para [Azure China 21Vianet](/azure/china/overview-operations) o [Azure Government](../azure-government/documentation-government-welcome.md).

* **Básico**: adecuada si se quiere utilizar solo el control de mensajes o actuar como un asociado de pequeña empresa que tenga una relación empresarial con una entidad empresarial mayor.

* **Estándar**: adecuada si se tienen relaciones B2B más complejas y un número de entidades mayor del que debe administrar.

<a name="data-retention"></a>

## <a name="data-retention"></a>Retención de datos

Excepto en el caso de las aplicaciones lógicas que se ejecutan en un Entorno del servicio de integración (ISE), todas las entradas y salidas que se almacenan en el historial de ejecuciones de la aplicación lógica se facturan en función del [período de retención de ejecuciones](logic-apps-limits-and-config.md#run-duration-retention-limits) de una aplicación lógica. Las aplicaciones lógicas que se ejecutan en un ISE no comportan gastos de retención de datos. Para ver las tarifas de precios, consulte los [precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

Para ayudarle a supervisar el consumo de almacenamiento de la aplicación lógica, haga lo siguiente:

* Consulte el número de unidades de almacenamiento en GB que la aplicación lógica usa cada mes.
* Consulte los tamaños de las entradas y salidas de una acción específica en el historial de ejecuciones de la aplicación lógica.

<a name="storage-consumption"></a>

### <a name="view-logic-app-storage-consumption"></a>Consulta del consumo de almacenamiento de la aplicación lógica

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, en **Supervisión**, seleccione **Métricas**.

1. En el panel de la derecha, en la lista **Métricas** de **Título del gráfico**, seleccione **Uso de facturación para las ejecuciones de consumo de almacenamiento**.

   Esta métrica señala el número de unidades de consumo de almacenamiento en GB al mes que se está facturando.

<a name="input-output-sizes"></a>

### <a name="view-action-input-and-output-sizes"></a>Consulta de los tamaños de entrada y salida de acciones

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, seleccione **Introducción**.

1. En el panel de la derecha, en **Historial de ejecuciones**, seleccione la ejecución que tiene las entradas y salidas que quiera consultar.

1. En **Ejecución de aplicación lógica**, elija **Detalles de ejecución**.

1. En el panel **Detalles de ejecución de la aplicación lógica**, seleccione la acción que quiera ver en la tabla de acciones, que muestra el estado y la duración de cada acción.

1. En el panel **Acción de aplicación lógica**, los tamaños de las entradas y las salidas de esa acción aparecen en **Vínculo de entradas** y **Vínculo de salidas** respectivamente.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre Azure Logic Apps](logic-apps-overview.md)
* [Creación de una nueva aplicación lógica mediante la conexión de servicios de SaaS](quickstart-create-first-logic-app-workflow.md)
