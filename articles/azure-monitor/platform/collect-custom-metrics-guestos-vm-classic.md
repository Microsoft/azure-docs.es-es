---
title: Envío de métricas de máquina virtual de Windows clásica a la base de datos de métricas de Azure Monitor
description: Envío de métricas de SO invitado al almacén de datos de Azure Monitor para una máquina virtual Windows (clásica)
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.subservice: ''
ms.openlocfilehash: 87277c0c61f6d63e453386724dd472d2663e3148
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045208"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metrics-database-for-a-windows-virtual-machine-classic"></a>Envío de métricas de SO invitado a la base de datos de métricas de Azure Monitor para una máquina virtual Windows (clásica)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

La [extensión Diagnostics](./diagnostics-extension-overview.md) de Azure Monitor (conocida como "WAD" o "Diagnostics") le permite recopilar métricas y registros del sistema operativo invitado (SO invitado) que se ejecuta como parte de un clúster de Service Fabric, un servicio en la nube o una máquina virtual. La extensión puede enviar datos de telemetría a [muchas ubicaciones diferentes](./data-platform.md?toc=/azure/azure-monitor/toc.json).

En este artículo se describe el proceso de envío de métricas de rendimiento del SO invitado para una máquina virtual Windows (clásica) a la base de datos de métricas de Azure Monitor. A partir de Diagnostics versión 1.11, puede escribir las métricas directamente en el almacén de métricas de Azure Monitor, donde ya se recopilan métricas de la plataforma estándar. 

Almacenarlas en esta ubicación permite acceder a las mismas acciones disponibles para las métricas de la plataforma. Las acciones incluyen la generación de alertas casi en tiempo real, la creación de gráficos, el enrutamiento, el acceso desde una API REST y mucho más. Anteriormente, la extensión Diagnostics se escribía en Azure Storage, pero no en el almacén de datos de Azure Monitor. 

El proceso descrito en este artículo solo funciona para máquinas virtuales clásicas que ejecutan el sistema operativo Windows.

## <a name="prerequisites"></a>Requisitos previos

- Debe ser [administrador de servicios o administrador](../../cost-management-billing/manage/add-change-subscription-administrator.md) en su suscripción de Azure. 

