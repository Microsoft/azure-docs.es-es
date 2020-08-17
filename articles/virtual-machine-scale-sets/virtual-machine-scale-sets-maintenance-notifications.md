---
title: Notificaciones de mantenimiento de conjuntos de escalado de máquinas virtuales de Azure
description: Vea las notificaciones de mantenimiento e inicie el mantenimiento de autoservicio de los conjuntos de escalado de máquinas virtuales de Azure.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 08/20/2019
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 5521e49c767a2510bf7c8c53cf6ac5e86b73b466
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837183"
---
# <a name="planned-maintenance-notifications-for-virtual-machine-scale-sets"></a>Notificaciones de mantenimiento planeado de conjuntos de escalado de máquinas virtuales


Azure realiza periódicamente actualizaciones para mejorar la confiabilidad, el rendimiento y la seguridad de la infraestructura host de las máquinas virtuales (VM). Las actualizaciones podrían incluir la aplicación de revisiones en el entorno de hospedaje o la actualización y la retirada de hardware. La mayoría de las actualizaciones no afectan a las máquinas virtuales hospedadas. Sin embargo, sí afectan a las máquinas virtuales en estos escenarios:

- Si el mantenimiento no requiere un reinicio, Azure usa la migración en contexto para pausar la máquina virtual mientras se actualiza el host. Las operaciones de mantenimiento que no requieren un reinicio se aplican por dominio de error. El progreso se detiene si no se reciben señales de estado de advertencia.

- Si el mantenimiento requiere un reinicio, el usuario recibe un aviso de cuándo está programado el mantenimiento. En estos casos, se le asigna también un período de tiempo (normalmente treinta y cinco días) donde puede iniciar el mantenimiento a la hora que le resulte más cómodo.


El mantenimiento planeado que requiere un reinicio se programa en olas. Cada ola tiene un ámbito diferente (regiones):

- Una ola comienza con una notificación a los clientes. De forma predeterminada, la notificación se envía al propietario de la suscripción y a los copropietarios. Con las [alertas del registro de actividad](../azure-monitor/platform/platform-logs-overview.md) de Azure puede agregar a las notificaciones destinatarios y opciones de mensajería, como correo electrónico, SMS y webhooks.  
- Con la notificación, aparece una *ventana de autoservicio*. Durante este período, que suele ser de treinta y cinco días, puede encontrar cuál de las máquinas virtuales se incluye en la ola. Puede iniciar el mantenimiento de forma anticipada según sus propias necesidades de programación.
- Después de la ventana de autoservicio, comienza una *ventana de mantenimiento programado*. Mientras está ventana está activa, Azure programa el mantenimiento necesario y lo aplica a la máquina virtual. 

El objetivo de tener dos ventanas es proporcionar tiempo suficiente para que pueda iniciar el mantenimiento y reiniciar la máquina virtual sabiendo cuándo iniciará Azure el mantenimiento automáticamente.

Puede usar Azure Portal, PowerShell, la API REST y la CLI de Azure para consultar las ventanas de mantenimiento de las máquinas virtuales del conjunto de escalado de máquinas virtuales e iniciar el mantenimiento de autoservicio.

## <a name="should-you-start-maintenance-during-the-self-service-window"></a>¿Debe comenzar el mantenimiento durante la ventana de autoservicio?  

Las directrices siguientes pueden ayudarlo a decidir si iniciar el mantenimiento a una hora que elija.

> [!NOTE] 
> Es posible que el mantenimiento de autoservicio no esté disponible para todas las máquinas virtuales. Para determinar si su máquina virtual se puede volver a implementar de forma proactiva, busque **Iniciar ahora** en el estado del mantenimiento. En este momento, el mantenimiento de autoservicio no está disponible para Azure Cloud Services (rol de trabajo o web) y Azure Service Fabric.


