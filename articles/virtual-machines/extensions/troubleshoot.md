---
title: Solución de errores de las extensiones de máquina virtual de Windows
description: Más información sobre la solución de problemas de la extensión de máquina virtual de Microsoft Azure
services: virtual-machines-windows
documentationcenter: ''
author: kundanap
manager: gwallace
editor: ''
tags: top-support-issue,azure-resource-manager
ms.assetid: 878ab9b6-c3e6-40be-82d4-d77fecd5030f
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/29/2016
ms.author: kundanap
ms.openlocfilehash: ad3197f20428ec751b4e3520af72dc5f8eb9ad28
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180362"
---
# <a name="troubleshooting-azure-windows-vm-extension-failures"></a>Solución de problemas de la extensión de máquina virtual de Microsoft Azure.
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## <a name="viewing-extension-status"></a>Consulta del estado de la extensión
Las plantillas de Azure Resource Manager se pueden ejecutar desde Azure PowerShell. Cuando se ejecute la plantilla, el estado de extensión se puede ver desde el Explorador de recursos de Azure o las herramientas de la línea de comandos.

Este es un ejemplo:

Azure PowerShell:

```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

Este es la salida de ejemplo:

```output
Extensions:  {
  "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
  "Name": "myCustomScriptExtension",
  "SubStatuses": [
    {
      "Code": "ComponentStatus/StdOut/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
          \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
          test.txt                          \\n\\n",
                  "Time": null
      },
    {
      "Code": "ComponentStatus/StdErr/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "",
      "Time": null
    }
  ]
}
```

## <a name="troubleshooting-extension-failures"></a>Solución de problemas de errores de extensión

### <a name="rerun-the-extension-on-the-vm"></a>Volver a ejecutar la extensión en la máquina virtual
Si está ejecutando los scripts en la máquina virtual mediante la extensión de script personalizada, a veces podría recibir un error en el que la máquina virtual se creó correctamente, pero en el que el script ha generado un error. En estas condiciones, la manera que se recomienda para recuperarse de este error es quitar la extensión y volver a ejecutar la plantilla.
Nota: En el futuro, esta funcionalidad se mejoraría para eliminar la necesidad de desinstalar la extensión.

#### <a name="remove-the-extension-from-azure-powershell"></a>Eliminación de la extensión de Azure PowerShell
```azurepowershell
Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"
```

Cuando se ha quitado la extensión, la plantilla puede volver a ejecutarse para ejecutar los scripts en la máquina virtual.

### <a name="trigger-a-new-goalstate-to-the-vm"></a>Desencadenamiento de un nuevo GoalState en la máquina virtual
Es posible que observe que una extensión no se ha ejecutado o que no se puede ejecutar porque falta el "generador de certificados CRP de Windows Azure" (ese certificado se utiliza para proteger el transporte de la configuración protegida de la extensión).
Ese certificado se volverá a generar automáticamente al reiniciar el agente invitado de Windows desde dentro de la máquina virtual:
- Abra el Administrador de tareas.
- Vaya a la pestaña Detalles.
- Busque el proceso WindowsAzureGuestAgent.exe.
- Haga clic con el botón derecho y seleccione "Finalizar tarea". El proceso se reiniciará de forma automática.


También puede desencadenar un nuevo objeto GoalState en la máquina virtual mediante la ejecución de una "actualización vacía":

Azure PowerShell:

```azurepowershell
$vm = Get-AzureRMVM -ResourceGroupName <RGName> -Name <VMName>  
Update-AzureRmVM -ResourceGroupName <RGName> -VM $vm  
```

CLI de Azure:

```azurecli
az vm update -g <rgname> -n <vmname>
```

Si no funcionó una "actualización vacía", puede agregar un nuevo disco de datos vacío a la máquina virtual desde el portal de administración de Azure y, después, quitarlo cuando el certificado se haya agregado de nuevo.
