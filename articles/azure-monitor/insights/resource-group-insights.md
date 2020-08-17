---
title: Información del grupo de recursos de Azure Monitor | Microsoft Docs
description: Comprenda el estado y el rendimiento de sus aplicaciones y servicios distribuidos en el nivel Grupo de recursos con Azure Monitor
ms.subservice: ''
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 09/19/2018
ms.reviewer: mbullwin
ms.openlocfilehash: bf8ea45f37e421d70a68c4a6e138f5203e92d78f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045767"
---
# <a name="monitor-resource-groups-with-azure-monitor-preview"></a>Supervisar los grupos de recursos con Azure Monitor (versión preliminar)

Las aplicaciones modernas son a menudo complejas y están altamente distribuidas, ya que cuentan con muchas partes discretas trabajando juntas para ofrecer un servicio dado. Gracias a esta complejidad, Azure Monitor proporciona información detallada de supervisión para los grupos de recursos. Esto hace que sea más fácil clasificar y diagnosticar cualquier problema que encuentren sus recursos individuales, a la vez que se ofrece un contexto en cuanto al estado y el rendimiento del grupo de recursos &mdash;y su aplicación&mdash; como un todo.

## <a name="access-insights-for-resource-groups"></a>Obtenga acceso a la información detallada de los grupos de recursos

1. Seleccione **Grupos de recursos** en la barra de navegación de la izquierda.
2. Elija el grupo de recursos que quiera explorar. (Si tiene un gran número de grupos de recursos, la opción de filtrado en función de la suscripción a veces puede ser útil).
3. Para obtener acceso a la información detallada de un grupo de recursos, haga clic en **Insights** (Detalles) en el menú del lado izquierdo de cualquier grupo de recursos.

![Captura de pantalla de la página de información general del grupo de recursos](./media/resource-group-insights/0001-overview.png)

## <a name="resources-with-active-alerts-and-health-issues"></a>Recursos con alertas activas y problemas de estado

La página de información general muestra cuántas alertas se han desencadenado y siguen estando activas, junto con la información de Azure Resource Health de cada recurso. Gracias a esta información puede detectar rápidamente cualquier recurso que tenga problemas. Las alertas le ayudarán a detectar problemas en el código y le indicarán cómo se ha configurado la infraestructura. Azure Resource Health indica un problema con la plataforma de Azure, que no es específico de las aplicaciones individuales.

![Captura de pantalla del panel de Azure Resource Health](./media/resource-group-insights/0002-overview.png)

### <a name="azure-resource-health"></a>Azure Resource Health

Para mostrar Azure Resource Health seleccione el cuadro **Mostrar Azure Resource Health** de la parte superior de la tabla. Esta columna está oculta de forma predeterminada para que la página se cargue rápidamente.

![Captura de pantalla con el gráfico de Resource Health agregado](./media/resource-group-insights/0003-overview.png)

De forma predeterminada, los recursos se agrupan por capa de aplicaciones y tipo de recurso. La **capa de aplicaciones** es una categorización simple de los tipos de recursos, que solo existe en el contexto de la página de información general de los grupos de recursos. Existen tipos de recursos relacionados con el código de la aplicación, la infraestructura informática, las redes, el almacenamiento y las bases de datos. Las herramientas de administración obtienen sus propias capas de aplicaciones, y el resto de recursos se clasifica como perteneciente a la capa de aplicaciones denominada **Otros**. Esta agrupación puede ayudarle a saber de un solo vistazo qué subsistemas de su aplicación son correctos y cuáles tienen problemas.

## <a name="diagnose-issues-in-your-resource-group"></a>Diagnosticar problemas en su grupo de recursos

La página de información del grupo de recursos proporciona otras herramientas con el objetivo de ayudarle a diagnosticar problemas.

   |         |          |
   | ---------------- |:-----|
   | [**Alertas**](../platform/alerts-overview.md)      |  Ver, crear y administrar las alertas. |
   | [**Métricas**](../platform/data-platform.md) | Visualizar y explorar los datos en función de las métricas.    |
   | [**Registros de actividad**](../platform/platform-logs-overview.md) | Eventos de nivel de suscripción que se han producido en Azure.  |
   | [**Mapa de aplicación**](../app/app-map.md) | Vaya a la topología de la aplicación distribuida para identificar los cuellos de botella en el rendimiento o las zonas activas de un error. |

## <a name="failures-and-performance"></a>Errores y rendimiento

