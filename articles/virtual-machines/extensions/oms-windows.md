---
title: Extensión de máquina virtual de Log Analytics para Windows
description: Implemente el agente de Log Analytics en la máquina virtual Windows con una extensión de máquina virtual.
services: virtual-machines-windows
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: feae6176-2373-4034-b5d9-a32c6b4e1f10
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/26/2020
ms.author: akjosh
ms.openlocfilehash: 501681a17603fb21e073b04e48d23f1b566f8128
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89067695"
---
# <a name="log-analytics-virtual-machine-extension-for-windows"></a>Extensión de máquina virtual de Log Analytics para Windows

Los registros de Azure Monitor proporcionan funcionalidades de supervisión de recursos locales y en la nube. Microsoft, como editor de la extensión de máquina virtual del agente de Log Analytics para Windows, es quien presta los servicios de soporte técnico para esta solución. La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics. En este documento se especifican las plataformas compatibles, configuraciones y opciones de implementación de la extensión de máquina virtual de Log Analytics para Windows.

## <a name="prerequisites"></a>Requisitos previos

### <a name="operating-system"></a>Sistema operativo

Para obtener más información acerca de los sistemas operativos Windows admitidos, consulte el artículo [Información general sobre los agentes de Azure Monitor](../../azure-monitor/platform/agents-overview.md#supported-operating-systems).

### <a name="agent-and-vm-extension-version"></a>Versión de extensión de agente y máquina virtual
En la tabla siguiente se proporciona una asignación de la versión de la extensión de VM de Windows Log Analytics y el paquete del agente de Log Analytics para cada versión. 

| Versión del conjunto de productos Windows para Log Analytics | Versión de extensión de VM Windows de Log Analytics | Fecha de la versión | Notas de la versión |
|--------------------------------|--------------------------|--------------------------|--------------------------|
| 10.20.18038 | 1.0.18040.2 | Agosto de 2020   | <ul><li>Resuelve un problema en Azure Arc</li></ul> |
| 10.20.18038 | 1.0.18038 | Abril de 2020   | <ul><li>Permite la conectividad a través de Private Link mediante ámbitos de Private Link de Azure Monitor.</li><li>Agrega la limitación de ingesta para evitar un influjo repentino y accidental de ingesta en un área de trabajo.</li><li>Agrega compatibilidad con regiones y nubes de Azure Government adicionales.</li><li>Resuelve un error en el que se bloqueaba HealthService.exe.</li></ul> |
| 10.20.18029 | 1.0.18029 | Marzo de 2020   | <ul><li>Agrega compatibilidad con la firma de código SHA-2.</li><li>Mejora la instalación y administración de extensiones de máquina virtual.</li><li>Resuelve un error en Azure Arc de integración de servidores.</li><li>Agrega una herramienta de solución de problemas integrada de soporte al cliente.</li><li>Agrega compatibilidad con más regiones de Azure Government.</li> |
| 10.20.18018 | 1.0.18018 | Octubre de 2019 | <ul><li> Correcciones de errores menores y mejoras de estabilización </li></ul> |
| 10.20.18011 | 1.0.18011 | Julio de 2019 | <ul><li> Correcciones de errores menores y mejoras de estabilización </li><li> Aumento de MaxExpressionDepth a 10 000 </li></ul> |
| 10.20.18001 | 1.0.18001 | Junio de 2019 | <ul><li> Correcciones de errores menores y mejoras de estabilización </li><li> Se agregó la posibilidad de deshabilitar las credenciales predeterminadas al realizar la conexión de proxy (compatibilidad con WINHTTP_AUTOLOGON_SECURITY_LEVEL_HIGH). </li></ul>|
| 10.19.13515 | 1.0.13515 | Marzo de 2019 | <ul><li>Correcciones menores de estabilización </li></ul> |
| 10.19.10006 | N/D | Diciembre de 2018 | <ul><li> Correcciones menores de estabilización </li></ul> | 
| 8.0.11136 | N/D | Septiembre de 2018 |  <ul><li> Se agregó compatibilidad con la detección de cambios en los identificadores de recursos al migrar las máquinas virtuales. </li><li> Se agregó compatibilidad con la notificación de identificadores de recursos cuando se usa una instalación sin extensiones. </li></ul>| 
| 8.0.11103 | N/D |  Abril de 2018 | |
| 8.0.11081 | 1.0.11081 | Noviembre de 2017 | | 
| 8.0.11072 | 1.0.11072 | Septiembre de 2017 | |
| 8.0.11049 | 1.0.11049 | Febrero de 2017 | |


### <a name="azure-security-center"></a>Azure Security Center

Azure Security Center aprovisiona automáticamente el agente de Log Analytics y lo conecta con el área de trabajo predeterminada de Log Analytics de la suscripción de Azure. Si utiliza Azure Security Center, no siga los pasos de este documento. Si lo hace, sobrescribirá el área de trabajo configurada e interrumpirá la conexión con Azure Security Center.

### <a name="internet-connectivity"></a>Conectividad de Internet
La extensión del agente de Log Analytics para Windows requiere que la máquina virtual de destino esté conectada a Internet. 

## <a name="extension-schema"></a>Esquema de extensión

En el siguiente JSON se muestra el esquema para la extensión del agente de Log Analytics. La extensión requiere el identificador y la clave del área de trabajo de destino de Log Analytics. Se pueden encontrar en la configuración del área de trabajo en Azure Portal. Como la clave del área de trabajo debe tratarse como datos confidenciales, debe almacenarse en una configuración protegida. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino. Tenga en cuenta que **workspaceId** y **workspaceKey** distinguen mayúsculas de minúsculas.

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

| Nombre | Valor / ejemplo |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.EnterpriseCloud.Monitoring |
| type | MicrosoftMonitoringAgent |
| typeHandlerVersion | 1.0 |
| workspaceId (e.g)* | 6f680a37-00c6-41c7-a93f-1437e3462574 |
| workspaceKey (p. ej.) | z4bU3p1/GrnWpQkky4gdabWXAhbWSTz70hm4m2Xt92XI+rSRgE8qVvRhsGo9TXffbrTahyrwv35W0pOqQAU7uQ== |

\* El identificador del área de trabajo se denomina identificador de consumidor en la API de Log Analytics.

> [!NOTE]
> Para obtener más propiedades, vea [Conexión de equipos Windows a Azure Monitor](../../azure-monitor/platform/agent-windows.md) de Azure.

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager para ejecutar el agente de Log Analytics durante la implementación de una plantilla de Azure Resource Manager. En la [Galería de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-oms-extension-windows-vm) encontrará una plantilla de ejemplo que incluye la extensión de VM del agente de Log Analytics. 

>[!NOTE]
>La plantilla no admite la especificación de más de un identificador y clave de área de trabajo cuando quiere configurar el agente para informar a varias áreas de trabajo. Para configurar el agente para que informe a varias áreas de trabajo, consulte [Adición o eliminación de un área de trabajo](../../azure-monitor/platform/agent-manage.md#adding-or-removing-a-workspace).  

El JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual, o colocada en la raíz o un nivel superior de una plantilla JSON de Resource Manager. La colocación de la plantilla JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md). 

En el siguiente ejemplo se da por supuesto que la extensión de Log Analytics está anidada dentro de los recursos de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto `"resources": []` de la máquina virtual.


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

Set-AzVMExtension -ExtensionName "MicrosoftMonitoringAgent" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS 
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

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
