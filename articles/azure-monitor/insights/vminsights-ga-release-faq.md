---
title: Preguntas frecuentes sobre Azure Monitor para VM (disponible de forma general) | Microsoft Docs
description: Azure Monitor para máquinas virtuales es una solución de Azure que combina la supervisión de rendimiento y de mantenimiento del sistema operativo de la VM de Azure, así como la detección automática de las dependencias y los componentes de aplicación con otros recursos, y asigna la comunicación entre ellos. En este artículo se responde a preguntas habituales sobre la versión disponible de forma general.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/31/2020
ms.openlocfilehash: be192a5807c40ea65ea6533ec6244183e5a4b644
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958803"
---
# <a name="azure-monitor-for-vms-generally-available-ga-frequently-asked-questions"></a>Preguntas frecuentes sobre Azure Monitor para VM disponible de forma general (GA): preguntas más frecuentes
En estas Preguntas más frecuentes acerca de la disponibilidad general se describen los cambios realizados en el cuarto trimestre de 2019 y el primer trimestre de 2020, a medida que los preparábamos para disponibilidad general.

## <a name="updates-for-azure-monitor-for-vms"></a>Actualizaciones de Azure Monitor para VM
Hemos publicado una versión nueva de Azure Monitor para VM en enero de 2020 antes de nuestro anuncio sobre la disponibilidad general. Los clientes que habiliten Azure Monitor para VM recibirán la nueva versión de disponibilidad general, mientras que a los clientes existentes que usan Azure Monitor para VM desde el cuarto trimestre de 2019 y versiones anteriores se les solicitará que realicen la actualización. En estas preguntas frecuentes se ofrece una guía para realizar una actualización a gran escala si tiene implementaciones de gran tamaño en varias áreas de trabajo.


Con esta actualización, los datos de rendimiento de Azure Monitor para VM se almacenan en la misma tabla *InsightsMetrics* que [Azure Monitor para contenedores](container-insights-overview.md), lo que facilita la consulta de los dos conjuntos de datos. Además, puede almacenar conjuntos de datos más diversos que no podíamos almacenar en la tabla utilizada anteriormente. 

Ahora nuestras vistas de rendimiento usan los datos que almacenamos en la tabla *InsightsMetrics*.  Si aún no ha llevado a cabo la actualización para usar la solución de VMInsights más reciente en el área de trabajo, los gráficos ya no mostrarán información.  Puede actualizar desde nuestra página **Get Started** como se describe a continuación.


## <a name="what-is-changing"></a>¿Qué está cambiando?
Se ha lanzado una nueva solución, llamada VMInsights, que incluye funcionalidades adicionales para la recopilación de datos, junto con una nueva ubicación para almacenar estos datos en el área de trabajo de Log Analytics. 

En el pasado, se habilitó la solución ServiceMap en el área de trabajo y se configuraron contadores de rendimiento en el área de trabajo de Log Analytics para enviar los datos a la tabla *Perf*. Esta nueva solución envía los datos a una tabla llamada *InsightsMetrics*, que también se usa en Azure Monitor para contenedores. El esquema de esta tabla nos permite almacenar métricas adicionales y conjuntos de datos de servicio que no son compatibles con el formato de la tabla *Perf*.

Se han actualizado los gráficos de rendimiento para usar los datos que almacenamos en la tabla *InsightsMetrics*. Puede actualizar para usar la tabla *InsightsMetrics* desde nuestra página **Comenzar** como se describe a continuación.


## <a name="how-do-i-upgrade"></a>¿Cómo realizar una actualización?
Cuando un área de trabajo de Log Analytics se actualiza a la versión más reciente de Azure Monitor a las máquinas virtuales, actualizará el agente de dependencias en cada una de las máquinas virtuales asociadas al área de trabajo. Cada máquina virtual que necesite actualización se identificará en la pestaña **Introducción** de Azure Monitor para VM en Azure Portal. Cuando elige actualizar una máquina virtual, se actualiza el área de trabajo de esa máquina virtual junto con cualquier otra máquina virtual conectada al área de trabajo. Puede seleccionar una o varias máquinas virtuales, grupos de recursos o suscripciones. 

Use el siguiente comando para actualizar un área de trabajo mediante PowerShell:

