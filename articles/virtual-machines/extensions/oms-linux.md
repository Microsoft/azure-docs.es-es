---
title: Extensión de máquina virtual de Log Analytics para Linux
description: Implemente el agente de Log Analytics en la máquina virtual Linux con una extensión de máquina virtual.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: c7bbf210-7d71-4a37-ba47-9c74567a9ea6
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/18/2020
ms.author: akjosh
ms.openlocfilehash: d7ebb23cfc0ffdcfcac7ecb4ecbd9b9a665e6130
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029100"
---
# <a name="log-analytics-virtual-machine-extension-for-linux"></a>Extensión de máquina virtual de Log Analytics para Linux

## <a name="overview"></a>Información general

Los registros de Azure Monitor proporcionan funcionalidades de supervisión, generación de alertas y corrección de alertas a los recursos locales y en la nube. Microsoft, como editor de la extensión de máquina virtual de Log Analytics para Linux, es quien presta los servicios de soporte técnico para esta solución. La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics. En este documento se especifican las plataformas compatibles, configuraciones y opciones de implementación de la extensión de máquina virtual de Log Analytics para Linux.

>[!NOTE]
>Como parte de la transición en curso de Microsoft Operations Management Suite (OMS) a Azure Monitor, se hará referencia al agente de OMS para Windows o Linux como el agente de Log Analytics para Windows y el agente de Log Analytics para Linux.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Requisitos previos

### <a name="operating-system"></a>Sistema operativo