- La suscripción debe estar registrada en [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Debe tener instalado [Azure PowerShell](/powershell/azure) o [Azure Cloud Shell](../../cloud-shell/overview.md).

- El recurso de máquina virtual debe estar en una [región que admita métricas personalizadas](metrics-custom-overview.md#supported-regions).

## <a name="create-a-classic-virtual-machine-and-storage-account"></a>Creación de una máquina virtual clásica y una cuenta de almacenamiento

1. Cree una máquina virtual clásica mediante Azure Portal.
   ![Crear una máquina virtual clásica](./media/collect-custom-metrics-guestos-vm-classic/create-classic-vm.png)

1. Al crear esta máquina virtual, elija la opción para crear una nueva cuenta de almacenamiento clásica. Usaremos esta cuenta de almacenamiento en pasos posteriores.

1. En Azure Portal, vaya a la hoja de recursos **Cuentas de almacenamiento**. Seleccione **Claves** y anote el nombre y la clave de la cuenta de almacenamiento. Necesitará esta información en pasos posteriores.
   ![Claves de acceso a Azure Storage](./media/collect-custom-metrics-guestos-vm-classic/storage-access-keys.png)

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Cree una entidad de servicio en el inquilino de Azure Active Directory según las instrucciones que encontrará en [Creación de una entidad de servicio](../../active-directory/develop/howto-create-service-principal-portal.md). Tenga en cuenta lo siguiente al realizar este proceso: 
- Cree un nuevo secreto de cliente para esta aplicación.
- Guarde la clave y el identificador de cliente para usarlos en pasos posteriores.

Asigne a esta aplicación permisos "Supervisión del publicador de métricas" para el recurso para el cual quiere emitir métricas. Puede usar un grupo de recursos o una suscripción completa.  

> [!NOTE]
> La extensión Diagnostics usa la entidad de servicio para autenticarse en Azure Monitor y emitir métricas para la máquina virtual clásica.

## <a name="author-diagnostics-extension-configuration"></a>Creación de la configuración de la extensión Diagnostics

1. Prepare el archivo de configuración de la extensión Diagnostics. Este archivo determina qué registros y contadores de rendimiento debe recopilar la extensión Diagnostics para la máquina virtual clásica. El siguiente es un ejemplo:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
            <Directories scheduledTransferPeriod="PT1M">
                <IISLogs containerName="wad-iis-logfiles" />
                <FailedRequestLogs containerName="wad-failedrequestlogs" />
            </Directories>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
                <PerformanceCounterConfiguration counterSpecifier="\Processor(*)\% Processor Time" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(*)\Disk Read Bytes/sec" sampleRate="PT15S" />
            </PerformanceCounters>
            <WindowsEventLog scheduledTransferPeriod="PT1M">
                <DataSource name="Application!*[System[(Level=1 or Level=2 or Level=3)]]" />
                <DataSource name="Windows Azure!*[System[(Level=1 or Level=2 or Level=3 or Level=4)]]" />
            </WindowsEventLog>
            <CrashDumps>
                <CrashDumpConfiguration processName="WaIISHost.exe" />
                <CrashDumpConfiguration processName="WaWorkerHost.exe" />
                <CrashDumpConfiguration processName="w3wp.exe" />
            </CrashDumps>
            <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Error" />
            <Metrics resourceId="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicCompute/virtualMachines/MyClassicVM">
                <MetricAggregation scheduledTransferPeriod="PT1M" />
                <MetricAggregation scheduledTransferPeriod="PT1H" />
            </Metrics>
        </DiagnosticMonitorConfiguration>
        <SinksConfig>
        </SinksConfig>
        </WadCfg>
        <StorageAccount />
    </PublicConfig>
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <StorageAccount name="" endpoint="" />
    </PrivateConfig>
    <IsEnabled>true</IsEnabled>
    </DiagnosticsConfiguration>
    ```
1. En la sección "SinksConfig" del archivo de diagnóstico, defina un nuevo receptor de Azure Monitor, de la forma siguiente:

    ```xml
    <SinksConfig>
        <Sink name="AzMonSink">
            <AzureMonitor>
                <ResourceId>Provide the resource ID of your classic VM </ResourceId>
                <Region>The region your VM is deployed in</Region>
            </AzureMonitor>
        </Sink>
    </SinksConfig>
    ```

1. En la sección del archivo de configuración donde se enumera la lista de contadores de rendimiento que se van a recopilar, enrute los contadores de rendimiento al receptor de Azure Monitor "AzMonSink".

    ```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
    ```

1. En la configuración privada, defina la cuenta de Azure Monitor. A continuación, agregue la información de la entidad de servicio que se usará para emitir métricas.

    ```xml
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <StorageAccount name="" endpoint="" />
        <AzureMonitorAccount>
            <ServicePrincipalMeta>
                <PrincipalId>clientId for your service principal</PrincipalId>
                <Secret>client secret of your service principal</Secret>
            </ServicePrincipalMeta>
        </AzureMonitorAccount>
    </PrivateConfig>
    ```

1. Guarde este archivo localmente.

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>Implementación de la extensión Diagnostics en el servicio en la nube

1. Inicie PowerShell e inicie sesión.

    ```powershell
    Login-AzAccount
    ```

1. Empiece por establecer el contexto en la máquina virtual clásica.

    ```powershell
    $VM = Get-AzureVM -ServiceName <VM’s Service_Name> -Name <VM Name>
    ```

1. Establezca el contexto de la cuenta de almacenamiento clásica que se creó con la VM.

    ```powershell
    $StorageContext = New-AzStorageContext -StorageAccountName <name of your storage account from earlier steps> -storageaccountkey "<storage account key from earlier steps>"
    ```

1.  Establezca la ruta de acceso del archivo de Diagnostics en una variable mediante el comando siguiente:

    ```powershell
    $diagconfig = “<path of the diagnostics configuration file with the Azure Monitor sink configured>”
    ```

1.  Prepare la actualización de la máquina virtual clásica con el archivo de diagnóstico que tenga configurado el receptor de Azure Monitor.

    ```powershell
    $VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $diagconfig -VM $VM -StorageContext $Storage_Context
    ```

1.  Implemente la actualización en la máquina virtual; para ello, ejecute el comando siguiente:

    ```powershell
    Update-AzureVM -ServiceName "ClassicVMWAD7216" -Name "ClassicVMWAD" -VM $VM_Update.VM
    ```

> [!NOTE]
> Sigue siendo obligatorio proporcionar una cuenta de almacenamiento como parte de la instalación de la extensión Diagnostics. Todos los registros o contadores de rendimiento especificados en el archivo de configuración de Diagnostics se escribirán en la cuenta de almacenamiento especificada.

## <a name="plot-the-metrics-in-the-azure-portal"></a>Trazado de métricas en Azure Portal

1.  Vaya a Azure Portal. 

1.  En el menú de la izquierda, seleccione **Monitor**.

1.  En la hoja **Monitor**, seleccione **Métricas**.

    ![Navegación por las métricas](./media/collect-custom-metrics-guestos-vm-classic/navigate-metrics.png)

1. En la lista desplegable de recursos, seleccione la máquina virtual clásica.

1. En el menú desplegable de espacios de nombres, seleccione **azure.vm.windows.guest**.

1. En el menú desplegable de métricas, seleccione **Memory\Committed Bytes in Use** (Memoria\bytes confirmados en uso).
   ![Trazado de las métricas](./media/collect-custom-metrics-guestos-vm-classic/plot-metrics.png)


## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de las [métricas personalizadas](metrics-custom-overview.md).
