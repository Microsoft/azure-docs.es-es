---
title: Instalación y configuración de la extensión de Azure Diagnostics (WAD) para Windows
description: Obtenga información sobre cómo recopilar datos de diagnóstico de Azure en una cuenta de Azure Storage para que pueda verlos con una de las diversas herramientas disponibles.
services: azure-monitor
author: bwren
ms.subservice: diagnostic-extension
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: bwren
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: ac087a7ba241534c08c4e5737973861727ab01ca
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069585"
---
# <a name="install-and-configure-windows-azure-diagnostics-extension-wad"></a>Instalación y configuración de la extensión de Azure Diagnostics (WAD) para Windows
[Azure Diagnostics Extension](diagnostics-extension-overview.md) es un agente de Azure Monitor que recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure y de otros recursos de proceso. En este artículo se proporcionan detalles sobre cómo instalar y configurar la extensión de diagnósticos para Windows y una descripción de cómo se almacenan los datos en la cuenta de Azure Storage.

La extensión de diagnósticos se implementa como una [extensión de máquina virtual](../../virtual-machines/extensions/overview.md) en Azure, por lo que admite las mismas opciones de instalación mediante las plantillas de Resource Manager, PowerShell y la CLI. Consulte [Características y extensiones de las máquinas virtuales para Windows](../../virtual-machines/extensions/features-windows.md) para más información sobre la instalación y el mantenimiento de extensiones de máquina virtual.

## <a name="overview"></a>Información general
Al configurar la extensión de diagnósticos de Windows Azure, debe especificar una cuenta de almacenamiento a la que se enviarán todos los datos especificados. Opcionalmente, puede agregar uno o más *receptores de datos* con el fin de enviar los datos a ubicaciones diferentes.

- Receptor de Azure Monitor: envíe datos de rendimiento de invitados a métricas de Azure Monitor.
- Receptor de centros de eventos: envíe datos de registro y de rendimiento de invitados a centros de eventos de Azure para reenviarlos fuera de Azure. No se puede configurar este receptor en Azure Portal.


## <a name="install-with-azure-portal"></a>Instalación con Azure Portal
Puede instalar y configurar la extensión de diagnósticos en una máquina virtual de Azure Portal que proporcione una interfaz en lugar de trabajar directamente con la configuración. Al habilitar la extensión de diagnósticos, se usará automáticamente una configuración predeterminada con los eventos y contadores de rendimiento más comunes. Esta configuración predeterminada se puede modificar de acuerdo con sus requisitos específicos.

> [!NOTE]
> A continuación se describe la configuración más común de la extensión de diagnósticos. Para obtener más información sobre la configuración, vea [Esquema de Windows Diagnostic Extension](diagnostics-extension-schema-windows.md).

1. Abra el menú de una máquina virtual en Azure Portal.

2. Haga clic en **Configuración de diagnóstico** en la sección **Supervisión** del menú de la máquina virtual.

3. Haga clic en **Habilitar supervisión a nivel de invitado** si aún no se ha habilitado la extensión de diagnósticos.

   ![Habilitar supervisión](media/diagnostics-extension-windows-install/enable-monitoring.png)

4. Se crea una cuenta de Azure Storage para la máquina virtual, donde el nombre se basa en el del grupo de recursos de la máquina virtual, y se seleccionará un conjunto predeterminado de contadores y registros de rendimiento de invitado.

   ![Configuración de diagnóstico](media/diagnostics-extension-windows-install/diagnostic-settings.png)

5. En la pestaña **Contadores de rendimiento**, seleccione las métricas de invitado que desea recopilar de esta máquina virtual. Use la opción **Personalizado** para realizar una selección más avanzada.

   ![Contadores de rendimiento](media/diagnostics-extension-windows-install/performance-counters.png)

6. En la pestaña **Registros**, seleccione los registros que se van a recopilar de la máquina virtual. Los registros se pueden enviar a centro de eventos o almacenamiento, pero no a Azure Monitor. Use el [agente de Log Analytics](log-analytics-agent.md) para recopilar los registros de invitados a Azure Monitor.

   ![Registros](media/diagnostics-extension-windows-install/logs.png)

7. En la pestaña **Volcados de memoria**, especifique cualquier proceso para recopilar volcados de memoria después de un bloqueo. Los datos se escribirán en la cuenta de almacenamiento para la configuración de diagnóstico y, opcionalmente, puede especificar un contenedor de blobs.

   ![Volcados de memoria](media/diagnostics-extension-windows-install/crash-dumps.png)

