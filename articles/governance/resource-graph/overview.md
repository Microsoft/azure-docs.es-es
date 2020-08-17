---
title: Información general de Azure Resource Graph
description: Conozca la forma en que el servicio Azure Resource Graph permite realizar consultas complejas de recursos a escala en suscripciones e inquilinos.
ms.date: 07/25/2020
ms.topic: overview
ms.openlocfilehash: 5a2be5e65ecd5590d992e1883f432c173660e78d
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541794"
---
# <a name="what-is-azure-resource-graph"></a>¿Qué es Azure Resource Graph?

Azure Resource Graph es un servicio de Azure diseñado para extender Azure Resource Management; para ello, proporciona una exploración de recursos eficaz y de alto rendimiento con la capacidad de consultar a escala a través de un conjunto especificado de suscripciones para que pueda controlar eficazmente el entorno. Estas consultas proporcionan las siguientes características:

- Capacidad para consultar recursos con propiedades complejas de filtrado, agrupación y ordenación por recursos.
- Capacidad de explorar recursos de forma iterativa en función de los requisitos de gobernanza.
- Capacidad de evaluar el impacto de aplicar directivas en un entorno de nube de gran capacidad.
- Capacidad de [detallar los cambios realizados en las propiedades de los recursos](./how-to/get-resource-changes.md) (versión preliminar).

En esta documentación, repasará cada característica en detalle.

