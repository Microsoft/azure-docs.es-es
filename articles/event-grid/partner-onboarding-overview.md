---
title: Incorporación como asociado de Azure Event Grid
description: Incorporación como tipo de tema de asociado de Azure Event Grid. Comprenda el modelo de recursos y el flujo de publicación para los temas de asociados.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 36f2178b7c21af016f9074d6f973a01cedb873d7
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87826796"
---
# <a name="onboard-as-an-azure-event-grid-partner"></a>Incorporación como asociado de Azure Event Grid

En este artículo se describe cómo usar de forma privada los recursos de asociados de Azure Event Grid y cómo hacer que un tipo de tema de asociado esté disponible públicamente.

No necesita permisos especiales para empezar a usar los tipos de recursos de Event Grid asociados a la publicación de eventos como asociado de Event Grid. De hecho, puede usarlos hoy mismo para publicar eventos de forma privada en sus propias suscripciones a Azure, así como para probar el modelo de recursos si está pensando en convertirse en asociado.

## <a name="become-an-event-grid-partner"></a>Convertirse en asociado de Event Grid

Si está interesado en convertirse en un asociado público de Event Grid, comience por rellenar [este formulario](https://aka.ms/gridpartnerform). A continuación, póngase en contacto con el equipo de Event Grid en [GridPartner@microsoft.com](mailto:gridpartner@microsoft.com).

## <a name="how-partner-topics-work"></a>Funcionamiento de los temas de asociados
En los temas de asociados se toma la arquitectura existente que Event Grid ya utiliza para publicar eventos de recursos de Azure, como Azure Storage y Azure IoT Hub, y hacer que esas herramientas estén disponibles públicamente para que las use cualquier persona. De manera predeterminada, el uso de estas herramientas es privado solo para su suscripción a Azure. Si quiere que los eventos estén disponibles públicamente, rellene el formulario y [póngase en contacto con el equipo de Event Grid](mailto:gridpartner@microsoft.com).

Los temas de asociados le permiten publicar eventos en Azure Event Grid para el consumo multiinquilino.

### <a name="onboarding-and-event-publishing-overview"></a>Introducción a la incorporación y publicación de eventos

#### <a name="partner-flow"></a>Flujo de asociados

1. Cree un inquilino de Azure si aún no tiene ninguno.
1. Use la CLI de Azure para crear una nueva instancia de Event Grid `partnerRegistration`. Este recurso incluye información como el nombre para mostrar, la descripción, el URI de configuración, etc.

    ![Creación de un tema de asociado](./media/partner-onboarding-how-to/create-partner-registration.png)

1. Cree uno o varios espacios de nombres de asociados en cada región en la que quiera publicar eventos. El servicio Event Grid aprovisiona un punto de conexión de publicación (por ejemplo, `https://contoso.westus-1.eventgrid.azure.net/api/events`) y las claves de acceso.

    ![Creación de un espacio de nombres de asociado](./media/partner-onboarding-how-to/create-partner-namespace.png)

1. Proporcione una manera para que los clientes se registren en el sistema en el que quieran un tema de asociado.
1. Póngase en contacto con el equipo de Event Grid si quiere que su tipo de tema de asociado pase a estar disponible públicamente.

#### <a name="customer-flow"></a>Flujo de clientes

1. El cliente visita Azure Portal para tomar nota del grupo de recursos y el identificador de la suscripción de Azure en los que le gustaría que se creara el tema del asociado.
1. El cliente solicita un tema de asociado a través del sistema. En respuesta a esto, puede crear un túnel de eventos para el espacio de nombres del asociado.
1. Event Grid crea un tema de asociado **pendiente** en el grupo de recursos y la suscripción de Azure del cliente.

    ![Creación de un canal de eventos](./media/partner-onboarding-how-to/create-event-tunnel-partner-topic.png)

1. El cliente activa el tema de asociado a través de Azure Portal. Los eventos ahora podrán fluir desde su servicio a la suscripción de Azure del cliente.

    ![Activación de un tema de asociado](./media/partner-onboarding-how-to/activate-partner-topic.png)

## <a name="resource-model"></a>Modelo de recursos


El siguiente modelo de recursos es para los temas de asociados.

### <a name="partner-registrations"></a>Registros de dispositivos
* Recurso: `partnerRegistrations`
* Usado por: Asociados
* Descripción: captura los metadatos globales del asociado de software como servicio (por ejemplo, el nombre, nombre para mostrar, descripción y URI de configuración).
    
    La creación o actualización de un registro de asociado es una operación de autoservicio para los asociados. Esta capacidad de autoservicio permite a los asociados compilar y probar el flujo completo.
    
    Los clientes solo pueden detectar los registros de asociados aprobados por Microsoft.
* Ámbito: creado en la suscripción de Azure del asociado. Los metadatos son visibles para los clientes después de hacerse públicos.

### <a name="partner-namespaces"></a>Espacios de nombres de los asociados
* Recurso: partnerNamespaces
* Usado por: Asociados
* Descripción: proporciona un recurso regional en el que publicar eventos de los clientes. Cada espacio de nombres asociado tiene un punto de conexión de publicación y claves de autenticación. El espacio de nombres también es la forma en que el asociado solicita un tema de asociado para un cliente determinado y enumera a los clientes activos.
* Ámbito: reside en la suscripción del asociado.

### <a name="event-channel"></a>Canal de eventos
* Recurso: `partnerNamespaces/eventChannels`
* Usado por: Asociados
* Descripción: los túneles de eventos son un reflejo del tema de asociado del cliente. Mediante la creación de un túnel de eventos y la especificación del grupo de recursos y la suscripción a Azure del cliente en los metadatos, indica a Event Grid que cree un tema de asociado para el cliente. Event Grid emitirá una llamada a ARM para crear el elemento partnerTopic correspondiente en la suscripción del cliente. El tema del asociado se crea con un estado "pendiente". Hay un vínculo uno a uno entre cada túnel de eventos y cada tema de asociado.
* Ámbito: reside en la suscripción del asociado.

### <a name="partner-topics"></a>Temas de asociados
* Recurso: `partnerTopics`
* Usado por: Clientes
* Descripción: los temas de asociados son similares a los temas personalizados y del sistema en Event Grid. Cada tema de asociado está relacionado con un origen específico (por ejemplo, `Contoso:myaccount`) y un tipo de tema de asociado específico (por ejemplo, Contoso). Los clientes crean suscripciones a eventos en el tema de asociado a fin de enrutar eventos a varios controladores de eventos.

    Los clientes no pueden crear directamente este recurso. La única forma de crear un tema de asociado es a través de una operación de asociado que crea un túnel de eventos.
* Ámbito: reside en la suscripción del cliente.

### <a name="partner-topic-types"></a>Tipos de temas de asociado
* Recurso: `partnerTopicTypes`
* Usado por: Clientes
* Descripción: los tipos de temas de asociados son tipos de recursos de todo el inquilino que permiten a los clientes detectar la lista de tipos de temas de asociados aprobados. La dirección URL tiene el aspecto siguiente: https://management.azure.com/providers/Microsoft.EventGrid/partnerTopicTypes).
* Ámbito: Global

## <a name="publish-events-to-event-grid"></a>Publicación de eventos en Event Grid
Cuando crea un elemento de espacio de nombres de asociado en una región de Azure, obtiene un punto de conexión regional y las claves de autenticación correspondientes. Publique lotes de eventos en este punto de conexión para todos los túneles de eventos de cliente de ese espacio de nombres. Según el campo de origen del evento, Azure Event Grid asigna cada evento con los temas de asociados correspondientes.

### <a name="event-schema-cloudevents-v10"></a>Esquema del evento: CloudEvents v1.0
Publique eventos en Azure Event Grid mediante el esquema CloudEvents 1.0. Event Grid admite el modo estructurado y por lotes. CloudEvents 1.0 es el único esquema de eventos que se admite para los espacios de nombres de los asociados.

### <a name="example-flow"></a>Ejemplo de flujo

1.  El servicio de publicación realiza una solicitud HTTP POST para `https://contoso.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01`.
1.  En la solicitud, incluya un valor de encabezado denominado aeg-sas-key que contenga una clave para la autenticación. Esta clave se aprovisiona durante la creación del espacio de nombres del asociado. Por ejemplo, un valor de encabezado válido es aeg-sas-key: VXbGWce53249Mt8wuotr0GPmyJ/nDT4hgdEj9DpBeRr38arnnm5OFg==.
1.  Establezca el encabezado Content-Type en "application/cloudevents-batch+json; charset=UTF-8a".
1.  Realice una solicitud HTTP POST a la dirección URL de publicación con el lote de eventos correspondiente a esa región. Por ejemplo:

``` json
[
{
    "specversion" : "1.0-rc1",
    "type" : "com.contoso.ticketcreated",
    "source" : " com.contoso.account1",
    "subject" : "tickets/123",
    "id" : "A234-1234-1234",
    "time" : "2019-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
          object-unique-to-each-publisher
    }
},
{
    "specversion" : "1.0-rc1",
    "type" : "com.contoso.ticketclosed",
    "source" : "https://contoso.com/account2",
    "subject" : "tickets/456",
    "id" : "A234-1234-1234",
    "time" : "2019-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
          object-unique-to-each-publisher
    }
}
]
```

Después de realizar una publicación en el punto de conexión partnerNamespace, recibirá una respuesta. La respuesta es un código de respuesta HTTP estándar. Algunas respuestas comunes son:

| Resultado                             | Response              |
|------------------------------------|-----------------------|
| Correcto                            | 200 OK                |
| Los datos del evento tienen un formato incorrecto    | 400 - Solicitud incorrecta       |
| Clave de acceso no válida                 | 401 No autorizado      |
| Punto de conexión incorrecto                 | 404 No encontrado         |
| La matriz o el evento superan los límites de tamaño | 413 Carga demasiado grande |

## <a name="references"></a>Referencias

  * [Swagger](https://github.com/ahamad-MS/azure-rest-api-specs/blob/master/specification/eventgrid/resource-manager/Microsoft.EventGrid/preview/2020-04-01-preview/EventGrid.json)
  * [Plantilla ARM](/azure/templates/microsoft.eventgrid/allversions)
  * [Esquema de plantilla de ARM](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2020-04-01-preview/Microsoft.EventGrid.json)
  * [API de REST](/rest/api/eventgrid/version2020-04-01-preview/partnernamespaces)
  * [Extensión de la CLI](/cli/azure/ext/eventgrid/?view=azure-cli-latest)

### <a name="sdks"></a>SDK
  * [.NET](https://www.nuget.org/packages/Microsoft.Azure.Management.EventGrid/5.3.1-preview)
  * [Python](https://pypi.org/project/azure-mgmt-eventgrid/3.0.0rc6/)
  * [Java](https://search.maven.org/artifact/com.microsoft.azure.eventgrid.v2020_04_01_preview/azure-mgmt-eventgrid/1.0.0-beta-3/jar)
  * [Ruby](https://rubygems.org/gems/azure_mgmt_event_grid/versions/0.19.0)
  * [JS](https://www.npmjs.com/package/@azure/arm-eventgrid/v/7.0.0)
  * [Go](https://github.com/Azure/azure-sdk-for-go)


## <a name="next-steps"></a>Pasos siguientes
- [Introducción a los temas de asociados](partner-topics-overview.md)
- [Formulario de incorporación de temas de asociados](https://aka.ms/gridpartnerform)
- [Tema de asociado de Auth0](auth0-overview.md)
- [Procedimientos para usar el tema de asociado de Auth0](auth0-how-to.md)
