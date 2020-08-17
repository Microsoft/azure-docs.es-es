---
title: Información general de alertas y supervisión de notificaciones en Azure
description: Información general de las alertas de Azure Alertas, alertas clásicas e interfaz de alertas.
ms.subservice: alerts
ms.topic: conceptual
ms.date: 01/28/2018
ms.openlocfilehash: e0741a23d7e5ece0898d83c53782afc353d9a7e5
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371607"
---
# <a name="overview-of-alerts-in-microsoft-azure"></a>Información general sobre las alertas en Microsoft Azure 

En este artículo se explica qué son las alertas, sus beneficios y cómo empezar a utilizarlas.  

## <a name="what-are-alerts-in-microsoft-azure"></a>¿Qué son las alertas en Microsoft Azure?
Las alertas le informan de forma proactiva cuando se detectan condiciones importantes en los datos que supervisa. Le permiten identificar y solucionar los problemas antes de que los usuarios del sistema puedan verlos. 

En este artículo se describe la experiencia unificada de alertas en Azure Monitor, que ahora incluye las alertas que antes administraban Log Analytics y Application Insights. La [experiencia de alerta anterior](alerts-classic.overview.md) y los tipos de alertas se denominan *alertas clásicas*. Para ver esta experiencia anterior y el tipo de alerta anterior, haga clic en **Ver alertas clásicas** en la parte superior de la página de alertas. 

## <a name="overview"></a>Información general

En el siguiente diagrama se representa el flujo de alertas. 

![Diagrama del flujo de alertas](media/alerts-overview/Azure-Monitor-Alerts.svg)

Las reglas de alertas están separadas de las alertas y las acciones que se realizan cuando se activa una alerta. La regla de alertas captura el destino y los criterios para las alertas. La regla de alertas puede tener el estado deshabilitado o habilitado. Las alertas solo se activan cuando están habilitadas. 

Los atributos clave de las reglas de alertas son:

**Recurso de destino**: define el ámbito y las señales disponibles para las alertas. Un destino puede ser cualquier recurso de Azure. Destinos de ejemplo: una máquina virtual, una cuenta de almacenamiento, un conjunto de escalado de máquinas virtuales, un área de trabajo de Log Analytics o un recurso de Application Insights. Para determinados recursos (por ejemplo, Virtual Machines), puede especificar varios recursos como destino de la regla de alertas.

**Señal**: la emite el recurso de destino. Las señales pueden ser de los siguientes tipos: métrica, registro de actividad, Application Insights y registro.

**Criterios**: Combinación de señales y lógica aplicadas en un recurso de destino. Ejemplos: 

- Porcentaje de la CPU superior al 70 %
- Tiempo de respuesta del servidor superior a 4 ms 
- Recuento de resultados de una consulta de registro superior a 100

**Nombre de la alerta**: nombre específico para la regla de alertas que haya configurado el usuario.

**Descripción de la alerta**: descripción de la regla de alertas que haya configurado el usuario.

**Gravedad**: gravedad de la alerta, una vez que se cumplen los criterios especificados en la regla de alertas. La gravedad puede tener un valor entre 0 y 4.

- Sev 0 = Crítica
- Sev 1 = Error
- Sev 2 - Advertencia
- Sev 3 = Informativa
- Sev 4 = Detallada 

**Acción**: una acción específica llevada a cabo al desencadenarse la alerta. Para más información, vea [Grupos de acciones](./action-groups.md).

## <a name="what-you-can-alert-on"></a>Tipo de alertas que se pueden realizar

Puede enviar alertas sobre métricas y registros tal y como se describe en el artículo sobre los [orígenes de datos de supervisión](./data-sources.md). Estas incluyen, pero no se limitan a:

- Valores de métrica
- Consultas de búsqueda de registros
- Eventos del registro de actividad
- Estado de la plataforma Azure subyacente
- Pruebas de disponibilidad del sitio web

Anteriormente, las métricas de Azure Monitor, Application Insights, Log Analytics y Service Health tenían funcionalidades independientes de generación de alertas. Con el tiempo, Azure ha mejorado y combinado la interfaz de usuario y los distintos métodos de generación de alertas. Esta consolidación aún está en proceso. Como resultado, algunas funcionalidades de alertas aún no se encuentran en el nuevo sistema de alertas.  

