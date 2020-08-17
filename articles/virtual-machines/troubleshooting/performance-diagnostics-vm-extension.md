---
title: Extensión de máquina virtual de diagnóstico de rendimiento de Azure para Windows | Microsoft Docs
description: Presenta la extensión de máquina virtual de diagnóstico de rendimiento de Azure para Windows.
services: virtual-machines-windows'
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 62a30fada32a23546323dae34f875ab9c7da228c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028556"
---
# <a name="azure-performance-diagnostics-vm-extension-for-windows"></a>Extensión de máquina virtual de diagnóstico de rendimiento de Azure para Windows

La extensión de máquina virtual de diagnósticos de rendimiento de Azure ayuda a recopilar datos de diagnóstico de rendimiento de máquinas virtuales Windows. La extensión realiza un análisis y proporciona un informe de resultados y recomendaciones para identificar y resolver problemas de rendimiento en la máquina virtual. Esta extensión instala una herramienta de solución de problemas denominada [PerfInsights](https://aka.ms/perfinsights).

> [!NOTE]
> Si desea ejecutar diagnósticos en la máquina virtual desde Azure Portal para máquinas virtuales no clásicas, se recomienda usar la nueva experiencia. Para más información, consulte [Diagnóstico de rendimiento para máquinas virtuales de Azure](performance-diagnostics.md). 

## <a name="prerequisites"></a>Requisitos previos

Esta extensión se puede instalar en Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 y Windows Server 2016. También se puede instalar en Windows 8.1 y Windows 10.

## <a name="extension-schema"></a>Esquema de extensión
El siguiente código JSON muestra el esquema de la extensión de máquina virtual de diagnósticos de rendimiento de Azure. Esta extensión requiere el nombre y la clave de una cuenta de almacenamiento para almacenar el resultado de diagnóstico y el informe. Estos valores son confidenciales. La clave de la cuenta de almacenamiento debe almacenarse en una configuración protegida. Los datos de configuración protegida de la extensión de máquina virtual de Azure están cifrados y solo se descifran en la máquina virtual de destino. Tenga en cuenta que **storageAccountName** y **storageAccountKey** distinguen mayúsculas de minúsculas. Otros parámetros necesarios se enumeran en la sección siguiente.

```JSON
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "storageAccountName": "[parameters('storageAccountName')]",
            "performanceScenario": "[parameters('performanceScenario')]",
            "traceDurationInSeconds": "[parameters('traceDurationInSeconds')]",
            "perfCounterTrace": "[parameters('perfCounterTrace')]",
            "networkTrace": "[parameters('networkTrace')]",
            "xperfTrace": "[parameters('xperfTrace')]",
            "storPortTrace": "[parameters('storPortTrace')]",
            "srNumber": "[parameters('srNumber')]",
            "requestTimeUtc":  "[parameters('requestTimeUtc')]",
            "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"        
        }
      }
    }
```

### <a name="property-values"></a>Valores de propiedad

|   **Nombre**   |**Valor/Ejemplo**|       **Descripción**      |
|--------------|-------------------|----------------------------|
|apiVersion|2015-06-15|La versión de la API.
|publisher|Microsoft.Azure.Performance.Diagnostics|El espacio de nombres del publicador correspondiente a la extensión.
|type|AzurePerformanceDiagnostics|El tipo de la extensión de máquina virtual.
|typeHandlerVersion|1.0|La versión del controlador de extensiones.
|performanceScenario|basic|El escenario de rendimiento para el que se capturan datos. Los valores válidos son: **basic**, **vmslow**, **azurefiles** y **custom**.
|traceDurationInSeconds|300|La duración de los seguimientos, si se ha seleccionado alguna de las opciones de seguimiento.
|perfCounterTrace|p|Opción para habilitar el seguimiento del contador de rendimiento. Los valores válidos son **p** o un valor vacío. Si no desea capturar este seguimiento, deje el valor vacío.
|networkTrace|n|Opción para habilitar el seguimiento de la red. Los valores válidos son **n** o un valor vacío. Si no desea capturar este seguimiento, deje el valor vacío.
|xperfTrace|x|Opción para habilitar el seguimiento de XPerf. Los valores válidos son **x** o un valor vacío. Si no desea capturar este seguimiento, deje el valor vacío.
|storPortTrace|s|Opción para habilitar el seguimiento de StorPort. Los valores válidos son **s** o un valor vacío. Si no desea capturar este seguimiento, deje el valor vacío.
|srNumber|123452016365929|El número de incidencia de soporte técnico, si está disponible. Deje el valor vacío si no dispone de él.
|requestTimeUtc|2017-09-28T22:08:53.736Z|Hora de fecha actual en UTC. No tiene que proporcionar este valor si usa el portal para instalar esta extensión.
|resourceId|/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}|El identificador único de una máquina virtual.
|storageAccountName|mystorageaccount|El nombre de la cuenta de almacenamiento para almacenar los registros de diagnóstico y los resultados.
|storageAccountKey|lDuVvxuZB28NNP…hAiRF3voADxLBTcc==|La clave de la cuenta de almacenamiento.

