---
title: Información general de Change Tracking e Inventario en Azure Automation
description: En este artículo se describe la característica Change Tracking e Inventario, que ayuda a identificar los cambios en el software y el servicio de Microsoft que se producen en su entorno.
services: automation
ms.subservice: change-inventory-management
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: 2fe92942e263cf53b9827ccbcb13a2d7bafc367c
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511055"
---
# <a name="change-tracking-and-inventory-overview"></a>Información general de Change Tracking e Inventario

En este artículo se presenta la solución Change Tracking e Inventario de Azure Automation. Esta característica realiza un seguimiento de los cambios en las máquinas virtuales y la infraestructura de servidor para ayudarle a identificar problemas operativos y ambientales con el software administrado por el administrador de paquetes de distribución. Los elementos cuyo seguimiento realiza Change Tracking e Inventario incluyen: 

- Software de Windows
- Software de Linux (paquetes)
- Archivos de Windows y Linux
- Clave del Registro de Windows
- Servicios de Microsoft
- Demonios de Linux

> [!NOTE]
> Para hacer el seguimiento de los cambios en las propiedades de Azure Resource Manager, consulte [Historial de cambios](../governance/resource-graph/how-to/get-resource-changes.md) de Azure Resource Graph.

Change Tracking e Inventario obtiene sus datos de Azure Monitor. Las máquinas virtuales conectadas a las áreas de trabajo de Log Analytics usan agentes de Log Analytics para recopilar datos sobre los cambios en el software instalado, los servicios de Microsoft, los archivos y el Registro de Windows, y demonios de Linux de los servidores supervisados. Cuando los datos están disponibles, los agentes los envían a Azure Monitor para su procesamiento. Azure Monitor aplica la lógica a los datos recibidos, los registra y hace que estén disponibles.

> [!NOTE]
> Change Tracking e Inventario requiere vincular un área de trabajo de Log Analytics a la cuenta de Automation. Para ver una lista definitiva de regiones admitidas, consulte [Asignaciones de áreas de trabajo](./how-to/region-mappings.md). Las asignaciones de regiones no afectan a la capacidad de administrar VM en una región independiente de la cuenta de Automation.

Actualmente, la solución Change Tracking e Inventario no admite los elementos siguientes:

- Recursión para el seguimiento del registro de Windows
- Sistemas de archivos de red
- Otros métodos de instalación
- Archivos * **.exe** para Windows

Otras limitaciones:

- Los valores y la columna **Tamaño máximo de archivos** no se usan en la implementación actual.
- Si recopila más de 2500 archivos en un ciclo de recopilación de 30 minutos, el rendimiento de Change Tracking e Inventario podría degradarse.
- Cuando el tráfico de red es elevado, los registros de cambios pueden tardar hasta seis horas en aparecer.
- Si modifica la configuración mientras un equipo está apagado, el equipo podría publicar cambios que pertenecían a la configuración anterior.

Actualmente, la solución Change Tracking e Inventario tiene los siguientes problemas:

- Las actualizaciones de revisiones no se recopilan en máquinas Windows Server 2016 Core RS3.

- Los demonios de Linux pueden mostrar un estado modificado aunque no se haya producido ningún cambio. Este problema se debe a la manera en que se capturan los datos de `SvcRunLevels` del registro [ConfigurationChange](/azure/azure-monitor/reference/tables/configurationchange) de Azure Monitor.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Change Tracking e Inventario se admite en todos los sistemas operativos que cumplen los requisitos del agente de Log Analytics. Las versiones de sistema operativo oficiales son Windows Server 2008 SP1 o posterior y Windows 7 SP1 o posterior. La característica también se admite en varios sistemas operativos Linux. Para obtener una lista de sistemas operativos compatibles, vea [Introducción al agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md).

