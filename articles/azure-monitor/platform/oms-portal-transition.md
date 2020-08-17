---
title: Traslado del portal de OMS a Azure | Microsoft Docs
description: El portal de OMS desaparecerá al trasladar toda la funcionalidad a Azure Portal. En este artículo se ofrecen detalles sobre esta transición.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/22/2019
ms.openlocfilehash: 9c4e9676208a5d2cfa8c0c04a97675a3c634e3d5
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446786"
---
# <a name="oms-portal-moving-to-azure"></a>Traslado del portal de OMS a Azure

> [!NOTE]
> Este artículo se aplica a la nube pública de Azure y a la nube del gobierno excepto donde se indique lo contrario.

**El portal de OMS para la nube pública de Azure se ha retirado oficialmente. El portal de OMS para la nube de Azure US Government se retiró oficialmente 15 de mayo de 2019.** Estamos entusiasmados con la mudanza a Azure Portal y esperamos que la transición sea fácil. Pero sabemos que los cambios son difíciles y pueden ser perturbadores. En el resto de este artículo se repasan los principales escenarios y la hoja de ruta para esta transición.

Azure Portal es el centro de todos los servicios de Azure y ofrece una rica experiencia de administración con funcionalidades tales como paneles de información de anclaje, búsqueda inteligente y etiquetado para la administración de recursos. Para consolidar y agilizar el flujo de trabajo de supervisión y administración, empezamos por agregar las funcionalidades del portal de OMS a Azure Portal. Todas las características del portal de OMS forman parte ahora de Azure Portal. De hecho, algunas de las nuevas características, como Análisis de tráfico, solo están disponibles en Azure Portal. Con Azure Portal podrá llevar a cabo todo lo que hacía en el portal de OMS y mucho más. Si aún no lo ha hecho, empiece a usar Azure Portal hoy mismo.

## <a name="what-is-changing"></a>¿Qué está cambiando? 
Se anuncian los siguientes cambios al quedar en desuso el portal de OMS. Cada uno de estos cambios se describe con más detalle en las siguientes secciones.

