---
title: Creación y uso compartido de paneles de datos de Azure Log Analytics | Microsoft Docs
description: Con este tutorial le resultará más fácil comprender cómo los paneles de Log Analytics pueden mostrar todas las consultas de registros guardadas, lo que le permite ver su entorno en una sola vista.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 05/28/2020
ms.custom: mvc
ms.openlocfilehash: d38a4ba04529e70ecfd772eef7f60179041f0273
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927375"
---
# <a name="create-and-share-dashboards-of-log-analytics-data"></a>Creación y uso compartido de paneles de datos de Log Analytics

Los paneles de Log Analytics permiten visualizar todas las consultas de registro guardadas, lo que le proporciona la posibilidad de buscar, correlacionar y compartir datos operativos de TI en la organización.  Este tutorial trata la creación de una consulta de registro que se usará para admitir un panel compartido al que accederá su equipo de soporte técnico de operaciones de TI.  Aprenderá a:

> [!div class="checklist"]
> * Crear un panel compartido en Azure Portal.
> * Visualizar una consulta de registro de rendimiento. 
> * Agregar una consulta de registro a un panel compartido. 
> * Personalizar un icono en un panel compartido.

Para completar el ejemplo en este tutorial, debe disponer de una máquina virtual existente [conectada al área de trabajo de Log Analytics](quick-collect-azurevm.md).  
 
## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-shared-dashboard"></a>Creación de un panel compartido
Seleccione **Panel** para abrir el [panel](../../azure-portal/azure-portal-dashboards.md) predeterminado. El panel tendrá un aspecto distinto al ejemplo siguiente.

![Panel de Portal de Azure](media/tutorial-logs-dashboards/log-analytics-portal-dashboard.png)

Aquí puede agrupar los datos operacionales que sean más importantes para la TI en todos los recursos de Azure, incluidos los datos de telemetría de Azure Log Analytics.  Antes de que pasemos a la visualización de una consulta de registro, creemos primero un panel y compartámoslo.  Luego, podemos centrarnos en la consulta de registro de rendimiento de ejemplo, que se representará como un gráfico de líneas, y lo agregaremos al panel.  

> [!NOTE]
> Los siguientes tipos de gráficos se admiten en los paneles de Azure mediante consultas de registro:
> - areachart
> - columnchart
> - piechart (se representará en el panel como un anillo)
> - scatterchart
> - timechart

Para crear un panel, haga clic en el botón **Panel nuevo** situado junto al nombre del panel actual.

![Creación de un nuevo panel en Azure Portal](media/tutorial-logs-dashboards/log-analytics-create-dashboard-01.png)

Esta acción crea un panel nuevo, vacío y privado, y activa el modo de personalización, donde puede asignar un nombre al panel y agregar o reorganizar los iconos. Edite el nombre del panel y especifique *Panel de ejemplo* para este tutorial y, después, seleccione **Personalización finalizada**.<br><br> ![Guardado del panel personalizado de Azure](media/tutorial-logs-dashboards/log-analytics-create-dashboard-02.png)

Al crear un panel, será privado de forma predeterminada, lo que significa que usted será la única persona que puede verlo. Para que puedan verlo otros usuarios, utilice el botón **Compartir** que aparece junto al resto de los comandos del panel.

![Uso compartido de un nuevo panel en Azure Portal](media/tutorial-logs-dashboards/log-analytics-share-dashboard.png) 

Se le pedirá que elija una suscripción y un grupo de recursos en el que se vaya a publicar el panel. Para mayor comodidad, la experiencia de publicación del portal lo guiará por un patrón donde colocará paneles en un grupo de recursos denominado " **paneles**".  Compruebe la suscripción seleccionada y, después, haga clic en **Publicar**.  El acceso a la información mostrada en el panel se controla con el [control de acceso basado en rol (Azure RBAC) de Azure](../../role-based-access-control/role-assignments-portal.md).   

## <a name="visualize-a-log-query"></a>Visualización de una consulta de registro
[Log Analytics](../log-query/get-started-portal.md) es un portal dedicado que se usa para trabajar con consultas de registro y sus resultados. Las características incluyen la capacidad para editar una consulta en varias líneas, ejecutar código de forma selectiva, Intellisense sensible al contexto y análisis inteligente. En este tutorial, usará Log Analytics para crear una vista de rendimiento en formato gráfico, la guardará para una consulta futura y la anclará al panel compartido creado anteriormente.

Seleccione **Registros** en el menú Azure Monitor para abrir Log Analytics. Se inicia con una nueva consulta en blanco.

![Página de inicio](media/tutorial-logs-dashboards/homepage.png)

Escriba la siguiente consulta para devolver registros de uso del procesador para los equipos de Windows y Linux, agrupada por Equipo y TimeGenerated, y mostrada en un gráfico visual. Haga clic en **Ejecutar** para ejecutar la consulta y ver el gráfico resultante.

```Kusto
Perf 
| where CounterName == "% Processor Time" and ObjectName == "Processor" and InstanceName == "_Total" 
| summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1hr), Computer 
| render timechart
```

Para guardar la consulta, seleccione el botón **Guardar** en la parte superior de la página.

![Guardar consulta](media/tutorial-logs-dashboards/save-query.png)

En el panel de control **Guardar consulta**, escriba un nombre como *Máquinas virtuales de Azure: uso del procesador* y una categoría como *Paneles* y, luego, haga clic en **Guardar**.  De este modo, puede crear una biblioteca con las consultas comunes que puede usar y modificar.  Por último, ánclela al panel compartido que se creó anteriormente. Para ello, seleccione el botón **Anclar al panel** en la esquina superior derecha de la página y seleccione el nombre del panel.

Ahora que tenemos una consulta anclada al panel, podrá observar que tiene un título genérico y un comentario debajo de él.

![Ejemplo de panel de Azure](media/tutorial-logs-dashboards/log-analytics-modify-dashboard-01.png)

 Debemos cambiarle el nombre a uno descriptivo que permita que las personas que lo vean lo entiendan fácilmente.  Haga clic en el botón Editar para personalizar el título y el subtítulo del icono y, a continuación, haga clic en **Actualizar**.  Aparecerá un banner en el que se le preguntará si desea publicar los cambios o rechazarlos.  Haga clic en **Guardar una copia**.  

![Configuración completada del panel de ejemplo](media/tutorial-logs-dashboards/log-analytics-modify-dashboard-02.png)

## <a name="next-steps"></a>Pasos siguientes
En este tutorial aprendió a crear un panel en Azure Portal y a agregarle una consulta de registro.  Vaya al siguiente tutorial para conocer las diferentes respuestas que puede implementar según los resultados de consulta de registro.  

> [!div class="nextstepaction"]
> [Respuesta a eventos con las alertas de Log Analytics](tutorial-response.md)
