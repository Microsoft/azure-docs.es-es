---
title: Supervisión de los eventos programados para las máquinas virtuales Windows en Azure
description: Aprenda a supervisar los eventos programados en las máquinas virtuales de Azure.
author: mysarn
ms.service: virtual-machines-windows
ms.subservice: monitoring
ms.date: 08/20/2019
ms.author: sarn
ms.topic: how-to
ms.openlocfilehash: 0806c6e0ed89c2c0f4712ec985599810119fcf89
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999027"
---
# <a name="monitoring-scheduled-events"></a>Supervisión de Scheduled Events

Las actualizaciones se aplican a diferentes partes de Azure cada día, para que los servicios que se ejecutan en ellas sean seguros y estén actualizados. Además de las actualizaciones planeadas, también se pueden producir eventos no planeados. Por ejemplo, si se detecta algún error o degradación del hardware, es posible que los servicios de Azure necesiten realizar un mantenimiento no planeado. Con la migración en vivo, la conservación de actualizaciones en la memoria y al mantener de forma general un control estricto acerca del impacto de las actualizaciones, en la mayoría de los casos estos eventos son casi transparentes para los clientes y no tienen ningún impacto o, como máximo, causan unos segundos de inmovilización en la máquina virtual. Sin embargo, para algunas aplicaciones, incluso unos pocos segundos de inmovilización en las máquinas virtuales podría tener un impacto. Es importante conocer por adelantado el próximo mantenimiento de Azure para garantizar la mejor experiencia para esas aplicaciones. El [servicio Scheduled Events](scheduled-events.md) proporciona una interfaz de programación para recibir notificaciones sobre el próximo mantenimiento y le permite administrar correctamente el mantenimiento. 

En este artículo, le mostraremos cómo puede usar los eventos programados para recibir notificaciones sobre los eventos de mantenimiento que podrían afectar a las máquinas virtuales y crear una automatización básica que pueda ayudar con la supervisión y el análisis.


## <a name="routing-scheduled-events-to-log-analytics"></a>Enrutamiento de eventos programados a Log Analytics

Scheduled Events está disponible como parte de [Azure Instance Metadata Service](instance-metadata-service.md), que está disponible en todas las máquinas virtuales de Azure. Los clientes pueden escribir automatizaciones para consultar el punto de conexión de sus máquinas virtuales para buscar notificaciones de mantenimiento programado y realizar mitigaciones, tales como guardar el estado y sacar la máquina virtual de la rotación. Se recomienda crear la automatización para registrar Scheduled Events de modo que pueda tener un registro de auditoría de los eventos de mantenimiento de Azure. 

En este artículo, le guiaremos a través de la captura de mantenimiento de Scheduled Events en Log Analytics. Después, se desencadenarán algunas acciones de notificación básicas, como enviar un correo electrónico al equipo y obtener una vista histórica de todos los eventos que han afectado a las máquinas virtuales. Para la agregación y automatización de eventos usaremos [Log Analytics](../../azure-monitor/learn/quick-create-workspace.md), pero puede usarse cualquier solución de supervisión para recopilar estos registros y desencadenar la automatización.

![Diagrama que muestra el ciclo de vida de los eventos](./media/notifications/events.png)

## <a name="prerequisites"></a>Requisitos previos

