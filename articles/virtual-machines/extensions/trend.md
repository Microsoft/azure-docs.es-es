---
title: Instalación de Trend Micro Deep Security en una máquina virtual
description: En este artículo se describe cómo instalar y configurar la seguridad de Trend Micro en una máquina virtual creada con el modelo de implementación clásica en Azure.
author: axayjo
tags: azure-service-management
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: article
ms.date: 04/20/2018
ms.author: akjosh
ms.openlocfilehash: bed1d567aa48a11c01ae952e6a4a2028e260e6e0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288525"
---
# <a name="how-to-install-and-configure-trend-micro-deep-security-as-a-service-on-a-windows-vm"></a>Instalación y configuración de Trend Micro Deep Security como servicio en una máquina virtual de Azure

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]
En este artículo se muestra cómo instalar y configurar Trend Micro Deep Security como servicio en una máquina virtual nueva o existente con Windows Server. Deep Security como servicio proporciona protección antimalware, firewall, sistema de prevención contra intrusiones y supervisión de integridad.

El cliente se instala como una extensión de seguridad usando el Agente de máquina virtual. En una nueva máquina virtual, instale el agente de Deep Security, ya que Azure Portal crea automáticamente el agente de máquina virtual.

Es posible que una máquina virtual existente que se creó mediante Azure Portal, la CLI de Azure o PowerShell no tenga un agente de máquina virtual. En una máquina virtual que no tenga el agente de máquina virtual, primero debe descargarlo e instalarlo. Este artículo trata ambas situaciones.

Si tiene una suscripción actual de Trend Micro para una solución local, puede usarla para ayudar a proteger sus máquinas virtuales de Azure. Si todavía no es cliente, puede suscribirse para una prueba. Para obtener más información acerca de esta solución, consulte la publicación del blog de Trend Micro [Microsoft Azure VM Agent Extension For Deep Security (Extensión del agente de máquina virtual de Microsoft Azure para Deep Security)](https://go.microsoft.com/fwlink/p/?LinkId=403945).

## <a name="install-the-deep-security-agent-on-a-new-vm"></a>Instalación del Agente de Deep Security en una máquina virtual nueva

[Azure Portal](https://portal.azure.com) le permite instalar la extensión de seguridad de Trend Micro cuando usa una imagen de **Marketplace** para crear la máquina virtual. Si crea una sola máquina virtual, usar el portal es una forma simple para agregar la protección de Trend Micro.

Mediante una entrada de **Marketplace** se abre un asistente que le ayuda a configurar la máquina virtual. Utilice la hoja **Configuración**, el tercer panel del asistente, para instalar la extensión de seguridad de Trend Micro.  Para obtener instrucciones generales, consulte [Creación de una máquina virtual que ejecuta Windows en Azure Portal](../windows/quick-create-portal.md).

Cuando se encuentre en la hoja **Configuración** del asistente, realice los pasos siguientes:

1. Haga clic en **Extensiones**, a continuación, haga clic en **Agregar extensión** en el panel siguiente.

   ![Empezar a agregar la extensión][1]

2. Seleccione **Agente de Deep Security** en el panel **Nuevo recurso**. En el panel de Agente de Deep Security, haga clic en **Crear**.

   ![Identificar agente de Deep Security][2]

3. Escriba el **identificador del inquilino** y la **contraseña de activación del inquilino** para la extensión. Opcionalmente, puede especificar un **identificador de directiva de seguridad**. A continuación, haga clic en **Aceptar** para agregar el cliente.

   ![Proporcionar detalles de la extensión][3]

## <a name="install-the-deep-security-agent-on-an-existing-vm"></a>Instalación del Agente de Deep Security en una máquina virtual existente
Para instalar el agente en una máquina virtual existente, necesitará los siguientes elementos:

* El módulo Azure PowerShell, con una versión 0.8.2 o posterior, instalado en el equipo local. Puede comprobar la versión de Azure PowerShell que ha instalado con el comando **Get-Module azure | format-table version** . Para obtener instrucciones y un vínculo a la versión más reciente, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/). Inicie sesión en la suscripción de Azure mediante `Add-AzureAccount`.
* El agente de máquina virtual instalado en la MV de destino.

En primer lugar, compruebe que el agente de máquina virtual ya está instalado. Introduzca el nombre de servicio de nube y el nombre de la máquina virtual y, a continuación, ejecute los siguientes comandos en un símbolo de sistema de Azure PowerShell con nivel de administrador. Reemplace todo el contenido dentro de las comillas, incluidos los caracteres < y >.

```azurepowershell
$CSName = "<cloud service name>"
$VMName = "<virtual machine name>"
$vm = Get-AzureVM -ServiceName $CSName -Name $VMName
write-host $vm.VM.ProvisionGuestAgent
```

Si no conoce el nombre del servicio en la nube y de la máquina virtual, ejecute **Get-AzureVM** para mostrar esa información para todas las máquinas virtuales de su suscripción actual.

Si el comando **write-host** devuelve **True**, el agente de máquina virtual está instalado. Si devuelve **False**, consulte las instrucciones y un vínculo a la descarga en la publicación del blog de Azure [VM Agent and Extensions - Part 2 (Agente de máquina virtual extensiones: parte 2)](https://go.microsoft.com/fwlink/p/?LinkId=403947).

Si el agente de máquina virtual está instalado, ejecute estos comandos.

```azurepowershell
$Agent = Get-AzureVMAvailableExtension TrendMicro.DeepSecurity -ExtensionName TrendMicroDSA

Set-AzureVMExtension -Publisher TrendMicro.DeepSecurity –Version $Agent.Version -ExtensionName TrendMicroDSA -VM $vm | Update-AzureVM
```

## <a name="next-steps"></a>Pasos siguientes
El agente tarda unos minutos en empezar la ejecución cuando se instala. Después de ello, debe activar Deep Security en la máquina virtual de forma que pueda ser administrado por Deep Security Manager. Consulte los siguientes artículos para obtener instrucciones adicionales:

* Artículo de Trend sobre esta solución, [Instant-On Cloud Security for Microsoft Azure (Seguridad en la nube instantánea para Microsoft Azure)](https://go.microsoft.com/fwlink/?LinkId=404101)
* [Script de Windows PowerShell de ejemplo](https://go.microsoft.com/fwlink/?LinkId=404100) para configurar la máquina virtual
* [Instrucciones](https://go.microsoft.com/fwlink/?LinkId=404099) para el ejemplo

## <a name="additional-resources"></a>Recursos adicionales
[Inicio de sesión en una máquina virtual con Windows Server]

[Características y extensiones de máquina virtual de Azure]

<!-- Image references -->
[1]: ./media/trend/new_vm_Blade3.png
[2]: ./media/trend/find_SecurityAgent.png
[3]: ./media/trend/SecurityAgentDetails.png

<!-- Link references -->
[Inicio de sesión en una máquina virtual con Windows Server]:../windows/classic/connect-logon.md
[Características y extensiones de máquina virtual de Azure]: features-windows.md
