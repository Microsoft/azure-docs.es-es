---
title: Extensión de máquina virtual de Azure Monitor para Windows | Microsoft Docs
description: Implemente el agente de Log Analytics en la máquina virtual Windows con una extensión de máquina virtual.
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: feae6176-2373-4034-b5d9-a32c6b4e1f10
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/29/2019
ms.author: roiyz
ms.openlocfilehash: 270b3ae49a815c9e12fce9377c8298192237f28a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790370"
---
# <a name="azure-monitor-virtual-machine-extension-for-windows"></a>Extensión de máquina virtual de Azure Monitor para Windows

Registros de Azure Monitor proporciona capacidades de supervisión en los activos en la nube y locales. Microsoft, como editor de la extensión de máquina virtual del agente de Log Analytics para Windows, es quien presta los servicios de soporte técnico para esta solución. La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics. Este documento detalla las plataformas compatibles, configuraciones y opciones de implementación para la extensión de máquina virtual de Azure Monitor para Windows.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Requisitos previos

### <a name="operating-system"></a>Sistema operativo

La extensión del agente de Log Analytics para Windows es compatible con las siguientes versiones del sistema operativo Windows:

- Windows Server 2019
- Windows Server 2008 R2, 2012, 2012 R2, 2016, versión 1709 y 1803

### <a name="agent-and-vm-extension-version"></a>Versión de extensión de agente y máquina virtual
En la tabla siguiente proporciona una asignación de la versión de la extensión de máquina virtual de Azure Monitor y el paquete del agente de Log Analytics para cada versión. 

| Versión de la extensión de máquina virtual Linux de Monitor Azure | Versión del paquete del agente de Log Analytics | Fecha de lanzamiento | Notas de la versión |
|--------------------------------|--------------------------|--------------------------|--------------------------|
| 8.0.11049.0 | 1.0.11049.1 | Febrero de 2017 | |
| 8.0.11072.0 | 1.0.11072.1 | Septiembre de 2017 | |
| 8.0.11081.0 | 1.0.11081.5 | Noviembre de 2017 | | 
| 8.0.11103.0 | N/D |  2018 de abril de 2018 | |
| 8.0.11136.0 | N/D | Septiembre de 2018 |  <ul><li> Se ha agregado compatibilidad para detectar el cambio de Id. de recurso en el movimiento de la máquina virtual </li><li> Se agregó compatibilidad para instalar identificador cuando se usa sin extensión de recurso de informes </li></ul>| 
| 10.19.10006.0 | N/D | Diciembre de 2018 | <ul><li> Correcciones menores de estabilización </li></ul> | 
| 10.19.13515.0 | 1.0.13515.1 | Marzo de 2019 | <ul><li>Correcciones menores de estabilización </li></ul> |

### <a name="azure-security-center"></a>Azure Security Center

Azure Security Center aprovisiona al agente de Log Analytics y lo conecta con el área de trabajo de Log Analytics predeterminado de la suscripción de Azure automáticamente. Si utiliza Azure Security Center, no siga los pasos de este documento. Si lo hace, sobrescribirá el área de trabajo configurada e interrumpirá la conexión con Azure Security Center.

### <a name="internet-connectivity"></a>Conectividad de Internet
La extensión del agente de Log Analytics para Windows requiere que la máquina virtual de destino esté conectada a Internet. 

## <a name="extension-schema"></a>Esquema de extensión

En el siguiente JSON se muestra el esquema para la extensión del agente de Log Analytics. La extensión requiere el Id. de área de trabajo y la clave de área de trabajo desde el área de trabajo de Log Analytics de destino. Se pueden encontrar en la configuración del área de trabajo en Azure Portal. Como la clave del área de trabajo debe tratarse como datos confidenciales, debe almacenarse en una configuración protegida. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino. Tenga en cuenta que **workspaceId** y **workspaceKey** distinguen mayúsculas de minúsculas.

```json
{
    "type": "extensions",
    "name": "OMSExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
        "type": "MicrosoftMonitoringAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "workspaceId": "myWorkSpaceId"
        },
        "protectedSettings": {
            "workspaceKey": "myWorkspaceKey"
        }
    }
}
```
### <a name="property-values"></a>Valores de propiedad

| NOMBRE | Valor / ejemplo |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.EnterpriseCloud.Monitoring |
| Tipo | MicrosoftMonitoringAgent |
| typeHandlerVersion | 1.0 |
| workspaceId (e.g)* | 6f680a37-00c6-41c7-a93f-1437e3462574 |
| workspaceKey (p. ej.) | z4bU3p1/GrnWpQkky4gdabWXAhbWSTz70hm4m2Xt92XI+rSRgE8qVvRhsGo9TXffbrTahyrwv35W0pOqQAU7uQ== |

\* El identificador del área de trabajo se denomina identificador de consumidor en la API de Log Analytics.

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager para ejecutar el agente de Log Analytics durante la implementación de una plantilla de Azure Resource Manager. Puede encontrar una plantilla de ejemplo que incluye la extensión de VM del agente de Log Analytics en la [Galería de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-oms-extension-windows-vm). 

>[!NOTE]
>La plantilla no admite la especificación de más de un identificador de área de trabajo y la clave de área de trabajo cuando desea configurar el agente para informar a varias áreas de trabajo. Para configurar el agente para informar a varias áreas de trabajo, consulte [agregando o quitando un área de trabajo](../../azure-monitor/platform/agent-manage.md#adding-or-removing-a-workspace).  

El JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual, o colocada en la raíz o un nivel superior de una plantilla JSON de Resource Manager. La colocación de la plantilla JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/resource-group-authoring-templates.md#child-resources). 

En el siguiente ejemplo se da por supuesto que la extensión de Azure Monitor está anidada dentro de los recursos de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto `"resources": []` de la máquina virtual.


```json
{
    "type": "extensions",
    "name": "OMSExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
        "type": "MicrosoftMonitoringAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "workspaceId": "myWorkSpaceId"
        },
        "protectedSettings": {
            "workspaceKey": "myWorkspaceKey"
        }
    }
}
```

Al colocar la plantilla JSON de la extensión en la raíz de la plantilla, el nombre de recurso incluye una referencia a la máquina virtual principal, y el tipo refleja la configuración anidada. 

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "<parentVmResource>/OMSExtension",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
        "type": "MicrosoftMonitoringAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "workspaceId": "myWorkSpaceId"
        },
        "protectedSettings": {
            "workspaceKey": "myWorkspaceKey"
        }
    }
}
```

## <a name="powershell-deployment"></a>Implementación de PowerShell

El comando `Set-AzVMExtension` puede utilizarse para implementar la extensión de máquina virtual del agente de Log Analytics en una máquina virtual existente. Antes de ejecutar el comando, las configuraciones públicas y privadas deben estar almacenadas en una tabla hash de PowerShell. 

```powershell
$PublicSettings = @{"workspaceId" = "myWorkspaceId"}
$ProtectedSettings = @{"workspaceKey" = "myWorkspaceKey"}

Set-AzVMExtension -ExtensionName "Microsoft.EnterpriseCloud.Monitoring" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS 
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y soporte técnico

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante el módulo Azure PowerShell. Para ver el estado de implementación de las extensiones de una VM determinada, ejecute el comando siguiente con el módulo de Azure PowerShell.

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

El resultado de la ejecución de las extensiones se registra en los archivos que se encuentran en el siguiente directorio:

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\
```

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
