---
title: Introducción a Update Management en Azure Automation
description: En este artículo se ofrece información general de la característica Update Management que implementa las actualizaciones de las máquinas Windows y Linux.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: ea568acdf356064dc603d30ffba7df3c02bde621
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449771"
---
# <a name="update-management-overview"></a>Introducción a Update Management

Puede usar Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas Windows y Linux de Azure, en entornos locales y en otros entornos en la nube. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todas las máquinas agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores.

Puede habilitar Update Management en máquinas virtuales de las siguientes maneras:

* Desde la [cuenta de Azure Automation](update-mgmt-enable-automation-account.md) para una o varias máquinas de Azure.
* Manualmente para máquinas que no son de Azure.
* Para una sola máquina virtual de Azure, desde la página de la máquina virtual en Azure Portal. Este escenario está disponible para las VM [Linux](../../virtual-machines/linux/tutorial-config-management.md#enable-update-management) y [Windows](../../virtual-machines/windows/tutorial-config-management.md#enable-update-management).
* Para [varias máquinas virtuales de Azure](update-mgmt-enable-portal.md), selecciónelas en la página Máquinas virtuales de Azure Portal.

> [!NOTE]
> Update Management requiere vincular un área de trabajo de Log Analytics a la cuenta de Automation. Para ver una lista definitiva de regiones admitidas, consulte [Asignaciones de áreas de trabajo](../how-to/region-mappings.md). Las asignaciones de regiones no afectan a la capacidad de administrar VM en una región independiente de la cuenta de Automation.

Hay disponible una [plantilla de Azure Resource Manager](update-mgmt-enable-template.md) para ayudarle a implementar Update Management en una cuenta de Automation nueva o existente, y en el área de trabajo de Log Analytics de la suscripción.

> [!NOTE]
> No se puede usar una máquina configurada con Update Management para ejecutar scripts personalizados desde Azure Automation. Esta máquina solo puede ejecutar el script de actualización con firma de Microsoft.

## <a name="about-update-management"></a>Acerca de Update Management

Las máquinas administradas por Update Management usan las siguientes configuraciones para realizar valoraciones y actualizar implementaciones:

* Agente de Log Analytics para Windows o Linux
* Extensión DSC (configuración de estado deseado) de PowerShell para Linux
* Hybrid Runbook Worker de Automation
* Microsoft Update o Windows Server Update Services (WSUS) para máquinas Windows

El siguiente diagrama muestra cómo Update Management evalúa y aplica las actualizaciones de seguridad a todas las máquinas Windows Server y Linux conectadas de un área de trabajo:

![Flujo de trabajo de Update Management](./media/update-mgmt-overview/update-mgmt-updateworkflow.png)

Se puede utilizar Update Management para implementar máquinas de forma nativa en varias suscripciones en el mismo inquilino.

Después de publicarse un paquete, la revisión tarda de 2 a 3 horas en aparecer en las máquinas Linux para su evaluación. Para las máquinas Windows, la revisión tarda de 12 a 15 horas en aparecer para su evaluación tras su publicación.

Después de que una máquina completa un examen de cumplimiento de las actualizaciones, el agente reenvía la información de forma masiva a los registros de Azure Monitor. En una máquina Windows, el examen de cumplimiento se ejecuta cada 12 horas de forma predeterminada.

Además del examen programado, el examen de cumplimiento de las actualizaciones se inicia a los 15 minutos del reinicio del agente de Log Analytics, así como antes y después de la instalación de actualizaciones.

En una máquina Linux, el examen de cumplimiento se realiza cada hora de manera predeterminada. Si se reinicia el agente de Log Analytics, se inicia un examen de cumplimiento al cabo de 15 minutos.

Update Management informa del grado de actualización de la máquina en función del origen configurado para la sincronización. Si la máquina Windows está configurada para informar a WSUS, en función de cuándo WSUS se sincronizó por última vez con Microsoft Update, los resultados pueden diferir de lo que se muestra en Microsoft Update. Este comportamiento es el mismo para las máquinas Linux que están configuradas para informar a un repositorio local en lugar de a uno público.

> [!NOTE]
> Para informar correctamente al servicio, Update Management requiere la habilitación de ciertas direcciones URL y puertos. Para más información sobre estos requisitos, consulte [Configuración de red](../automation-hybrid-runbook-worker.md#network-planning).

Puede implementar e instalar las actualizaciones de software en las máquinas que requieren las actualizaciones mediante la creación de una implementación programada. Las actualizaciones clasificadas como opcionales no se incluyen en el ámbito de implementación en máquinas Windows. Solo se incluyen las actualizaciones necesarias.

La implementación programada define qué máquina de destino reciben las actualizaciones aplicables. Para ello, se especifican de forma explícita determinadas máquinas o se selecciona un [grupo de equipos](../../azure-monitor/platform/computer-groups.md) que se basa en las búsquedas de registros de un conjunto determinado de máquinas (o en una [consulta de Azure]update-mgmt-view-logs.md) que selecciona de forma dinámica las máquinas virtuales de Azure en función de los criterios especificados). Estos grupos difieren de la [configuración de ámbito](../../azure-monitor/insights/solution-targeting.md), que se usa para controlar los destinos de las máquinas que reciben la configuración para habilitar Update Management. Esto evita que realicen la comprobación de actualizaciones e informen sobre la misma, además de instalar las actualizaciones necesarias aprobadas.

Al definir una implementación, también se especifica una programación para aprobar y establecer un período de tiempo durante el que se pueden instalar actualizaciones. Este período se denomina ventana de mantenimiento. Un intervalo de 20 minutos de la ventana de mantenimiento se reserva para los reinicios, suponiendo que sea necesario reiniciar y que haya seleccionado la opción de reinicio adecuada. Si la aplicación de revisiones tarda más de lo esperado y la ventana de mantenimiento dura menos de 20 minutos, no se producirá un reinicio.

Los Runbooks instalan las actualizaciones en Azure Automation. No puede ver estos runbooks, que no requieren ninguna configuración. Cuando se crea una implementación de actualizaciones, esta crea una programación que inicia un runbook de actualización maestro a la hora especificada para las máquinas incluidas. El runbook maestro inicia un runbook secundario en cada agente para instalar las actualizaciones necesarias.

En la fecha y hora especificadas en la implementación de actualizaciones, las máquinas de destino ejecutan la implementación en paralelo. Antes de la instalación, se ejecuta un examen para comprobar que las actualizaciones siguen siendo necesarias. En las máquinas cliente de WSUS, si no se aprueban las actualizaciones en WSUS, se produce un error en la implementación de actualizaciones.

No se permite tener registrada una máquina para Update Management en más de un área de trabajo de Log Analytics (también conocido como hospedaje múltiple).

## <a name="clients"></a>Clientes

### <a name="supported-client-types"></a>Tipos de cliente admitidos

En la tabla siguiente se muestra una lista de sistemas operativos compatibles para evaluaciones de actualizaciones y parches. La aplicación de revisiones requiere Hybrid Runbook Worker. Para obtener información sobre los requisitos de Hybrid Runbook Worker, consulte [Implementación de Hybrid Runbook Worker en Windows](../automation-windows-hrw-install.md) e [Implementación de Hybrid Runbook Worker en Linux](../automation-linux-hrw-install.md).

> [!NOTE]
> La evaluación de la actualización de máquinas Linux solo se admite en determinadas regiones tal como se muestra en la [tabla de asignaciones](../how-to/region-mappings.md#supported-mappings) del área de trabajo de Log Analytics y la cuenta de Automation. 

|Sistema operativo  |Notas  |
|---------|---------|
|Windows Server 2019 (Datacenter/Datacenter Core/Standard)<br><br>Windows Server 2016 (Datacenter/Datacenter Core/Standard)<br><br>Windows Server 2012 R2 (Datacenter/Standard)<br><br>Windows Server 2012 ||
|Windows Server 2008 R2 (RTM and SP1 Standard)| Update Management admite evaluaciones y parches para este sistema operativo. [Hybrid Runbook Worker](../automation-windows-hrw-install.md) es compatible con Windows Server 2008 R2. |
|CentOS 6 (x86/x64) y 7 (x64)      | Los agentes de Linux requieren acceso a un repositorio de actualización. La aplicación de revisiones basada en la clasificación requiere `yum` para devolver los datos de seguridad que CentOS no tiene en sus versiones RTM. Para más información sobre la aplicación de revisiones basadas en clasificaciones en CentOS, consulte [Actualización de clasificaciones en Linux](update-mgmt-view-update-assessments.md#linux).          |
|Red Hat Enterprise (x86/x64) 6 y 7 (x64)     | Los agentes de Linux requieren acceso a un repositorio de actualización.        |
|SUSE Linux Enterprise Server 11 (x86/x64) y 12 (x64)     | Los agentes de Linux requieren acceso a un repositorio de actualización.        |
|Ubuntu 14.04 LTS, 16.04 LTS y 18.04 (x86/x64)      |Los agentes de Linux requieren acceso a un repositorio de actualización.         |

> [!NOTE]
> Los conjuntos de escalado de máquinas virtuales de Azure se pueden administrar a través de Update Management. Update Management funciona en las propias instancias y no en la imagen base. Deberá programar las actualizaciones de forma incremental, de modo que no todas las instancias de VM se actualicen a la vez. Puede agregar nodos para conjuntos de escalado de máquinas virtuales siguiendo los pasos que se describen en [Incorporación de una máquina que no es de Azure a la característica Change Tracking e Inventario](../automation-tutorial-installed-software.md#add-a-non-azure-machine-to-change-tracking-and-inventory).

### <a name="unsupported-client-types"></a>Tipos de cliente no admitidos

En la tabla siguiente se enumeran los sistemas operativos no admitidos:

|Sistema operativo  |Notas  |
|---------|---------|
|Cliente Windows     | No se admiten sistemas operativos cliente (por ejemplo, Windows 7 y Windows 10).<br> En el caso de Windows Virtual Desktop(WVD) de Azure, el método recomendado<br> para administrar las actualizaciones es usar [Microsoft Endpoint Configuration Manager](../../virtual-desktop/configure-automatic-updates.md) para la administración de revisiones de máquinas cliente de Windows 10. |
|Windows Server 2016 Nano Server     | No compatible.       |
|Nodos de Azure Kubernetes Service | No compatible. Use el proceso de aplicación de revisiones que se describe en [Aplicación de actualizaciones de kernel y seguridad en los nodos de Linux en Azure Kubernetes Service (AKS)](../../aks/node-updates-kured.md)|

### <a name="client-requirements"></a>Requisitos del cliente

La información siguiente describe los requisitos del cliente específicos del sistema operativo. Para obtener más instrucciones, vea [Planeamiento de red](#ports). Para comprender los requisitos de cliente para TLS 1.2, consulte el artículo sobre el [cumplimiento de TLS 1.2 para Azure Automation](../automation-managing-data.md#tls-12-enforcement-for-azure-automation).

#### <a name="windows"></a>Windows

Los agentes de Windows deben estar configurados para comunicarse con un servidor de WSUS o requieren acceso a Microsoft Update. Para más información sobre cómo instalar el agente de Log Analytics para Windows, consulte [Conexión de equipos Windows a Azure Monitor](../../azure-monitor/platform/agent-windows.md).

Puede usar Update Management con Microsoft Endpoint Configuration Manager. Para más información sobre escenarios de integración, consulte este artículo sobre la [integración de Update Management con Endpoint Configuration Manager de Windows](update-mgmt-mecmintegration.md). El [agente de Log Analytics para Windows](../../azure-monitor/platform/agent-windows.md) es necesario para los servidores Windows administrados por sitios en su entorno de Configuration Manager. 

De forma predeterminada, las VM Windows implementadas desde Azure Marketplace se establecen para recibir actualizaciones automáticas del servicio de Windows Update. Este comportamiento no cambia cuando se agregan VM Windows al área de trabajo. Si no administra activamente las actualizaciones mediante Update Management, se aplica el comportamiento predeterminado (las actualizaciones se aplican automáticamente).

> [!NOTE]
> Puede modificar la directiva de grupo para que los reinicios de la máquina solo los pueda realizar el usuario, no el sistema. Es posible que las máquinas administradas se bloqueen si Update Management no tiene derechos para reiniciar la máquina sin la intervención manual del usuario. Para obtener más información, consulte el tema sobre la [configuración de la directiva de grupo para actualizaciones automáticas](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates).

#### <a name="linux"></a>Linux

En Linux, la máquina requiere acceso a un repositorio de actualización, privado o público. Se requiere TLS 1.1 o TLS 1.2 para interactuar con Update Management. Update Management no admite un agente de Log Analytics para Linux que esté configurado para informar a varias áreas de trabajo de Log Analytics. La máquina también debe tener instalado Python 2.x.

> [!NOTE]
> La evaluación de la actualización de máquinas Linux solo se admite en determinadas regiones. Consulte la [tabla de asignaciones](../how-to/region-mappings.md#supported-mappings) del área de trabajo de Log Analytics y la cuenta de Automation.

Para más información sobre cómo instalar el agente de Log Analytics para Linux y descargar la versión más reciente, consulte [Agente de Log Analytics para Linux](../../azure-monitor/platform/agent-linux.md).

Las VM que se crearon a partir de las imágenes a petición de Red Hat Enterprise Linux (RHEL) que están disponibles en Azure Marketplace están registradas para acceder a la instancia de [Red Hat Update Infrastructure (RHUI)](../../virtual-machines/workloads/redhat/redhat-rhui.md) implementada en Azure. Cualquier otra distribución de Linux se debe actualizar desde el repositorio de archivos en línea de la distribución mediante los métodos que admite esta.

## <a name="permissions"></a>Permisos

Para crear y administrar implementaciones de actualizaciones, necesita permisos concretos. Para más información sobre estos permisos, consulte [Acceso basado en rol: Update Management](../automation-role-based-access-control.md#update-management-permissions).

## <a name="update-management-components"></a>Componentes de Update Management

Update Management usa los recursos descritos en esta sección. Estos recursos se agregan automáticamente a la cuenta de Automation al habilitar Update Management.

### <a name="hybrid-runbook-worker-groups"></a>Grupos de Trabajos híbridos de runbook

Después de habilitar Update Management, las máquinas Windows conectadas directamente al área de trabajo de Log Analytics se configuran automáticamente como Hybrid Runbook Worker para admitir los runbooks que admiten a su vez Update Management.

Cada máquina Windows administrada por Update Management se muestra en el panel Grupos de Hybrid Worker como un grupo Hybrid Worker del sistema para la cuenta de Automation. Los grupos usan la convención de nomenclatura `Hostname FQDN_GUID`. No puede usar estos grupos como destino con runbooks de su cuenta. Si lo intenta, se producirá un error. Estos grupos están diseñados únicamente para admitir Update Management.

Puede agregar la máquina Windows a un grupo de Hybrid Runbook Worker en la cuenta de Automation para admitir runbooks de Automation si usa la misma cuenta para Update Management y la pertenencia a grupos de Hybrid Runbook Worker. Esta funcionalidad se agregó en la versión 7.2.12024.0 de Hybrid Runbook Worker.

### <a name="management-packs"></a>Módulos de administración

Si el grupo de administración de Operations Manager está [conectado a un área de trabajo de Log Analytics](../../azure-monitor/platform/om-agents.md), se instalarán los siguientes módulos de administración en Operations Manager. Estos módulos de administración también se instalan para Update Management en las máquinas Windows conectadas directamente. No es necesario configurar ni administrar dichos módulos.

* Intelligence Pack Update Assessment de Microsoft System Center Advisor (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* Módulo de administración de Update Deployment

> [!NOTE]
> Si tiene un grupo de administración de Operations Manager 1807 o 2019 conectado a un área de trabajo de Log Analytics con agentes configurados en el grupo de administración para recopilar los datos de registro, debe invalidar el parámetro `IsAutoRegistrationEnabled` y establecerlo en True en la regla **Microsoft.IntelligencePacks.AzureAutomation.HybridAgent.Init**.

Para más información sobre las actualizaciones de los módulos de administración, consulte [Conexión de Operations Manager con registros de Azure Monitor](../../azure-monitor/platform/om-agents.md).

> [!NOTE]
> Para que Update Management administre totalmente las máquinas que tengan el agente de Log Analytics, debe actualizar al agente de Log Analytics para Windows o Linux. Para aprender a actualizar el agente, consulte [Actualización de Operations Manager Agent](/system-center/scom/deploy-upgrade-agents). En entornos que usan Operations Manager, debe ejecutar System Center Operations Manager 2012 R2 UR 14 o posterior.

## <a name="data-collection"></a>datos, recopilación

### <a name="supported-sources"></a>Orígenes compatibles

En la tabla siguiente se describen los orígenes conectados compatibles con Update Management:

| Origen conectado | Compatible | Descripción |
| --- | --- | --- |
| Agentes de Windows |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de Windows e inicia la instalación de las actualizaciones necesarias. |
| Agentes de Linux |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de Linux e inicia la instalación de las actualizaciones necesarias en las distribuciones admitidas. |
| Grupo de administración de Operations Manager |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de un grupo de administración conectado.<br/><br/>No se requiere ninguna conexión directa entre el agente de Operations Manager y los registros de Azure Monitor. Los datos se reenvían desde el grupo de administración al área de trabajo de Log Analytics. |

### <a name="collection-frequency"></a>Frecuencia de recopilación

Update Management examina los datos de las máquinas administradas con las siguientes reglas. Puede que transcurran entre 30 minutos y 6 horas antes de que se muestren los datos actualizados de las máquinas administradas.

* Cada máquina Windows: Update Management realiza un examen dos veces al día de cada máquina. Cada 15 minutos, consulta la API de Windows para consultar la hora de la última actualización y determinar si ha cambiado el estado. Si el estado ha cambiado, Update Management inicia un examen de cumplimiento.

* Cada máquina Linux: Update Management realiza un examen cada hora.

El uso medio de datos de los registros de Azure Monitor para una máquina que utiliza Update Management es aproximadamente de 25 MB al mes. Este valor es solo una aproximación y está sujeto a cambios en función de su entorno. Se recomienda supervisar el entorno para realizar un seguimiento del uso exacto. Para más información sobre cómo analizar el uso de datos, consulte [Administración del uso y los costos](../../azure-monitor/platform/manage-cost-storage.md).

## <a name="network-planning"></a><a name="ports"></a>Planeamiento de red

Las direcciones siguientes se requieren específicamente para Update Management. La comunicación con estas direcciones se realiza a través del puerto 443.

|Azure Public  |Azure Government  |
|---------|---------|
|`*.ods.opinsights.azure.com`    | `*.ods.opinsights.azure.us`        |
|`*.oms.opinsights.azure.com`     | `*.oms.opinsights.azure.us`        |
|`*.blob.core.windows.net` | `*.blob.core.usgovcloudapi.net`|
|`*.azure-automation.net` | `*.azure-automation.us`|

Al crear reglas de seguridad de grupo de red o configurar Azure Firewall para permitir el tráfico en el servicio Automation y en el área de trabajo de Log Analytics, utilice la [etiqueta de servicio](../../virtual-network/service-tags-overview.md#available-service-tags) **GuestAndHybridManagement** y **AzureMonitor**. De esta forma, se simplifica la administración continua de las reglas de seguridad de red. Para conectarse al servicio Automation desde las máquinas virtuales de Azure de forma segura y privada, revise [Uso de Azure Private Link](../how-to/private-link-security.md). Para obtener información de intervalos y la etiqueta de servicio actual para incluirla como parte de las configuraciones del firewall local, consulte [Archivos JSON descargables](../../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

En el caso de máquinas Windows, también debe permitir el tráfico a los puntos de conexión que necesita Windows Update. Puede encontrar una lista actualizada de los puntos de conexión necesarios en [Problemas relacionados con HTTP/Proxy](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy). Si tiene un [servidor de Windows Update](/windows-server/administration/windows-server-update-services/plan/plan-your-wsus-deployment) local, también debe permitir el tráfico al servidor especificado en la [clave de WSUS](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry).

En el caso de máquinas de Red Hat Linux, consulte las [direcciones IP de los servidores de entrega de contenido de RHUI](../../virtual-machines/workloads/redhat/redhat-rhui.md#the-ips-for-the-rhui-content-delivery-servers) de los puntos de conexión necesarios. Para otras distribuciones de Linux, consulte la documentación del proveedor.

Para más información sobre los puertos que Hybrid Runbook Worker requiere, consulte [Direcciones de Update Management para Hybrid Runbook Worker](../automation-hybrid-runbook-worker.md#update-management-addresses-for-hybrid-runbook-worker).

Si las directivas de seguridad de TI no permiten que las máquinas de la red se conecten a Internet, puede configurar una [puerta de enlace de Log Analytics](../../azure-monitor/platform/gateway.md) y, luego, configurar la máquina para conectarse a Azure Monitor y a Azure Automation mediante la puerta de enlace.

## <a name="update-classifications"></a>Clasificaciones de actualizaciones

En la tabla siguiente se definen las clasificaciones que Update Management admite para las actualizaciones de Windows. 

|clasificación  |Descripción  |
|---------|---------|
|Actualizaciones críticas     | Actualización para un problema específico que resuelve un error crítico no relacionado con la seguridad.        |
|Actualizaciones de seguridad     | Actualización para un problema específico del producto relacionado con la seguridad.        |
|Paquetes acumulativos de actualizaciones     | Conjunto acumulativo de revisiones que se empaquetan para facilitar la implementación.        |
|Feature Packs     | Nuevas características del producto que se distribuyen fuera de una versión del producto.        |
|Service Packs     | Un conjunto acumulativo de revisiones que se aplican a una aplicación.        |
|Actualizaciones de definiciones     | Una actualización de archivos de definiciones de virus o de otra índole.        |
|Herramientas     | Utilidad o característica que ayuda a realizar una o más tareas.        |
|Actualizaciones     | Actualización de una aplicación o archivo que están instalados actualmente.        |

En la tabla siguiente se definen las clasificaciones admitidas para las actualizaciones de Linux.

|clasificación  |Descripción  |
|---------|---------|
|Actualizaciones críticas y de seguridad     | Actualizaciones para un problema específico o un problema de un producto específico relacionado con la seguridad.         |
|Otras actualizaciones     | Todas las demás actualizaciones que ni son críticas por naturaleza ni son actualizaciones de seguridad.        |

>[!NOTE]
>La clasificación de actualizaciones para máquinas Linux solo está disponible cuando se usa en las regiones de la nube pública de Azure admitidas. Al usar Update Management en las siguientes regiones de la nube nacional:
>* Azure US Government
>* 21Vianet en China
>
> No hay ninguna clasificación de las actualizaciones de Linux y se muestran en la categoría **Otras actualizaciones**. Update Management usa los datos publicados por las distribuciones admitidas, en concreto, sus archivos de [OVAL](https://oval.mitre.org/) (Open Vulnerability and Assessment Language). Dado que el acceso a Internet está restringido desde estas nubes nacionales, Update Management no puede tener acceso a estos archivos ni consumirlos.

En Linux, Update Management puede distinguir entre actualizaciones críticas y de seguridad en la nube y mostrar al mismo tiempo datos de evaluación debido al enriquecimiento de datos en la nube. Para aplicar revisiones, Update Management se basa en los datos de clasificación disponibles en la máquina. A diferencia de otras distribuciones, CentOS no dispone de esta información en la versión RTM. Si tiene máquinas de CentOS configuradas para devolver datos de seguridad para el siguiente comando, Update Management puede aplicar revisiones basadas en clasificaciones.

```bash
sudo yum -q --security check-update
```

Actualmente no hay ningún método compatible para habilitar la disponibilidad de datos de clasificación nativos en CentOS. En este momento, solo se proporciona soporte técnico a clientes que puedan haber habilitado esta característica por su cuenta.

Para clasificar las actualizaciones de la versión 6 de Red Hat Enterprise, debe instalar el complemento de seguridad de yum. En Red Hat Enterprise Linux 7, el complemento ya forma parte de yum y no es necesario instalar nada. Para más información, consulte el siguiente [artículo de conocimientos](https://access.redhat.com/solutions/10021) de Red Hat.

## <a name="integrate-update-management-with-configuration-manager"></a>Integración de Update Management con Configuration Manager.

Los clientes que han invertido en Microsoft Endpoint Configuration Manager para administrar equipos, servidores y dispositivos móviles también dependen de la fuerza y la madurez de Configuration Manager para que les ayude a administrar las actualizaciones de software. Para aprender a integrar Update Management con Configuration Manager, consulte este artículo sobre la [integración de Update Management con Endpoint Configuration Manager de Windows](update-mgmt-mecmintegration.md).

## <a name="third-party-updates-on-windows"></a>Actualizaciones de terceros en Windows

Update Management se basa en el repositorio de actualización configurado localmente para actualizar los sistemas Windows compatibles, ya sea WSUS o Windows Update. Herramientas como [System Center Updates Publisher](/configmgr/sum/tools/updates-publisher) le permiten importar y publicar actualizaciones personalizadas con WSUS. Este escenario permite que Update Management actualice las máquinas que usan Configuration Manager como repositorio de actualizaciones con software de terceros. Para obtener información sobre cómo configurar Updates Publisher, consulte [Instalar Updates Publisher](/configmgr/sum/tools/install-updates-publisher).

## <a name="enable-update-management"></a>Habilitación de la Administración de actualizaciones

Hay disponible una [plantilla de Azure Resource Manager](update-mgmt-enable-template.md) para ayudarle a implementar Update Management en una cuenta de Automation nueva o existente, y en el área de trabajo de Azure Monitor Log Analytics de la suscripción. No configura el ámbito de las máquinas que deben administrarse; esto se realiza como un paso independiente después de usar la plantilla.

Estas son las formas en que puede habilitar Update Management y seleccionar las máquinas que se van a administrar:

* [Desde una máquina virtual](update-mgmt-enable-vm.md)
* [Navegando desde diferentes máquinas](update-mgmt-enable-portal.md)
* [Desde una cuenta de Azure Automation](update-mgmt-enable-automation-account.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo trabajar con Update Management, consulte [Administración de actualizaciones para las máquinas virtuales](update-mgmt-manage-updates-for-vm.md).

* Revise las preguntas más frecuentes sobre Update Management en [Preguntas más frecuentes sobre Azure Automation](../automation-faq.md).