Para más información sobre las distribuciones de Linux admitidas, consulte el artículo [Introducción al agente de Log Analytics](../../azure-monitor/platform/log-analytics-agent.md#supported-linux-operating-systems).

### <a name="agent-and-vm-extension-version"></a>Versión de extensión de agente y máquina virtual
En la tabla siguiente se proporciona una asignación de la versión de la extensión de VM de Log Analytics y el paquete del agente de Log Analytics para cada versión. Se incluye un vínculo a las notas de la versión del paquete del agente de Log Analytics. Las notas de versión incluyen detalles sobre corrección de errores y nuevas características que están disponibles para una versión de agente determinada.  

| Versión de extensión de máquina virtual Linux de Log Analytics | Versión del paquete del agente de Log Analytics | 
|--------------------------------|--------------------------|
| 1.13.13 | [1.13.7-0](https://github.com/microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.13.7-0) |
| 1.12.25 | [1.12.15-0](https://github.com/microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.12.15-0) |
| 1.11.15 | [1.11.0-9](https://github.com/microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.11.0-9) |
| 1.10.0 | [1.10.0-1](https://github.com/microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.10.0-1) |
| 1.9.1 | [1.9.0-0](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.9.0-0) |
| 1.8.11 | [1.8.1-256](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.8.1.256)| 
| 1.8.0 | [1.8.0-256](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/1.8.0-256)| 
| 1.7.9 | [1.6.1-3](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.6.1.3)| 
| 1.6.42.0 | [1.6.0-42](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.6.0-42)| 
| 1.4.60.2 | [1.4.4-210](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.4-210)| 
| 1.4.59.1 | [1.4.3-174](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.3-174)|
| 1.4.58.7 | [14.2-125](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.2-125)|
| 1.4.56.5 | [1.4.2-124](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.2-124)|
| 1.4.55.4 | [1.4.1-123](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.1-123)|
| 1.4.45.3 | [1.4.1-45](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.1-45)|
| 1.4.45.2 | [1.4.0-45](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.0-45)|
| 1.3.127.5 | [1.3.5-127](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201705-v1.3.5-127)| 
| 1.3.127.7 | [1.3.5-127](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201705-v1.3.5-127)|
| 1.3.18.7 | [1.3.4-15](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201704-v1.3.4-15)|  

### <a name="azure-security-center"></a>Azure Security Center

Azure Security Center aprovisiona automáticamente el agente de Log Analytics y lo conecta con el área de trabajo de Log Analytics predeterminada creada por ASC en su suscripción a Azure. Si utiliza Azure Security Center, no siga los pasos de este documento. Si lo hace, sobrescribirá el área de trabajo configurada e interrumpirá la conexión con Azure Security Center.

### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión del agente de Log Analytics para Linux requiere que la máquina virtual de destino esté conectada a Internet. 

## <a name="extension-schema"></a>Esquema de extensión

El siguiente JSON muestra el esquema para la extensión del agente de Log Analytics. La extensión requiere el identificador y la clave del área de trabajo de Log Analytics de destino, valores que se pueden [encontrar en el área de trabajo de Log Analytics](../../azure-monitor/learn/quick-collect-linux-computer.md#obtain-workspace-id-and-key) en Azure Portal. Como la clave del área de trabajo debe tratarse como datos confidenciales, debe almacenarse en una configuración protegida. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino. Tenga en cuenta que **workspaceId** y **workspaceKey** distinguen mayúsculas de minúsculas.

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "OMSExtension",
  "apiVersion": "2018-06-01",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.13",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

>[!NOTE]
>En el esquema anterior se da por supuesto que se colocará en el nivel raíz de la plantilla. Si lo coloca en el recurso de máquina virtual en la plantilla, debe cambiar las propiedades `type` y `name` tal como se describe [a continuación](#template-deployment).

### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo |
| ---- | ---- |
| apiVersion | 2018-06-01 |
| publisher | Microsoft.EnterpriseCloud.Monitoring |
| type | OmsAgentForLinux |
| typeHandlerVersion | 1.7 |
| workspaceId (p.ej) | 6f680a37-00c6-41c7-a93f-1437e3462574 |
| workspaceKey (p. ej.) | z4bU3p1/GrnWpQkky4gdabWXAhbWSTz70hm4m2Xt92XI+rSRgE8qVvRhsGo9TXffbrTahyrwv35W0pOqQAU7uQ== |


## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas resultan ideales cuando se implementan una o varias máquinas virtuales que requieren configuración tras la implementación, como por ejemplo, su incorporación a los registros de Azure Monitor. Puede encontrar una plantilla de Resource Manager de ejemplo que incluye la extensión de máquina virtual del agente de Log Analytics en la [Galería de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-oms-extension-ubuntu-vm). 

La configuración JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual o colocada en la raíz o nivel superior de una plantilla JSON de Resource Manager. La colocación de la configuración JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md). 

En el siguiente ejemplo se da por supuesto que la extensión de VM está anidada dentro de los recursos de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto `"resources": []` de la máquina virtual.

```json
{
  "type": "extensions",
  "name": "OMSExtension",
  "apiVersion": "2018-06-01",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.7",
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

Al colocar la plantilla JSON de la extensión en la raíz de la plantilla, el nombre de recurso incluye una referencia a la máquina virtual principal, y el tipo refleja la configuración anidada.  

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "<parentVmResource>/OMSExtension",
  "apiVersion": "2018-06-01",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.7",
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

La CLI de Azure puede utilizarse para implementar la extensión de máquina virtual del agente de Log Analytics en una máquina virtual. Reemplace el valor *myWorkspaceKey* siguiente por la clave del área de trabajo y el valor *myWorkspaceId* por el identificador del área de trabajo. Estos valores se pueden encontrar en el área de trabajo de Log Analytics en Azure Portal en *Configuración avanzada*. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name OmsAgentForLinux \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --version 1.10.1 --protected-settings '{"workspaceKey":"myWorkspaceKey"}' \
  --settings '{"workspaceId":"myWorkspaceId"}'
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

El resultado de la ejecución de las extensiones se registra en el archivo siguiente:

```
/opt/microsoft/omsagent/bin/stdout
```

### <a name="error-codes-and-their-meanings"></a>Códigos de error y su significado

| Código de error | Significado | Acción posible |
| :---: | --- | --- |
| 9 | Habilitar llamado antes de tiempo | [Actualice el agente de Linux de Azure](./update-linux-agent.md) a la versión más reciente disponible. |
| 10 | La máquina virtual ya está conectada a un área de trabajo de Log Analytics | Para conectar la máquina virtual al área de trabajo especificada en el esquema de extensión, establezca stopOnMultipleConnections en false en la configuración pública o quite esta propiedad. Esta máquina virtual se factura una vez por cada área de trabajo a la que se conecta. |
| 11 | Configuración no válida proporcionada a la extensión | Siga los ejemplos anteriores para configurar todos los valores de propiedad necesarios para la implementación. |
| 17 | Error al instalar el paquete de Log Analytics | 
| 19 | Error al instalar el paquete de OMI | 
| 20 | Error al instalar el paquete de SCX |
| 51 | Esta extensión no se admite en el sistema operativo de la máquina virtual | |
| 55 | No se puede conectar al servicio Azure Monitor o faltan los paquetes necesarios o el administrador de paquetes dpkg está bloqueado| Compruebe que el sistema tiene acceso a Internet o que se ha proporcionado un servidor proxy HTTP válido. Además, compruebe la validez del identificador del área de trabajo y compruebe que las utilidades curl y tar está instaladas. |

Puede encontrar información adicional para solucionar problemas en la [Guía de solución de problemas del agente de Log Analytics para Linux](../../azure-monitor/platform/vmext-troubleshoot.md).

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
