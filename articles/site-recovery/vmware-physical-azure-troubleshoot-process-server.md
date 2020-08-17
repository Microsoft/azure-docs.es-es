---
title: Solucionar problemas del servidor de procesos de Azure Site Recovery
description: En este artículo se describe cómo solucionar problemas relacionados con el servidor de procesos de Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: troubleshooting
ms.date: 09/09/2019
ms.author: raynew
ms.openlocfilehash: 4816b597d66aea3bbe7f834004f924b5108de939
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499762"
---
# <a name="troubleshoot-the-process-server"></a>Solucionar problemas del servidor de procesos

El servidor de procesos [Site Recovery](site-recovery-overview.md) se usa para configurar la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos locales. En este artículo se describe cómo solucionar problemas con el servidor de procesos, incluidos los problemas de replicación y conectividad.

[Más información](vmware-physical-azure-config-process-server-overview.md) sobre el servidor de procesos.

## <a name="before-you-start"></a>Antes de comenzar

Antes de empezar a solucionar problemas:

1. Asegúrese de que comprende cómo [supervisar los servidores de procesos](vmware-physical-azure-monitor-process-server.md).
2. Revise los procedimientos recomendados siguientes.
3. Asegúrese de seguir las [consideraciones de capacidad](site-recovery-plan-capacity-vmware.md#capacity-considerations) y use la guía de ajuste de tamaño para el [servidor de configuración](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-configuration-server-and-inbuilt-process-server) o los [servidores de procesos independientes](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-process-server).

## <a name="best-practices-for-process-server-deployment"></a>Procedimientos recomendados para la implementación del servidor de procesos

Hemos resumido una serie de procedimientos recomendados generales para el rendimiento óptimo de los servidores de procesos.

**Procedimiento recomendado** | **Detalles**
--- |---
**Uso** | Asegúrese de que el servidor de configuración o el servidor de procesos independiente solo se usen para la finalidad prevista. No ejecute nada más en el equipo.
**Dirección IP** | Asegúrese de que el servidor de procesos tenga una dirección IPv4 estática y no tenga NAT configurado.
**Controlar el uso de memoria y CPU** |Conserve el uso de CPU y memoria por debajo del 70 %.
**Garantizar el espacio libre** | El espacio libre hace referencia al espacio en disco de caché en el servidor de procesos. Los datos de replicación se almacenan en caché antes de cargarlos en Azure.<br/><br/> Conserve el espacio libre por encima del 25 %. Si cae por debajo del 20 %, la replicación se limita para las máquinas replicadas asociadas con el servidor de procesos.

## <a name="check-process-server-health"></a>Comprobar el estado del servidor de procesos

El primer paso para la solución de problemas es comprobar el estado del servidor de procesos. Para ello, revise todas las alertas, compruebe que los servicios necesarios están en ejecución y compruebe que haya latido del servidor de procesos. En el gráfico siguiente, se resumen estos pasos, seguido de los procedimientos que le ayudarán a realizarlos.

![Solución de problemas del estado del servidor de procesos](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-process-server-health.png)

## <a name="step-1-troubleshoot-process-server-health-alerts"></a>Paso 1: Solucionar problemas de las alertas de estado del servidor de procesos

El servidor de procesos genera un número de alertas de estado. Estas alertas y las acciones recomendadas se resumen en la tabla siguiente.

**Tipo de alerta** | **Error** | **Solución de problemas**
--- | --- | --- 
![Healthy][green] | None  | El servidor de procesos está conectado y en buen estado.
![Advertencia][yellow] | No se ejecutan los servicios especificados. | 1. Compruebe que se están ejecutando los servicios.<br/> 2. Si los servicios se ejecutan según lo previsto, siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).
![Advertencia][yellow]  | Uso de CPU > 80 % durante los últimos 15 minutos. | 1. No agregue nuevas máquinas.<br/>2. Compruebe que el número de máquinas virtuales que usan el servidor de procesos se alinea con los [límites definidos](site-recovery-plan-capacity-vmware.md#capacity-considerations) y considere la posibilidad de configurar un [servidor de procesos adicional](vmware-azure-set-up-process-server-scale.md).<br/>3. Siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).
![Crítico][red] |  Uso de CPU > 95 % durante los últimos 15 minutos. | 1. No agregue nuevas máquinas.<br/>2. Compruebe que el número de máquinas virtuales que usan el servidor de procesos se alinea con los [límites definidos](site-recovery-plan-capacity-vmware.md#capacity-considerations) y considere la posibilidad de configurar un [servidor de procesos adicional](vmware-azure-set-up-process-server-scale.md).<br/>3. Siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).<br/> 4. Si el problema persiste, ejecute [Deployment Planner](https://aka.ms/asr-v2a-deployment-planner) para la replicación del servidor de VMware y físico.
![Advertencia][yellow] | Uso de memoria > 80 % durante los últimos 15 minutos. |  1. No agregue nuevas máquinas.<br/>2. Compruebe que el número de máquinas virtuales que usan el servidor de procesos se alinea con los [límites definidos](site-recovery-plan-capacity-vmware.md#capacity-considerations) y considere la posibilidad de configurar un [servidor de procesos adicional](vmware-azure-set-up-process-server-scale.md).<br/>3. Siga las instrucciones asociadas con la advertencia.<br/> 4. Si el problema persiste, siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).
![Crítico][red] | Uso de memoria > 95 % durante los últimos 15 minutos. | 1. No agregue nuevas máquinas y considere la posibilidad de configurar un [servidor de procesos adicional](vmware-azure-set-up-process-server-scale.md).<br/> 2. Siga las instrucciones asociadas con la advertencia.<br/> 3. 4. Si continúa el problema, siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).<br/> 4. Si el problema persiste, ejecute [Deployment Planner](https://aka.ms/asr-v2a-deployment-planner) para los problemas de replicación del servidor de VMware y físico.
![Advertencia][yellow] | Espacio libre de la carpeta de caché < 30 % durante los últimos 15 minutos. | 1. No agregue nuevas máquinas y considere la posibilidad de configurar un [servidor de procesos adicional](vmware-azure-set-up-process-server-scale.md).<br/>2. Compruebe que el número de máquinas virtuales que usan el servidor de procesos se alinea con las [directrices](site-recovery-plan-capacity-vmware.md#capacity-considerations).<br/> 3. Siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).
![Crítico][red] |  Espacio libre < 25 % durante los últimos 15 minutos | 1. Siga las instrucciones asociadas con la advertencia para este problema.<br/> 2. 3. Siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).<br/> 3. Si el problema persiste, ejecute [Deployment Planner](https://aka.ms/asr-v2a-deployment-planner) para la replicación del servidor de VMware y físico.
![Crítico][red] | No hay latido del servidor de procesos durante 15 minutos o más. El servicio tmansvs no se comunica con el servidor de configuración. | 1) Asegúrese de que el servidor de procesos esté en funcionamiento.<br/> 2. Compruebe que tmassvc se ejecuta en el servidor de procesos.<br/> 3. Siga las instrucciones siguientes para [solucionar problemas de conectividad y replicación](#check-connectivity-and-replication).


![Clave de tabla](./media/vmware-physical-azure-troubleshoot-process-server/table-key.png)


## <a name="step-2-check-process-server-services"></a>Paso 2: Comprobar los servicios del servidor de procesos

Los servicios que deben estar en ejecución en el servidor de procesos se resumen en la siguiente tabla. Hay ligeras diferencias entre servicios, en función de cómo está implementado el servidor de procesos. 

Para todos los servicios, a excepción del agente de Microsoft Azure Recovery Services(obengine), compruebe que StartType se define en **Automático** o **Automático (inicio retrasado)** .
 
**Implementación** | **Servicios en ejecución**
--- | ---
**Servidor de procesos en el servidor de configuración** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Servicio de carga de registro (LogUpload); Servicio Aplicación InMage Scout; Agente de Microsoft Azure Recovery Services (obengine); InMage Scout VX Agent-Sentinel/Outpost (svagents); tmansvc; Servicio de publicación World Wide Web (W3SVC); MySQL; Servicio Microsoft Azure Site Recovery (dra)
**Servidor de procesos que se ejecuta como servidor independiente** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Servicio de carga de registro (LogUpload); Servicio Aplicación InMage Scout; Agente de Microsoft Azure Recovery Services (obengine); InMage Scout VX Agent-Sentinel/Outpost (svagents); tmansvc.
**Servidor de procesos implementado en Azure para conmutación por recuperación** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Servicio de carga de registro (LogUpload)


## <a name="step-3-check-the-process-server-heartbeat"></a>Paso 3: Comprobar el latido del servidor de procesos

Si no hay ningún latido del servidor de procesos (código de error 806), realice lo siguiente:

1. Compruebe que la máquina virtual del servidor de procesos esté en funcionamiento.
2. Compruebe estos registros en busca errores.

    C:\ProgramData\ASR\home\svsystems\eventmanager *.log C\ProgramData\ASR\home\svsystems\monitor_protection*.log

## <a name="check-connectivity-and-replication"></a>Comprobar la conectividad y replicación

 Los errores iniciales y continuados de la replicación se deben a menudo a problemas de conectividad entre las máquinas de origen y el servidor de procesos, o entre el servidor de procesos y Azure. En el gráfico siguiente, se resumen estos pasos, seguido de los procedimientos que le ayudarán a realizarlos.

![Diagrama de flujo que muestra los pasos para solucionar problemas de conectividad y replicación.](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-connectivity-replication.png)


## <a name="step-4-verify-time-sync-on-source-machine"></a>Paso 4: Comprobar la sincronización de hora en la máquina de origen

Asegúrese de que la fecha y hora del sistema de la máquina replicada están sincronizadas. [Más información](/windows-server/networking/windows-time-service/accurate-time)

## <a name="step-5-check-anti-virus-software-on-source-machine"></a>Paso 5: Comprobar el software antivirus en la máquina de origen

Compruebe que ningún software antivirus en la máquina replicada bloquee Site Recovery. Si tiene que excluir Site Recovery de programas antivirus, revise [este artículo](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program).

## <a name="step-6-check-connectivity-from-source-machine"></a>Paso 6: Comprobar la conectividad de la máquina de origen


1. Instale el [cliente Telnet](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771275(v=ws.10)) en la máquina de origen, si es necesario. No use Ping.
2. En la máquina de origen, haga ping en el servidor de procesos en el puerto HTTPS con Telnet. De forma predeterminada, 9443 es el puerto HTTPS para el tráfico de replicación.

    `telnet <process server IP address> <port>`

3. Compruebe si la conexión es correcta.


**Conectividad** | **Detalles** | **Acción**
--- | --- | ---
**Correcto** | Telnet muestra una pantalla en blanco y el servidor de procesos es accesible. | No se requiere ninguna acción adicional.
**Incorrecto** | No se puede conectar. | Asegúrese de que se permite el puerto de entrada 9443 en el servidor de procesos. Por ejemplo, si tiene una red perimetral o una subred con pantalla. Vuelva a comprobar la conectividad.
**Parcialmente correcto** | Puede conectarse, pero la máquina de origen notifica que no se puede alcanzar el servidor de procesos. | Continúe con el siguiente procedimiento para solucionar problemas.

## <a name="step-7-troubleshoot-an-unreachable-process-server"></a>Paso 7: Solución de problemas de un servidor de procesos inaccesible

Si el servidor de procesos no es accesible desde la máquina de origen, se mostrará el error 78186. Si no se soluciona, este problema dará lugar a que no se generen los puntos de recuperación coherentes con la aplicación y coherentes con el bloqueo según lo previsto.

Para solucionar el problema, compruebe si la máquina de origen puede llegar a la dirección IP del servidor de procesos y ejecute la herramienta cxpsclient en la máquina de origen para comprobar la conexión de un extremo a otro.


### <a name="check-the-ip-connection-on-the-process-server"></a>Comprobar la conexión IP en el servidor de procesos

Si telnet es correcto, pero la máquina de origen notifica que no se puede alcanzar el servidor de procesos, compruebe si puede llegar a la dirección IP del servidor de procesos.

1. En un explorador web, intente conectarse a la dirección IP https://<IP_SP>:<Puerto_datos_SP>/.
2. Si esta comprobación muestra un error de certificado HTTPS, es normal. Si ignora el error, debería ver 400 - Solicitud incorrecta. Esto significa que el servidor no puede atender la solicitud del explorador y que la conexión HTTPS estándar está bien.
3. Si esta comprobación no funciona, tenga en cuenta el mensaje de error del explorador. Por ejemplo, un error 407 indicará un problema con la autenticación proxy.

### <a name="check-the-connection-with-cxpsclient"></a>Comprobar la conexión con cxpsclient

Además, puede ejecutar la herramienta cxpsclient para comprobar la conexión de un extremo a otro.

1. Ejecute la herramienta de la manera siguiente:

    ```
    <install folder>\cxpsclient.exe -i <PS_IP> -l <PS_Data_Port> -y <timeout_in_secs:recommended 300>
    ```

2. En el servidor de procesos, compruebe los registros generados en estas carpetas:

    C:\ProgramData\ASR\home\svsystems\transport\log\cxps.err  C:\ProgramData\ASR\home\svsystems\transport\log\cxps.xfer



### <a name="check-source-vm-logs-for-upload-failures-error-78028"></a>Compruebe los registros de la máquina virtual de origen para buscar errores de carga (error 78028)

El problema del bloqueo de cargas de datos desde máquinas de origen al servicio de proceso puede dar lugar a que no se generen puntos de recuperación coherentes con el bloqueo y coherentes con la aplicación. 

1. Para solucionar problemas de errores de carga de red, puede buscar errores en este registro:

    C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\agent\svagents*.log 

2. Use el resto de los procedimientos descritos en este artículo como ayuda para resolver problemas de carga de datos.



## <a name="step-8-check-whether-the-process-server-is-pushing-data"></a>Paso 8: Comprobar si el servidor de procesos está insertando datos

Compruebe si el servidor de procesos está insertando datos activamente en Azure.

  1. En el servidor de procesos, abra el Administrador de tareas (presione Ctrl+Mayús+Esc).
  2. Seleccione la pestaña **Rendimiento** > **Abrir el Monitor de recursos**.
  3. En la página **Monitor de recursos**, seleccione la pestaña **Red**. En **Procesos con actividad de red**, compruebe si cbengine.exe está enviando un gran volumen de datos.

       ![Captura de pantalla que muestra un gran número de volúmenes en procesos con actividad de red.](./media/vmware-physical-azure-troubleshoot-process-server/cbengine.png)

  Si cbengine.exe no está enviando un gran volumen de datos, realice los pasos de las secciones siguientes.

## <a name="step-9-check-the-process-server-connection-to-azure-blob-storage"></a>Paso 9: Comprobar la conexión entre el servidor de procesos y Azure Blob Storage

1. En el monitor de recursos, seleccione **cbengine.exe**.
2. En **Conexiones TCP**, compruebe si hay conectividad entre el servidor de procesos y Azure Storage.

  ![Captura de pantalla que muestra la conectividad entre cbengine.exe y la dirección URL de Azure Blob Storage.](./media/vmware-physical-azure-troubleshoot-process-server/rmonitor.png)

### <a name="check-services"></a>Comprobar servicios

Si no hay conectividad entre el servidor de procesos y la dirección URL de Azure Blob Storage, compruebe si los servicios están en ejecución.

1. En el Panel de Control, seleccione **Servicios**.
2. Compruebe que los siguientes servicios se están ejecutando:

    - cxprocessserver
    - InMage Scout VX Agent - Sentinel/Outpost
    - Agente de Microsoft Azure Recovery Services
    - Servicio Microsoft Azure Site Recovery
    - tmansvc

3. Inicie o reinicie cualquier servicio que no esté en ejecución.
4. Compruebe que el servidor de procesos está conectado y es accesible. 

## <a name="step-10-check-the-process-server-connection-to-azure-public-ip-address"></a>Paso 10: comprobar la conexión entre el servidor de procesos y la dirección IP pública de Azure

1. En el servidor de procesos, en **%programfiles%\Microsoft Azure Recovery Services Agent\Temp**, abra el archivo CBEngineCurr.errlog más reciente.
2. En el archivo, busque **443** o la cadena **Error en el intento de conexión**.

  ![Registros de errores en la carpeta Temp](./media/vmware-physical-azure-troubleshoot-process-server/logdetails1.png)

3. Si ve errores, localice la dirección IP pública de Azure en el archivo CBEngineCurr.currLog con el puerto 443:

  `telnet <your Azure Public IP address as seen in CBEngineCurr.errlog>  443`

5. En la línea de comandos del servidor de procesos, use Telnet para hacer ping a la dirección IP pública de Azure.
6. Si no se puede conectar, siga el procedimiento siguiente.

## <a name="step-11-check-process-server-firewall-settings"></a>Paso 11: Compruebe la configuración de firewall del servidor de procesos. 

Compruebe si el firewall basado en la dirección IP en el servidor de procesos bloquea el acceso.

1. Para las reglas de firewall basadas en direcciones IP:

    a) Descargue la lista completa de [intervalos de direcciones IP del centro de datos de Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).

    b) Agregue los intervalos de direcciones IP a la configuración del firewall para asegurarse de que el firewall permite la comunicación con Azure (y con el puerto HTTPS predeterminado, 443).

    c) Permita intervalos de direcciones IP correspondientes a la región de Azure de la suscripción y a la región Oeste de EE. UU. de Azure (se usan para controlar el acceso y administrar las identidades).

2. Para firewalls basados en la dirección URL, agregue las direcciones URL enumeradas en la tabla siguiente a la configuración del firewall.

    [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]  


## <a name="step-12-verify-process-server-proxy-settings"></a>Paso 12: Comprobar la configuración de proxy del servidor de procesos 

1. Si usa un servidor proxy, asegúrese de que el servidor DNS resuelva el nombre de dicho servidor. Compruebe el valor que proporcionó al configurar el servidor de configuración en la clave del Registro **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure Site Recovery\ProxySettings**.
2. Asegúrese de que el agente de Azure Site Recovery usa la misma configuración para enviar datos.

    a) Busque **Microsoft Azure Backup**.

    b) Abra **Microsoft Azure Backup** y seleccione **Acción** > **Cambiar propiedades**.

    c) En la pestaña **Configuración de proxy**, la dirección del proxy debe ser igual que la que se muestra en la configuración del Registro. Si no es así, cámbiela por la misma dirección.

## <a name="step-13-check-bandwidth"></a>Paso 13: Comprobar ancho de banda

Aumente el ancho de banda entre el servidor de procesos y Azure y, luego, compruebe si el problema persiste.


## <a name="next-steps"></a>Pasos siguientes

Si necesita más ayuda, puede publicar su pregunta en la [Página de preguntas y respuestas de Microsoft sobre Azure Site Recovery](/answers/topics/azure-site-recovery.html). 

[green]: ./media/vmware-physical-azure-troubleshoot-process-server/green.png
[yellow]: ./media/vmware-physical-azure-troubleshoot-process-server/yellow.png
[red]: ./media/vmware-physical-azure-troubleshoot-process-server/red.png