Para comprender los requisitos de cliente para TLS 1.2, consulte el artículo sobre el [cumplimiento de TLS 1.2 para Azure Automation](automation-managing-data.md#tls-12-enforcement-for-azure-automation).

## <a name="network-requirements"></a>Requisitos de red

Change Tracking e Inventario requiere específicamente las direcciones de red que se enumeran en la tabla siguiente. Las comunicaciones con estas direcciones usan el puerto 443.

|Azure Public  |Azure Government  |
|---------|---------|
|*.ods.opinsights.azure.com    | *.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|*.blob.core.windows.net | *.blob.core.usgovcloudapi.net|
|*.azure-automation.net | *.azure-automation.us|

## <a name="change-tracking-and-inventory-user-interface"></a>Interfaz de usuario de Change Tracking e Inventario

Utilice Change Tracking e Inventario en Azure Portal para ver el resumen de cambios de los equipos supervisados. La característica está disponible al seleccionar una de las opciones de incorporación de máquinas virtuales para **Change Tracking** o **Inventario** en **Administración de configuración** en su cuenta de Automation.  

![Panel de Change Tracking](./media/change-tracking/change-tracking-dash01.png)

Hay disponibles listas desplegables en la parte superior del panel para limitar el gráfico y la información detallada de Change Tracking en función del tipo de cambio y los intervalos de tiempo. También puede hacer clic y arrastrar en el gráfico para seleccionar un intervalo de tiempo personalizado. 

Puede hacer clic en un cambio o un evento para mostrar sus detalles. Los tipos de cambios disponibles son:

- Eventos
- Demonios
- Archivos
- Registro
- Software
- Servicios de Microsoft

Puede agregar, modificar o quitar cualquier cambio. En el ejemplo siguiente se muestra un cambio en el tipo de inicio de un servicio de Manual a Automático.

![Detalles de Change Tracking e Inventario](./media/change-tracking/change-tracking-details.png)

## <a name="fim-support-in-azure-security-center"></a>Compatibilidad de FIM en Azure Security Center

Change Tracking e Inventario utiliza la característica [Supervisión de la integridad de los archivos (FIM) de Azure Security Center](../security-center/security-center-file-integrity-monitoring.md). Mientras que FIM solo supervisa archivos y Registros, la solución Change Tracking e Inventario completa también incluye:

- Cambios de software
- Servicios de Microsoft
- Demonios de Linux

> [!NOTE]
> La habilitación de la solución completa Change Tracking e Inventario puede provocar cargos adicionales. Consulte [Precios de Automation](https://azure.microsoft.com/pricing/details/automation/). Es posible eliminar FIM de la [lista de soluciones de supervisión instaladas](../azure-monitor/insights/solutions.md#list-installed-monitoring-solutions) disponibles en Azure Portal. Consulte [Eliminación de una solución de supervisión](../azure-monitor/insights/solutions.md#remove-a-monitoring-solution).

## <a name="tracking-of-file-changes"></a>Seguimiento de cambios en los archivos

Para realizar el seguimiento de los cambios en los archivos de Windows y Linux, Change Tracking e Inventario utiliza los valores hash MD5 de los archivos. La característica usa los valores hash para detectar si se han realizado cambios desde el último inventario.

## <a name="tracking-of-file-content-changes"></a>Seguimiento de cambios en el contenido de los archivos

Change Tracking e Inventario le permite ver el contenido de un archivo de Windows o Linux. Para cada cambio en un archivo, Change Tracking e Inventario almacena el contenido del archivo en una [cuenta de Azure Storage](../storage/common/storage-account-create.md). Cuando realiza el seguimiento de un archivo, puede ver su contenido antes o después de un cambio. El contenido del archivo se puede ver en línea o en paralelo. 

![Visualización de los cambios en un archivo](./media/change-tracking/view-file-changes.png)

## <a name="tracking-of-registry-keys"></a>Seguimiento de las claves del Registro

Change Tracking e Inventario permite la supervisión de los cambios en las claves del Registro de Windows. La supervisión permite identificar los puntos de extensibilidad en los que se puede activar malware y código de terceros. En la tabla siguiente se enumeran las claves del Registro preconfiguradas (pero no habilitadas). Para realizar el seguimiento de estas claves, debe habilitar cada una de ellas.

> [!div class="mx-tdBreakAll"]
> |Clave del Registro | Propósito |
> | --- | --- |
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup` | Supervisa los scripts que se ejecutan al arrancar.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown` | Supervisa los scripts que se ejecutan al apagar el equipo.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run` | Supervisa las claves que se cargan antes de que el usuario inicie sesión en la cuenta de Windows. La clave se usa para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Microsoft\Active Setup\Installed Components` | Supervisa los cambios en la configuración de la aplicación.
> |`HKEY\LOCAL\MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers` | Supervisa los controladores de los menús contextuales que se enlazan directamente con el Explorador de Windows y se suelen ejecutar en el proceso con **explorer.exe**.
> |`HKEY\LOCAL\MACHINE\Software\Classes\Directory\Shellex\CopyHookHandlers` | Supervisa los controladores de enlace de copia que se enlazan directamente con el Explorador de Windows y se suelen ejecutar en el proceso con **explorer.exe**.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers` | Supervisa el registro del controlador de superposición de iconos.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers` | Supervisa el registro del controlador de superposición de iconos para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects` | Supervisa si hay nuevos complementos de objetos auxiliares de explorador para Internet Explorer. Se utiliza para acceder a Document Object Model (DOM) de la página actual y controlar la navegación.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects` | Supervisa si hay nuevos complementos de objetos auxiliares de explorador para Internet Explorer. Se usa para acceder a Document Object Model (DOM) de la página actual y controlar la navegación de aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Internet Explorer\Extensions` | Supervisa si hay nuevas extensiones de Internet Explorer, como menús de la herramienta personalizada y botones de la barra de herramientas personalizada.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions` | Supervisa si hay nuevas extensiones de Internet Explorer, como menús de la herramienta personalizada y botones de la barra de herramientas personalizada para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Drivers32` | Supervisa los controladores de 32 bits asociados a wavemapper, wave1 y wave2, msacm.imaadpcm, .msadpcm, .msgsm610 y vidc. Es similar a la sección [drivers] del archivo **system.ini**.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32` | Supervisa los controladores de 32 bits asociados a wavemapper, wave1 y wave2, msacm.imaadpcm, .msadpcm, .msgsm610 y vidc para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits. Es similar a la sección [drivers] del archivo **system.ini**.
> |`HKEY\LOCAL\MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDlls` | Supervisa la lista de DDL del sistema conocidas o utilizadas habitualmente. La supervisión impide que las personas se aprovechen de permisos de directorio de aplicaciones débiles colocando versiones de troyanos en archivos DLL del sistema.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify` | Supervisa la lista de paquetes que pueden recibir notificaciones de eventos de **winlogon.exe**, el modelo de compatibilidad de inicio de sesión interactivo para Windows.

## <a name="recursion-support"></a>Compatibilidad con la recursión

Change Tracking e Inventory es compatible con la recursión, lo que permite especificar comodines para simplificar el seguimiento entre directorios. La recursión también proporciona variables de entorno que permiten realizar un seguimiento de los archivos en entornos con nombres de unidad varios o dinámicos. En la lista siguiente se incluye información común que debe conocer al configurar la recursión:

- Los caracteres comodín son necesarios para realizar el seguimiento de varios archivos.

- Puede usar caracteres comodín solo en el último segmento de la ruta de acceso de un archivo, como **c:\carpeta\\archivo*** o **/etc/*.conf**.

- Si una variable de entorno tiene una ruta de acceso no válida, la validación será correcta pero se producirá un error en dicha ruta durante la ejecución.

- Al establecer la ruta de acceso, debe evitar los nombres de ruta de acceso generales, ya que daría lugar a que se recorrieran demasiadas carpetas.

## <a name="change-tracking-and-inventory-data-collection"></a>Recopilación de datos de Change Tracking e Inventory

En la tabla siguiente se muestra la frecuencia de recopilación de datos para los tipos de cambios que admite Change Tracking e Inventario. Para cada tipo, la instantánea de datos del estado actual también se actualiza al menos cada 24 horas.

| **Tipo de cambio** | **Frecuencia** |
| --- | --- |
| Registro de Windows | 50 minutos |
| Archivo de Windows | 30 minutos |
| Archivo de Linux | 15 minutos |
| Servicios de Microsoft | 10 segundos hasta 30 minutos</br> Valor predeterminado: 30 minutos |
| Demonios de Linux | 5 minutos |
| Software de Windows | 30 minutos |
| Software Linux | 5 minutos |

En la siguiente tabla se muestran los límites de elementos sometidos a seguimiento por máquina para Change Tracking e Inventario.

| **Recurso** | **Límite** |
|---|---|---|
|Archivo|500|
|Registro|250|
|Software de Windows (sin incluir revisiones) |250|
|Paquetes Linux|1250|
|Servicios|250|
|Demonios|250|

El uso medio de datos de Log Analytics para una máquina con Change Tracking e Inventario es aproximadamente de 40 MB al mes, según su entorno. Con la característica Uso y costos estimados del área de trabajo de Log Analytics, puede ver los datos ingeridos por Change Tracking e Inventario en un gráfico de uso. Use esta vista de datos para evaluar el uso de los datos y determinar cómo afecta a su factura. Consulte [Información útil del uso y los costos estimados](../azure-monitor/platform/manage-cost-storage.md#understand-your-usage-and-estimate-costs).

### <a name="microsoft-service-data"></a>Datos de servicio de Microsoft

La frecuencia de recopilación predeterminada para los servicios de Microsoft es de 30 minutos. Puede configurar la frecuencia con un control deslizante en la pestaña **Servicios de Microsoft** bajo **Editar configuración**.

![Control deslizante de Servicios de Microsoft](./media/change-tracking/windowservices.png)

Para optimizar el rendimiento, el agente de Log Analytics solo realiza el seguimiento de los cambios. Al establecer un umbral alto, se pueden perder los cambios si el servicio se revierte a su estado original. Si la frecuencia se establece en un valor menor, es posible detectar los cambios que de otra forma podrían pasar desapercibidos.

> [!NOTE]
> Aunque el agente puede seguir los cambios hasta en intervalos de 10 segundos, los datos todavía tardan unos minutos en mostrarse en Azure Portal. Durante el tiempo que lleva la visualización en el portal, se continúa realizando el seguimiento y registro de los cambios que se producen.

## <a name="support-for-alerts-on-configuration-state"></a>Compatibilidad con alertas sobre el estado de configuración

Una funcionalidad clave de Change Tracking e Inventario son las alertas sobre los cambios en el estado de configuración de su entorno híbrido. Existen muchas acciones útiles disponibles para desencadenar en respuesta a las alertas, como las acciones en Azure Functions, runbooks de Automation, webhooks y similares. Las alertas sobre los cambios en el archivo **c:\windows\system32\drivers\etc\hosts** de una máquina es una buena aplicación de alertas para los datos de Change Tracking e Inventario. También hay muchos más escenarios para las alertas, incluidos los escenarios de consulta que se definen en la tabla siguiente.

|Consultar  |Descripción  |
|---------|---------|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Files" and FileSystemPath contains " c:\\windows\\system32\\drivers\\"|Útil para el seguimiento de cambios en archivos críticos del sistema.|
|ConfigurationChange <br>&#124; where FieldsChanged contains "FileContentChecksum" and FileSystemPath == "c:\\windows\\system32\\drivers\\etc\\hosts"|Útil para el seguimiento de las modificaciones de los archivos de configuración de claves.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "WindowsServices" and SvcName contains "w3svc" and SvcState == "Stopped"|Útil para el seguimiento de cambios en servicios críticos del sistema.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Daemons" and SvcName contains "ssh" and SvcState!= "Running"|Útil para el seguimiento de cambios en servicios críticos del sistema.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Software" and ChangeCategory == "Added"|Útil para entornos que necesitan configuraciones de software bloqueadas.|
|ConfigurationData <br>&#124; where SoftwareName contains "Monitoring Agent" and CurrentVersion!= "8.0.11081.0"|Útil para ver qué máquinas tienen instalada una versión de software obsoleta o incompatible. Esta consulta notifica el último estado de configuración notificado, no los cambios.|
|ConfigurationChange <br>&#124; where RegistryKey == @"HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\QualityCompat"| Útil para el seguimiento de los cambios en claves de antivirus fundamentales.|
|ConfigurationChange <br>&#124; where RegistryKey contains @"HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\SharedAccess\\Parameters\\FirewallPolicy"| Útil para el seguimiento de los cambios en la configuración del firewall.|

## <a name="next-steps"></a>Pasos siguientes

- Para habilitar la característica desde una cuenta de Automation, vea [Habilitación de Change Tracking e Inventario desde una cuenta de Automation](automation-enable-changes-from-auto-acct.md).

- Para habilitar la característica a través de Azure Portal, consulte [Habilitación de Change Tracking e Inventario desde Azure Portal](automation-enable-changes-from-browse.md).

- Para habilitar la característica desde un runbook, vea [Habilitación de Change Tracking e Inventario desde un runbook](automation-enable-changes-from-runbook.md).

- Para habilitar la característica desde una máquina virtual de Azure, vea [Habilitación de Change Tracking e Inventario desde una máquina virtual de Azure](automation-enable-changes-from-vm.md).
