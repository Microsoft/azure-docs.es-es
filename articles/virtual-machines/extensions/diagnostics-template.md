---
title: Adición de supervisión y diagnósticos a una máquina virtual de Azure
description: Use una plantilla de Azure Resource Manager para crear una máquina virtual Windows con la extensión Azure Diagnostics.
services: virtual-machines-windows
documentationcenter: ''
author: mimckitt
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 8cde8fe7-977b-43d2-be74-ad46dc946058
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 05/31/2017
ms.author: mimckitt
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 31f690277675650323763a7bc6872ad736f5776c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837013"
---
# <a name="use-monitoring-and-diagnostics-with-a-windows-vm-and-azure-resource-manager-templates"></a>Uso de la supervisión y el diagnóstico con una máquina virtual Windows y plantillas de Azure Resource Manager
La extensión Azure Diagnostics proporciona funcionalidades de supervisión y diagnóstico en una máquina virtual de Azure basada en Windows. Para habilitar estas funcionalidades en la máquina virtual, incluya la extensión como parte de la plantilla de Azure Resource Manager. Para obtener más información sobre cómo incluir cualquier extensión como parte de una plantilla de máquina virtual, consulte [Creación de plantillas del Administrador de recursos de Azure con extensiones de máquina virtual](../windows/template-description.md#extensions) . En este artículo se describe cómo agregar la extensión de Diagnósticos de Azure a una plantilla de máquina virtual de Windows.  

## <a name="add-the-azure-diagnostics-extension-to-the-vm-resource-definition"></a>Incorporación de la extensión de Diagnósticos de Azure a la definición de recursos de máquina virtual
Para habilitar la extensión Diagnostics en una máquina virtual Windows, debe agregar la extensión como un recurso de máquina virtual en la plantilla de Resource Manager.

En una máquina virtual sencilla basada en el Administrador de recursos, agregue la configuración de extensión a la matriz *resources* de la máquina virtual: 

```json
"resources": [
    {
        "name": "Microsoft.Insights.VMDiagnosticsSettings",
        "type": "extensions",
        "location": "[resourceGroup().location]",
        "apiVersion": "2015-06-15",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "tags": {
            "displayName": "AzureDiagnostics"
        },
        "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.5",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), variables('vmName'), variables('wadcfgxend')))]",
                "storageAccount": "[parameters('existingdiagnosticsStorageAccountName')]"
            },
            "protectedSettings": {
                "storageAccountName": "[parameters('existingdiagnosticsStorageAccountName')]",
                "storageAccountKey": "[listkeys(variables('accountid'), '2015-05-01-preview').key1]",
                "storageAccountEndPoint": "https://core.windows.net"
            }
        }
    }
]
```

Otra costumbre común es agregar la configuración de extensión en el nodo de recursos raíz de la plantilla en lugar de definirla en el nodo de recursos de la máquina virtual. Con este enfoque tiene que especificar explícitamente una relación jerárquica entre la extensión y la máquina virtual con los valores *name* y *type*. Por ejemplo: 

```json
"name": "[concat(variables('vmName'),'Microsoft.Insights.VMDiagnosticsSettings')]",
"type": "Microsoft.Compute/virtualMachines/extensions",
```

La extensión siempre se asocia a la máquina virtual y puede definirla directamente en el nodo de recursos de la máquina virtual o en el nivel base y usar la convención de nomenclatura jerárquica para asociarla a la máquina virtual.

Para conjuntos de escala de máquina virtual, la configuración de extensión se especifica en la propiedad *extensionProfile* de *VirtualMachineProfile*.

La propiedad *publisher* con el valor de **Microsoft.Azure.Diagnostics** y la propiedad *type* con el valor **IaaSDiagnostics** identifican de forma única la extensión de Diagnósticos de Azure.

El valor de la propiedad *name* se puede usar para hacer referencia a la extensión en el grupo de recursos. Al establecerla específicamente en **Microsoft.Insights.VMDiagnosticsSettings** es posible identificarla fácilmente mediante Azure Portal, lo que garantiza que los gráficos de supervisión se muestren correctamente en Azure Portal.

*typeHandlerVersion* especifica la versión de la extensión que quiere usar. Establecer la versión secundaria de *autoUpgradeMinorVersion* en **true** garantiza la obtención de la versión secundaria más reciente de la extensión que está disponible. Se recomienda establecer siempre *autoUpgradeMinorVersion* en **true** para que siempre obtenga la extensión de diagnósticos más reciente disponible con todas las nuevas características y soluciones de errores. 

El elemento *settings* contiene propiedades de configuración de la extensión que se pueden establecer y leer de la extensión (lo que se conoce en ocasiones como configuración pública). La propiedad *xmlcfg* contiene configuración basada en XML para los registros de diagnóstico, los contadores de rendimiento, etc., que se recopilan con el agente de diagnóstico. Consulte [Esquema de configuración de diagnósticos](../../azure-monitor/platform/diagnostics-extension-schema-windows.md) para obtener más información sobre el propio esquema XML. Es una práctica común almacenar la configuración XML real como una variable en la plantilla del Administrador de recursos de Azure y luego concatenarla y codificarla como base64 para establecer el valor de *xmlcfg*. Consulte la sección sobre las [variables de configuración de diagnóstico](#diagnostics-configuration-variables) para obtener más información sobre cómo almacenar el XML en variables. La propiedad *storageAccount* especifica el nombre de la cuenta de almacenamiento a la que se transfieren los datos de diagnóstico. 

Las propiedades de *protectedSettings* (conocida en ocasiones como configuración privada) se pueden establecer, pero no se pueden leer después de que se han establecido. El carácter de solo escritura de *protectedSettings* hace que sea útil para almacenar secretos, como la clave de la cuenta de almacenamiento donde se escriben los datos de diagnóstico.    

## <a name="specifying-diagnostics-storage-account-as-parameters"></a>Especificación de la cuenta de almacenamiento de diagnóstico como parámetro
El fragmento de código JSON de la extensión Diagnostics anterior presupone dos parámetros *existingdiagnosticsStorageAccountName* y *existingdiagnosticsStorageResourceGroup* para especificar la cuenta de almacenamiento de diagnóstico donde se almacenan los datos de diagnóstico. La especificación de la cuenta de almacenamiento de diagnóstico como parámetro hace que sea fácil cambiarla entre distintos entornos; por ejemplo, puede que quiera usar una cuenta de almacenamiento de diagnóstico diferente para prueba y otra para la implementación en producción.  

```json
"existingdiagnosticsStorageAccountName": {
    "type": "string",
    "metadata": {
"description": "The name of an existing storage account to which diagnostics data is transfered."
    }
},
"existingdiagnosticsStorageResourceGroup": {
    "type": "string",
    "metadata": {
"description": "The resource group for the storage account specified in existingdiagnosticsStorageAccountName"
    }
}
```

Resulta aconsejable especificar una cuenta de almacenamiento de diagnóstico en un grupo de recursos diferente al grupo de recursos de la máquina virtual. Un grupo de recursos se puede considerar una unidad de implementación con su propio ciclo de vida; una máquina virtual se puede implementar y volver a implementar a medida que se realizan nuevas actualizaciones de configuraciones en ella, pero puede que quiera seguir almacenando los datos de diagnóstico en la misma cuenta de almacenamiento en esas implementaciones de máquina virtual. Tener la cuenta de almacenamiento en un recurso diferente hace posible que dicha cuenta acepte datos de varias implementaciones de máquina virtual, lo que facilita la solución de problemas entre las distintas versiones.

> [!NOTE]
> Si crea una plantilla de máquina virtual Windows desde Visual Studio, la cuenta de almacenamiento predeterminada se podría configurar para usar la misma cuenta de almacenamiento en la que se carga la VHD de máquina virtual. Esto sirve para simplificar la configuración inicial de la máquina virtual. Vuelva a factorizar la plantilla para usar una cuenta de almacenamiento diferente que se pueda pasar como un parámetro. 
> 
> 

## <a name="diagnostics-configuration-variables"></a>variables de configuración de diagnóstico
El fragmento json de la extensión de diagnósticos anterior define una variable *accountid* para simplificar la obtención de la clave de la cuenta de almacenamiento para el almacenamiento de diagnóstico:   

```json
"accountid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/',parameters('existingdiagnosticsStorageResourceGroup'), '/providers/','Microsoft.Storage/storageAccounts/', parameters('existingdiagnosticsStorageAccountName'))]"
```

La propiedad *xmlcfg* de la extensión de diagnósticos se define mediante diversas variables que se concatenan juntas. Los valores de estas variables están en XML, así que se deben incluir correctamente entre secuencias de escape al definir las variables json.

El siguiente ejemplo describe el XML de configuración de diagnóstico que recopila contadores de rendimiento de nivel de sistema estándar junto con algunos registros de eventos de Windows y registros de infraestructura de diagnóstico. Se ha incluido correctamente entre secuencias de escape y se le ha dado el formato adecuado para que la configuración se pueda pegar directamente en la sección de variables de su plantilla. Para obtener un ejemplo más claro para el usuario del XML de configuración, consulte el [Esquema de configuración de diagnósticos](../../azure-monitor/platform/diagnostics-extension-schema-windows.md) .

```json
"wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" /></WindowsEventLog>",
"wadperfcounters1": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Processor Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU utilization\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Privileged Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU privileged time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% User Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU user time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor Information(_Total)\\Processor Frequency\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"CPU frequency\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\System\\Processes\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Processes\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Threads\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Handle Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Handles\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\% Committed Bytes In Use\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Memory usage\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Available Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory available\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Committed Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory committed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Commit Limit\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory commit limit\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active time\" locale=\"en-us\"/></PerformanceCounterConfiguration>",
"wadperfcounters2": "<PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Read Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active read time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Write Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active write time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Transfers/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Reads/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk read operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Writes/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk write operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Read Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk read speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Write Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk write speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\LogicalDisk(_Total)\\% Free Space\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk free space (percentage)\" locale=\"en-us\"/></PerformanceCounterConfiguration></PerformanceCounters>",
"wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters1'), variables('wadperfcounters2'), '<Metrics resourceId=\"')]",
"wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name , '/providers/', 'Microsoft.Compute/virtualMachines/')]",
"wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>"
```

El nodo de XML de definición de métricas de la configuración anterior es un elemento de configuración importante ya que define cómo se agregarán y almacenarán los contadores de rendimiento definidos anteriormente en el XML en el nodo *PerformanceCounter*. 

> [!IMPORTANT]
> Estas métricas dirigen los gráficos de supervisión y alertas en el Portal de Azure.  El nodo **Métricas** con *resourceID* y **MetricAggregation** deben incluirse en la configuración de diagnóstico para la máquina virtual si quiere ver los datos de supervisión de la máquina virtual en Azure Portal. 
> 
> 

A continuación se muestra un ejemplo del XML de las definiciones de métricas: 

```xml
<Metrics resourceId="/subscriptions/subscription().subscriptionId/resourceGroups/resourceGroup().name/providers/Microsoft.Compute/virtualMachines/vmName">
    <MetricAggregation scheduledTransferPeriod="PT1H"/>
    <MetricAggregation scheduledTransferPeriod="PT1M"/>
</Metrics>
```

El atributo *resourceID* identifica de forma única la máquina virtual de su suscripción. Asegúrese de usar las funciones subscription() y resourceGroup() para que la plantilla actualice automáticamente esos valores en función de la suscripción y el grupo de recursos en el que va a realizar la implementación.

Si va a crear varias máquinas virtuales en un bucle, tiene que rellenar el valor *resourceID* con una función copyIndex() para diferenciar correctamente cada máquina virtual de forma individual. El valor *xmlCfg* se puede actualizar para admitir esto de la manera siguiente:  

```json
"xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), concat(parameters('vmNamePrefix'), copyindex()), variables('wadcfgxend')))]", 
```

El valor MetricAggregation de *PT1M* y *PT1H* indica una agregación durante un minuto y una agregación durante una hora, respectivamente.

## <a name="wadmetrics-tables-in-storage"></a>Tablas de WADMetrics en almacenamiento
La configuración de métricas anterior genera tablas en la cuenta de almacenamiento de diagnóstico con las convenciones de nomenclatura siguientes:

* **WADMetrics**: prefijo estándar para todas las tablas WADMetrics.
* **PT1H** o **PT1M**: significa que la tabla contiene datos agregados durante una hora o un minuto.
* **P10D**: significa que la tabla contendrá datos de diez días de cuando la tabla comenzó a recopilar datos.
* **V2S**: constante de cadena.
* **yyyymmdd**: la fecha en la que la tabla comenzó a recopilar datos.

Ejemplo: *WADMetricsPT1HP10DV2S20151108* contiene datos de métricas agregados durante una hora durante 10 días a partir del 11 de noviembre de 2015.    

Cada tabla WADMetrics contiene las columnas siguientes:

* **PartitionKey**: la clave de partición se construye a partir del valor *resourceID* para identificar de forma única el recurso de máquina virtual. Por ejemplo: `002Fsubscriptions:<subscriptionID>:002FresourceGroups:002F<ResourceGroupName>:002Fproviders:002FMicrosoft:002ECompute:002FvirtualMachines:002F<vmName>`  
* **RowKey**: sigue el formato `<Descending time tick>:<Performance Counter Name>`. El cálculo de la graduación de tiempo descendente es igual a la graduación de tiempo máxima menos el tiempo de comienzo del período de agregación. Por ejemplo, si el período de ejemplo empezó el 10 de noviembre de 2015 a las 00:00 horas UTC, entonces el cálculo sería: `DateTime.MaxValue.Ticks - (new DateTime(2015,11,10,0,0,0,DateTimeKind.Utc).Ticks)`. Para el contador de rendimiento de bytes de memoria disponible, la clave de fila será similar a: `2519551871999999999__:005CMemory:005CAvailable:0020Bytes`
* **CounterName**: nombre del contador de rendimiento. Coincide con el valor de *counterSpecifier* definido en el archivo de configuración XML.
* **Maximum**: el valor máximo del contador de rendimiento durante el período de agregación.
* **Mínimos**: el valor mínimo del contador de rendimiento durante el período de agregación.
* **Total**: la suma de todos los valores del contador de rendimiento notificados durante el período de agregación.
* **Recuento**: el número total de valores notificados para el contador de rendimiento.
* **Average**: el valor medio (total/count) del contador de rendimiento durante el período de agregación.

## <a name="next-steps"></a>Pasos siguientes
* Para obtener una plantilla de ejemplo de una máquina virtual Windows con la extensión Diagnostics, consulte [201-vm-monitoring-diagnostics-extension](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-monitoring-diagnostics-extension).   
* Implemente la plantilla de Azure Resource Manager mediante [Azure PowerShell](../windows/ps-template.md) o la [línea de comandos de Azure](../linux/create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Obtenga más información sobre la [creación de plantillas del Administrador de recursos de Azure](../../azure-resource-manager/templates/template-syntax.md)