| **Origen de supervisión** | **Tipo de señal**  | **Descripción** |
|-------------|----------------|-------------|
| Estado del servicio | Registro de actividades  | No compatible. Consulte [Creación de alertas del registro de actividad en notificaciones del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md).  |
| Application Insights | Pruebas de disponibilidad web | No compatible. Consulte [Alertas de pruebas web](../app/monitor-web-app-availability.md). Disponible para cualquier sitio web instrumentado para enviar datos a Application Insights. Reciba una notificación cuando la disponibilidad o la capacidad de respuesta de un sitio web está por debajo de las expectativas. |

## <a name="manage-alerts"></a>Administrar alertas
Puede establecer el estado de una alerta para especificar dónde se encuentra en el proceso de resolución. Cuando se cumplen los criterios especificados en la regla de alertas, se crea o se desencadena una alerta que tiene el estado *Nueva*. Puede cambiar el estado cuando se confirma una alerta y cuando se cierra. Todos los cambios de estado se almacenan en el historial de la alerta.

Se admiten los siguientes estados de alerta.

| State | Descripción |
|:---|:---|
| Nuevo | Se acaba de detectar el problema y todavía no se ha revisado. |
| Confirmado | Un administrador revisó la alerta y empezó a trabajar en ella. |
| Closed | Se resolvió el problema. Después de cerrar una alerta, puede volver a abrirla mediante el cambio a otro estado. |

