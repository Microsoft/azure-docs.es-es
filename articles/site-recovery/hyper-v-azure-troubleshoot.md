---
title: Solución de problemas para la recuperación ante desastres en Hyper-V con Azure Site Recovery
description: Se describe cómo solucionar problemas de recuperación ante desastres con la replicación de Hyper-V a Azure mediante Azure Site Recovery.
services: site-recovery
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/14/2019
ms.author: sharrai
ms.openlocfilehash: 9e8a1246a2e48c3c569b82845c713ba25ee2467a
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426129"
---
# <a name="troubleshoot-hyper-v-to-azure-replication-and-failover"></a>Solución de problemas de la replicación y la conmutación por error de Hyper-V en Azure

En este artículo se describen problemas comunes que pueden surgir al replicar máquinas virtuales de Hyper-V locales en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

## <a name="enable-protection-issues"></a>Problemas para habilitar la protección

Si experimenta problemas al habilitar la protección para VM de Hyper-V, consulte las siguientes recomendaciones:

1. Compruebe que los hosts de Hyper-V y las VM cumplen con todos los [requisitos y requisitos previos](hyper-v-azure-support-matrix.md).
2. Si se los servidores de Hyper-V se ubican en nubes de System Center Virtual Machine Manager (VMM), compruebe que ha preparado el [servidor VMM](hyper-v-prepare-on-premises-tutorial.md#prepare-vmm-optional).
3. Compruebe que el servicio de administración de máquinas virtuales de Hyper-V se está ejecutando en los hosts de Hyper-V.
4. Compruebe si hay problemas que aparecen en el inicio de sesión de Hyper-V-VMMS\Admin en la VM. Este registro se encuentra en **Registros de aplicaciones y servicios** > **Microsoft** > **Windows**.
5. En la máquina virtual invitada, compruebe que WMI esté habilitado y accesible.
   - [Información acerca de](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) pruebas básicas de WMI.
   - [Solución de problemas](https://aka.ms/WMiTshooting) de WMI.
   - [Solución de problemas](/previous-versions/tn-archive/ff406382(v=msdn.10)#H22) con los scripts y servicios de WMI.
6. En la máquina virtual invitada, asegúrese de que se está ejecutando la versión más reciente de Integration Services.
    - [Compruebe](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services) que dispone de la versión más reciente.
    - [Mantenga](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#keep-integration-services-up-to-date) Integration Services actualizado.
    
## <a name="replication-issues"></a>Problemas de replicación

Para solucionar los problemas relacionados con la replicación inicial y en curso considere lo siguiente:

1. Asegúrese de que está ejecutando la [versión más reciente](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx) de los servicios de Site Recovery.
2. Verificar si la replicación está en pausa:
   - Compruebe el estado de mantenimiento de la máquina virtual en la consola del Administrador de Hyper-V.
   - Si el problema es crítico, haga clic con el botón derecho en la máquina virtual > **Replicación** > **Ver mantenimiento de la replicación**.
   - Si la replicación está en pausa, haga clic en **Reanudar replicación**.
3. Compruebe que se están ejecutando los servicios necesarios. Si no es así,reinícielos.
    - Si se está replicando Hyper-V sin VMM, compruebe que estos servicios se están ejecutando en el host de Hyper-V:
        - Servicio de administración de máquinas virtuales
        - Servicio Agente de Microsoft Azure Recovery Services
        - Servicio de Microsoft Azure Site Recovery
        - Servicio de host del proveedor de WMI
    - Si replica con VMM en el entorno, compruebe que estos servicios se están ejecutando:
        - En el host de Hyper-V, compruebe que se están ejecutando el servicio de administración de máquinas virtuales, el Agente de Microsoft Azure Recovery Services y el servicio de Host del proveedor de WMI.
        - En el servidor VMM, asegúrese de que se está ejecutando el servicio System Center Virtual Machine Manager.
4. Compruebe la conectividad entre el servidor de Hyper-V y Azure. Para comprobar la conectividad, abra el Administrador de tareas en el host de Hyper V. Vaya a la pestaña **Rendimiento** haga clic en el vínculo **Abrir el Monitor de recursos**. En la pestaña **Red** > **Proceso con actividad de red**, compruebe si cbengine.exe está enviando activamente grandes volúmenes (MB) de datos.
5. Compruebe si los hosts de Hyper-V pueden conectarse a la dirección URL del blob de almacenamiento de Azure. Para comprobar si los hosts pueden conectarse, seleccione y compruebe **cbengine.exe**. Revise las **conexiones TCP** para comprobar la conectividad desde el host al blob de almacenamiento de Azure.
6. Compruebe los problemas de rendimiento, como se describe a continuación.
    
### <a name="performance-issues"></a>Problemas de rendimiento

Las limitaciones de ancho de banda de red pueden afectar a la replicación. Para solucionar problemas proceda de la manera siguiente:

1. [Compruebe](https://support.microsoft.com/help/3056159/how-to-manage-on-premises-to-azure-protection-network-bandwidth-usage) si hay restricciones de ancho de banda o de limitación en su entorno.
2. Ejecute el [generador de perfiles Deployment Planner](hyper-v-deployment-planner-run.md).
3. Después de ejecutar el generador de perfiles, siga las recomendaciones sobre [ancho de banda](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input) y [almacenamiento](hyper-v-deployment-planner-analyze-report.md#vm-storage-placement-recommendation).
4. Compruebe [limitaciones de la modificación de datos](hyper-v-deployment-planner-analyze-report.md#azure-site-recovery-limits). Si observa una alta modificación datos en una máquina virtual, haga lo siguiente:
   - Compruebe si la máquina virtual está marcada para la resincronización.
   - Siga [estos pasos](https://techcommunity.microsoft.com/t5/virtualization/bg-p/Virtualization) para investigar el origen de la modificación.
   - La modificación puede producirse cuando los archivos de registro HRL superan el 50 % del espacio en disco disponible. Si este es el problema, aprovisione más espacio de almacenamiento para todas las máquinas virtuales en las que se produce el problema.
   - Compruebe que la replicación no está en pausa. Si es así, la replicación continúa escribiendo los cambios en el archivo hrl, lo que puede contribuir a su aumento de tamaño.
 

## <a name="critical-replication-state-issues"></a>Problemas de estado de replicación críticos

1. Para comprobar el estado de la replicación, conéctese a la consola del Administrador de Hyper-V local, seleccione la máquina virtual y vea el estado.

    ![Estado de la replicación](media/hyper-v-azure-troubleshoot/replication-health1.png)
    

2. Haga clic en **Ver mantenimiento de la replicación** para ver los detalles:

    - Si la replicación está en pausa, haga clic con el botón derecho en la máquina virtual > **Replicación** > **Reanudar replicación**.
    - Si una máquina virtual en un host de Hyper-V configurado en Site Recovery se migra a otro host de Hyper-V en el mismo clúster o a un equipo independiente, la replicación para la máquina virtual no resulta afectada. Compruebe que el nuevo host de Hyper-V cumpla todos los requisitos previos y se configure mediante Site Recovery.

## <a name="app-consistent-snapshot-issues"></a>Problemas de instantáneas coherentes con la aplicación

Una instantánea coherente con la aplicación es una instantánea en un momento dado de los datos de la aplicación dentro de la máquina virtual. El Servicio de instantáneas de volumen (VSS) garantiza que las aplicaciones en la máquina virtual se encuentre en un estado coherente cuando se toma la instantánea.  En esta sección se detallan algunos de los problemas comunes que puede experimentar.

### <a name="vss-failing-inside-the-vm"></a>Fallo de VSS en la máquina virtual

1. Asegúrese de que la versión más reciente de Integration Services está instalada y ejecutándose.  Compruebe si hay disponible una actualización ejecutando el comando siguiente desde un símbolo del sistema de PowerShell con privilegios elevados en el host de Hyper-V: **get-vm | select Name, State, IntegrationServicesState**.
2. Compruebe que los servicios VSS se están ejecutando y están en buen estado:
   - Para comprobar los servicios, inicie sesión en la VM de invitado. A continuación, abra un símbolo del sistema de administrador y ejecute los siguientes comandos para comprobar si todas las instancias de VSS Writer están en buenas condiciones.
       - **Vssadmin list writers**
       - **Vssadmin list shadows**
       - **Vssadmin list providers**
   - Compruebe los resultados. Si las instancias de VSS Writer se encuentran en estado erróneo, haga lo siguiente:
       - Compruebe el registro de eventos de aplicación en la máquina virtual para los errores de operación de VSS.
   - Pruebe a reiniciar estos servicios asociados con la instancia Writer que se encuentra en estado erróneo:
     - Instantáneas de volumen
       - Proveedor VSS de Azure Site Recovery
   - Después de hacer esto, espere un par de horas para ver si las instantáneas coherentes con la aplicación se generan correctamente.
   - Como último recurso reinicie la máquina virtual. Esto puede resolver servicios que no responden.
3. Compruebe que no tiene discos dinámicos en la máquina virtual. Las instantáneas consistentes con la aplicación no admiten esto. Puede comprobar en Administración de discos (diskmgmt.msc).

    ![Disco dinámico](media/hyper-v-azure-troubleshoot/dynamic-disk.png)
    
4. Compruebe que no tiene un disco iSCSI asociado a la máquina virtual. No es una opción admitida.
5. Compruebe que el servicio de copia de seguridad está habilitado. Compruebe que está habilitada en **Configuración de Hyper-V** > **Integration Services**.
6. Asegúrese de que no hay conflictos con aplicaciones que estén realizando instantáneas VSS. Si varias aplicaciones están intentando realizar instantáneas VSS al mismo tiempo se pueden producir conflictos. Por ejemplo, si una aplicación de copia de seguridad está realizando instantáneas VSS cuando Site Recovery está programado por la directiva de replicación para realizar una instantánea.   
7. Compruebe si la máquina virtual está experimentando una tasa de modificación elevada:
    - Puede medir la tasa de cambio de datos diaria para las máquinas virtuales invitadas con los contadores de rendimiento en el host de Hyper-V. Para medir la tasa de cambio de datos, habilite el siguiente contador. Agregue un ejemplo de este valor a los discos de máquina virtual de 5 a 15 minutos, para obtener la renovación de la máquina virtual.
        - Categoría: "Dispositivo de almacenamiento virtual de Hyper-V"
        - Contador: "Bytes de escritura/s"</br>
        - Esta tasa de modificación de datos aumentará o permanecer en un nivel alto, dependiendo de cómo de estén de ocupadas la máquina virtual o sus aplicaciones.
        - El promedio de modificación de datos del disco de origen es 2 MB/s de almacenamiento estándar para Site Recovery. [Más información](hyper-v-deployment-planner-analyze-report.md#azure-site-recovery-limits)
    - Además, puede [comprobar los objetivos de escalabilidad de Storage](../storage/common/scalability-targets-standard-account.md).
8. Asegúrese de que, si usa un servidor basado en Linux, tiene habilitada en él la coherencia de la aplicación. [Más información](./site-recovery-faq.md#replication)
9. Ejecute [Deployment Planner](hyper-v-deployment-planner-run.md).
10. Revise las recomendaciones para [red](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input) y [almacenamiento](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input).


### <a name="vss-failing-inside-the-hyper-v-host"></a>Fallo de VSS en el host de Hyper-V

1. Busque en los registros de eventos errores VSS y recomendaciones:
    - En el servidor host de Hyper-V, abra el registro de eventos de administrador de Hyper-V en **Visor de eventos** > **Registros de aplicaciones y servicios** > **Microsoft** > **Windows** > **Hyper-V** > **Administrador**.
    - Compruebe si existen eventos que indiquen errores de instantánea coherentes con la aplicación.
    - Un error habitual es: "Hyper-V failed to generate VSS snapshot set for virtual machine: More data is available. The writer experienced a non-transient error (Hyper-V no pudo generar el conjunto de instantáneas VSS para la máquina virtual: hay más datos disponibles. El escritor ha experimentado un error transitorio). Restarting the VSS service might resolve issues if the service is unresponsive". (Hyper-V no ha podido generar el conjunto de instantáneas VSS para la máquina virtual "XYZ": El escritor ha experimentado un error no transitorio. Reiniciar el servicio VSS puede resolver el problema si el servicio no responde)

2. Para generar instantáneas VSS para la máquina virtual, compruebe que los servicios de integración de Hyper-V están instalados en la máquina virtual y que está habilitado el servicio de integración de copia de seguridad (VSS).
    - Asegúrese de que los servicios/demonios de VSS de Integration Services se ejecutan en el invitado y se encuentran en un estado **Aceptar**.
    - Para comprobarlo, inicie una sesión de PowerShell con privilegios elevados en el host de Hyper-V con el comando **Get-VMIntegrationService -VMName\<VMName>-nombre VSS**. Otra manera de obtener esta información es iniciar sesión en la máquina virtual invitada. [Más información](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services).
    - Asegúrese de que los servicios de integración de VSS/copia de seguridad en la máquina virtual se están ejecutando y se encuentran en buen estado. Si no es así, reinicie estos servicios y el servicio de solicitante de instantáneas de volumen de Hyper-V en el servidor host de Hyper-V.

### <a name="common-errors"></a>Errores comunes

**Código de error** | **Mensaje** | **Detalles**
--- | --- | ---
**0x800700EA** | "Hyper-V failed to generate VSS snapshot set for virtual machine: More data is available (Hyper-V no pudo generar el conjunto de instantáneas de VSS para la máquina virtual: hay más datos disponibles). (0x800700EA). VSS snapshot set generation can fail if backup operation is in progress. Replication operation for virtual machine failed: More data is available" (La generación de un conjunto de instantáneas VSS puede producir un error si la operación de copia de seguridad está en curso).<br/><br/> Replication operation for virtual machine failed: More data is available" (Error en la operación de replicación de la máquina virtual: hay más datos disponibles) | Compruebe si la máquina virtual tiene habilitado un disco dinámico. No es una opción admitida.
**0x80070032** | "Hyper-V Volume Shadow Copy Requestor failed to connect to virtual machine <./VMname> because the version does not match the version expected by Hyper-V" (El solicitante de instantáneas de volumen de Hyper-V no ha podido conectar con la máquina virtual <./VMname> porque la versión no coincide con la versión esperada por Hyper-V) | Compruebe que las actualizaciones más recientes de Windows están instaladas.<br/><br/> [Actualice](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#keep-integration-services-up-to-date) a la versión más reciente de Integration Services.



## <a name="collect-replication-logs"></a>Recopilación de registros de replicación

Todos los eventos de replicación de Hyper-V se registran en el registro Hyper-V-VMMS\Admin ubicado en **Registros de aplicaciones y servicios** > **Microsoft** > **Windows**. Además, puede habilitar un registro analítico para el servicio de administración de máquinas virtuales de Hyper-V de la forma siguiente:

1. Asegúrese de que los registros analíticos y de depuración puedan verse en el Visor de eventos. Para que los registros estén disponibles, en el Visor de eventos, haga clic en **Vista** > **Mostrar registros analíticos y de depuración**. Puede verse el registro analítico en **Hyper-V-VMMS**.
2. En el panel **Acciones**, haga clic en **Habilitar registro**. 

    ![Habilitar registro](media/hyper-v-azure-troubleshoot/enable-log.png)
    
3. Una vez habilitado, aparece en **Monitor de rendimiento** como una **Sesión de seguimiento de eventos** en **Conjuntos de recopiladores de datos**. 
4. Para ver la información recopilada, deshabilite el registro para detener la sesión de seguimiento. Después guarde el registro y vuelva a abrirlo en el Visor de eventos, o use otras herramientas para convertirlo, según proceda.


### <a name="event-log-locations"></a>Ubicación de registros de eventos

**Registro de eventos** | **Detalles** |
--- | ---
**Registros de aplicaciones y servicios/Microsoft/VirtualMachineManager/Server/Admin** (Servidor VMM) | Registros para solucionar problemas de VMM.
**Registros de aplicaciones y servicios/MicrosoftAzureRecoveryServices/Replication** (Host de Hyper-V) | Registros para solucionar problemas del agente de Microsoft Azure Recovery Services. 
**Registros de aplicaciones y servicios/Microsoft/Azure Site Recovery/Provider/Operational** (Host de Hyper-V)| Registros para solucionar problemas del servicio de Microsoft Azure Site Recovery.
**Registros de aplicaciones y servicios/Microsoft/Windows/Hyper-V-VMMS/Admin** (Host de Hyper-V) | Registros para solucionar problemas de administración de máquinas virtuales de Hyper-V.

### <a name="log-collection-for-advanced-troubleshooting"></a>Recopilación de registros para solución de problemas avanzada

Estas herramientas pueden ayudar a la solución de problemas avanzada:

-   Para VMM, realice la recopilación de registros de Site Recovery mediante la [herramienta de la plataforma de diagnóstico de soporte técnico (SDP)](https://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx).
-   Para Hyper-V sin VMM [descargue esta herramienta](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab) y ejecútela en el host de Hyper-V para recopilar los registros.