¿Qué ocurre si ha observado que la aplicación funciona con lentitud o si los usuarios notifican errores? Lleva mucho tiempo analizar todos los recursos para aislar los problemas.

Las pestañas **Rendimiento** y **Errores** simplifican este proceso al reunir las vistas de diagnóstico de rendimiento y errores de muchos tipos de recursos comunes.

La mayoría de los tipos de recursos abrirán una galería de plantillas del libro de Azure Monitor. Cada libro que cree se puede personalizar, guardar, compartir con el equipo y reutilizar en el futuro, para diagnosticar problemas similares.

### <a name="investigate-failures"></a>Investigar los errores

Para probar la pestaña Errores, seleccione **Errores** en **Investigar**, en el menú de la izquierda.

La barra de menú del lado izquierdo cambia después de que realice su selección, y le ofrece nuevas opciones.

![Captura de pantalla del panel de descripción de errores](./media/resource-group-insights/00004-failures.png)

Cuando elige el servicio de aplicación, se muestra una galería de plantillas del libro de Azure Monitor.

![Captura de pantalla de la galería de libros de aplicaciones](./media/resource-group-insights/0005-failure-insights-workbook.png)

Al elegir la plantilla Detalles de errores, se abrirá el libro.

![Captura de pantalla del informe de errores](./media/resource-group-insights/0006-failure-visual.png)

Puede seleccionar cualquiera de las filas. A continuación, se muestra la selección en una vista gráfica de detalles.

![Captura de pantalla de los detalles del error](./media/resource-group-insights/0007-failure-details.png)

Los libros evitan la dificultad que supone crear informes personalizados y visualizaciones, y ofrecen un formato fácil de usar. Como es posible que algunos usuarios quieran ajustar los parámetros precompilados, los libros son completamente personalizables.

Para tener una idea de cómo funciona este libro de forma interna, seleccione **Editar** en la barra superior.

![Captura de pantalla de la opción de edición adicional](./media/resource-group-insights/0008-failure-edit.png)

Aparecen varios cuadros **Editar** cerca de los diversos elementos del libro. Seleccione el cuadro **Editar** debajo de la tabla de operaciones.

![Captura de pantalla de los cuadros de edición](./media/resource-group-insights/0009-failure-edit-graph.png)

Esto revela la consulta de registro subyacente que está administrando la visualización de la tabla.

 ![Captura de pantalla de la ventana de consulta de registro](./media/resource-group-insights/0010-failure-edit-query.png)

Puede modificar la consulta directamente. O puede usarla como referencia y tomarla prestado cuando diseñe su propio libro de parámetros personalizado.

### <a name="investigate-performance"></a>Investigar el rendimiento

El rendimiento le ofrece su propia galería de libros. En cuanto a App Service, el libro Rendimiento de la aplicación le ofrece la siguiente vista:

 ![Captura de pantalla de la vista de rendimiento](./media/resource-group-insights/0011-performance.png)

En este caso, si selecciona Edición verá que este conjunto de visualizaciones funciona con las métricas de Azure Monitor.

 ![Captura de pantalla de la vista de rendimiento con Azure Metrics](./media/resource-group-insights/0012-performance-metrics.png)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="enabling-access-to-alerts"></a>Habilitación del acceso a las alertas

Para ver las alertas en Azure Monitor para grupos de recursos, un usuario con un rol de propietario o colaborador en esta suscripción debe abrir Azure Monitor para grupos de recursos para cualquier grupo de recursos de la suscripción. Esto permitirá a cualquier usuario con acceso de lectura ver alertas en Azure Monitor para grupos de recursos en todos los grupos de recursos de la suscripción. Si tiene un rol de propietario o colaborador, dedique unos minutos a actualizar esta página.

Azure Monitor para grupos de recursos se basa en el sistema de administración de alertas de Azure Monitor para recuperar el estado de alerta. La administración de alertas no se configura de forma predeterminada para todos los grupos de recursos y las suscripciones, y solo puede habilitarla un usuario con un rol de propietario o colaborador. Se puede habilitar de dos maneras:
* Abriendo Azure Monitor para grupos de recursos con cualquier grupo de recursos de la suscripción.
* O bien, yendo a la suscripción, haciendo clic en **Proveedores de recursos** y luego en **Register for Alerts.Management** (Registrarse con Alerts.Management).

## <a name="next-steps"></a>Pasos siguientes

- [Libros de Azure Monitor](../platform/workbooks-overview.md)
- [Azure Resource Health](../../service-health/resource-health-overview.md)
- [Alertas de Azure Monitor](../platform/alerts-overview.md)
