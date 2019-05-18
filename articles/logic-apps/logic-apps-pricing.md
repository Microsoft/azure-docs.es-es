---
title: 'Precios y facturación: Azure Logic Apps | Microsoft Docs'
description: Descubra cómo funcionan los precios y la facturación en Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: logic-apps
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
ms.assetid: f8f528f5-51c5-4006-b571-54ef74532f32
ms.topic: article
ms.date: 03/25/2019
ms.openlocfilehash: c3d06d3c0f9c86b4fe6495632b48051fd69dc663
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544539"
---
# <a name="pricing-model-for-azure-logic-apps"></a>Modelo de precios de Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) le ayuda a crear y ejecutar flujos de trabajo de integración automatizada que puede escalar en la nube. En este artículo se describe cómo funcionan los precios y facturación para Azure Logic Apps. Para obtener información de precios, consulte [precios de Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

<a name="consumption-pricing"></a>

## <a name="consumption-pricing-model"></a>Modelo de precios de consumo

Para las nuevas aplicaciones lógicas que se ejecutan en el servicio de Azure Logic Apps "global" o público, pagar solo por lo que usa. Estas aplicaciones lógicas usan un plan de consumo y el modelo de precios. En la definición de la aplicación lógica, cada paso es una acción. Por ejemplo, las acciones son:

* Desencadenadores, que son acciones especiales. Todas las aplicaciones lógicas requieren un desencadenador como primer paso.
* Acciones "Integradas" o nativas como HTTP, las llamadas a funciones de Azure y API Management etc.
* Llamadas a conectores como Outlook 365, Dropbox etc.
* Controlar los pasos del flujo, como bucles, instrucciones condicionales etc.

