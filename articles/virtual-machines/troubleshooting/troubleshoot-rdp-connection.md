---
title: No es posible conectarse mediante RDP a una máquina virtual Windows en Azure | Microsoft Docs
description: Solución de problemas cuando no se puede conectar a la máquina virtual Windows en Azure mediante Escritorio remoto
keywords: Error de escritorio remoto, error de conexión del escritorio remoto, no se puede conectar a la máquina virtual, solución de problemas con el escritorio remoto
services: virtual-machines-windows
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 0d740f8e-98b8-4e55-bb02-520f604f5b18
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 03/23/2018
ms.author: akjosh
ms.openlocfilehash: 831ce1ccb2c09a85ddfff8fa65172b1871119a61
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079900"
---
# <a name="troubleshoot-remote-desktop-connections-to-an-azure-virtual-machine"></a>Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure
La conexión de Protocolo de escritorio remoto (RDP) a la máquina virtual (VM) de Azure basada en Windows puede presentar errores por varios motivos que le impedirán acceder a la máquina virtual. El problema puede originar en el servicio de Escritorio remoto de la máquina virtual, la conexión de red o el cliente de Escritorio remoto en el equipo host. Este artículo le guía por algunos de los métodos más comunes para resolver problemas de conexión de RDP. 

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.

 

<a id="quickfixrdp"></a>

## <a name="quick-troubleshooting-steps"></a>Pasos rápidos para solucionar problemas
Después de cada paso de solución de problemas, intente volver a conectarse a la máquina virtual:

1. Restablezca la configuración de Escritorio remoto.
2. Compruebe las reglas de grupo de seguridad de red/puntos de conexión de Cloud Services.
3. Revise los registros de la consola de máquina virtual.
4. Restablezca el NIC para la máquina virtual.
5. Compruebe el estado del recurso de la máquina virtual.
6. Restablezca la contraseña de la máquina virtual.
7. Reinicie la máquina virtual.
8. Vuelva a implementar la máquina virtual.

Si necesita instrucciones más detalladas y explicaciones, siga leyendo. Compruebe que el equipo de red local, como enrutadores y firewalls, no están bloqueando el puerto TCP 3389 saliente, como se indicó en [los escenarios de solución de problemas de RDP detallados](detailed-troubleshoot-rdp.md).

> [!TIP]
> Si el botón **Conectar** de la máquina virtual está atenuado en el portal y no está conectado a Azure a través de una conexión [Express Route](../../expressroute/expressroute-introduction.md) o [VPN de sitio a sitio](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), deberá crear y asignar a la máquina virtual una dirección IP pública antes de poder usar RDP. Aquí puede leer más sobre [direcciones IP públicas en Azure](../../virtual-network/public-ip-addresses.md).


## <a name="ways-to-troubleshoot-rdp-issues"></a>Formas de solucionar problemas de RDP
Puede solucionar los problemas de las máquinas virtuales creadas mediante el modelo de implementación de Resource Manager con uno de los métodos siguientes:

* Azure Portal: este método es perfecto si necesita restablecer rápidamente las credenciales de usuario o de configuración de RDP y no tiene instaladas las herramientas de Azure.
* Azure PowerShell: si está familiarizado con el símbolo del sistema de PowerShell, restablezca rápidamente las credenciales de usuario o la configuración de RDP mediante los cmdlets de Azure PowerShell.