El mantenimiento de autoservicio no se recomienda para implementaciones que usan *conjuntos de disponibilidad*. Los conjuntos de disponibilidad son configuraciones de alta disponibilidad en las que solo un dominio de actualización resulta afectado en cualquier momento. Para los conjuntos de disponibilidad:

- Deje que Azure desencadene el mantenimiento. Para el mantenimiento que requiere un reinicio, el proceso se realiza por dominio de actualización. Los dominios de actualización no reciben necesariamente el mantenimiento de forma secuencial. Hay una pausa de 30 minutos entre dominios de actualización.
- Si una pérdida temporal de una parte de su capacidad (1/recuento de dominios de actualización) supone un problema, se puede compensar fácilmente mediante la asignación de instancias adicionales durante el período de mantenimiento.
- Para el mantenimiento que no requiere el reinicio, las actualizaciones se aplican en el nivel de dominio de error. 
    
**No** utilice el mantenimiento de autoservicio en los escenarios siguientes: 

- Si apaga las máquinas virtuales con frecuencia, ya sea manualmente, mediante DevTest Labs, con el apagado automático o siguiendo una programación. El mantenimiento de autoservicio en estos escenarios podría revertir el estado de mantenimiento y provocar tiempo de inactividad adicional.
- En las máquinas virtuales de corta duración que sepa que se van a eliminar antes del final del mantenimiento. 
- Para cargas de trabajo con un estado grande almacenadas en el disco local (efímero) en el que se desea realizar el mantenimiento tras la actualización. 
- Si con frecuencia cambia de tamaño la máquina virtual. Este escenario puede revertir el estado de mantenimiento. 
- Si ha realizado eventos programados que permiten una conmutación por error proactiva o un apagado ordenado de la carga de trabajo, 15 minutos antes de que comience el apagado de mantenimiento.

**Use** el mantenimiento de autoservicio si tiene planeado ejecutar la máquina virtual de forma ininterrumpida durante la fase de mantenimiento programado y no se aplica ninguna de las contraindicaciones anteriores. 

Se recomienda usar el mantenimiento de autoservicio en los siguientes casos:

- Es preciso comunicar una ventana de mantenimiento exacta al equipo directivo o al cliente. 
- Es preciso completar el mantenimiento para una fecha determinada. 
- Es preciso controlar la secuencia de mantenimiento, por ejemplo, en una aplicación de niveles múltiples, para garantizar la recuperación segura.
- Se necesitan más de 30 minutos de recuperación de la máquina virtual entre dos dominios de actualización. Para controlar el tiempo entre dominios de actualización, debe desencadenar el mantenimiento en las máquinas virtuales en un dominio de actualización a la vez.

 
## <a name="view-virtual-machine-scale-sets-that-are-affected-by-maintenance-in-the-portal"></a>Visualización de los conjuntos de escalado de máquinas virtuales afectados por el mantenimiento en el portal

Cuando se programa una ola de mantenimiento planeado, puede ver la lista de conjuntos de escalado de máquinas virtuales que resultan afectados por la siguiente ola de mantenimiento mediante Azure Portal. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú izquierdo, seleccione **Todos los servicios** y, luego, seleccione **Conjuntos de escalado de máquinas virtuales**.
3. En **Conjuntos de escalado de máquinas virtuales**, seleccione **Editar columnas** para abrir la lista de columnas disponibles.
4. En la sección **Columnas disponibles**, seleccione **Mantenimiento de autoservicio** y, luego, muévalo a la lista **Columnas seleccionadas**. Seleccione **Aplicar**.  

    Para que el elemento **Mantenimiento de autoservicio** sea más fácil de encontrar, puede cambiar la opción de lista desplegable en la sección **Columnas disponibles** de **Todas** a **Propiedades**.

La columna **Mantenimiento de autoservicio** aparece ahora en la lista de conjuntos de escalado de máquinas virtuales. Cada conjunto de escalado de máquinas virtuales puede tener uno de los siguientes valores para la columna de mantenimiento de autoservicio:

| Value | Descripción |
|-------|-------------|
| Sí | Al menos una máquina virtual del conjunto de escalado de máquinas virtuales está en una ventana de autoservicio. Puede iniciar el mantenimiento en cualquier momento durante esta ventana de autoservicio. | 
| No | No hay ninguna máquina virtual en una ventana de autoservicio en el conjunto de escalado de máquinas virtuales afectado. | 
| - | Los conjuntos de escalado de máquinas virtuales no forman parte de una ola de mantenimiento planeado.| 

## <a name="notification-and-alerts-in-the-portal"></a>Notificaciones y alertas en el portal

Azure comunica una programación para el mantenimiento planeado enviando un correo electrónico al grupo de propietario y copropietarios de la suscripción. Puede agregar destinatarios y canales a esta comunicación mediante la creación de alertas de registro de actividad. Para más información, consulte [Supervise la actividad de suscripción con Azure Activity Log](../azure-monitor/platform/platform-logs-overview.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú de la izquierda, seleccione **Monitor**. 
3. En el panel **Monitor - Alertas (clásico)** , seleccione **+ Agregar alerta de registro de actividad**.
4. En la página **Agregar alerta de registro de actividad**, seleccione o escriba la información solicitada. En **Criterios**, asegúrese de establecer los valores siguientes:
   - **Categoría de eventos**: Seleccione **Service Health**.
   - **Servicios**: Seleccione **Virtual Machine Scale Sets y Virtual Machines**.
   - **Tipo**: Seleccione **Mantenimiento planeado**. 
    
Para más información sobre cómo configurar las alertas de registro de actividad, consulte [Creación de alertas de registro de actividad](../azure-monitor/platform/activity-log-alerts.md).
    
    
## <a name="start-maintenance-on-your-virtual-machine-scale-set-from-the-portal"></a>Inicio del mantenimiento en el conjunto de escalado de máquinas virtuales desde el portal

Puede ver más detalles relacionados con el mantenimiento en la información general de los conjuntos de escalado de máquinas virtuales. Si se incluye al menos una máquina virtual del conjunto de escalado de máquinas virtuales en la ola de mantenimiento planeado, se agrega una nueva cinta de notificaciones cerca de la parte superior de la página. Seleccione la cinta de notificaciones para ir a la página **Mantenimiento**. 

En la página **Mantenimiento**, puede ver qué instancia de máquina virtual resulta afectada por el mantenimiento planeado. Para iniciar el mantenimiento, active la casilla que corresponde a la máquina virtual afectada. A continuación, seleccione **Iniciar el mantenimiento**.

Una vez que comience el mantenimiento, las máquinas virtuales afectadas del conjunto de escalado de máquinas virtuales se someterán a mantenimiento y no estarán disponibles temporalmente. Si perdió la ventana de autoservicio, seguirá pudiendo ver la ventana de tiempo cuando Azure mantenga el conjunto de escalado de máquinas virtuales.
 
## <a name="check-maintenance-status-by-using-powershell"></a>Comprobación del estado de mantenimiento con PowerShell

Puede usar Azure PowerShell para ver cuándo está programado el mantenimiento de las máquinas virtuales de sus conjuntos de escalado. La información de mantenimiento planeado está disponible mediante el cmdlet [Get-AzVmss](/powershell/module/az.compute/get-azvmss) cuando se usa el parámetro `-InstanceView`.
 
La información de mantenimiento se devuelve solo si hay mantenimiento planeado. Si no está programado ningún mantenimiento que afecte a la instancia de la máquina virtual, el cmdlet no devuelve ninguna información de mantenimiento. 

```powershell
Get-AzVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -InstanceView
```

Las siguientes propiedades se devuelven en **MaintenanceRedeployStatus**: 

| Value | Descripción   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indica si puede iniciar el mantenimiento en la máquina virtual en este momento. |
| PreMaintenanceWindowStartTime         | El comienzo de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual. |
| PreMaintenanceWindowEndTime           | El final de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual. |
| MaintenanceWindowStartTime            | El comienzo del mantenimiento programado en que Azure inicia el mantenimiento de la máquina virtual. |
| MaintenanceWindowEndTime              | El final de la ventana de mantenimiento programado en la que Azure inicia el mantenimiento en la máquina virtual. |
| LastOperationResultCode               | El resultado del último intento de iniciar el mantenimiento en la máquina virtual. |



### <a name="start-maintenance-on-your-vm-instance-by-using-powershell"></a>Inicio del mantenimiento en la instancia de máquina virtual mediante PowerShell

Puede iniciar el mantenimiento en una máquina virtual si **IsCustomerInitiatedMaintenanceAllowed** está establecido en **true**. Use el cmdlet [Set-AzVmss](/powershell/module/az.compute/set-azvmss) con el parámetro `-PerformMaintenance`.

```powershell
Set-AzVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -PerformMaintenance 
```

## <a name="check-maintenance-status-by-using-the-cli"></a>Comprobación del estado de mantenimiento mediante la CLI

Puede ver la información de mantenimiento planeado con el cmdlet [az vmss list-instances](/cli/azure/vmss?view=azure-cli-latest#az-vmss-list-instances).
 
La información de mantenimiento se devuelve solo si hay mantenimiento planeado. Si no está programado ningún mantenimiento que afecte a la instancia de máquina virtual, el comando no devuelve ninguna información de mantenimiento. 

```azurecli
az vmss list-instances -g rgName -n vmssName --expand instanceView
```

Las siguientes propiedades se devuelven en **MaintenanceRedeployStatus** para cada instancia de máquina virtual: 

| Value | Descripción   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indica si puede iniciar el mantenimiento en la máquina virtual en este momento. |
| PreMaintenanceWindowStartTime         | El comienzo de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual. |
| PreMaintenanceWindowEndTime           | El final de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual. |
| MaintenanceWindowStartTime            | El comienzo del mantenimiento programado en que Azure inicia el mantenimiento de la máquina virtual. |
| MaintenanceWindowEndTime              | El final de la ventana de mantenimiento programado en la que Azure inicia el mantenimiento en la máquina virtual. |
| LastOperationResultCode               | El resultado del último intento de iniciar el mantenimiento en la máquina virtual. |


### <a name="start-maintenance-on-your-vm-instance-by-using-the-cli"></a>Inicio del mantenimiento en la instancia de máquina virtual mediante la CLI

La siguiente llamada inicia el mantenimiento en una instancia de máquina virtual si `IsCustomerInitiatedMaintenanceAllowed` está establecido en **true**:

```azurecli
az vmss perform-maintenance -g rgName -n vmssName --instance-ids id
```

## <a name="faq"></a>Preguntas más frecuentes

**P: ¿Por qué es necesario reiniciar las máquinas virtuales ahora?**

**R:** Aunque la mayoría de actualizaciones y mejoras de la plataforma Azure no afectan a la disponibilidad de las máquinas virtuales, en algunos casos no se puede evitar el reinicio de las máquinas virtuales hospedadas en Azure. Hemos acumulado varios cambios que requieren el reinicio de nuestros servidores, lo que conllevará el reinicio de las máquinas virtuales.

**P: Si sigo las recomendaciones para lograr una alta disponibilidad mediante un conjunto de disponibilidad, ¿estoy seguro?**

**R:** Las máquinas virtuales implementadas en un conjunto de disponibilidad o en conjuntos de escalado de máquinas virtuales usan dominios de actualización. Al realizar mantenimiento, Azure respeta la restricción de dominios de actualización y no reinicia las máquinas virtuales desde un dominio de actualización diferente (dentro del mismo conjunto de disponibilidad). Azure también espera al menos 30 minutos antes de pasar al siguiente grupo de máquinas virtuales. 

Para más información acerca de la alta disponibilidad, consulte [Regiones y disponibilidad de máquinas virtuales en Azure](../virtual-machines/availability.md).

**P: ¿Cómo puedo recibir notificaciones acerca del mantenimiento planeado?**

**R:** Una oleada de mantenimiento planeado se inicia mediante el establecimiento de una programación en una o varias regiones de Azure. Poco después, se envía una notificación por correo electrónico a los propietarios de las suscripciones (un correo electrónico por suscripción). Puede agregar canales y destinatarios para esta notificación mediante alertas de registro de actividad. Si implementa una máquina virtual en una región en la que ya está programado el mantenimiento planeado, no recibe la notificación. En su lugar, compruebe el estado de mantenimiento de la máquina virtual.

**P: No veo ninguna indicación de mantenimiento planeado en el portal, PowerShell o la CLI. ¿Cuál es el problema?**

**R:** La información relacionada con el mantenimiento planeado está disponible durante una ola de mantenimiento planeado solo para las máquinas virtuales que van a verse afectadas por él. Si no ve datos, puede que la ola de mantenimiento ya haya acabado (o no se haya iniciado) o que la máquina virtual ya esté hospedada en un servidor actualizado.

**P: ¿Existe alguna manera de saber exactamente cuándo resultará afectada mi máquina virtual?**

**R:** Al establecer la programación, definimos una ventana de tiempo de varios días. Se desconoce la secuencia exacta de los servidores (y las máquinas virtuales) en esta ventana. Si quiere conocer el momento exacto en que se actualizarán las máquinas virtuales, puede usar [eventos programados](../virtual-machines/windows/scheduled-events.md). Cuando usa eventos programados, puede realizar consultas dentro de la máquina virtual y recibir una notificación de 15 minutos antes de reiniciar la máquina virtual.

**P: ¿Cuánto tiempo tardarán en reiniciar mi máquina virtual?**

**R:**  En función del tamaño de la máquina virtual, el reinicio podría tardar hasta varios minutos durante la ventana de mantenimiento de autoservicio. Durante los reinicios ejecutados por Azure en la ventana de mantenimiento programado, el proceso suele tardar unos 25 minutos. Si usa Cloud Services (roles de trabajo o web), conjuntos de escalado de máquinas virtuales o conjuntos de disponibilidad, dispone de 30 minutos entre cada grupo de máquinas virtuales (dominio de actualización) durante la ventana de mantenimiento programado. 

**P: No veo información de mantenimiento en mis máquinas virtuales. ¿A qué se debe?**

**R:** Existen varios motivos por los que podría no ver información de mantenimiento en sus máquinas virtuales:
   - Usa una suscripción marcada como *interna de Microsoft*.
   - Las máquinas virtuales no están programadas para mantenimiento. Es posible que la ola de mantenimiento finalizara, se cancelara o modificara para que las máquinas virtuales ya no resulten afectadas por ella.
   - La columna **Mantenimiento** no se ha agregado a la vista de la lista de máquinas virtuales. Aunque hemos agregado esta columna a la vista predeterminada, si configura la vista para ver las columnas no predeterminadas, debe agregar manualmente la columna **Mantenimiento** a la vista de lista de máquinas virtuales.

**P: Está programado que se realice un segundo mantenimiento programado de mi máquina virtual. ¿Por qué?**

**R:** En varios casos de uso, la máquina virtual se programa para mantenimiento después de que ya ha completado su mantenimiento y la ha vuelto a implementar:
   - Hemos cancelado la oleada de mantenimiento y la hemos reiniciado con otra carga útil. Es posible que hayamos detectado una carga útil con errores y que simplemente necesitemos implementar una carga adicional.
   - El *servicio de una máquina virtual se ha reparado* en otro nodo debido a un error de hardware.
   - Ha seleccionado detener (desasignar) y reiniciar la máquina virtual.
   - El **apagado automático** se ha activado en la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a registrarse para eventos de mantenimiento desde la máquina virtual mediante [eventos programados](../virtual-machines/windows/scheduled-events.md).