El *estado de alerta* es diferente e independiente de la *condición de supervisión*. El estado de alerta lo establece el usuario. La condición de supervisión la establece el sistema. Cuando se desencadena una alerta, la condición de supervisión de la alerta se establece en *desencadenada*. Cuando desaparece la condición subyacente que provocó que se desencadenara la alerta, la condición de supervisión se establece en *resuelta*. El estado de alerta no cambia hasta que el usuario lo cambia. Obtenga información sobre [cómo cambiar el estado de las alertas y de los grupos inteligentes](https://aka.ms/managing-alert-smart-group-states).

## <a name="smart-groups"></a>Grupos inteligentes 

Los grupos inteligentes son agregaciones de alertas basadas en algoritmos de aprendizaje automático que pueden ayudar a reducir el ruido de las alertas y contribuyen a la solución de problemas. [Más información acerca de los grupos inteligentes](https://aka.ms/smart-groups) y [cómo administrar los grupos inteligentes](https://aka.ms/managing-smart-groups).


## <a name="alerts-experience"></a>Experiencia de alertas 
La página de alertas predeterminada proporciona un resumen de las alertas que se crean dentro un intervalo de tiempo determinado. Muestra el total de alertas de cada gravedad, con columnas que identifican el número total de alertas en cada estado para cada gravedad. Seleccione cualquiera de las gravedades para abrir la página [Todas las alertas](#all-alerts-page) filtrada según esa gravedad.

Como alternativa, puede [enumerar mediante programación las instancias de alertas generadas en las suscripciones mediante las API de REST](#manage-your-alert-instances-programmatically).

> [!NOTE]
   >  Solo puede acceder a las alertas generadas en los últimos 30 días.

No muestra ni realiza un seguimiento de las alertas clásicas. Puede cambiar las suscripciones o los parámetros de filtro para actualizar la página. 

![Captura de pantalla de la página de alertas](media/alerts-overview/alerts-page.png)

Para filtrar esta vista, seleccione los valores en los menús desplegables que aparecen en la parte superior de la página.

| Columna | Descripción |
|:---|:---|
| Subscription | Seleccione las suscripciones a Azure de las que quiere ver las alertas. Si lo desea, puede seleccionar todas las suscripciones. Solo las alertas a las que tiene acceso de las suscripciones seleccionadas se incluyen en la vista. |
| Resource group | Seleccione un solo grupo de recursos. Solo las alertas con destinos en el grupo de recursos seleccionado se incluyen en la vista. |
| Intervalo de horas | Solo las alertas activadas dentro del intervalo de tiempo seleccionado se incluyen en la vista. Los valores compatibles son Última hora, Últimas 24 horas, Últimos 7 días y Últimos 30 días. |

Seleccione los valores siguientes en la parte superior de la página de alertas para abrir otra página:

| Value | Descripción |
|:---|:---|
| Total de alertas | Número total de las alertas que coinciden con los criterios seleccionados. Seleccione este valor para abrir sin ningún filtro la vista Todas las alertas. |
| Grupos inteligentes | Número total de grupos inteligentes creados a partir de las alertas que coinciden con los criterios seleccionados. Seleccione este valor para abrir la lista de grupos inteligentes en la vista Todas las alertas.
| Total de reglas de alertas | Número total de reglas de alertas en la suscripción y el grupo de recursos que seleccionó. Seleccione este valor para abrir la vista Reglas filtrada en la suscripción y el grupo de recursos que seleccionó.


## <a name="manage-alert-rules"></a>Administración de reglas de alerta
Haga clic en **Administrar reglas de alertas** para mostrar la página **Reglas**. La página de reglas es una sola página que permite administrar todas las reglas de alertas de las suscripciones a Azure. Se enumeran todas las reglas de alertas que se pueden ordenar según los recursos de destino, los grupos de recursos, el nombre de regla o el estado. También puede editar, habilitar o deshabilitar las reglas de alertas desde esta página.  

 ![Captura de pantalla de la página de reglas](./media/alerts-overview/alerts-preview-rules.png)


## <a name="create-an-alert-rule"></a>Crear una regla de alerta
Puede crear alertas uniformes independientemente del tipo de señal o del servicio de supervisión.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4tflw]

 
Aquí se explica cómo crear una nueva regla de alerta:
1. Elija el _destino_ de la alerta.
1. Seleccione la _señal_ de las señales disponibles para el destino.
1. Especifique la _lógica_ que se aplicará a los datos desde la señal.
 
Este proceso simplificado de creación ya no requiere que el usuario sepa las señales o el origen de supervisión admitidos antes de seleccionar un recurso de Azure. La lista de señales disponibles se filtra automáticamente según el recurso de destino que seleccione. También en función de ese destino, se le guía por la definición de la lógica de la regla de alerta automáticamente.  

Puede obtener más información sobre cómo crear reglas de alertas en [Creación, visualización y administración de alertas mediante Azure Monitor](./alerts-metric.md).

Las alertas están disponibles a través de varios servicios de supervisión de Azure. Para obtener más información sobre cómo y cuándo usar cada uno de estos servicios, consulte [Supervisión de aplicaciones y recursos de Azure](../overview.md). 


## <a name="all-alerts-page"></a>Página Todas las alertas 
Haga clic en **Total de alertas** para ver la página de **todas las alertas**. Aquí puede ver una lista de las alertas que se crearon en el intervalo de tiempo seleccionado. Puede ver una lista de las alertas individuales o una lista de los grupos inteligentes que contienen las alertas. Seleccione el banner en la parte superior de la página para alternar entre las vistas.

![Captura de pantalla de la página de alertas](media/alerts-overview/all-alerts-page.png)

Para filtrar la vista, seleccione los valores siguientes en los menús desplegables que aparecen en la parte superior de la página:

| Columna | Descripción |
|:---|:---|
| Subscription | Seleccione las suscripciones a Azure de las que quiere ver las alertas. Si lo desea, puede seleccionar todas las suscripciones. Solo las alertas a las que tiene acceso de las suscripciones seleccionadas se incluyen en la vista. |
| Resource group | Seleccione un solo grupo de recursos. Solo las alertas con destinos en el grupo de recursos seleccionado se incluyen en la vista. |
| Tipo de recurso | Seleccione uno o varios tipos de recurso. Solo las alertas con destinos del tipo seleccionado se incluyen en la vista. Esta columna solo está disponible tras especificar un grupo de recursos. |
| Resource | Seleccione un recurso. Solo las alertas con ese recurso como destino se incluyen en la vista. Esta columna solo está disponible tras especificar un tipo de recurso. |
| severity | Seleccione un nivel de gravedad de alerta o seleccione **Todo** para incluir alertas de todos los niveles de gravedad. |
| Condición de supervisión | Seleccione una condición de supervisión o seleccione **Todo** para incluir alertas de todas las condiciones. |
| Estado de alerta | Seleccione un estado de alerta o seleccione **Todo** para incluir alertas de todos los estados. |
| Servicio de supervisión | Seleccione un servicio o seleccione **Todo** para incluir todos los servicios. Solo se incluyen las alertas que se crean mediante las reglas que usan ese servicio como destino. |
| Intervalo de horas | Solo las alertas activadas dentro del intervalo de tiempo seleccionado se incluyen en la vista. Los valores compatibles son Última hora, Últimas 24 horas, Últimos 7 días y Últimos 30 días. |

Seleccione **Columnas** en la parte superior de la página para seleccionar las columnas que quiere mostrar. 

## <a name="alert-details-page"></a>Página Detalles de la alerta
Cuando seleccione una alerta, esta página le proporcionará detalles de la alerta y le permitirá cambiar su estado.

![Captura de pantalla de la página de detalles de alertas](media/alerts-overview/alert-detail2.png)

La página de detalles de alertas incluye las secciones siguientes:

| Sección | Descripción |
|:---|:---|
| Resumen | Muestra las propiedades y otra información importante sobre la alerta. |
| Historial | Muestra cada acción que realiza la alerta y los cambios hechos en la alerta. Actualmente se limita a cambios de estado. |
| Diagnóstico | Información sobre el grupo inteligente en el que se incluye la alerta. El *recuento de alertas* se refiere al número de alertas incluidas en el grupo inteligente. Incluye en el mismo grupo inteligente otras alertas que se crearon en los últimos 30 días, con independencia del filtro de tiempo de la página de lista de alertas. Seleccione una alerta para ver sus detalles. |

## <a name="role-based-access-control-rbac-for-your-alert-instances"></a>Control de acceso basado en rol (RBAC) para las instancias de alertas

El consumo y la administración de instancias de alertas requiere que el usuario tenga los roles integrados de Azure de [colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) o [lector de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-reader). Estos roles se admiten en cualquier ámbito de Azure Resource Manager, desde el nivel de la suscripción hasta las asignaciones pormenorizadas en el nivel de un recurso. Por ejemplo, si un usuario solo tiene acceso de "colaborador de supervisión" para la máquina virtual `ContosoVM1`, puede consumir y administrar solo las alertas generadas en `ContosoVM1`.

## <a name="manage-your-alert-instances-programmatically"></a>Administrar las instancias de alertas mediante programación

Es posible que quiera consultar mediante programación las alertas generadas en la suscripción. Esto podría llevarse a cabo para crear vistas personalizadas fuera de Azure Portal, o bien para analizar las alertas a fin de identificar patrones y tendencias.

Puede consultar las alertas generadas en sus suscripciones, ya sea mediante la [API de REST de Alert Management](https://aka.ms/alert-management-api), [Azure Resource Graph](../../governance/resource-graph/overview.md) o la [API de REST de Recursos](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources).

La API de REST de Resource Graph para Recursos permite consultar instancias de alertas a escala. Esto se recomienda cuando tiene que administrar las alertas generadas en muchas suscripciones. 

La siguiente solicitud de ejemplo a la API de REST de Resource Graph devuelve el recuento de alertas de una suscripción:

```json
{
  "subscriptions": [
    <subscriptionId>
  ],
  "query": "AlertsManagementResources | where type =~ 'Microsoft.AlertsManagement/alerts' | summarize count()"
}
```

También puede ver el resultado de esta consulta de Resource Graph en el portal con el explorador de Azure Resource Graph: [portal.azure.com](https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/AlertsManagementResources%20%7C%20where%20type%20%3D~%20%27Microsoft.AlertsManagement%2Falerts%27%20%7C%20summarize%20count()).

Las alertas se pueden consultar para sus campos [básicos](alerts-common-schema-definitions.md#essentials).

Puede usar la [API de REST de Alert Management](https://aka.ms/alert-management-api) para obtener más información sobre alertas específicas, incluidos los campos [contexto de alerta](alerts-common-schema-definitions.md#alert-context) correspondientes.

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre los grupos inteligentes](https://aka.ms/smart-groups)
- [Información sobre los grupos de acciones](./action-groups.md)
- [Administración de instancias de alertas en Azure](https://aka.ms/managing-alert-instances)
- [Administración de grupos inteligentes](https://aka.ms/managing-smart-groups)
- [Más información acerca de los precios de alertas de Azure](https://azure.microsoft.com/pricing/details/monitor/)
