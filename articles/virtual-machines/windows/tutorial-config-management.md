---
title: 'Tutorial: Administración de la configuración de una máquina virtual Windows en Azure'
description: En este tutorial aprenderá a identificar los cambios y a administrar las actualizaciones de paquetes en una máquina virtual Windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/05/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 2b88c053e1f9029f916d9a180221a3bb58629608
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448860"
---
# <a name="tutorial-monitor-changes-and-update-a-windows-virtual-machine-in-azure"></a>Tutorial: Supervisión de cambios y actualización de una máquina virtual Windows en Azure

Con Azure [Change Tracking](../../automation/change-tracking.md) y [Update Management](../../automation/update-management/update-mgmt-overview.md), es fácil identificar los cambios realizados en las máquinas virtuales Windows en Azure y administrar las actualizaciones del sistema operativo de esas mismas máquinas virtuales.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Administrar actualizaciones de Windows.
> * Supervisar los cambios y el inventario.

## <a name="open-azure-cloud-shell"></a>Apertura de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en su cuenta de Azure.

Para abrir cualquier bloque de código en Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha del bloque.

También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Seleccione **Copiar** para copiar los bloques de código, péguelos en la pestaña Cloud Shell y seleccione la tecla ENTRAR para ejecutar el código.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Para configurar la supervisión y la administración de actualizaciones de Azure en este tutorial, necesita una máquina virtual Windows en Azure.

En primer lugar, establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

A continuación, cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). El siguiente ejemplo crea una máquina virtual llamada `myVM` en la ubicación `East US`. Si aún no existen, se crea el grupo de recursos `myResourceGroupMonitor` y los recursos de red de soporte:

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "East US" `
    -Credential $cred
```

Los recursos y la máquina virtual tardan unos minutos en crearse.

## <a name="manage-windows-updates"></a>Administrar actualizaciones de Windows

Update Management le ayuda a administrar las actualizaciones y revisiones de las máquinas virtuales Windows de Azure. Directamente desde la máquina virtual, puede:

- Evaluar el estado de las actualizaciones disponibles.
- Programar la instalación de las actualizaciones necesarias.
- Examinar los resultados de la implementación para comprobar que las actualizaciones se han aplicado correctamente a la máquina virtual.