## <a name="install-the-extension"></a>Instalación de la extensión

Siga estas instrucciones para instalar la extensión en máquinas virtuales Windows:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione la máquina virtual en la que desee instalar esta extensión.

    ![Captura de pantalla de Azure Portal, donde aparece resaltado Virtual Machines](media/performance-diagnostics-vm-extension/select-the-virtual-machine.png)
3. Seleccione la hoja **Extensiones** y seleccione **Agregar**.

    ![Captura de pantalla de la hoja Extensiones, donde aparece resaltado Agregar](media/performance-diagnostics-vm-extension/select-extensions.png)
4. Seleccione **Diagnósticos de rendimiento de Azure**, revise los términos y condiciones y seleccione **Crear**.

    ![Captura de pantalla de la pantalla Nuevo recurso, donde aparece resaltado Diagnósticos de rendimiento de Azure](media/performance-diagnostics-vm-extension/create-azure-performance-diagnostics-extension.png)
5. Proporcione los valores de parámetros para la instalación y seleccione **Aceptar** para instalar la extensión. Para más información sobre los escenarios compatibles, consulte [Cómo usar PerfInsights](how-to-use-perfinsights.md#supported-troubleshooting-scenarios). 

    ![Captura de pantalla del cuadro de diálogo Instalar extensión](media/performance-diagnostics-vm-extension/install-the-extension.png)
6. Si la instalación se realiza correctamente, verá un mensaje que indica este estado.

    ![Captura de pantalla del mensaje Aprovisionamiento realizado correctamente](media/performance-diagnostics-vm-extension/provisioning-succeeded-message.png)

    > [!NOTE]
    > La extensión se ejecuta cuando el aprovisionamiento se ha realizado correctamente. En escenarios básicos, tarda dos minutos o menos en completarse. En otros escenarios, se ejecuta durante el tiempo especificado en el momento de la instalación.

## <a name="remove-the-extension"></a>Eliminación de la extensión
Para quitar la extensión desde una máquina virtual, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione la máquina virtual de la que quiere quitar esta extensión y, luego, seleccione la hoja **Extensiones**. 
2. Seleccione ( **…** ) en la entrada de la lista de la extensión de diagnósticos de rendimiento y seleccione **Desinstalar**.

    ![Captura de pantalla de la hoja Extensiones, donde se resalta Desinstalar](media/performance-diagnostics-vm-extension/uninstall-the-extension.png)

    > [!NOTE]
    > También puede seleccionar la entrada de la extensión y seleccionar la opción **Desinstalar**.

## <a name="template-deployment"></a>Implementación de plantilla
Las extensiones de máquina virtual de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager. Este esquema ejecuta la extensión de máquina virtual de diagnósticos de rendimiento de Azure durante la implementación de una plantilla de Azure Resource Manager. Esta es una plantilla de ejemplo:

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "yourVMName"
    },
    "location": {
      "type": "string",
      "defaultValue": "southcentralus"
    },
    "storageAccountName": {
      "type": "securestring"
      "defaultValue": "yourStorageAccount"
    },
    "storageAccountKey": {
      "type": "securestring"
      "defaultValue": "yourStorageAccountKey"
    },
    "performanceScenario": {
      "type": "string",
      "defaultValue": "basic"
    },
    "srNumber": {
      "type": "string",
      "defaultValue": ""
    },
    "traceDurationInSeconds": {
      "type": "int",
    "defaultValue": 300
    },
    "perfCounterTrace": {
      "type": "string",
      "defaultValue": "p"
    },
    "networkTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "xperfTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "storPortTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "requestTimeUtc": {
      "type": "string",
      "defaultValue": "10/2/2017 11:06:00 PM"
    }       
  },
  "resources": [
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "storageAccountName": "[parameters('storageAccountName')]",
            "performanceScenario": "[parameters('performanceScenario')]",
            "traceDurationInSeconds": "[parameters('traceDurationInSeconds')]",
            "perfCounterTrace": "[parameters('perfCounterTrace')]",
            "networkTrace": "[parameters('networkTrace')]",
            "xperfTrace": "[parameters('xperfTrace')]",
            "storPortTrace": "[parameters('storPortTrace')]",
            "srNumber": "[parameters('srNumber')]",
            "requestTimeUtc":  "[parameters('requestTimeUtc')]",
            "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
```

## <a name="powershell-deployment"></a>Implementación de PowerShell
El comando `Set-AzVMExtension` puede usarse para implementar la extensión de máquina virtual de diagnósticos de rendimiento de Azure en una máquina virtual existente.

PowerShell

```
$PublicSettings = @{ "storageAccountName"="mystorageaccount";"performanceScenario"="basic";"traceDurationInSeconds"=300;"perfCounterTrace"="p";"networkTrace"="";"xperfTrace"="";"storPortTrace"="";"srNumber"="";"requestTimeUtc"="2017-09-28T22:08:53.736Z";"resourceId"="VMResourceId" }
$ProtectedSettings = @{"storageAccountKey"="mystoragekey" }

Set-AzVMExtension -ExtensionName "AzurePerformanceDiagnostics" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Performance.Diagnostics" `
    -ExtensionType "AzurePerformanceDiagnostics" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS
```

## <a name="information-on-the-data-captured"></a>Información sobre los datos capturados
La herramienta PerfInsights recopila distintos registros, configuraciones y datos de diagnóstico, según el escenario seleccionado. Para más información, consulte la [documentación de PerfInsights](https://aka.ms/perfinsights).

## <a name="view-and-share-the-results"></a>Visualización y uso compartido de resultados

Puede encontrar la salida de la extensión en un archivo ZIP que se cargó en la cuenta de almacenamiento especificada durante la instalación y que se comparte durante 30 días mediante el uso de [Firmas de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md). Este archivo ZIP contiene registros de diagnóstico y un informe con conclusiones y recomendaciones. Puede encontrar un vínculo SAS al archivo ZIP de salida dentro de un archivo de texto denominado *zipfilename*_saslink.txt, en la carpeta **C:\Packages\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\\\<version>** . Cualquier persona que tenga este vínculo puede descargar el archivo ZIP.

Para ayudar al ingeniero de soporte técnico a trabajar en su incidencia, Microsoft puede usar este vínculo SAS para descargar los datos de diagnóstico.

Para ver el informe, extraiga el archivo ZIP y abra el archivo **PerfInsights Report.html**.

También puede descargar el archivo ZIP directamente del portal seleccionando la extensión.

![Captura de pantalla del estado detallado de diagnósticos de rendimiento](media/performance-diagnostics-vm-extension/view-detailed-status.png)

> [!NOTE]
> Puede que el vínculo SAS que se muestra en el portal a veces no funcione. El motivo puede ser una dirección URL con un formato incorrecto durante las operaciones de codificación y decodificación. Si este es su caso, obtenga el vínculo directamente del archivo *_saslink.txt de la máquina virtual.

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

- El estado de implementación de la extensión (en el área de notificación) puede mostrar "Implementación en curso" a pesar de que se haya realizado un aprovisionamiento correcto de la extensión.

    Este problema puede omitirse de forma segura siempre que el estado de la extensión indique que la extensión se ha aprovisionado correctamente.
- Puede solucionar algunos problemas durante la instalación mediante los registros de extensión. El resultado de la ejecución de las extensiones se registra en los archivos que se encuentran en el siguiente directorio:

    `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\<version>`

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**. Para más información sobre el uso del soporte técnico de Azure, lea las [preguntas más frecuentes](https://azure.microsoft.com/support/faq/).