```PowerShell
Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resource-group-name> -WorkspaceName <workspace-name> -IntelligencePackName "VMInsights" -Enabled $True
```

## <a name="what-should-i-do-about-the-performance-counters-in-my-workspace-if-i-install-the-vminsights-solution"></a>¿Qué debo hacer con los contadores de rendimiento de mi área de trabajo si instalo la solución VMInsights?

El método anterior de habilitación de Azure Monitor para VM usaba contadores de rendimiento en el área de trabajo. La versión actual almacena estos datos en una tabla denominada `InsightsMetrics`. Puede elegir deshabilitar estos contadores de rendimiento en el área de trabajo si ya no necesita usarlos. 

>[!NOTE]
>Si tiene reglas de alertas que hagan referencia a estos contadores en la tabla `Perf`, deberá actualizarlas para que hagan referencia a los nuevos datos almacenados en la tabla `InsightsMetrics`. Consulte nuestra documentación para obtener ejemplos de consultas de registro que puede usar y que hacen referencia a esta tabla.
>

Si decide mantener los contadores de rendimiento habilitados, se le facturarán los datos ingeridos y conservados en la tabla `Perf` según los [precios de Log Analytics[(https://azure.microsoft.com/pricing/details/monitor/).

## <a name="how-will-this-change-affect-my-alert-rules"></a>¿Cómo afectará este cambio a las reglas de alerta?

Si ha creado [alertas del registro](../platform/alerts-unified-log.md) que consultan los contadores de rendimiento dirigidos a la tabla `Perf` que se han habilitado en el área de trabajo, debe actualizar estas reglas para que hagan referencia a la tabla `InsightsMetrics` en su lugar. Esta guía también se aplica a las reglas de búsqueda de registros mediante `ServiceMapComputer_CL` y `ServiceMapProcess_CL`, ya que esos conjuntos de datos se mueven a las tablas `VMComputer` y `VMProcess`.

Actualizaremos tanto este documento con preguntas frecuentes como nuestra documentación para incluir reglas de alertas de búsqueda de registros de ejemplo para los conjuntos de datos que recopilamos.

## <a name="how-will-this-affect-my-bill"></a>¿Cómo afectará esto a la factura?

La facturación todavía se basa en los datos ingeridos y retenidos en el área de trabajo de Log Analytics.

Los datos de rendimiento de nivel de máquina que se recopilan son los mismos, tienen un tamaño similar a los datos almacenados en la tabla `Perf` y costarán aproximadamente lo mismo.

## <a name="what-if-i-only-want-to-use-service-map"></a>¿Qué pasa si solo deseo usar Service Map?

No hay problema. Verá en Azure Portal mensajes acerca de la próxima actualización cuando vea Azure Monitor para VM. Una vez que se lance, recibirá un mensaje en el que se le solicitará que actualice a la nueva versión. Si prefiere usar solo la característica [Maps](vminsights-maps.md), puede optar por no realizar la actualización y seguir usando la característica Maps en Azure Monitor para VM y la solución Service Map a la que se accede desde el área de trabajo o desde el icono del panel.

Si elige habilitar manualmente los contadores de rendimiento en el área de trabajo, es posible que pueda ver los datos en algunos de los gráficos de rendimiento que se ven en Azure Monitor. Una vez que se lance la nueva solución, actualizaremos los gráficos de rendimiento para consultar los datos almacenados en la tabla `InsightsMetrics`. Si desea ver los datos de esa tabla en estos gráficos, tendrá que realizar la actualización a la nueva versión de Azure Monitor para VM.

Los cambios para mover datos desde `ServiceMapComputer_CL` y `ServiceMapProcess_CL` afectarán tanto a Service Map como a Azure Monitor para VM, por lo que debe planear esta actualización.

Aunque decida no realizar la actualización a la solución **VMInsights**, seguiremos proporcionando versiones heredadas de los libros de rendimiento que hacen referencia a los datos de la tabla `Perf`.  

## <a name="will-the-service-map-data-sets-also-be-stored-in-insightsmetrics"></a>¿Se almacenarán también en InsightsMetrics los conjuntos de datos de Service Map?

Los conjuntos de datos no se duplicarán si usa ambas soluciones. Ambas ofertas comparten los conjuntos de datos que se almacenarán en las tablas `VMComputer` (anteriormente ServiceMapComputer_CL), `VMProcess` (anteriormente ServiceMapProcess_CL), `VMConnection` y `VMBoundPort` para almacenar los conjuntos de datos del mapa que recopilemos.  

La tabla `InsightsMetrics` almacenará los conjuntos de datos de máquinas virtuales, procesos y servicios que recopilemos y solo se rellenará si se usa Azure Monitor para VM y la solución VM Insights. La solución Service Map no recopilará ni almacenará datos en la tabla `InsightsMetrics`.

## <a name="will-i-be-double-charged-if-i-have-the-service-map-and-vminsights-solutions-in-my-workspace"></a>¿Se me cobrará el doble si tengo las soluciones Service Map y VMInsights en mi área de trabajo?

No, las dos soluciones comparten los conjuntos de datos del mapa que almacenamos en `VMComputer` (anteriormente ServiceMapComputer_CL), `VMProcess` (anteriormente ServiceMapProcess_CL), `VMConnection` y `VMBoundPort`. Si tiene ambas soluciones en el área de trabajo, no se le cobrará el doble por estos datos.

## <a name="if-i-remove-either-the-service-map-or-vminsights-solution-will-it-remove-my-data"></a>Si elimino cualquiera de las dos soluciones, Service Map o VMInsights, ¿se eliminarán mis datos?

No, las dos soluciones comparten los conjuntos de datos del mapa que almacenamos en `VMComputer` (anteriormente ServiceMapComputer_CL), `VMProcess` (anteriormente ServiceMapProcess_CL), `VMConnection` y `VMBoundPort`. Si elimina una de las soluciones, estos conjuntos de datos observan que aún hay una solución que utiliza los datos y que permanece en el área de trabajo de Log Analytics. Para que los datos se eliminen del área de trabajo, es preciso eliminar las dos soluciones del área de trabajo.

## <a name="health-feature-is-in-limited-public-preview"></a>La característica Health está en versión preliminar pública limitada.

Hemos recibido muchos comentarios de los clientes sobre nuestro conjunto de características Health de la máquina virtual. Existe gran interés en torno a esta característica y entusiasmo por su potencial para admitir la supervisión de flujos de trabajo. Estamos planeando realizar varios cambios para agregar funcionalidad y dar respuesta a los comentarios que hemos recibido. 

Para minimizar el impacto de estos cambios en los clientes nuevos, hemos trasladado esta característica a una **versión preliminar pública limitada**. Esta actualización se produjo en octubre de 2019.

Tenemos previsto volver a lanzar la característica Health en 2020, después de que Azure Monitor para VM se encuentre en disponibilidad general.

## <a name="how-do-existing-customers-access-the-health-feature"></a>¿Cómo acceden los clientes existentes a la característica Health?

Los clientes existentes que usen la característica Health seguirán teniendo acceso a ella, pero no se ofrecerán a los nuevos clientes.  

Para acceder a la característica, puede agregar la siguiente marca de la característica `feature.vmhealth=true` a la dirección URL de Azure Portal, [https://portal.azure.com](https://portal.azure.com). Ejemplo: `https://portal.azure.com/?feature.vmhealth=true`.

También puede usar esta dirección URL corta, que establece la marca de la característica automáticamente: [https://aka.ms/vmhealthpreview](https://aka.ms/vmhealthpreview).

Como cliente existente, puede seguir usando la característica Health en las máquinas virtuales conectadas a una configuración de área de trabajo existente con la funcionalidad de estado.  

## <a name="i-use-vm-health-now-with-one-environment-and-would-like-to-deploy-it-to-a-new-one"></a>Ahora utilizo la característica Health de la máquina virtual con un entorno y desearía implementarla en uno nuevo.

Los clientes existentes que usan la característica Health y desean usarla para una nueva puesta en servicio, póngase en contacto con nosotros en vminsights@microsoft.com para solicitar instrucciones.

## <a name="next-steps"></a>Pasos siguientes

Para conocer los requisitos y los métodos para habilitar la supervisión de máquinas virtuales, consulte [Implementación de Azure Monitor para VM](vminsights-enable-overview.md).