Para más información acerca de los precios, consulte [Precios de automatización de la administración de actualizaciones](https://azure.microsoft.com/pricing/details/automation/).

### <a name="enable-update-management"></a>Habilitación de la Administración de actualizaciones

Para habilitar Update Management en una máquina virtual:

1. En el lado izquierdo de la ventana, seleccione **Máquinas virtuales**.
1. Elija una de las máquinas virtuales de la lista.
1. En el panel **Operaciones** de la ventana de la máquina virtual, seleccione **Administración de actualizaciones**.
1. Se abre la ventana **Habilitar la administración de actualizaciones**.

Se realiza la validación para determinar si Update Management está habilitado para esta máquina virtual. La validación incluye comprobaciones de un área de trabajo de Log Analytics, para una cuenta de Automation vinculada, y si la solución está en el área de trabajo.

Use un área de trabajo de [Log Analytics](../../azure-monitor/log-query/log-query-overview.md) para recopilar los datos que se generan mediante características y servicios como Update Management. El área de trabajo proporciona una única ubicación para revisar y analizar datos desde varios orígenes.

Para llevar a cabo en las máquinas virtuales acciones adicionales que requieran actualizaciones, puede usar Azure Automation para ejecutar runbooks en máquinas virtuales. Estas acciones incluyen la descarga o la aplicación de actualizaciones.

El proceso de validación también comprueba si la máquina virtual se aprovisiona con Microsoft Monitoring Agent (MMA) y Hybrid Runbook Worker de Azure Automation. Use el agente para comunicarse con la máquina virtual y obtener información sobre el estado de la actualización.

En la ventana **Habilitar la administración de actualizaciones**, elija el área de trabajo de Azure Log Analytics y la cuenta de Automation, y seleccione **Habilitar**. La solución tarda un máximo de 15 minutos en habilitarse.

Los requisitos previos siguientes que falten durante la incorporación se agregarán automáticamente:

* Área de trabajo de [Log Analytics](../../azure-monitor/log-query/log-query-overview.md)
* [Automation](../../automation/index.yml)
* [Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md), que está habilitado en la máquina virtual

Una vez habilitada la solución, se abre la ventana **Administración de actualizaciones**. Configure la ubicación, el área de trabajo de Log Analytics y la cuenta de Automation que va a usar y seleccione **Habilitar**. Si estas opciones aparecen atenuadas, significa que otra solución de automatización está habilitada en la máquina virtual y que deben usarse el área de trabajo de esa solución y la cuenta de Automation.

![Habilitación de la solución Update Management](./media/tutorial-monitoring/manageupdates-update-enable.png)

La solución Update Management puede tardar un máximo de 15 minutos en habilitarse. Durante este tiempo, no cierre la ventana del explorador. Después de habilitar la solución, la información sobre las actualizaciones que faltan en la máquina virtual se pasa a los registros de Azure Monitor. Los datos pueden tardar entre 30 minutos y 6 horas en estar disponibles para el análisis.

### <a name="view-an-update-assessment"></a>Visualización de la evaluación de la actualización

Una vez que Update Management esté habilitado, se abrirá la ventana **Administración de actualizaciones**. Una vez que acabe la evaluación de las actualizaciones, verá una lista de las que falten en la pestaña **Actualizaciones que faltan**.

 ![Ver el estado de la actualización](./media/tutorial-monitoring/manageupdates-view-status-win.png)

### <a name="schedule-an-update-deployment"></a>Programación de una implementación de actualizaciones

Para instalar actualizaciones, programe una implementación que se ajuste a su ventana de programación y servicio de versiones. Puede elegir los tipos de actualizaciones que desea incluir en la implementación. Por ejemplo, puede incluir actualizaciones de seguridad o críticas y excluir paquetes acumulativos de actualizaciones.

Para programar una nueva implementación de actualizaciones en la máquina virtual, seleccione **Programar implementación de actualizaciones**  en la parte superior de la pantalla **Administración de actualizaciones**. En la ventana **Nueva implementación de actualización**, especifique la siguiente información:

| Opción | Descripción |
| --- | --- |
| **Nombre** |proporcione un nombre único para identificar la implementación de actualizaciones. |
|**Sistema operativo**| Seleccione **Linux** o **Windows**.|
| **Grupos que se deben actualizar** |En el caso de las máquinas virtuales hospedadas en Azure, defina una consulta basada en una combinación de suscripción, grupos de recursos, ubicaciones y etiquetas. Esta consulta crea un grupo dinámico de máquinas virtuales hospedadas en Azure que se van a incluir en la implementación. </br></br>En el caso de las máquinas virtuales no hospedadas en Azure, seleccione una búsqueda guardada existente. Con esta búsqueda, puede seleccionar un grupo de estas máquinas virtuales para incluirlas en la implementación. </br></br> Para más información, consulte [Grupos dinámicos](../../automation/update-management/update-mgmt-groups.md).|
| **Máquinas para actualizar** |Seleccione **Búsqueda guardada**, **Grupo importado** o **Máquinas**.<br/><br/>Si selecciona **Máquinas**, puede elegir máquinas individuales en la lista desplegable. La preparación de la máquina se muestra en la columna **PREPARACIÓN DE ACTUALIZACIONES DEL AGENTE** de la tabla.</br></br> Para información sobre los distintos métodos de creación de grupos de equipos en los registros de Azure Monitor, consulte el artículo sobre los [Grupos de equipos en los registros de Azure Monitor](../../azure-monitor/platform/computer-groups.md) |
|**Clasificaciones de actualizaciones**|Elija todas las clasificaciones de actualizaciones necesarias.|
|**Incluir o excluir las actualizaciones**|Seleccione esta opción para abrir el panel **Incluir o excluir**. Las actualizaciones que se van a incluir y las que se van a excluir están en pestañas separadas. Para más información sobre cómo se controla la inclusión, consulte [Programación de una implementación de actualizaciones](../../automation/update-management/update-mgmt-deploy-updates.md#schedule-an-update-deployment). |
|**Configuración de la programación**|Elija la hora de inicio y seleccione **Una vez** o **Periódico**.|
| **Scripts previos + scripts posteriores**|Elija los scripts que se van a ejecutar antes y después de la implementación.|
| **Ventana de mantenimiento** | Escriba el número de minutos establecido para las actualizaciones. Los valores válidos oscilan entre 30 y 360 minutos. |
| **Control de reinicio**| Seleccione cómo se controlan los reinicios. Las opciones disponibles son:<ul><li>**Reiniciar si es necesario**</li><li>**Reiniciar siempre**</li><li>**No reiniciar nunca**</li><li>**Solo reiniciar**</li></ul>**Reiniciar si es necesario** es la opción predeterminada. Si selecciona **Solo reiniciar**, no se instalarán las actualizaciones.|

Después de que haya terminado de configurar la programación, haga clic en **Crear** para volver al panel de estado. En la tabla **Programada** se muestra la programación de implementaciones que ha creado.

También se pueden crear implementaciones de actualizaciones mediante programación. Para aprender a crear una implementación de actualizaciones con la API REST, consulte [Configuraciones de actualizaciones de software: crear](/rest/api/automation/softwareupdateconfigurations/create). También hay un runbook de ejemplo que se puede usar para crear una implementación de actualizaciones semanal. Para más información acerca de este runbook, consulte [Create a weekly update deployment for one or more VMs in a resource group](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1) (Crear una implementación de actualización semanal para una o más VM en un grupo de recursos).

### <a name="view-results-of-an-update-deployment"></a>Visualización de los resultados de una implementación de actualizaciones

Después de que se inicie la implementación programada, puede ver su estado en la pestaña **Implementaciones de actualizaciones** de la ventana **Administración de actualizaciones**.

Si la implementación se está ejecutando, aparece el estado "En curso". Una vez que finaliza correctamente, el estado cambia a "Correcto". Pero si se produce un error en cualquiera de las actualizaciones de la implementación, el estado es "Error parcial".

Seleccione la implementación de actualizaciones completada para ver el panel de esa implementación.

![Panel de estado de Implementación de actualizaciones de la implementación específica](./media/tutorial-monitoring/manageupdates-view-results.png)

El icono **Resultados de actualización** muestra un resumen del número total de actualizaciones y los resultados de la implementación en la máquina virtual. En la tabla de la derecha se muestra un desglose detallado de cada actualización y los resultados de la instalación. Cada resultado tiene uno de los siguientes valores:

* **No intentado**: La actualización no está instalada. No había suficiente tiempo disponible según la duración de la ventana de mantenimiento definida.
* **Correcto**: actualización realizada correctamente.
* **Error**: error en la actualización.

Seleccione **Todos los registros** para ver todas las entradas de registro que creó la implementación.

Seleccione el icono **Salida** para ver el flujo de trabajo del runbook responsable de administrar la implementación de actualizaciones en la máquina virtual de destino.

Seleccione **Errores** para ver información detallada acerca de los errores en la implementación.

## <a name="monitor-changes-and-inventory"></a>Supervisión de cambios y del inventario

Puede recopilar y ver un inventario de software, archivos, demonios de Linux, servicios de Windows y claves del Registro de Windows en los equipos. Realizar un seguimiento de las configuraciones de sus máquinas le ayuda a identificar problemas de funcionamiento en el entorno y comprender mejor el estado de las máquinas.

### <a name="enable-change-and-inventory-management"></a>Habilitación de la administración de cambios y del inventario

Para habilitar la administración de cambios y del inventario para la máquina virtual:

1. En el lado izquierdo de la ventana, seleccione **Máquinas virtuales**.
1. Elija una de las máquinas virtuales de la lista.
1. En **Operaciones**, en la ventana de la máquina virtual, seleccione **Inventario** o **Seguimiento de cambios** .
1. Se abre la página **Habilitar Change Tracking e Inventario**.

Configure la ubicación, el área de trabajo de Log Analytics y la cuenta de Automation que va a usar, y seleccione **Habilitar**. Si las opciones aparecen atenuadas, significa que ya hay una solución de automatización habilitada para la máquina virtual. En ese caso, se debe emplear la cuenta de Automation y el área de trabajo ya utilizados.

Aunque las soluciones aparezcan por separado en el menú, son la misma solución. Al habilitar una, se habilitan ambas para la máquina virtual.

![Habilitación del seguimiento de cambios e inventario](./media/tutorial-monitoring/manage-inventory-enable.png)

Una vez habilitada la solución, el inventario puede tardar un tiempo en recopilarse en la máquina virtual hasta que aparecen los datos.

### <a name="track-changes"></a>Control de cambios

En la máquina virtual, en **OPERACIONES**, seleccione **Change Tracking** y, después, seleccione **Editar configuración**. Se abre el panel **Change Tracking**. Seleccione el tipo de configuración a la que desea realizar un seguimiento y, después, seleccione **+ Agregar** para configurar las opciones.

Las opciones de configuración disponibles para Windows son:

* Registro de Windows
* Archivos de Windows

Para obtener información detallada sobre Change Tracking, consulte [Solución de los problemas de una máquina virtual](../../automation/automation-tutorial-troubleshoot-changes.md).

### <a name="view-inventory"></a>Visualización del inventario

EN la máquina virtual, seleccione **Inventario** en **OPERACIONES**. En la pestaña **Software**, hay una tabla que muestra el software que se ha encontrado. En la tabla aparece información de alto nivel de todos y cada uno de los registros de software. Esta información incluye el nombre del software, su versión, el editor y la última vez que se actualizó.

![Visualización del inventario](./media/tutorial-monitoring/inventory-view-results.png)

### <a name="monitor-activity-logs-and-changes"></a>Supervisión de los registros de actividad y de los cambios

En la ventana **Change Tracking** de la máquina virtual, seleccione **Administrar conexión de registro de actividad** para abrir el panel **Registro de actividad de Azure**. Seleccione **Conectar** para conectar Change Tracking al registro de actividad de Azure de la máquina virtual.

Después de que se habilite Change Tracking, vaya al panel **Información general** de la máquina virtual y seleccione **Detener** para detenerla. Cuando se le solicite, seleccione **Sí** para detener la máquina virtual. Después de que se haya desasignado la máquina virtual, seleccione **Iniciar** para reiniciar la máquina virtual.

Si se detiene y se reinicia una máquina virtual, se registrará un evento en su registro de actividad. Vuelva al panel **Change Tracking** y seleccione la pestaña **Eventos** en la parte inferior del panel. Después de un tiempo, los eventos aparecerán en el gráfico y en la tabla. Puede seleccionar cada uno de los eventos para ver información detallada del mismo.

![Visualización de cambios en el registro de actividad](./media/tutorial-monitoring/manage-activitylog-view-results.png)

El gráfico anterior muestra los cambios que se han producido con el tiempo. Después de agregar una conexión del registro de actividad de Azure, el gráfico de líneas de la parte superior muestra los eventos del registro de actividad de Azure.

Cada una de las filas de los gráficos de barras representa un tipo de cambio diferente al que se puede realizar un seguimiento. Estos tipos son demonios de Linux, archivos, claves del Registro de Windows, software y servicios de Windows. La pestaña **Cambiar** muestra los detalles de los cambios. Los cambios aparecen en el orden en que han tenido lugar. Por tanto, el primero que aparece es el último que se ha producido.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha configurado y examinado Change Tracking y Update Management en una máquina virtual. Ha aprendido a:

> [!div class="checklist"]
> * Crear un grupo de recursos y una máquina virtual.
> * Administrar actualizaciones de Windows.
> * Supervisar los cambios y el inventario.

En el siguiente tutorial se explica cómo supervisar una máquina virtual.

> [!div class="nextstepaction"]
> [Supervisión de máquinas virtuales](tutorial-monitor.md)