- Las [áreas de trabajo](#new-workspaces) nuevas solo se pueden crear en Azure Portal.
- La nueva experiencia de Alert Management [sustituye a la solución Alert Management](#changes-to-alerts).
- La [administración de acceso de los usuarios](#user-access-and-role-migration) se realiza en Azure Portal con el control de acceso basado en rol de Azure.
- [Application Insights Connector ya no se necesita](#application-insights-connector-and-solution), ya que la misma funcionalidad está habilitada mediante las consultas de un área de trabajo a otra.
- Se va a dejar de utilizar la [aplicación móvil de OMS](#oms-mobile-app). 
- La [solución NSG se reemplaza](#azure-network-security-group-analytics) por las funcionalidades mejoradas disponibles a través de la solución Análisis de tráfico.
- Las nuevas conexiones de System Center Operations Manager con Log Analytics requieren [módulos de administración actualizados](#system-center-operations-manager).
- Consulte [Migración de las implementaciones de actualizaciones de OMS a Azure](../../automation/migrate-oms-update-deployments.md) para más información sobre los cambios de [Update Management](../../automation/update-management/update-mgmt-overview.md).


## <a name="what-should-i-do-now"></a>¿Qué debo hacer a continuación?
Aunque la mayoría de las características seguirán funcionando sin necesidad de migración, debe realizar las siguientes tareas:

- Deberá [migrar sus permisos de usuario](#user-access-and-role-migration) a Azure Portal.
- Consulte [Migración de las implementaciones de actualizaciones de OMS a Azure](../../automation/migrate-oms-update-deployments.md) para más información sobre la transición de la solución Update Management.

Consulte [Preguntas comunes para la transición desde el portal de OMS a Azure Portal para los usuarios de Log Analytics](../overview.md) para información sobre cómo realizar la transición a Azure Portal. 

## <a name="user-access-and-role-migration"></a>Migración de roles y acceso de usuario
La administración de acceso a Azure Portal es más completa y eficaz que la administración de acceso al portal de OMS. Consulte [Diseño del área de trabajo de registros de Azure Monitor](design-logs-deployment.md) para obtener más información sobre la administración de acceso en Log Analytics.

> [!NOTE]
> En las versiones anteriores de este artículo se indica que los permisos pasarían automáticamente del portal de OMS a Azure Portal. Esta conversión automática ya no está prevista y debe realizarla el usuario.

Puede que ya tenga el acceso adecuado en Azure Portal, en cuyo caso no es necesario realizar ningún cambio. Hay un par de casos en los que es posible que no tenga el acceso adecuado, en cuyo caso el administrador debe asignar los permisos.

- Tiene permisos de usuario de solo lectura en el portal de OMS, pero ninguno en Azure Portal. 
- Tiene permisos de colaborador en el portal de OMS, pero de solo lectura en Azure Portal.
 
En ambos casos, el administrador debe asignar manualmente el rol adecuado de la tabla siguiente. Se recomienda asignar este rol en el nivel de suscripción o de grupo de recursos.  Se proporcionarán instrucciones más normativas en breve para ambos casos.

| Permiso del portal de OMS | Rol de Azure |
|:---|:---|
| ReadOnly | Lector de Log Analytics |
| Colaborador | Colaborador de Log Analytics |
| Administrador | Propietario | 
 

## <a name="new-workspaces"></a>Nuevas áreas de trabajo
Ya no podrá crear áreas de trabajo mediante el portal de OMS. Siga las instrucciones de [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md) para crear un área de trabajo en Azure Portal.

## <a name="changes-to-alerts"></a>Cambios a las alertas

### <a name="alert-extension"></a>Extensión de alerta  

Las alertas se han [ampliado a Azure Portal](./alerts-unified-log.md) Las alertas existentes seguirán apareciendo en el portal de OMS, pero solo se pueden administrar en Azure Portal. Si obtiene acceso a las alertas mediante programación con la API de REST para alertas de Log Analytics o la plantilla de recursos de alerta de Log Analytics, tendrá que usar grupos de acciones en lugar de acciones en las llamadas API, las plantillas de Azure Resource Manager y los comandos de PowerShell.

### <a name="alert-management-solution"></a>solución de administración de alertas
Se ha producido un cambio en un anuncio anterior, y es que la [solución de administración de alertas](alert-management-solution.md) seguirá estando disponible y se admitirá completamente en Azure Portal. Puede seguir instalando la solución desde Azure Marketplace.

Aunque la solución Administración de alertas sigue estando disponible, le animamos a usar la [interfaz de alerta unificada de Azure Monitor](alerts-overview.md) para visualizar y administrar todas las alertas en Azure. Esta nueva experiencia agrega de forma nativa alertas que proceden de varios orígenes en Azure, como, por ejemplo, alertas de registro de Log Analytics. Si va a usar la interfaz de alerta unificada de Azure Monitor, la solución Administración de alertas solo es necesaria para permitir la integración de alertas de System Center Operation Manager en Azure. En la interfaz de alerta unificada de Azure Monitor, puede ver las distribuciones de las alertas, aprovechar la agrupación automatizada de alertas relacionadas mediante grupos inteligentes y ver alertas en varias suscripciones mientras aplica filtros complejos. De esta nueva experiencia se derivarán principalmente futuros avances en la administración de alertas. 

Los datos recopilados por la solución Administración de alertas (registros con un tipo de alerta) seguirán estando en Log Analytics siempre que la solución se instale en el área de trabajo. 

## <a name="oms-mobile-app"></a>Aplicación móvil de OMS
La aplicación móvil de OMS desaparecerá con el portal de OMS. En lugar de la aplicación móvil de OMS, para el acceso a información sobre la infraestructura de TI, los paneles de información y las consultas guardadas, tiene acceso a Azure Portal directamente desde el explorador del dispositivo móvil. Para obtener alertas, debe configurar [grupos de acciones de Azure](action-groups.md) para recibir notificaciones en forma de SMS o llamada de voz

## <a name="application-insights-connector-and-solution"></a>Application Insights Connector y solución
[Application Insights Connector](app-insights-connector.md) ofrece una forma de incorporar datos de Application Insights a un área de trabajo de Log Analytics. Esta duplicación de datos era necesaria para permitir la visibilidad de los datos de la infraestructura y las aplicaciones. Gracias a la compatibilidad con la retención de datos ampliada de Application Insights en marzo de 2019 y la posibilidad de realizar [consultas entre recursos](../log-query/cross-workspace-query.md), además de poder [ver diferentes recursos de Azure Monitor Application Insights](../log-query/unify-app-resource-data.md), no es necesario duplicar los datos de los recursos de Application Insights y enviarlos a Log Analytics. Además, el conector envía un subconjunto de las propiedades de las aplicaciones a Log Analytics, mientras que las consultas entre recursos permiten una mayor flexibilidad.  

Como tal, Application Insights Connector ha quedado en desuso y se ha quitado de Azure Marketplace, y lo mismo sucederá con el portal de OMS el 30 de marzo de 2019. Las conexiones existentes continuarán funcionando hasta el 30 de junio de 2019. Con el desuso del portal de OMS, no hay manera de configurar y quitar las conexiones existentes del portal. Para ello, se necesitará la API REST que estará disponible en enero de 2019, y se publicará una notificación en las [actualizaciones de Azure](https://azure.microsoft.com/updates/). 

## <a name="azure-network-security-group-analytics"></a>Azure Network Security Group Analytics
La [solución de Azure Network Security Group Analytics](../insights/azure-networking-analytics.md#azure-network-security-group-analytics-solution-in-azure-monitor) se reemplazará con la solución [Análisis de tráfico](https://azure.microsoft.com/blog/traffic-analytics-in-preview/) lanzada recientemente, que ofrece visibilidad de la actividad de usuario y de la aplicación en redes en la nube. Análisis de tráfico le ayuda a auditar la actividad de red de la organización, a proteger las aplicaciones y los datos, a optimizar el rendimiento de la carga de trabajo y a satisfacer los requisitos de cumplimiento. 

Esta solución analiza los registros de flujo de NSG y ofrece información sobre lo siguiente.

- Flujo de tráfico a través de las redes entre regiones de la nube pública, redes virtuales y subredes de Azure e Internet.
- Aplicaciones y protocolos de la red, sin necesidad de rastreadores ni aparatos de recopilación de flujo dedicados.
- Top Talkers, aplicaciones comunicativas, conversaciones de máquinas virtuales en la nube, zonas activas de tráfico.
- Orígenes y destinos del tráfico entre redes virtuales, interrelaciones entre servicios y aplicaciones críticos para el negocio.
- Seguridad, que incluye tráfico malintencionado, puertos abiertos a Internet, aplicaciones o máquinas virtuales que intentan tener acceso a Internet.
- Utilización de la capacidad, que ayuda a eliminar problemas de sobreaprovisionamiento o infrautilización.

Aún puede contar con la configuración de diagnóstico para enviar los registros de NSG a Log Analytics para que las búsquedas, las alertas y los paneles de información guardados continúen funcionando. Los clientes que ya tengan instalada la solución pueden seguir utilizándola hasta nuevo aviso. A partir del 5 de septiembre, la solución Network Security Group Analytics se quitará de Marketplace y estará disponible en la comunidad como [plantilla de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Operationalinsights).

## <a name="system-center-operations-manager"></a>System Center Operations Manager
Si ha [conectado el grupo de administración de Operations Manager a Log Analytics](om-agents.md), seguirá funcionando sin cambios. Aunque para las nuevas conexiones, debe seguir las instrucciones de [Microsoft System Center Operations Manager Management Pack to configure Operations Management Suite](https://techcommunity.microsoft.com/t5/system-center-blog/bg-p/SystemCenterBlog) (Módulo de administración de Microsoft System Center Operations Manager para configurar Operations Management Suite).

## <a name="next-steps"></a>Pasos siguientes
- Vea [Common questions for transition from OMS portal to Azure portal for Log Analytics users](../overview.md) (Preguntas frecuentes sobre la transición del portal de OMS a Azure Portal para usuarios de Log Analytics) a fin de obtener instrucciones sobre cómo trasladarse del portal de OMS a Azure Portal.
