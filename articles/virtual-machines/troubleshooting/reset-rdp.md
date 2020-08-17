---
title: Restablecer los Servicios de Escritorio remoto o su contraseña de administrador en una máquina virtual Windows | Microsoft Docs
description: Aprenda a restablecer la contraseña de una cuenta o los Servicios de Escritorio remoto en una máquina virtual Windows mediante Azure Portal o Azure PowerShell.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 45c69812-d3e4-48de-a98d-39a0f5675777
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 03/25/2019
ms.author: genli
ms.openlocfilehash: 80bcac36271d86334e11cb3cd161cd96777e6fe5
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832457"
---
# <a name="reset-remote-desktop-services-or-its-administrator-password-in-a-windows-vm"></a>Restablecer los Servicios de Escritorio remoto o su contraseña de administrador en una máquina virtual Windows
Si no puede conectarse a una máquina virtual Windows, puede restablecer la contraseña de administrador local o la configuración de los Servicios de Escritorio remoto (esto no se admite en los controladores de dominio de Windows). Para restablecer la contraseña, use Azure Portal o la extensión de acceso de máquina virtual en Azure PowerShell. Una vez haya iniciado sesión en la máquina virtual, restablezca la contraseña del administrador local.  
Si usa PowerShell, asegúrese de tener [instalado y configurado el módulo de PowerShell más reciente](/powershell/azure/) y de haber iniciado sesión en su suscripción a Azure. También puede [realizar estos pasos para máquinas virtuales creadas con el modelo de implementación clásica](/previous-versions/azure/virtual-machines/windows/classic/reset-rdp).

Puede restablecer los Servicios de Escritorio remoto y las credenciales de las siguientes maneras:

- [Restablecimiento mediante Azure Portal](#reset-by-using-the-azure-portal)

- [Restablecimiento mediante la extensión de VMAccess y PowerShell](#reset-by-using-the-vmaccess-extension-and-powershell)

## <a name="reset-by-using-the-azure-portal"></a>Restablecimiento mediante Azure Portal

En primer lugar, inicie sesión en [Azure Portal](https://portal.azure.com) y, en el menú izquierdo, haga clic en **Máquinas virtuales**. 

### <a name="reset-the-local-administrator-account-password"></a>**Restablecer la contraseña de cuenta de administración local**

1. Seleccione su máquina virtual Windows y, a continuación, seleccione **Restablecer contraseña** en **Support + Troubleshooting** (Soporte técnico y solución de problemas). Aparecerá la ventana para **restablecer la contraseña**.

2. Seleccione la opción **Restablecer contraseña**, escriba un nombre de usuario y una contraseña, y seleccione **Actualizar**. 

3. Trate de conectarse de nuevo a la máquina virtual.

### <a name="reset-the-remote-desktop-services-configuration"></a>**Restablecer la configuración de los Servicios de Escritorio remoto**

Este proceso habilitará el servicio de Escritorio remoto en la máquina virtual y creará una regla de firewall para el puerto RDP predeterminado 3389.

1. Seleccione su máquina virtual Windows y, a continuación, seleccione **Restablecer contraseña** en **Support + Troubleshooting** (Soporte técnico y solución de problemas). Aparecerá la ventana para **restablecer la contraseña**. 

2. Seleccione **Reset configuration only** (Restablecer solo la configuración) y, a continuación, seleccione **Actualizar**. 

3. Trate de conectarse de nuevo a la máquina virtual.

## <a name="reset-by-using-the-vmaccess-extension-and-powershell"></a>Restablecimiento mediante la extensión de VMAccess y PowerShell

En primer lugar, asegúrese de tener [instalado y configurado el módulo de PowerShell más reciente](/powershell/azure/) y de haber iniciado sesión en su suscripción de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

### <a name="reset-the-local-administrator-account-password"></a>**Restablecer la contraseña de cuenta de administración local**

- Restablezca la contraseña de administrador o el nombre de usuario con el comando de PowerShell [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension). La configuración de `typeHandlerVersion` debe ser 2.0 o posterior, ya que la versión 1 está en desuso. 

    ```powershell
    $SubID = "<SUBSCRIPTION ID>" 
    $RgName = "<RESOURCE GROUP NAME>" 
    $VmName = "<VM NAME>" 
    $Location = "<LOCATION>" 
 
    Connect-AzAccount 
    Select-AzSubscription -SubscriptionId $SubID 
    Set-AzVMAccessExtension -ResourceGroupName $RgName -Location $Location -VMName $VmName -Credential (get-credential) -typeHandlerVersion "2.0" -Name VMAccessAgent 
    ```

    > [!NOTE] 
    > Si escribe un nombre distinto al de la cuenta de administrador local actual en la máquina virtual, la extensión VMAccess agregará una cuenta de administrador local con ese nombre y asignará la contraseña especificada a esa cuenta. Si la cuenta de administrador local en la máquina virtual ya existe, la extensión VMAccess la restablecerá. Si la cuenta está deshabilitada, la extensión VMAccess la habilitará de nuevo.

### <a name="reset-the-remote-desktop-services-configuration"></a>**Restablecer la configuración de los Servicios de Escritorio remoto**

1. Restablezca el acceso remoto a la VM con el cmdlet de PowerShell [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension). En el ejemplo siguiente se habilita se restablece la extensión de acceso llamada `myVMAccess` en la máquina virtual denominada `myVM` en el grupo de recursos `myResourceGroup`:

    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResoureGroup" -VMName "myVM" -Name "myVMAccess" -Location WestUS -typeHandlerVersion "2.0" -ForceRerun
    ```

    > [!TIP]
    > En cualquier momento, una VM puede tener un solo agente de acceso a VM. Para establecer las propiedades del agente de acceso a la máquina virtual, use la opción `-ForceRerun`. Cuando use `-ForceRerun`, asegúrese de usar el mismo nombre que haya usado en cualquier comando anterior para el agente de acceso a la máquina virtual.

1. Si sigue sin poder conectarse de forma remota a la máquina virtual, consulte [Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure con Windows](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Si pierde la conexión con el controlador de dominio de Windows, deberá restaurarla a partir de una copia de seguridad del controlador de dominio.

## <a name="next-steps"></a>Pasos siguientes

- Si la extensión de acceso de la máquina virtual de Azure no responde y no puede restablecer la contraseña, puede [restablecer la contraseña de Windows local sin tener conexión](reset-local-password-without-agent.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Este método es un proceso más avanzado y requiere que se conecte el disco duro virtual de la máquina virtual problemática a otra máquina virtual. Si los pasos indicados no surten efecto, primero siga los pasos descritos en este artículo e intente realizar solamente el método de restablecimiento de contraseña sin conexión.

- [Más información sobre las características y extensiones de la máquina virtual de Azure](../extensions/features-windows.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

- [Conexión a una máquina virtual de Azure con RDP o SSH](/previous-versions/azure/dn535788(v=azure.100))

- [Solucionar problemas de conexiones de Escritorio remoto a una máquina virtual de Azure basada en Windows](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)