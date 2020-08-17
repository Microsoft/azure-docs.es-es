---
title: Creación de implementaciones de actualizaciones para Update Management en Azure Automation
description: En este artículo se describe cómo programar implementaciones de actualizaciones y revisar su estado.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 2a81376b284e0d1df84a69b969335c0e63999a00
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449807"
---
# <a name="how-to-deploy-updates-and-review-results"></a>Implementación de actualizaciones y revisión de los resultados

En este artículo se describe cómo programar una implementación de actualizaciones y revisar el proceso una vez completada la implementación.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [Portal de Azure](https://portal.azure.com)

## <a name="schedule-an-update-deployment"></a>Programación de una implementación de actualizaciones

Cuando se programa una implementación de actualizaciones, se crea un recurso de [programación](../shared-resources/schedules.md) vinculado al runbook **Patch-MicrosoftOMSComputers** que controla la implementación de actualizaciones en las máquinas de destino. Tiene que programar una implementación que se ajuste a la programación y a la ventana de servicio de su versión para instalar las actualizaciones. Puede elegir los tipos de actualizaciones que desea incluir en la implementación. Por ejemplo, puede incluir actualizaciones de seguridad o críticas y excluir paquetes acumulativos de actualizaciones.

>[!NOTE]
>Si elimina el recurso de programación desde Azure Portal o mediante PowerShell después de crear la implementación, esta operación de eliminación interrumpe la implementación de actualizaciones programada y, cuando se intenta volver a configurar el recurso de programación desde el portal, aparece un error. Solo se puede eliminar el recurso de programación si se elimina la programación de implementaciones correspondiente.  

Para programar una implementación de actualizaciones:

1. En la cuenta de Automation, vaya a **Update Management** y, a continuación,**** seleccione **Programar implementación de actualizaciones**.

2. En **Nueva implementación de la actualización**, utilice el campo **Nombre** para escribir un nombre único para la implementación.

3. Seleccione el sistema operativo para la implementación de actualizaciones.

4. En el área **Grupos que se deben actualizar (versión preliminar)** , defina una consulta que combine una suscripción, grupos de recursos, ubicaciones y etiquetas con el fin de crear un grupo dinámico de máquinas virtuales de Azure e incluirlo en su implementación. Para más información, consulte [Uso de grupos dinámicos con Update Management](update-mgmt-groups.md).

5. En el área **Máquinas para actualizar**, seleccione una búsqueda guardada, un grupo importado, o bien la opción **Máquinas** del menú desplegable, y elija máquinas específicas. Mediante esta opción, puede ver el grado de preparación del agente de Log Analytics para cada máquina. Para más información sobre los distintos métodos de creación de grupos de equipos en los registros de Azure Monitor, consulte el artículo sobre los [grupos de equipos en los registros de Azure Monitor](../../azure-monitor/platform/computer-groups.md).

6. Use el área **Clasificaciones de actualizaciones** para especificar [clasificaciones de actualizaciones](update-mgmt-view-update-assessments.md#work-with-update-classifications) para productos. En cada producto, anule la selección de todas las clasificaciones de actualizaciones compatibles, salvo las que se incluirán en la implementación de actualizaciones.

7. Use el área **Incluir o excluir actualizaciones** para seleccionar actualizaciones específicas con fines de implementación. En la página Incluir o excluir, se muestran las actualizaciones con arreglo a los números de identificador de artículo de KB que se van a incluir o excluir.
    
   > [!IMPORTANT]
   > Es importante recordar que las exclusiones tienen prioridad sobre las inclusiones. Por ejemplo, si define una regla de exclusión con `*`, Update Management excluirá todas las revisiones o los paquetes de la instalación. Las revisiones excluidas aparecen todavía como que faltan en la máquina. En el caso de las máquinas Linux, si incluye un paquete que tiene un paquete dependiente excluido, Update Management no instalará el paquete principal.

   > [!NOTE]
   > No se pueden especificar actualizaciones reemplazadas para incluirlas en la implementación de actualizaciones.

8. Seleccione **Configuración de programación**. La hora de inicio predeterminada es 30 minutos después de la hora actual. Puede establecer la hora de inicio en cualquier momento a partir de 10 minutos en el futuro.

9. Use el campo **Periodicidad** para especificar si la implementación se producirá una vez o periódicamente y luego seleccione **Aceptar**.

10. En el área **Scripts previos + scripts posteriores (versión preliminar)** , seleccione los scripts que se van a ejecutar antes y después de la implementación. Para más información, consulte [Administración de scripts previos y posteriores](update-mgmt-pre-post-scripts.md).
    
11. Use el campo **Ventana de mantenimiento (minutos)** con el fin de indicar la cantidad de tiempo permitido para la instalación de actualizaciones. Tenga en cuenta los siguientes detalles al especificar una ventana de mantenimiento:

    * Las ventanas de mantenimiento controlan el número de actualizaciones que se instalan.
    * Update Management no detiene la instalación de nuevas actualizaciones si se está aproximando el final de una ventana de mantenimiento.
    * Update Management no finaliza las actualizaciones en curso si se supera la ventana de mantenimiento.
    * Normalmente, si se supera la ventana de mantenimiento de Windows, se debe a que una actualización de Service Pack tarda mucho tiempo en instalarse.

    > [!NOTE]
    > Para evitar que las actualizaciones se apliquen en Ubuntu fuera de la ventana de mantenimiento, vuelva a configurar el paquete `Unattended-Upgrade` para deshabilitar las actualizaciones automáticas. Para más información sobre cómo configurar el paquete, consulte el tema sobre [actualizaciones automáticas en la guía de Ubuntu Server](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).

12. En el campo **Opciones de reinicio**, especifique cómo abordar los reinicios durante la implementación. Están disponibles las siguientes opciones: 
    * Reboot if necessary (default) [Reiniciar si es necesario (valor predeterminado)]
    * Always reboot (Reiniciar siempre)
    * Never reboot (No reiniciar nunca)
    * Solo reiniciar (no se instalarán actualizaciones)

    > [!NOTE]
    > Las claves del Registro que se muestran en la sección [Claves del Registro usadas para administrar reinicios](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) pueden provocar un evento de reinicio si el valor de **Opciones de reinicio** es **No reiniciar nunca**.

13. Cuando haya terminado de configurar la implementación, seleccione **Crear**.

    ![Panel de configuración de la programación de actualizaciones](./media/update-mgmt-deploy-updates/manageupdates-schedule-win.png)

14. Volverá al panel de estado. Seleccione **Implementaciones de actualizaciones programadas** para mostrar la programación de implementación que ha creado.

## <a name="schedule-an-update-deployment-programmatically"></a>Programación de una implementación de actualizaciones mediante programación

Para aprender a crear una implementación de actualizaciones con la API REST, consulte [Configuraciones de actualizaciones de software: crear](/rest/api/automation/softwareupdateconfigurations/create).

También puede usar un runbook de ejemplo para crear una implementación semanal de actualizaciones. Para más información acerca de este runbook, consulte [Create a weekly update deployment for one or more VMs in a resource group](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1) (Crear una implementación de actualización semanal para una o más VM en un grupo de recursos).

## <a name="check-deployment-status"></a>Comprobar el estado de implementación

Una vez que se ha iniciado la implementación programada, puede ver el estado en la pestaña **Implementaciones de actualizaciones** en **Update Management**. Si la implementación se está ejecutando, su estado es **En curso**. Cuando la implementación finaliza correctamente, el estado cambia a **Correcto**. Si se producen errores con una o varias actualizaciones en la implementación, el estado es **Error parcial**.

## <a name="view-results-of-a-completed-update-deployment"></a>Visualización de los resultados de una implementación de actualizaciones que se ha completado

Una vez finalizada la implementación, puede seleccionarla para consultar los resultados.

[ ![Panel de estado de implementación de actualizaciones para una implementación específica](./media/update-mgmt-deploy-updates/manageupdates-view-results.png)](./media/update-mgmt-deploy-updates/manageupdates-view-results-expanded.png#lightbox)

**Resultados de actualización** ofrece un resumen del número total de actualizaciones y de los resultados de la implementación en las máquinas virtuales de destino. En la tabla de la derecha, se muestra un análisis detallado de las actualizaciones y de los resultados individuales de la instalación.

Los valores disponibles son:

* **No intentado**: la actualización no se instaló porque no se disponía de tiempo suficiente, de acuerdo con la duración especificada para la ventana de mantenimiento.
* **No seleccionado**: la actualización no se seleccionó para su implementación.
* **Correcta**: la actualización se realizó correctamente.
* **Error**: la actualización generó un error.

Seleccione **Todos los registros** para ver todas las entradas de registro creadas por la implementación.

Seleccione **Salida** para ver el flujo de trabajo del runbook que administra la implementación de actualizaciones en las máquinas virtuales de destino.

Seleccione **Errores** para ver información detallada acerca de los errores de la implementación.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo crear alertas que le notifiquen los resultados de la implementación de actualizaciones, consulte [Creación de alertas para Update Management](update-mgmt-configure-alerts.md).