8. En la pestaña **Receptores**, especifique si desea enviar los datos a ubicaciones distintas de Azure Storage. Si selecciona **Azure Monitor**, se enviarán los datos de rendimiento de los invitados a las métricas de Azure Monitor. No se puede configurar el receptor de centros de eventos mediante Azure Portal.

   ![Receptores](media/diagnostics-extension-windows-install/sinks.png)
   
   Si no ha habilitado una identidad asignada por el sistema configurada para la máquina virtual, es posible que vea la siguiente advertencia al guardar una configuración con el receptor de Azure Monitor. Haga clic en el banner para habilitar la identidad asignada por el sistema.
   
   ![Entidad administrada](media/diagnostics-extension-windows-install/managed-entity.png)

9. En **Agente**, puede cambiar la cuenta de almacenamiento, establecer la cuota de disco y especificar si desea recopilar registros de infraestructura de diagnóstico.  

   ![Agente](media/diagnostics-extension-windows-install/agent.png)

10. Para guardar la configuración, haga clic en **Guardar**. 

> [!NOTE]
> Aunque se puede dar formato a la configuración de la extensión de diagnósticos en JSON o XML, cualquier configuración realizada en Azure Portal se almacenará siempre como JSON. Si usa XML con otro método de configuración y, luego, cambia la configuración con Azure Portal, esta cambiará a JSON.