> [!NOTE]
> Azure Resource Graph se usa en la barra de búsqueda de Azure Portal, en la nueva experiencia de exploración de "Todos los recursos" y en el [historial de cambios](../policy/how-to/determine-non-compliance.md#change-history)
> _diferencia visual_ de Azure Policy. Se ha diseñado para ayudar a los clientes a administrar entornos a gran escala.

[!INCLUDE [azure-lighthouse-supported-service](../../../includes/azure-lighthouse-supported-service.md)]

## <a name="how-does-resource-graph-complement-azure-resource-manager"></a>Cómo Resource Graph complementa Azure Resource Manager

Actualmente, Resource Manager admite consultas sobre campos de recursos básicos, específicamente: nombre del recurso, identificador, tipo, grupo de recursos, suscripción y ubicación. Resource Manager también proporciona los medios para llamar a proveedores de recursos individuales a fin de obtener las propiedades detalladas de los recursos de uno en uno.

Con Azure Resource Graph, puede tener acceso a estas propiedades que devuelven los proveedores de recursos sin necesidad de realizar llamadas individuales a cada proveedor de recursos. Para ver una lista de los tipos de recursos admitidos, revise la [referencia de tabla y tipo de recurso](./reference/supported-tables-resources.md). Una manera alternativa de ver los tipos de recursos admitidos es mediante el [explorador de esquemas de Azure Resource Graph Explorer](./first-query-portal.md#schema-browser).

Con Azure Resource Graph, puede:

- Acceder a las propiedades que devuelven los proveedores de recursos sin necesidad de realizar llamadas individuales a cada uno de ellos.
- Ver los últimos 14 días del historial de cambios realizados en el recurso para ver qué propiedades cambiaron y cuándo lo hicieron. (versión preliminar)

## <a name="how-resource-graph-is-kept-current"></a>Cómo se mantiene actualizado Resource Graph

Cuando se actualiza un recurso de Azure, Resource Manager informa a Resource Graph del cambio.
A continuación, Resource Graph actualiza su base de datos. Resource Graph también realiza un _examen completo_ regular. Este examen garantiza que los datos de Resource Graph sean actuales en caso de que falten notificaciones o al actualizarse un recurso fuera de Resource Manager.

> [!NOTE]
> Resource Graph usa una operación `GET` en la API de la versión no preliminar más reciente de cada proveedor de recursos para recopilar propiedades y valores. Como resultado, es posible que la propiedad esperada no esté disponible. En algunos casos, la versión de la API usada se ha invalidado para proporcionar propiedades más actuales o ampliamente utilizadas en los resultados. Consulte el ejemplo de [Mostrar la versión de API para cada tipo de recurso](./samples/advanced.md#apiversion) para obtener una lista completa de su entorno.

## <a name="the-query-language"></a>El lenguaje de consulta

Ahora que comprende mejor qué es Azure Resource Graph, vamos a profundizar en cómo crear consultas.

Es importante entender que el lenguaje de consulta de Azure Resource Graph se basa en el [lenguaje de consulta de Kusto](/azure/data-explorer/data-explorer-overview) que usa Azure Data Explorer.

En primer lugar, para obtener información sobre las operaciones y funciones que se pueden usar con Azure Resource Graph, consulte [Lenguaje de consulta de Resource Graph](./concepts/query-language.md). Para examinar los recursos, consulte [Explorar recursos](./concepts/explore-resources.md).

## <a name="permissions-in-azure-resource-graph"></a>Permisos en Azure Resource Graph

Para usar Resource Graph, debe tener los derechos adecuados en el [Control de acceso basado en rol](../../role-based-access-control/overview.md) (RBAC) con al menos acceso de lectura a los recursos que quiera consultar. Sin tener al menos permisos del tipo `read` para el objeto de Azure o el grupo de objetos, no se devolverán los resultados.

> [!NOTE]
> Resource Graph usa las suscripciones disponibles de una entidad de seguridad durante el inicio de sesión. Para ver los recursos de una nueva suscripción que se agregan durante una sesión activa, la entidad de seguridad debe actualizar el contexto. Esta acción se produce automáticamente al cerrar sesión y volver a iniciarla posteriormente.

La CLI de Azure y Azure PowerShell usan suscripciones a las que el usuario tiene acceso. Si se usa la API REST directamente, el usuario es quien proporciona la lista de suscripciones. Si el usuario tiene acceso a cualquiera de las suscripciones de la lista, los resultados de la consulta se devuelven a las suscripciones a las que el usuario tiene acceso. Este comportamiento es el mismo que cuando se llama a [Grupo de recursos: Lista](/rest/api/resources/resourcegroups/list) \-, se obtienen grupos de recursos a los que se tiene acceso sin indicar que el resultado puede ser parcial. Si no hay suscripciones en la lista de suscripciones para las que el usuario tiene los derechos adecuados, la respuesta es _403_ (prohibido).

> [!NOTE]
> En la **versión preliminar** de la API REST, versión `2020-04-01-preview`, es posible que la lista de suscripciones se haya omitido.
> Si las propiedades `subscriptions` y `managementGroupId` no se definen en la solicitud, el _ámbito_ se establece en el inquilino. Para más información, consulte [Ámbito de la consulta](./concepts/query-language.md#query-scope).

## <a name="throttling"></a>Limitaciones

Como servicio gratuito, las consultas a Resource Graph se limitan para proporcionar la mejor experiencia y tiempo de respuesta para todos los clientes. Si la organización quiere usar Resource Graph API para consultas frecuentes y a gran escala, use el portal "Comentarios" de la [página Resource Graph del portal](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyMenuBlade/ResourceGraph).
Proporcione su caso de negocio y seleccione la casilla "Microsoft puede ponerse en contacto con usted por correo electrónico en relación con sus comentarios" para que el equipo pueda ponerse en contacto con usted.

Resource Graph limita las consultas en el nivel de usuario. La respuesta del servicio contiene estos encabezados HTTP:

- `x-ms-user-quota-remaining` (int): la cuota de recurso restante para el usuario. Este valor se asigna al número de consultas.
- `x-ms-user-quota-resets-after` (hh:mm:ss): tiempo transcurrido hasta que se restablece el consumo de la cuota de un usuario.

Para más información, consulte [Guía para soluciones limitadas](./concepts/guidance-for-throttled-requests.md).

## <a name="running-your-first-query"></a>Ejecución de la primera consulta

Azure Resource Graph Explorer, que forma parte de Azure Portal, permite ejecutar consultas de Resource Graph directamente en Azure Portal. Ancle los resultados como gráficos dinámicos para proporcionar información dinámica en tiempo real al flujo de trabajo del portal. Para más información, consulte [Primera consulta con Azure Resource Graph Explorer](./first-query-portal.md).

Resource Graph admite la CLI de Azure, Azure PowerShell, y el SDK de Azure para Python, entre otros muchos. La consulta se estructura igual para cada lenguaje. Aprenda a habilitar Resource Graph con:

- [Azure Portal y Resource Graph Explorer](./first-query-portal.md) 
- [CLI de Azure](./first-query-azurecli.md#add-the-resource-graph-extension)
- [Azure PowerShell](./first-query-powershell.md#add-the-resource-graph-module)
- [Python](./first-query-python.md#add-the-resource-graph-library)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [lenguaje de consulta](./concepts/query-language.md).
- Vea el lenguaje en uso en[Consultas básicas](./samples/starter.md).
- Vea los usos avanzados en [Consultas avanzadas](./samples/advanced.md).