En este ejemplo, tendrá que crear una [máquina virtual Windows en un conjunto de disponibilidad](tutorial-availability-sets.md). Scheduled Events proporciona notificaciones sobre los cambios que pueden afectar a cualquiera de las máquinas virtuales del conjunto de disponibilidad, el servicio en la nube, el conjunto de escalado de máquinas virtuales o a las máquinas virtuales independientes. Se va a ejecutar un [servicio](https://github.com/microsoft/AzureScheduledEventsService) que sondea los eventos programados en una de las máquinas virtuales, la cual actuará como recopilador para obtener los eventos de todas las demás máquinas virtuales del conjunto de disponibilidad.    

No elimine el grupo de recursos del grupo al final del tutorial.

También necesitará [crear un área de trabajo de Log Analytics](../../azure-monitor/learn/quick-create-workspace.md) que se usará para agregar información de las máquinas virtuales del conjunto de disponibilidad.

## <a name="set-up-the-environment"></a>Configuración del entorno

Ahora debería tener dos máquinas virtuales iniciales en un conjunto de disponibilidad. Ahora debemos crear una tercera máquina virtual, denominada myCollectorVM, en el mismo conjunto de disponibilidad. 

```azurepowershell-interactive
New-AzVm `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Name "myCollectorVM" `
   -Location "East US" `
   -VirtualNetworkName "myVnet" `
   -SubnetName "mySubnet" `
   -SecurityGroupName "myNetworkSecurityGroup" `
   -OpenPorts 3389 `
   -PublicIpAddressName "myPublicIpAddress3" `
   -AvailabilitySetName "myAvailabilitySet" `
   -Credential $cred
```
 

Descargue el archivo .ZIP de instalación del proyecto de [GitHub](https://github.com/microsoft/AzureScheduledEventsService/archive/master.zip).

Conéctese a **myCollectorVM**, copie el archivo .ZIP en la máquina virtual y extraiga todos los archivos. En la máquina virtual, abra un símbolo del sistema de PowerShell. Mueva el símbolo del sistema a la carpeta que contiene `SchService.ps1`, por ejemplo: `PS C:\Users\azureuser\AzureScheduledEventsService-master\AzureScheduledEventsService-master\Powershell>`, y configure el servicio.

```powershell
.\SchService.ps1 -Setup
```

Inicie el servicio.

```powershell
.\SchService.ps1 -Start
```

Ahora, el servicio comenzará a realizar un sondeo cada 10 segundos para cualquier evento programado y aprobará los eventos para acelerar su mantenimiento.  Scheduled Events captura los eventos Inmovilizar, Reiniciar, Reimplementar y Reemplazar.   Tenga en cuenta que puede extender el script para desencadenar algunas mitigaciones antes de aprobar el evento.

Valide el estado del servicio y asegúrese de que se está ejecutando.

```powershell
.\SchService.ps1 -status  
```

Esto debería devolver el valor `Running`.

Ahora, el servicio comenzará a realizar un sondeo cada 10 segundos para cualquier evento programado y aprobará los eventos para acelerar su mantenimiento.  Scheduled Events captura los eventos Inmovilizar, Reiniciar, Reimplementar y Reemplazar. Puede extender el script para desencadenar algunas mitigaciones antes de aprobar el evento.

Cuando el servicio de eventos de programación captura cualquiera de los eventos anteriores, se registra en los valores de Event Status, Event Type, Resources (nombres de máquina virtual) y NotBefore (período mínimo de aviso) del registro de eventos de la aplicación. Puede buscar los eventos con el id. 1234 en el registro de eventos de la aplicación.

Una vez que el servicio se ha configurado e iniciado, registrará los eventos en los registros de aplicación de Windows.   Para comprobar que funciona, reinicie una de las máquinas virtuales en el conjunto de disponibilidad y debería ver que se está registrando un evento en el Visor de eventos en Registros de Windows > Registro de aplicación, donde se muestra que la máquina virtual se reinició. 

![Captura de pantalla del visor de eventos.](./media/notifications/event-viewer.png)

Cuando el servicio Scheduled Events captura eventos, se registran en el registro de eventos de la aplicación con los valores de Event Status, Event Type, Resources (nombre de máquina virtual) y NotBefore (período mínimo de aviso). Puede buscar los eventos con el id. 1234 en el registro de eventos de la aplicación.

> [!NOTE] 
> En este ejemplo, las máquinas virtuales se encontraban en un conjunto de disponibilidad, por lo que pudimos designar una única máquina virtual como recopilador para escuchar y enrutar los eventos programados a nuestro espacio de trabajo de Log Analytics. Si tiene máquinas virtuales independientes, puede ejecutar el servicio en todas las máquinas virtuales y, a continuación, conectarlas individualmente a su área de trabajo de Log Analytics.
>
> Para esta configuración, elegimos Windows, pero puede diseñar una solución similar en Linux.

En cualquier momento puede detener o quitar el servicio Scheduled Event los modificadores `–stop` y `–remove`.

## <a name="connect-to-the-workspace"></a>Conexión a un área de trabajo


Ahora queremos conectar un área de trabajo de Log Analytics a la máquina virtual del recopilador. El área de trabajo de Log Analytics actúa como repositorio y la recopilación de registros de eventos se configurará para capturar los registros de aplicaciones de la máquina virtual del recopilador. 

 Para enrutar Scheduled Events al registro de eventos (que el servicio guardará como registro de aplicaciones), deberá conectar la máquina virtual a su área de trabajo de Log Analytics.  
 
1. Abra la página del área de trabajo que creó.
1. En **Conectarse a un origen de datos** seleccione **Máquinas virtuales de Azure (VM)** .

    ![Conexión con una máquina virtual como origen de datos](./media/notifications/connect-to-data-source.png)

1. Busque y seleccione **myCollectorVM**. 
1. En la página nueva de **myCollectorVM**, seleccione **Conectar**.

Se instalará [Microsoft Monitoring Agent](../extensions/oms-windows.md) en la máquina virtual. La máquina virtual tardará unos minutos en conectarse al área de trabajo e instalar la extensión. 

## <a name="configure-the-workspace"></a>Configuración del área de trabajo

1. Abra la página del área de trabajo y seleccione **Configuración avanzada**.
1. Seleccione **Datos** en el menú de la izquierda y, a continuación, **Registros de eventos de Windows**.
1. En **Recopilar eventos de los registros de eventos siguientes**, empiece a escribir *aplicación* y, a continuación, seleccione **Aplicación** en la lista.

    ![Seleccione Configuración avanzada.](./media/notifications/advanced.png)

1. Deje seleccionada la opción **ERROR**, **ADVERTENCIA** e **INFORMACIÓN** y, a continuación, seleccione **Guardar** para guardar la configuración.


> [!NOTE]
> Habrá cierto retraso y el registro puede tardar hasta 10 minutos en estar disponible. 


## <a name="creating-an-alert-rule-with-azure-monitor"></a>Creación de una regla de alerta con Azure Monitor 


Una vez que los eventos se insertan en Log Analytics, puede ejecutar la [consulta](../../azure-monitor/log-query/get-started-portal.md) siguiente para buscar los eventos de programación.

1. En la parte superior de la página, seleccione **Registros** y pegue lo siguiente en el cuadro de texto:

    ```
    Event
    | where EventLog == "Application" and Source contains "AzureScheduledEvents" and RenderedDescription contains "Scheduled" and RenderedDescription contains "EventStatus" 
    | project TimeGenerated, RenderedDescription
    | extend ReqJson= parse_json(RenderedDescription)
    | extend EventId = ReqJson["EventId"]
    ,EventStatus = ReqJson["EventStatus"]
    ,EventType = ReqJson["EventType"]
    ,NotBefore = ReqJson["NotBefore"]
    ,ResourceType = ReqJson["ResourceType"]
    ,Resources = ReqJson["Resources"]
    | project-away RenderedDescription,ReqJson
    ```

1. Seleccione **Guardar** y, a continuación, escriba *logQuery* para el nombre, deje **Consulta** como tipo, escriba *VMLogs* como **Categoría** y después seleccione **Guardar**. 

    ![Guardar la consulta](./media/notifications/save-query.png)

1. Seleccione **Nueva regla de alertas**. 
1. En la página **Crear regla**, deje `collectorworkspace` como **Recurso**.
1. En **Condición**, seleccione la entrada *Whenever the customer log search<login undefined>* (Cada vez que la búsqueda de registros del cliente sea). Se abrirá la página **Configurar lógica de señal**.
1. En **Valor del umbral**, escriba *0* y seleccione **Listo**.
1. En **Acciones**, seleccione **Crear grupo de acciones**. Se abrirá la página **Agregar grupo de acciones**.
1. En **Nombre del grupo de acciones**, escriba *myActionGroup*.
1. En **Nombre corto**, escriba **myActionGroup**.
1. En **Grupo de recursos**, seleccione **myResourceGroupAvailability**.
1. En Acciones, en **NOMBRE DE ACCIÓN** escriba **Correo electrónico** y, a continuación, seleccione **Email/SMS/Push/Voice** (Correo electrónico/SMS/Push/Voz). Se abre la página **Email/SMS/Push/Voice** (Correo electrónico/SMS/Push/Voz).
1. Seleccione **Correo electrónico**, escriba su dirección de correo electrónico y después haga clic en **Aceptar**.
1. En la página **Agregar grupo de acciones**, seleccione **Aceptar**. 
1. En la página **Crear regla**, en **DETALLES DE ALERTA**, escriba *myAlert* como **Nombre de la regla de alertas** y escriba *Regla de alertas de correo electrónico* en la **Descripción**.
1. Cuando finalice, seleccione **Crear regla de alertas**.
1. Reinicie una de las máquinas virtuales del conjunto de disponibilidad. En unos minutos, recibirá un mensaje de correo electrónico que indica que se desencadenó la alerta.

Para administrar las reglas de alertas, vaya al grupo de recursos, seleccione **Alertas** en el menú de la izquierda y, a continuación, seleccione **Administrar las reglas de alertas** en la parte superior de la página.

     
## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la página del [servicio de Scheduled Events](https://github.com/microsoft/AzureScheduledEventsService) en GitHub.