También puede encontrar los pasos para solucionar problemas con las máquinas virtuales creadas mediante el [modelo de implementación clásica](#troubleshoot-vms-created-using-the-classic-deployment-model).

<a id="fix-common-remote-desktop-errors"></a>

## <a name="troubleshoot-using-the-azure-portal"></a>Solución de problemas mediante Azure Portal
Después de cada paso de solución de problemas, intente conectarse de nuevo a la máquina virtual. Si sigue sin poder conectarse, pruebe el paso siguiente.

1. **Restablezca la conexión RDP**. En este paso para solucionar problemas se restablece la configuración de RDP cuando las conexiones remotas están deshabilitadas o las reglas de Firewall de Windows están bloqueando el RDP, por ejemplo.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Restablecer contraseña**. Establezca el **Modo** en **Reset configuration only** (Configuración del restablecimiento solo) y, después, haga clic en el botón **Actualizar**:
   
    ![Restablecimiento de la configuración de RDP en Azure Portal](./media/troubleshoot-rdp-connection/reset-rdp.png)
2. **Compruebe las reglas del grupo de seguridad de red**. Use la [verificación del flujo IP](../../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) para confirmar si una regla en un grupo de seguridad de red está bloqueando el tráfico hacia o desde una máquina virtual. También puede revisar cómo crear reglas de grupo de seguridad eficaces para garantizar que exista la regla NSG "Permitir" de entrada y tenga prioridad para el puerto RDP (valor predeterminado 3389). Para más información, vea [Uso de las reglas de seguridad vigentes para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-traffic-filter-problem.md).

3. **Revise los diagnósticos de arranque de la máquina virtual**. En este paso para solucionar problemas se revisan los registros de la consola de la máquina virtual para determinar si la máquina virtual está notificando de un problema. No todas las máquinas virtuales tienen diagnósticos de arranque habilitados, por lo que este paso para solucionar problemas puede ser opcional.
   
    Los pasos para solucionar problemas específicos quedan fuera del ámbito de este artículo, pero pueden indicar un problema más amplio que está afectando a la conectividad RDP. Para más información acerca de cómo revisar los registros de la consola y la captura de pantalla de la máquina virtual, consulte la entrada del blog [Boot Diagnostics for VMs](boot-diagnostics.md) (Diagnósticos de arranque para máquinas virtuales).

4. **Restablezca el NIC para la máquina virtual**. Para más información, consulte [Procedimiento para establecer el NIC para la máquina virtual de Windows Azure](./reset-network-interface.md).
5. **Compruebe el estado del recurso de la máquina virtual**. En este paso para solucionar problemas se comprueba que no hay ningún problema conocido con la plataforma Azure que pueda afectar a la conectividad de la máquina virtual.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Estado de los recursos**. Una máquina virtual correcta se notifica como **Disponible**:
   
    ![Comprobación del estado del recurso de máquina virtual en Azure Portal](./media/troubleshoot-rdp-connection/check-resource-health.png)
6. **Restablezca las credenciales de usuario**. En este paso para solucionar problemas se restablece la contraseña en una cuenta de administrador local cuando no está seguro de las credenciales o las ha olvidado.  Una vez que haya iniciado sesión en la máquina virtual, debe restablecer la contraseña para ese usuario.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Restablecer contraseña**. Asegúrese de que el **Modo** está establecido en **Restablecer contraseña** y, después, escriba su nombre de usuario y una nueva contraseña. Finalmente, haga clic en el botón **Actualizar**:
   
    ![Restablecimiento de las credenciales de usuario en Azure Portal](./media/troubleshoot-rdp-connection/reset-password.png)
7. **Reinicie la máquina virtual**. En este paso para solucionar problemas se puede corregir cualquier problema subyacente que tenga la misma máquina virtual.
   
    Seleccione la máquina virtual en Azure Portal y haga clic en la pestaña **Información general**. Haga clic en el botón **Reiniciar**:
   
    ![Reinicio de máquina virtual en Azure Portal](./media/troubleshoot-rdp-connection/restart-vm.png)
8. **Vuelva a implementar la máquina virtual**. En este paso para solucionar problemas se vuelve a implementar la máquina virtual en otro host dentro de Azure para corregir cualquier problema de red o de plataforma subyacente.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Volver a implementar** y, después, haga clic en **Volver a implementar**:
   
    ![Reimplementación de la máquina virtual en Azure Portal](./media/troubleshoot-rdp-connection/redeploy-vm.png)
   
    Cuando finalice esta operación, se pierden datos de disco efímeros y se actualizan las direcciones IP dinámicas que están asociadas a la máquina virtual.

9. **Compruebe el enrutamiento**. Use la funcionalidad [Próximo salto](../../network-watcher/diagnose-vm-network-routing-problem.md) de Network Watcher para confirmar que una ruta no impide que el tráfico se enrute hacia una máquina virtual o desde esta. También puede revisar las rutas efectivas para ver todas las rutas efectivas de una interfaz de red. Para más información, consulte [Uso de rutas eficaces para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-routing-problem.md).

10. Asegúrese de que cualquier firewall local, o firewall del equipo, permite el tráfico saliente del puerto TCP 3389 a Azure.

Si sigue teniendo problemas con RDP, puede [abrir una solicitud de soporte técnico](https://azure.microsoft.com/support/options/) o leer [más pasos y conceptos detallados de solución de problemas de RDP](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-using-azure-powershell"></a>Solución de problemas mediante Azure PowerShell
Si aún no lo ha hecho, [instale y configure la última versión de Azure PowerShell](/powershell/azure/).

Los ejemplos siguientes usan variables como `myResourceGroup`, `myVM` y `myVMAccessExtension`. Reemplace estos nombres de variables y las ubicaciones por sus propios valores.

> [!NOTE]
> Restablezca las credenciales de usuario y la configuración de RDP mediante el cmdlet [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension) de PowerShell. En los ejemplos siguientes, `myVMAccessExtension` es un nombre que ha especificado como parte del proceso. Si ha trabajado previamente con VMAccessAgent, use `Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"` para obtener el nombre de la extensión existente y comprobar las propiedades de la máquina virtual. Para ver el nombre, busque en la sección "Extensiones" de la salida.

Después de cada paso de solución de problemas, intente conectarse de nuevo a la máquina virtual. Si sigue sin poder conectarse, pruebe el paso siguiente.

1. **Restablezca la conexión RDP**. En este paso para solucionar problemas se restablece la configuración de RDP cuando las conexiones remotas están deshabilitadas o las reglas de Firewall de Windows están bloqueando el RDP, por ejemplo.
   
    En el ejemplo siguiente se restablece la conexión RDP en una máquina virtual denominada `myVM` en la ubicación `WestUS` y en el grupo de recursos denominado `myResourceGroup`:
   
    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location Westus -Name "myVMAccessExtension"
    ```
2. **Compruebe las reglas del grupo de seguridad de red**. En este paso para solucionar problemas se comprueba que dispone de una regla en el grupo de seguridad de red para permitir el tráfico RDP. El puerto predeterminado para RDP es el puerto TCP 3389. No se creará automáticamente una regla que permita el tráfico RDP cuando se crea la máquina virtual.
   
    En primer lugar, asigne todos los datos de configuración para el grupo de seguridad de red a la variable `$rules`. En el ejemplo siguiente se obtiene información sobre el grupo de seguridad de red denominado `myNetworkSecurityGroup` en el grupo de recursos denominado `myResourceGroup`:
   
    ```powershell
    $rules = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" `
        -Name "myNetworkSecurityGroup"
    ```
   
    Ahora, vea las reglas que están configuradas para este grupo de seguridad de red. Compruebe que existe una regla que permite las conexiones de entrada en el puerto TCP 3389, tal como se indica a continuación:
   
    ```powershell
    $rules.SecurityRules
    ```
   
    En el ejemplo siguiente se muestra una regla de seguridad válida que permita el tráfico de RDP. Puede ver que `Protocol`, `DestinationPortRange`, `Access` y `Direction` están configurados correctamente:
   
    ```powershell
    Name                     : default-allow-rdp
    Id                       : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/default-allow-rdp
    Etag                     : 
    ProvisioningState        : Succeeded
    Description              : 
    Protocol                 : TCP
    SourcePortRange          : *
    DestinationPortRange     : 3389
    SourceAddressPrefix      : *
    DestinationAddressPrefix : *
    Access                   : Allow
    Priority                 : 1000
    Direction                : Inbound
    ```
   
    Si no dispone de una regla que permita el tráfico RDP, [cree una regla de grupo de seguridad de red](../windows/nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Permita el puerto TCP 3389.
3. **Restablezca las credenciales de usuario**. En este paso para solucionar problemas se restablece la contraseña en una cuenta de administrador local que haya especificado cuando no está seguro de las credenciales o las haya olvidado.
   
    En primer lugar, especifique el nombre de usuario y una contraseña nueva mediante la asignación de credenciales a la variable `$cred`, tal como se indica a continuación:
   
    ```powershell
    $cred=Get-Credential
    ```
   
    Ahora, actualice las credenciales en la máquina virtual. En el ejemplo siguiente se actualizan las credenciales en una máquina virtual denominada `myVM` en la ubicación `WestUS` y en el grupo de recursos `myResourceGroup`:
   
    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location WestUS -Name "myVMAccessExtension" `
        -UserName $cred.GetNetworkCredential().Username `
        -Password $cred.GetNetworkCredential().Password
    ```
4. **Reinicie la máquina virtual**. En este paso para solucionar problemas se puede corregir cualquier problema subyacente que tenga la misma máquina virtual.
   
    En el ejemplo siguiente se reinicia la máquina virtual llamada `myVM` en el grupo de recursos `myResourceGroup`:
   
    ```powershell
    Restart-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
    ```
5. **Vuelva a implementar la máquina virtual**. En este paso para solucionar problemas se vuelve a implementar la máquina virtual en otro host dentro de Azure para corregir cualquier problema de red o de plataforma subyacente.
   
    En el ejemplo siguiente se vuelve a implementar la máquina virtual denominada `myVM` en la ubicación `WestUS` y en el grupo de recursos `myResourceGroup`:
   
    ```powershell
    Set-AzVM -Redeploy -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

6. **Compruebe el enrutamiento**. Use la funcionalidad [Próximo salto](../../network-watcher/diagnose-vm-network-routing-problem.md) de Network Watcher para confirmar que una ruta no impide que el tráfico se enrute hacia una máquina virtual o desde esta. También puede revisar las rutas efectivas para ver todas las rutas efectivas de una interfaz de red. Para más información, consulte [Uso de rutas eficaces para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-routing-problem.md).

7. Asegúrese de que cualquier firewall local, o firewall del equipo, permite el tráfico saliente del puerto TCP 3389 a Azure.

Si sigue teniendo problemas con RDP, puede [abrir una solicitud de soporte técnico](https://azure.microsoft.com/support/options/) o leer [más pasos y conceptos detallados de solución de problemas de RDP](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-vms-created-using-the-classic-deployment-model"></a>Solución de problemas de máquinas virtuales creadas con el modelo de implementación clásica

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


Después de cada paso de solución de problemas, intente volver a conectarse a la máquina virtual.

1. **Restablezca la conexión RDP**. En este paso para solucionar problemas se restablece la configuración de RDP cuando las conexiones remotas están deshabilitadas o las reglas de Firewall de Windows están bloqueando el RDP, por ejemplo.
   
    Seleccione la máquina virtual en Azure Portal. Haga clic en el botón **...Más** y, después, haga clic en **Restablecer acceso remoto**:
   
    ![Restablecimiento de la configuración de RDP en Azure Portal](./media/troubleshoot-rdp-connection/classic-reset-rdp.png)
2. **Compruebe los puntos de conexión de Cloud Services**. En este paso para solucionar problemas se comprueba que dispone de puntos de conexión en Cloud Services para permitir el tráfico RDP. El puerto predeterminado para RDP es el puerto TCP 3389. No se creará automáticamente una regla que permita el tráfico RDP cuando se crea la máquina virtual.
   
   Seleccione la máquina virtual en Azure Portal. Haga clic en el botón **Puntos de conexión** para ver los puntos de conexión configurados para la máquina virtual. Compruebe que hay puntos de conexión que permiten el tráfico RDP en el puerto TCP 3389.
   
   En el ejemplo siguiente se muestran los puntos de conexión válidos que permiten el tráfico RDP:
   
   ![Comprobación de los puntos de conexión de Cloud Services en Azure Portal](./media/troubleshoot-rdp-connection/classic-verify-cloud-services-endpoints.png)
   
   Si no tiene un punto de conexión que permita el tráfico RDP, [cree un punto de conexión de Cloud Services](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints). Permita TCP en el puerto 3389 privado.
3. **Revise los diagnósticos de arranque de la máquina virtual**. En este paso para solucionar problemas se revisan los registros de la consola de la máquina virtual para determinar si la máquina virtual está notificando de un problema. No todas las máquinas virtuales tienen diagnósticos de arranque habilitados, por lo que este paso para solucionar problemas puede ser opcional.
   
    Los pasos para solucionar problemas específicos quedan fuera del ámbito de este artículo, pero pueden indicar un problema más amplio que está afectando a la conectividad RDP. Para más información acerca de cómo revisar los registros de la consola y la captura de pantalla de la máquina virtual, consulte la entrada del blog [Boot Diagnostics for VMs](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/) (Diagnósticos de arranque para máquinas virtuales).
4. **Compruebe el estado del recurso de la máquina virtual**. En este paso para solucionar problemas se comprueba que no hay ningún problema conocido con la plataforma Azure que pueda afectar a la conectividad de la máquina virtual.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Estado de los recursos**. Una máquina virtual correcta se notifica como **Disponible**:
   
    ![Comprobación del estado del recurso de máquina virtual en Azure Portal](./media/troubleshoot-rdp-connection/classic-check-resource-health.png)
5. **Restablezca las credenciales de usuario**. En este paso para solucionar problemas se restablece la contraseña en la cuenta de administrador local que haya especificado cuando no está seguro de las credenciales o las haya olvidado.  Una vez que haya iniciado sesión en la máquina virtual, debe restablecer la contraseña para ese usuario.
   
    Seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo en el panel de configuración a la sección **Soporte y solución de problemas** cerca de la parte inferior de la lista. Haga clic en el botón **Restablecer contraseña**. Escriba su nombre de usuario y una nueva contraseña. Finalmente, haga clic en el botón **Guardar**:
   
    ![Restablecimiento de las credenciales de usuario en Azure Portal](./media/troubleshoot-rdp-connection/classic-reset-password.png)
6. **Reinicie la máquina virtual**. En este paso para solucionar problemas se puede corregir cualquier problema subyacente que tenga la misma máquina virtual.
   
    Seleccione la máquina virtual en Azure Portal y haga clic en la pestaña **Información general**. Haga clic en el botón **Reiniciar**:
   
    ![Reinicio de máquina virtual en Azure Portal](./media/troubleshoot-rdp-connection/classic-restart-vm.png)

7. Asegúrese de que cualquier firewall local, o firewall del equipo, permite el tráfico saliente del puerto TCP 3389 a Azure.

Si sigue teniendo problemas con RDP, puede [abrir una solicitud de soporte técnico](https://azure.microsoft.com/support/options/) o leer [más pasos y conceptos detallados de solución de problemas de RDP](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="troubleshoot-specific-rdp-errors"></a>Solución de errores específicos de RDP
Es posible que al intentar conectarse a la máquina virtual mediante RDP, encuentre un mensaje error específico. Los siguientes son los mensajes de error más comunes:

* [Se desconectó la sesión remota porque no hay servidores de licencias de Escritorio remoto disponibles para proporcionar una licencia](troubleshoot-specific-rdp-errors.md#rdplicense).
* [Escritorio remoto no encuentra el equipo "nombre"](troubleshoot-specific-rdp-errors.md#rdpname).
* [Error de autenticación. No se puede conectar con la autoridad de seguridad local](troubleshoot-specific-rdp-errors.md#rdpauth).
* [Error de seguridad de Windows: Las credenciales no han funcionado](troubleshoot-specific-rdp-errors.md#wincred).
* [Este equipo no se puede conectar al equipo remoto](troubleshoot-specific-rdp-errors.md#rdpconnect).

## <a name="additional-resources"></a>Recursos adicionales
Si no se ha producido ninguno de estos errores y sigue sin poder conectarse a la máquina virtual a través de Escritorio remoto, consulte [Solucionar problemas de conexiones de Escritorio remoto a una máquina virtual de Azure basada en Windows](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Para ver los pasos sobre cómo solucionar problemas de acceso a las aplicaciones, consulte [Solucionar problemas de acceso a una aplicación que se ejecuta en una máquina virtual de Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json).
* Si tiene problemas con Secure Shell (SSH) para conectarse a una máquina virtual Linux en Azure, consulte el artículo sobre cómo [solucionar problemas con las conexiones SSH en una máquina virtual Linux en Azure](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).
