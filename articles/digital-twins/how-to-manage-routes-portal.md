---
title: Administración de puntos de conexión y rutas (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar y administrar puntos de conexión y rutas de eventos para datos de Azure Digital Twins mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 87b674a9c6b7f7d591b39f1baf54c4d54082f306
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87987314"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-portal"></a>Administración de puntos de conexión y rutas en Azure Digital Twins (portal)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

En Azure Digital Twins, se pueden enrutar [notificaciones de eventos](how-to-interpret-event-data.md) a los servicios de bajada o recursos de proceso conectados. Para ello, primero se configuran los **puntos de conexión** que pueden recibir los eventos. Después, puede crear [**rutas de eventos**](concepts-route-events.md) que especifiquen los eventos generados por Azure Digital Twins que se entregan a los puntos de conexión.

Este artículo le guía por el proceso de creación de puntos de conexión y rutas mediante [Azure Portal](https://portal.azure.com).

Los puntos de conexión y las rutas se administran con las [API EventRoutes](how-to-use-apis-sdks.md), el [SDK de .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) o la [CLI de Azure Digital Twins](how-to-use-cli.md). Se puede encontrar una versión de este artículo donde se usan estos mecanismos en lugar del portal en [*Procedimiento: Administración de puntos de conexión y rutas (API y CLI)* ](how-to-manage-routes-apis-cli.md).

## <a name="prerequisites"></a>Requisitos previos

* Necesitará una **cuenta de Azure** (puede configurar una de forma gratuita [aquí](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)).
* Necesitará una **instancia de Azure Digital Twins** en su suscripción de Azure. Si no dispone de una instancia, puede crear una con los pasos de [*Procedimiento: Configuración de una instancia y autenticación*](how-to-set-up-instance-scripted.md). Tenga a mano los siguientes valores de configuración para usarlos más adelante en este artículo:
    - Nombre de instancia
    - Grupo de recursos

Estos detalles se pueden encontrar en [Azure Portal](https://portal.azure.com) después de configurar la instancia. Inicie sesión en el portal y busque el nombre de la instancia en la barra de búsqueda del portal.
 
:::image type="content" source="media/how-to-manage-routes-portal/search-field-portal.png" alt-text="Captura de pantalla de la barra de búsqueda de Azure Portal.":::

En los resultados, seleccione la instancia para ver la página de detalles:

:::image type="content" source="media/how-to-manage-routes-portal/instance-details.png" alt-text="Captura de pantalla de detalles de la instancia de ADT." border="false":::

## <a name="create-an-endpoint-for-azure-digital-twins"></a>Creación de un punto de conexión para Azure Digital Twins

Estos son los tipos de puntos de conexión admitidos que puede crear para la instancia:
* [Event Grid](../event-grid/overview.md) 
* [Event Hubs](../event-hubs/event-hubs-about.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Para obtener más información sobre los distintos tipos de puntos de conexión, vea [*Elección entre los distintos servicios de mensajería de Azure*](../event-grid/compare-messaging-services.md).

Para vincular un punto de conexión a Azure Digital Twins, es necesario que ya exista el tema de Event Grid, Event Hubs o Service Bus que esté usando para el punto de conexión. 

### <a name="create-an-event-grid-endpoint"></a>Creación de un punto de conexión de Event Grid

**Requisito previo**: Cree un tema de Event Grid siguiendo los pasos descritos en [la sección *Creación de un tema personalizado*](../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic) del inicio rápido *Eventos personalizados* de Event Grid.

Después de crear el tema, puede vincularlo a Azure Digital Twins desde la página de la instancia de este servicio en [Azure Portal](https://portal.azure.com) (para encontrar la instancia, escriba su nombre en la barra de búsqueda del portal).

En el menú de la instancia, seleccione _Endpoints_ (Puntos de conexión). Luego, en la página *Endpoints* (Puntos de conexión) que le sigue, seleccione *+ Create an endpoint* (+ Crear un punto de conexión). 

En la página *Create an Endpoint* (Crear un punto de conexión) que se abre, puede crear un punto de conexión de tipo _Event Grid_ seleccionando el botón de radio correspondiente. Complete los demás detalles: escriba un nombre para el punto de conexión en el campo _Name_ (Nombre), elija la subscription en la lista desplegable _Subscription_ (Suscripción) y, en _Event Grid Topic_ (Tema de Event Grid), seleccione el tema creado anteriormente en la tercera lista desplegable.

A continuación, haga clic en _Save_ (Guardar) para crear el punto de conexión.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-grid.png" alt-text="Captura de pantalla de la creación de un punto de conexión de tipo Event Grid.":::

Para comprobar que el punto de conexión se ha creado correctamente, examine el icono de notificación de la barra superior de Azure Portal: 

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-notifications.png" alt-text="Captura de pantalla del icono de notificación para comprobar la creación del punto de conexión." border="false":::

También se puede ver el punto de conexión que se ha creado de vuelta en la página *Endpoints* (Puntos de conexión) de la instancia de Azure Digital Twins.

Si se produce un error en la creación del punto de conexión, observe el mensaje de error y vuelva a intentarlo al cabo de unos minutos.

Ahora, el tema de Event Grid está disponible como punto de conexión dentro de Azure Digital Twins, con el nombre especificado en el campo _Name_ (Nombre). Normalmente se usará ese nombre como destino de una **ruta de evento**, que se creará [más tarde en este artículo](#event-routes).

### <a name="create-an-event-hubs-endpoint"></a>Creación de un punto de conexión de Event Hubs

**Requisitos previos**: 
* Se necesita un _espacio de nombres de Event Hubs_ y un _centro de eventos_. Para crearlos, siga los pasos del inicio rápido [*Creación de un centro de eventos*](../event-hubs/event-hubs-create.md) de Event Hubs.
* Se necesita una _regla de autorización_. Para crearla, consulte el artículo [*Autorización del acceso a recursos de Event Hubs mediante firmas de acceso compartido*](../event-hubs/authorize-access-shared-access-signature.md) de Event Hubs.

Vaya a la página de detalles de la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com) (para buscarla, escriba su nombre en la barra de búsqueda del portal).

En el menú de la instancia, seleccione _Endpoints_ (Puntos de conexión). Luego, en la página *Endpoints* (Puntos de conexión) que le sigue, seleccione *+ Create an endpoint* (+ Crear un punto de conexión). 

En la página *Create an Endpoint* (Crear un punto de conexión) que se abre, puede crear un punto de conexión de tipo _Event Hub_ seleccionando el botón de radio correspondiente. Escriba un nombre para el punto de conexión en el campo _Name_ (Nombre). Después, en las listas desplegables _Subscription_ (Suscripción), _Event hub namespace_ (Espacio de nombres del centro de eventos), _Event Hub_ (Centro de eventos) y _Authorization rule_ (Regla de autorización), seleccione la suscripción, el espacio de nombres del centro de eventos creado anteriormente, el centro de eventos y la regla de autorización, respectivamente.

A continuación, haga clic en _Save_ (Guardar) para crear el punto de conexión.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-hub.png" alt-text="Captura de pantalla de la creación de un punto de conexión de tipo Event Hubs.":::

Para comprobar que el punto de conexión se ha creado correctamente, examine el icono de notificación de la barra superior de Azure Portal. 

Si se produce un error en la creación del punto de conexión, observe el mensaje de error y vuelva a intentarlo al cabo de unos minutos.

Ahora, el centro de eventos está disponible como punto de conexión dentro de Azure Digital Twins, con el nombre especificado en el campo _Name_ (Nombre). Normalmente se usará ese nombre como destino de una **ruta de evento**, que se creará [más tarde en este artículo](#event-routes).

### <a name="create-a-service-bus-endpoint"></a>Creación de un punto de conexión de Service Bus

**Requisitos previos**: 
* Se necesita un _espacio de nombres de Service Bus_ y un _tema de Service Bus_. Para crear ambos, siga los pasos del inicio rápido [*Creación de temas y suscripciones*](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md) de Service Bus. No es necesario completar la sección [*Crear suscripciones al tema*](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md#create-subscriptions-to-the-topic).
* Se necesita una _regla de autorización_. Para crearla, consulte el artículo [*Autenticación y autorización*](../service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature) de Service Bus.

Vaya a la página de detalles de la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com) (para buscarla, escriba su nombre en la barra de búsqueda del portal).

En el menú de la instancia, seleccione _Endpoints_ (Puntos de conexión). Luego, en la página *Endpoints* (Puntos de conexión) que le sigue, seleccione *+ Create an endpoint* (+ Crear un punto de conexión). 

En la página *Create an Endpoint* (Crear un punto de conexión) que se abre, puede crear un punto de conexión de tipo _Service Bus_ seleccionando el botón de radio correspondiente. Escriba un nombre para el punto de conexión en el campo _Name_ (Nombre). Después, en las listas desplegables _Subscription_ (Suscripción), _Service Bus namespace_ (Espacio de nombres de Service Bus), _Service Bus topic_ (Tema de Service Bus) y _Authorization rule_ (Regla de autorización), seleccione la suscripción, el espacio de nombres de Service Bus creado anteriormente, el tema de Service Bus y la regla de autorización, respectivamente.

A continuación, haga clic en _Save_ (Guardar) para crear el punto de conexión.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-service-bus.png" alt-text="Captura de pantalla de la creación de un punto de conexión de tipo Service Bus.":::

Para comprobar que el punto de conexión se ha creado correctamente, examine el icono de notificación de la barra superior de Azure Portal. 

Si se produce un error en la creación del punto de conexión, observe el mensaje de error y vuelva a intentarlo al cabo de unos minutos.

Ahora, el tema de Service Bus está disponible como punto de conexión dentro de Azure Digital Twins, con el nombre especificado en el campo _Name_ (Nombre). Normalmente se usará ese nombre como destino de una **ruta de evento**, que se creará [más tarde en este artículo](#event-routes).

## <a name="event-routes"></a>Rutas de eventos

Para enviar datos de Azure Digital Twins a un punto de conexión, debe definir una **ruta de eventos**. Estas rutas permiten a los desarrolladores conectar el flujo de eventos a lo largo del sistema y a los servicios de nivel inferior. Obtenga más información sobre las rutas de eventos en [*Conceptos: Enrutamiento de eventos de Azure Digital Twins*](concepts-route-events.md).

**Requisito previo**: Debe crear puntos de conexión como se describió anteriormente en este artículo para poder continuar con la creación de una ruta. Puede continuar con la creación de una ruta de evento una vez finalizada la configuración de los puntos de conexión.

>[!NOTE]
>Si ha implementado recientemente los puntos de conexión, compruebe que se haya completado la implementación **antes** de intentar usarlos para una nueva ruta de evento. Si no puede configurar la ruta porque los puntos de conexión no están listos, espere unos minutos y vuelva a intentarlo.

### <a name="create-an-event-route"></a>Crear una ruta de eventos 

Una definición de ruta de eventos contiene estos elementos:
* El nombre de ruta que quiere usar.
* Nombre del punto de conexión que quiere usar.
* Filtro que define los eventos que se envían al punto de conexión.
    - Para deshabilitar la ruta de modo que no se envíe ningún evento, use un valor de filtro de `false`.
    - Para habilitar una ruta que no tenga filtrado específico, use un valor de filtro de `true`.
    - Para más información sobre cualquier otro tipo de filtro, consulte la sección [*Eventos de filtro*](#filter-events) a continuación.

Una ruta puede permitir que se seleccionen varias notificaciones y tipos de eventos.

Para crear una ruta de eventos, vaya a la página de detalles de la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com) (para buscar la instancia, escriba su nombre en la barra de búsqueda del portal).

En el menú de la instancia, seleccione _Event routes_ (Rutas de eventos). Luego, en la página *Event routes* (Rutas de eventos) que le sigue, seleccione *+ + Create an event route* (+ Crear una ruta de evento). 

En la página *Create an event route* (Crear una ruta de evento) que se abre, elija como mínimo:
* Un nombre para la ruta en el campo _Name_ (Nombre).
* El _punto de conexión_ que le gustaría usar para crear la ruta. 

Para que la ruta esté habilitada, también se debe **agregar un filtro de ruta de evento** de al menos `true`. (Si se deja el valor predeterminado de `false`, se creará la ruta, pero no se le enviarán eventos). Para ello, cambie el botón de conmutación de _Advanced editor_ (Editor avanzado) para habilitarlo y escriba `true` en el cuadro *Filter* (Filtro).

:::image type="content" source="media/how-to-manage-routes-portal/create-event-route-no-filter.png" alt-text="Captura de pantalla de la creación de una ruta de evento para la instancia." lightbox="media/how-to-manage-routes-portal/create-event-route-no-filter.png":::

Cuando termine, pulse el botón _Save_ (Guardar) para crear la ruta de evento.

### <a name="filter-events"></a>Filtrado de eventos

Tal y como se ha descrito anteriormente, las rutas tienen un campo **Filter** (Filtro). Si el valor de filtro de la ruta es `false`, no se enviará ningún evento al punto de conexión. 

Después de habilitar el filtro mínimo de `true`, los puntos de conexión recibirán una variedad de eventos de Azure Digital Twins:
* Telemetría que desencadena [Digital Twins](concepts-twins-graph.md) mediante la API del servicio Azure Digital Twins.
* Notificaciones relativas a cambios de propiedades de gemelos, desencadenadas en los cambios de una propiedad de cualquier gemelo en la instancia de Azure Digital Twins.
* Eventos de ciclo de vida, desencadenados cuando se crean o se eliminan gemelos o relaciones.
* Eventos de cambio de modelo, desencadenados cuando se agregan o eliminan [modelos](concepts-models.md) en una instancia de Azure Digital Twins.

Se pueden restringir los tipos de eventos que se envían mediante la definición de un filtro más específico.

Para agregar un filtro de evento mientras se crea una ruta de evento, use la sección _Add an event route filter_ (Agregar un filtro de ruta de evento) de la página *Create an event route* (Crear una ruta de evento). 

Puede seleccionar entre algunas opciones básicas de filtros comunes o usar las opciones avanzadas de filtro para escribir sus propios filtros personalizados.

#### <a name="use-the-basic-filters"></a>Uso de los filtros básicos

Para usar los filtros básicos, expanda la opción _Event types_ (Tipos de evento) y active las casillas correspondientes a los eventos que quiere enviar a su punto de conexión. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-1.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro básico. Activación de las casillas de los eventos.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

El cuadro de texto de filtro se rellena automáticamente con el texto del filtro que ha seleccionado:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-2.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro básico. Se muestra el texto del filtro rellenado automáticamente después de seleccionar los eventos.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

#### <a name="use-the-advanced-filters"></a>Uso de los filtros avanzados

También puede usar la opción de filtro avanzado para escribir sus propios filtros personalizados.

Para crear una ruta de evento con opciones avanzadas de filtro, cambie el botón de conmutación de _Advanced editor_ (Editor avanzado) para habilitarlo. Después, puede escribir sus propios filtros de eventos en el cuadro *Filter* (Filtro):

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-advanced.png" alt-text="Captura de pantalla de la creación de una ruta de evento con un filtro avanzado.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Estos son los filtros de ruta admitidos. Los detalles de la columna *Filter Text Schema* (Esquema de texto de filtro) es el texto que se puede escribir en el cuadro de filtro.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Pasos siguientes

Lea acerca de los diferentes tipos de mensajes de evento que se pueden recibir:
* [*Procedimiento: Interpretación de los datos de evento*](how-to-interpret-event-data.md)