## <a name="resource-manager-template"></a>Plantilla de Resource Manager
Consulte [Uso de la supervisión y el diagnóstico con una máquina virtual Windows y plantillas de Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md) para información sobre la implementación de la extensión de diagnósticos con plantillas de Azure Resource Manager. 

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure
La CLI de Azure se puede usar para implementar la extensión de Azure Diagnostics en una máquina virtual existente mediante [az vm extension set](/cli/azure/vm/extension?view=azure-cli-latest#az-vm-extension-set), como en el ejemplo siguiente. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name IaaSDiagnostics \
  --publisher Microsoft.Azure.Diagnostics \
  --protected-settings protected-settings.json \
  --settings public-settings.json 
```

La configuración protegida se define en el [elemento PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element) del esquema de configuración. A continuación se indica un ejemplo mínimo de un archivo de configuración protegida que define la cuenta de almacenamiento. Consulte [Configuración de ejemplo](diagnostics-extension-schema-windows.md#privateconfig-element) para ver detalles completos de la configuración privada.

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
}
```

La configuración pública se define en el [elemento Public](diagnostics-extension-schema-windows.md#publicconfig-element) del esquema de configuración. A continuación se ofrece un ejemplo mínimo de un archivo de configuración pública que permite la recopilación de registros de infraestructura de diagnóstico, un único contador de rendimiento y un único registro de eventos. Consulte [Configuración de ejemplo](diagnostics-extension-schema-windows.md#publicconfig-element) para información detallada sobre la configuración pública.

```JSON
{
  "StorageAccount": "mystorageaccount",
  "WadCfg": {
    "DiagnosticMonitorConfiguration": {
      "overallQuotaInMB": 5120,
      "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": [
          {
            "counterSpecifier": "\\Processor Information(_Total)\\% Processor Time",
            "unit": "Percent",
            "sampleRate": "PT60S"
          }
        ]
      },
      "WindowsEventLog": {
        "scheduledTransferPeriod": "PT1M",
        "DataSource": [
          {
            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
          }
        ]
      }
    }
  }
}
```



## <a name="powershell-deployment"></a>Implementación de PowerShell
PowerShell se puede usar para implementar la extensión de Azure Diagnostics en una máquina virtual existente mediante [Set-AzVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension), como en el ejemplo siguiente. 

```powershell
Set-AzVMDiagnosticsExtension -ResourceGroupName "myvmresourcegroup" `
  -VMName "myvm" `
  -DiagnosticsConfigurationPath "DiagnosticsConfiguration.json"
```

La configuración privada se define en el [elemento PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element), mientras que la configuración pública se define en el [elemento Public](diagnostics-extension-schema-windows.md#publicconfig-element) del esquema de configuración. También puede optar por especificar los detalles de la cuenta de almacenamiento como parámetros del cmdlet Set-AzVMDiagnosticsExtension en lugar de incluirlos en la configuración privada.

A continuación se ofrece un ejemplo mínimo de un archivo de configuración que permite la recopilación de registros de infraestructura de diagnóstico, un único contador de rendimiento y un único registro de eventos. Consulte [Configuración de ejemplo](diagnostics-extension-schema-windows.md#publicconfig-element) para información detallada sobre la configuración pública y privada. 

```JSON
{
    "PublicConfig": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
                "overallQuotaInMB": 10000,
                "DiagnosticInfrastructureLogs": {
                    "scheduledTransferLogLevelFilter": "Error"
                },
                "PerformanceCounters": {
                    "scheduledTransferPeriod": "PT1M",
                    "PerformanceCounterConfiguration": [
                        {
                            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                            "sampleRate": "PT3M",
                            "unit": "percent"
                        }
                    ]
                },
                "WindowsEventLog": {
                    "scheduledTransferPeriod": "PT1M",
                        "DataSource": [
                        {
                            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                        }
                    ]
                }
            }
        },
        "StorageAccount": "mystorageaccount",
        "StorageType": "TableAndBlob"
    },
    "PrivateConfig": {
        "storageAccountName": "mystorageaccount",
        "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
    }
}
```

Consulte también [Uso de PowerShell para habilitar Diagnósticos de Azure en una máquina virtual con Windows](../../virtual-machines/extensions/diagnostics-windows.md).

## <a name="data-storage"></a>Almacenamiento de datos
En la tabla siguiente se enumeran los distintos tipos de datos recopilados por la extensión de diagnósticos y se indica si se almacenan como una tabla o un blob. Los datos almacenados en tablas también se pueden almacenar en blobs en función del [valor de StorageType](diagnostics-extension-schema-windows.md#publicconfig-element) de la configuración pública.


| data | Tipo de almacenamiento | Descripción |
|:---|:---|:---|
| WADDiagnosticInfrastructureLogsTable | Tabla | Monitor de diagnóstico y cambios de configuración. |
| WADDirectoriesTable | Tabla | Directorios que supervisa el monitor de diagnóstico.  Esto incluye los registros de IIS, los registros de solicitudes con error de IIS y los directorios personalizados.  La ubicación del archivo de registro de blob se especifica en el campo de contenedor y el nombre del blob está en el campo RelativePath.  El campo AbsolutePath indica la ubicación y el nombre del archivo tal como existía en la máquina virtual de Azure. |
| WadLogsTable | Tabla | Registros escritos en código mediante la escucha de seguimiento. |
| WADPerformanceCountersTable | Tabla | Contadores de rendimiento. |
| WADWindowsEventLogsTable | Tabla | Registros de eventos de Windows. |
| wad-iis-failedreqlogfiles | Blob | Contiene información de los registros de solicitudes con error de IIS. |
| wad-iis-logfiles | Blob | Contiene información sobre los registros de IIS. |
| "custom" | Blob | Un contenedor personalizado basado en la configuración de directorios supervisados por el monitor de diagnóstico.  El nombre de este contenedor de blob se especificará en WADDirectoriesTable. |

## <a name="tools-to-view-diagnostic-data"></a>Herramientas para ver los datos de diagnóstico
Existen varias herramientas para ver los datos una vez que se transfieren al almacenamiento. Por ejemplo:

* El Explorador de servidores en Visual Studio: si instaló Azure Tools para Microsoft Visual Studio, puede usar el nodo de Azure Storage en el Explorador de servidores para ver los datos de tabla y blob de solo lectura de las cuentas de Azure Storage. Puede mostrar datos de la cuenta del emulador de almacenamiento local y también desde cuentas de almacenamiento que haya creado para Azure. Para obtener más información, consulte [Exploración y administración de recursos de almacenamiento con el Explorador de servidores](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage).
* [Explorador de Microsoft Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente que permite trabajar fácilmente con los datos de Azure Storage en Windows, OSX y Linux.
* [Azure Management Studio](https://www.cerebrata.com/products/azure-management-studio/introduction) incluye Azure Diagnostics Manager, que permite ver, descargar y administrar los datos de diagnóstico recopilados por las aplicaciones que se ejecutan en Azure.

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Envío de datos desde la extensión Azure Diagnostics para Windows a Event Hubs](diagnostics-extension-stream-event-hubs.md) para más información sobre el reenvío de datos de supervisión a Azure Event Hubs.