Azure Logic Apps mide todas las acciones que se ejecutan en la aplicación lógica. Más información sobre cómo funciona la facturación para [desencadenadores](#triggers) y [acciones](#actions).

<a name="fixed-pricing"></a>

## <a name="fixed-pricing-model"></a>Modelo de precios fijos

Un [ *entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) proporciona una manera privada, aislada y dedicada para crear y ejecutar aplicaciones lógicas que pueden acceder a recursos en una red virtual de Azure. Para las nuevas aplicaciones lógicas que se ejecutan dentro de una instancia de ISE, paga un precio mensual fijo para los desencadenadores y acciones integradas y también para los conectores estándares.

El ISE también incluye un conector empresarial gratis, que incluye tantas conexiones como desee. El uso de conectores empresariales adicionales se cobra según el precio del consumo de empresa.

> [!NOTE]
> En una instancia de ISE, desencadenadores integrados y acciones mostrará el **Core** etiquetar y ejecutar en el ISE mismo como las aplicaciones lógicas. Conectores estándar y empresariales que muestren el **ISE** etiqueta que se ejecuta en el ISE mismo como las aplicaciones lógicas. Los conectores que no muestran la etiqueta ISE se ejecutan en el servicio global de aplicaciones lógicas.

La unidad básica de ISE ha se ha corregido la capacidad, por lo que si necesita más rendimiento, puede [agregar más unidades de escalado](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#add-capacity), ya sea durante la creación o después. 

Para obtener información de precios, consulte [precios de Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

<a name="connectors"></a>

## <a name="connectors"></a>Conectores

Los conectores de Azure Logic Apps ayudan a su lógica de acceso a aplicaciones, servicios y sistemas en la nube o locales proporcionando [desencadenadores](#triggers), [acciones](#actions), o ambos. Los conectores se clasifican como Standard o Enterprise. Para obtener información general acerca de estos conectores, consulte [conectores para Azure Logic Apps](../connectors/apis-list.md). Las secciones siguientes proporcionan más información acerca de cómo la facturación de desencadenadores y acciones funcionan.

<a name="triggers"></a>

## <a name="triggers"></a>Desencadenadores

Los desencadenadores son acciones especiales que crean una instancia de aplicación lógica cuando se produce un evento específico. Los desencadenadores actúan de formas distintas, que afectan al modo en que la aplicación lógica se mide. Estos son los distintos tipos de desencadenadores que se encuentran en Azure Logic Apps:

* **Desencadenador de sondeo**: Este desencadenador sondea continuamente un punto de conexión para los mensajes que cumplen los criterios para crear una instancia de aplicación lógica e iniciar el flujo de trabajo. Logic Apps mide cada solicitud de sondeo como una ejecución, incluso cuando no se crea ninguna instancia de aplicación lógica. Para especificar el intervalo de sondeo, configure el desencadenador a través del Diseñador de aplicaciones lógicas.

  [!INCLUDE [logic-apps-polling-trigger-non-standard-metering](../../includes/logic-apps-polling-trigger-non-standard-metering.md)]

* **Desencadenador de Webhook**: Este desencadenador espera a que un cliente enviar una solicitud a un extremo específico. Cada solicitud enviada al punto de conexión de webhook se considera una ejecución de acción. Por ejemplo, el desencadenador de solicitud y de webhook HTTP son ambos desencadenadores de webhook.

* **Desencadenador de periodicidad**: Este desencadenador crea una instancia de aplicación lógica según el intervalo de periodicidad que configuró en el desencadenador. Por ejemplo, puede establecer un desencadenador de periodicidad que se ejecuta cada tres días o según una programación más compleja.

<a name="actions"></a>

## <a name="actions"></a>Acciones

Azure Logic Apps mide "integradas" acciones, como HTTP, como acciones nativas. Por ejemplo, acciones integradas incluyen llamadas HTTP, las llamadas de Azure Functions o API Management y controlan los pasos del flujo tales como las condiciones, bucles e instrucciones switch. Cada acción tiene su propio tipo de acción. Por ejemplo, las acciones que llaman a [conectores](https://docs.microsoft.com/connectors) tienen el tipo "ApiConnection". Estos conectores se clasifican como Standard o conectores de empresa, que se miden en función de sus respectivas [precios](https://azure.microsoft.com/pricing/details/logic-apps). Conectores de empresa en *Preview* se cobran como conectores estándar.

Azure Logic Apps mide todas las acciones correctas e incorrectas como ejecuciones. Sin embargo, las aplicaciones lógicas no medir estas acciones:

* Acciones que se omitan debido a condiciones no satisfechas
* Acciones que no se ejecuten porque la aplicación lógica se detuviera antes de finalizar

Para las acciones que se ejecutan dentro de los bucles, Azure Logic Apps recuentos de cada acción de cada ciclo del bucle. Por ejemplo, supongamos que tiene un bucle "for each" que procesa una lista. Logic Apps mide una acción de ese bucle al multiplicar el número de elementos de lista por el número de acciones del bucle, y agrega la acción que inicia el bucle. Por lo tanto, el cálculo para obtener una lista de 10 elementos es (10 * 1) + 1, lo que da como resultado 11 ejecuciones de acción.

## <a name="disabled-logic-apps"></a>Aplicaciones lógicas deshabilitadas

Deshabilita la lógica de aplicaciones no se cobran porque no se crean nuevas instancias mientras se está deshabilitado.
Después de deshabilitar una aplicación lógica, las instancias que se estén ejecutando pueden tardar un tiempo antes de detenerse por completo.

## <a name="integration-accounts"></a>Cuentas de integración

Precios de consumo se aplican a [cuentas de integración](logic-apps-enterprise-integration-create-integration-account.md) donde puede explorar, desarrollar y probar el [B2B y EDI](logic-apps-enterprise-integration-b2b.md) y [procesamiento XML](logic-apps-enterprise-integration-xml.md) características en Azure Logic Apps sin ningún costo adicional.
Puede tener una cuenta de integración en cada región de Azure. Cada cuenta de integración puede almacenar [números específicos de artefactos](../logic-apps/logic-apps-limits-and-config.md), como entidades, contratos, mapas, esquemas, ensamblados, certificados y configuraciones por lotes, etc.

Azure Logic Apps también ofrece cuentas de integración de básico y estándar con el SLA de aplicaciones lógicas compatibles. Estas son formas que puede elegir si desea utilizar una cuenta de integración básico o estándar:

* Utilice cuentas de integración básicas cuando simplemente desea control de mensajes o actuar como un partner de pequeña empresa que tiene una relación de socios comerciales con una entidad empresarial mayor.

* Utilice cuentas de integración estándar cuando tiene relaciones más complejas de B2B y quiere aumentar el número de entidades que puede administrar.

Para obtener información de precios, consulte [precios de Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

<a name="data-retention"></a>

## <a name="data-retention"></a>Retención de datos

Todas las entradas y salidas que se almacenan en el historial de ejecución de la aplicación lógica se facturan en función de una aplicación lógica [ejecutar el período de retención](logic-apps-limits-and-config.md#run-duration-retention-limits). Para obtener información de precios, consulte [precios de Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps).

Para ayudarle a supervisar el consumo de almacenamiento de la aplicación lógica, hacer lo siguiente:

* Ver el número de unidades de almacenamiento en GB que la aplicación lógica usa cada mes.
* Ver los tamaños de las entradas y salidas de una acción específica en el historial de ejecución de la aplicación lógica.

<a name="storage-consumption"></a>

### <a name="view-logic-app-storage-consumption"></a>Vista de consumo de almacenamiento de aplicación lógica

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, en **supervisión**, seleccione **métricas**.

1. En el panel derecho, bajo **título del gráfico**, desde el **métrica** lista, seleccione **uso de facturación para las ejecuciones de consumo de almacenamiento**.

   Esta métrica indica el número de consumo de unidades de almacenamiento en GB al mes que se están facturando.

<a name="input-output-sizes"></a>

### <a name="view-action-input-and-output-sizes"></a>Ver la entrada de la acción y los tamaños de salida

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, seleccione **Introducción**.

1. En el panel derecho, bajo **historial de ejecuciones**, seleccione la ejecución que tiene las entradas y salidas que desea comprobar.

1. En **ejecución de aplicación lógica**, elija **detalles de ejecución**.

1. En el **detalles de ejecución de aplicación lógica** panel, seleccione la acción que desea ver en la tabla de acciones, que muestra el estado y la duración de cada acción.

1. En el **acción de aplicación lógica** panel, busque los tamaños de las entradas y salidas de esa acción aparecen respectivamente en **vínculo entradas** y **vínculo de salidas**.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre Azure Logic Apps](logic-apps-overview.md)
* [Creación de una nueva aplicación lógica mediante la conexión de servicios de SaaS](quickstart-create-first-logic-app-workflow.md)