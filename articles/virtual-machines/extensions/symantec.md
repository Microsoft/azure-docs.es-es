---
title: Instalación de Symantec Endpoint Protection en una máquina virtual Windows en Azure
description: Aprenda a instalar y configurar la extensión de seguridad de Symantec Endpoint Protection en una máquina virtual de Azure nueva o existente creada con el modelo de implementación clásica.
author: roiyz
tags: azure-service-management
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: article
ms.date: 03/31/2017
ms.author: akjosh
ms.openlocfilehash: a89ec99af02b32d452fba1dffb0387ae6013a7c8
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292435"
---
# <a name="how-to-install-and-configure-symantec-endpoint-protection-on-a-windows-vm"></a>Instalación y configuración de Endpoint Protection en una máquina virtual de Azure
[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Azure tiene dos modelos de implementación diferentes para crear recursos y trabajar con ellos: [Resource Manager y el clásico](../../azure-resource-manager/management/deployment-models.md). En este artículo se trata el modelo de implementación clásico. Microsoft recomienda que las implementaciones más recientes usen el modelo de Resource Manager.

En este artículo se muestra cómo instalar y configurar el cliente Symantec Endpoint Protection en una máquina virtual existente con Windows Server. Este es el cliente completo, que incluye servicios como protección contra virus y spyware, firewall y prevención de intrusiones. El cliente se instala como una extensión de seguridad usando el Agente de máquina virtual.

Si tiene una suscripción existente de Symantec para una solución local, puede usarla para proteger sus máquinas virtuales de Azure. Si todavía no es cliente, puede suscribirse para una prueba. Para obtener más información sobre esta solución, consulte la información sobre [Symantec Endpoint Protection en la plataforma de Microsoft Azure][Symantec]. Esta página también tiene vínculos a información de licencia e instrucciones para instalar el cliente si ya es un cliente de Symantec.

## <a name="install-symantec-endpoint-protection-on-an-existing-vm"></a>Instalación de Symantec Endpoint Protection en una máquina virtual existente
Antes de comenzar, necesitará lo siguiente:

* El módulo Azure PowerShell, versión 0.8.2 o posteriores, en el equipo de trabajo. Puede comprobar la versión de Azure PowerShell que ha instalado con el comando **Get-Module azure | format-table version** . Para obtener instrucciones y un vínculo a la versión más reciente, consulte [Instalación y configuración de Azure PowerShell][PS]. Inicie sesión en la suscripción de Azure mediante `Add-AzureAccount`.
* El agente de máquina virtual que se ejecuta en la máquina virtual de Azure.

En primer lugar, compruebe que el agente de máquina virtual ya está instalado en la máquina virtual. Introduzca el nombre de servicio de nube y el nombre de la máquina virtual y, a continuación, ejecute los siguientes comandos en un símbolo de sistema de Azure PowerShell con nivel de administrador. Reemplace todo el contenido dentro de las comillas, incluidos los caracteres < y >.

> [!TIP]
> Si no conoce los nombres del servicio en la nube y de la máquina virtual, ejecute **Get-AzureVM** para mostrar los nombres de todas las máquinas virtuales de su suscripción actual.

```powershell
$CSName = "<cloud service name>"
$VMName = "<virtual machine name>"
$vm = Get-AzureVM -ServiceName $CSName -Name $VMName
write-host $vm.VM.ProvisionGuestAgent
```

Si el comando **write-host** muestra **True**, el agente de la máquina virtual está instalado. Si muestra **False**, consulte las instrucciones y un vínculo a la descarga en la publicación del blog de Azure [VM Agent and Extensions - Part 2 (Agente de máquina virtual extensiones: parte 2)][Agent].

Si está instalado el agente de VM, ejecute estos comandos para instalar al agente de Symantec Endpoint Protection.

```powershell
$Agent = Get-AzureVMAvailableExtension -Publisher Symantec -ExtensionName SymantecEndpointProtection

Set-AzureVMExtension -Publisher Symantec –Version $Agent.Version -ExtensionName SymantecEndpointProtection \
    -VM $vm | Update-AzureVM
```

Para comprobar que la extensión de seguridad de Symantec se ha instalado y está actualizada:

1. Iniciar sesión en la nueva máquina virtual. Para obtener instrucciones, vea [Iniciar sesión en una máquina virtual con Windows Server][Logon].
2. Para Windows Server 2008 R2, haga clic en **Inicio > Symantec Endpoint Protection**. Para Windows Server 2012 o Windows Server 2012 R2, en la pantalla de inicio, escriba **Symantec** y luego haga clic en **Symantec Endpoint Protection**.
3. En la pestaña **Estado** de la ventana **Status-Symantec Endpoint Protection**, aplique actualizaciones o reinicie en caso necesario.

## <a name="additional-resources"></a>Recursos adicionales
[Inicio de sesión en una máquina virtual con Windows Server][Logon]

[Características y extensiones de máquina virtual de Azure][Ext]

<!--Link references-->
[Symantec]: https://www.symantec.com/connect/blogs/symantec-endpoint-protection-now-microsoft-azure

[Create]:../windows/classic/tutorial.md

[PS]: /powershell/azure/

[Agent]: https://go.microsoft.com/fwlink/p/?LinkId=403947

[Logon]:../windows/classic/connect-logon.md

[Ext]: features-windows.md
