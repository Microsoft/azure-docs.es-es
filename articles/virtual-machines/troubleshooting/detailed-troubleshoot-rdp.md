---
title: Solución detallada de problemas de Escritorio remoto en Azure | Microsoft Docs
description: Revise los pasos detallados de solución de problemas para errores de Escritorio remoto en los que no se puede conectar a máquinas virtuales Windows en Azure
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
keywords: no se puede conectar a un escritorio remoto, solucionar problemas de escritorio remoto, no puede conectarse a escritorio remoto, errores de escritorio remotos, solución de problemas de escritorio remoto, problemas de escritorio remoto
ms.assetid: 9da36f3d-30dd-44af-824b-8ce5ef07e5e0
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 91f15e32866cca008553286f7585247909d9a4ba
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87009873"
---
# <a name="detailed-troubleshooting-steps-for-remote-desktop-connection-issues-to-windows-vms-in-azure"></a>Pasos detallados para solucionar problemas de conexión a Escritorio remoto a máquinas virtuales Windows en Azure
Este artículo ofrece pasos detallados de solución de problemas para diagnosticar y corregir errores complejos de Escritorio remoto en máquinas virtuales de Azure basadas en Windows.

> [!IMPORTANT]
> Para eliminar los errores más comunes de Escritorio remoto, asegúrese de leer [el artículo sobre la solución de problemas básicos de Escritorio remoto](troubleshoot-rdp-connection.md) antes de continuar.

Puede que se encuentre un mensaje de error de Escritorio remoto que no se parezca a ninguno de los tratados en [la guía básica de solución de problemas de Escritorio remoto](troubleshoot-rdp-connection.md). Siga estos pasos para determinar por qué el cliente de Escritorio remoto (RDP) no se puede conectar con el servicio RDP en la máquina virtual de Azure.


Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y haga clic en **Obtener soporte**. Para obtener información sobre el uso del soporte técnico de Azure, lea las [Preguntas más frecuentes del soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="components-of-a-remote-desktop-connection"></a>Componentes de una conexión de Escritorio remoto
Los siguientes componentes participan en una conexión RDP:

![Diagrama que muestra los componentes que participan en una conexión a Escritorio remoto (RDP).](./media/detailed-troubleshoot-rdp/tshootrdp_0.png)

Antes de continuar, podría resultar útil revisar mentalmente qué ha cambiado desde la última conexión correcta de Escritorio remoto a la máquina virtual. Por ejemplo:

* La dirección IP pública de la máquina virtual o del servicio en la nube que contiene la máquina virtual (también llamada dirección IP virtual [VIP](https://en.wikipedia.org/wiki/Virtual_IP_address)) ha cambiado. El error de RDP podría deberse a que la memoria caché del cliente DNS sigue teniendo la *dirección IP antigua* registrada para el nombre DNS. Vacíe la memoria caché del cliente DNS e intente conectar la máquina virtual de nuevo. O bien, intente conectarse directamente con la VIP nueva.
* Está usando una aplicación de terceros para administrar las conexiones a Escritorio remoto en lugar de la conexión generada por Azure Portal. Compruebe que la configuración de la aplicación incluya el puerto TCP correcto para el tráfico de Escritorio remoto. Puede comprobar este puerto para una máquina virtual clásica en el [portal de Azure](https://portal.azure.com), haciendo clic en Configuración de la máquina virtual > Puntos de conexión.

## <a name="preliminary-steps"></a>Pasos previos
Antes de continuar con la solución de problemas detallada,

* compruebe el estado de la máquina virtual en Azure Portal para detectar cualquier error obvio.
* Siga los [pasos de corrección rápida para los errores comunes de RDP en la guía básica de solución de problemas](troubleshoot-rdp-connection.md#quick-troubleshooting-steps).
* Para las imágenes personalizadas, asegúrese de que el disco duro virtual se prepara correctamente antes de cargarlo. Para más información, consulte [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](../windows/prepare-for-upload-vhd-image.md).


Vuelva a conectarse a la máquina virtual a través de Escritorio remoto después de realizar estos pasos.

## <a name="detailed-troubleshooting-steps"></a>Pasos de la solución de problemas detallada
Puede que el cliente de Escritorio remoto no pueda ponerse en contacto con el servicio de Escritorio remoto en la máquina virtual de Azure debido a problemas en los orígenes siguientes:

* [Equipo cliente de Escritorio remoto](#source-1-remote-desktop-client-computer)
* [Dispositivo perimetral de intranet de la organización](#source-2-organization-intranet-edge-device)
* [Extremo de servicio en la nube y lista de control de acceso (ACL)](#source-3-cloud-service-endpoint-and-acl)
* [Grupos de seguridad de red](#source-4-network-security-groups)
* [Máquina virtual de Azure basada en Windows](#source-5-windows-based-azure-vm)

## <a name="source-1-remote-desktop-client-computer"></a>Causa 1: Equipo cliente de Escritorio remoto
Compruebe que el equipo puede establecer conexiones de Escritorio remoto a otro equipo local, basado en Windows.

![Diagrama de los componentes de una conexión a Escritorio remoto (RDP) con el cliente RDP resaltado y una flecha que apunta a otro equipo local que indica una conexión.](./media/detailed-troubleshoot-rdp/tshootrdp_1.png)

Si no es posible, compruebe si la configuración del equipo incluye lo siguiente:

* Una configuración del firewall local que bloquea el tráfico de Escritorio remoto
* Software de proxy de cliente instalado localmente que impide las conexiones a Escritorio remoto
* Software de supervisión de red instalado localmente que impide las conexiones a Escritorio remoto
* Otro tipo de software de seguridad, por ejemplo, para supervisar el tráfico o para permitir o impedir ciertos tipos de tráfico, que imposibilita las conexiones a Escritorio remoto

En todos estos casos, deshabilite temporalmente el software e intente conectarse a un equipo local a través de Escritorio remoto. Si puede averiguar la causa real mediante este método, trabaje con el administrador de red para corregir la configuración del software para permitir conexiones a Escritorio remoto.

## <a name="source-2-organization-intranet-edge-device"></a>Causa 2: Dispositivo perimetral de intranet de la organización
Compruebe que un equipo conectado directamente a Internet puede establecer conexiones de Escritorio remoto a la máquina virtual de Azure.

![Diagrama de los componentes de una conexión a Escritorio remoto (RDP) con un cliente RDP que está conectado a Internet resaltado y una flecha que apunta a una máquina virtual de Azure que indica una conexión.](./media/detailed-troubleshoot-rdp/tshootrdp_2.png)

Si no tiene un equipo conectado directamente a Internet, cree y pruebe con una nueva máquina virtual de Azure en un grupo de recursos o servicio en la nube. Para más información, consulte [Creación de una máquina virtual Windows en Azure](../windows/quick-create-portal.md). Puede eliminar la máquina virtual y el grupo de recursos o el servicio en la nube después de la prueba.

Si puede crear una conexión a Escritorio remoto con un equipo conectado directamente a Internet, compruebe si el dispositivo perimetral de intranet de la organización tiene lo siguiente:

* Un firewall interno que bloquea las conexiones HTTPS a Internet
* Un servidor proxy que impide las conexiones a Escritorio remoto
* Software de detección de intrusiones o supervisión de red en ejecución en los dispositivos de la red perimetral que impide las conexiones a Escritorio remoto

Trabaje con el administrador de red para corregir la configuración del dispositivo perimetral de intranet de la organización para permitir conexiones a Escritorio remoto a Internet basadas en HTTPS.

## <a name="source-3-cloud-service-endpoint-and-acl"></a>Causa 3: Punto de conexión de servicio en la nube y ACL

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Para las máquinas virtuales creadas mediante el modelo de implementación clásica, compruebe que otra máquina virtual de Azure que se encuentre en el mismo servicio en la nube o la misma red virtual pueda establecer conexiones a Escritorio remoto a su máquina virtual de Azure.

![Diagrama de los componentes de una conexión a Escritorio remoto (RDP) con una máquina virtual de Azure resaltada y una flecha que apunta a otra máquina virtual de Azure dentro del mismo servicio en la nube que indica una conexión.](./media/detailed-troubleshoot-rdp/tshootrdp_3.png)

> [!NOTE]
> Para las máquinas virtuales creadas en Resource Manager, vaya a [Causa 4: grupos de seguridad de red](#source-4-network-security-groups).

Si no tiene otra máquina virtual en el mismo servicio en la nube o red virtual, cree una. Siga los pasos en el artículo sobre la [creación de una máquina virtual que ejecuta Windows en Azure](../windows/quick-create-portal.md). Una vez completada la prueba, elimine la máquina virtual de prueba.

Si puede conectarse a una máquina virtual en el mismo servicio en la nube o la misma red virtual mediante Escritorio remoto, compruebe las siguientes configuraciones:

* La configuración del punto de conexión para el tráfico de Escritorio remoto en la máquina virtual de destino: el puerto TCP privado del punto de conexión debe coincidir con el puerto TCP en el que está escuchando el servicio de Escritorio remoto de la máquina virtual (el valor predeterminado es 3389).
* La ACL del punto de conexión del tráfico de Escritorio remoto en la máquina virtual de destino: las ACL permiten especificar el tráfico entrante de Internet que se permite o se deniega en función de la dirección IP de origen. Las ACL mal configuradas pueden impedir el tráfico entrante de Escritorio remoto al extremo. Compruebe las ACL para asegurarse de que está permitido el tráfico entrante desde las direcciones IP públicas del proxy o de otro servidor perimetral. Para obtener más información, vea [qué es una lista de control de acceso (ACL) de red](/previous-versions/azure/virtual-network/virtual-networks-acl)

Para comprobar si el punto de conexión es la causa del problema, quite el punto de conexión actual y cree uno nuevo. Para ello, elija un puerto aleatorio en el intervalo que va entre 49152 y 65535 para el número de puerto externo. Para más información, consulte [Configuración de puntos de conexión en una máquina virtual](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints?toc=/azure/virtual-machines/windows/classic/toc.json).

## <a name="source-4-network-security-groups"></a>Causa 4: Grupos de seguridad de red
Los grupos de seguridad de red permiten un control pormenorizado del tráfico entrante y saliente permitido. Puede crear reglas que abarquen subredes y servicios en la nube en una red virtual de Azure.

Use la [verificación del flujo IP](../../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) para confirmar si una regla en un grupo de seguridad de red está bloqueando el tráfico hacia o desde una máquina virtual. También puede revisar cómo crear reglas de grupo de seguridad eficaces para garantizar que exista la regla NSG "Permitir" de entrada y tenga prioridad para el puerto RDP (valor predeterminado 3389). Para más información, vea [Uso de las reglas de seguridad vigentes para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-traffic-filter-problem.md).

## <a name="source-5-windows-based-azure-vm"></a>Causa 5: VM de Azure basada en Windows
![Diagrama de los componentes de una conexión a Escritorio remoto (RDP) con una máquina virtual de Azure resaltada en un servicio en la nube y un mensaje que podría ser un "posible origen de problemas". Una línea azul indica que las reglas del grupo de seguridad de red podrían estar bloqueando el tráfico hacia o desde la máquina virtual de Azure.](./media/detailed-troubleshoot-rdp/tshootrdp_5.png)

Siga las instrucciones de [este artículo](./reset-rdp.md). En este artículo se restablece el servicio de Escritorio remoto en la máquina virtual:

* Se habilitará la regla predeterminada «Escritorio remoto» del firewall de Windows (puerto TCP 3389).
* Se habilitarán las conexiones a Escritorio remoto estableciendo en 0 el valor HKLM\System\CurrentControlSet\Control\Terminal Server\fDenyTSConnections del Registro.

Pruebe de nuevo la conexión desde su equipo. Si todavía no puede conectarse mediante Escritorio remoto, compruebe los siguientes problemas posibles:

* El servicio de Escritorio remoto no se está ejecutando en la máquina virtual de destino.
* El servicio de Escritorio remoto no está escuchando en el puerto TCP 3389.
* El firewall de Windows u otro firewall local tiene una regla de salida que impide el tráfico de Escritorio remoto.
* El software de detección de intrusiones o supervisión de red que se ejecuta en la máquina virtual de Azure impide las conexiones a Escritorio remoto.

Para las máquinas virtuales creadas con el modelo de implementación clásico, puede utilizar una sesión remota de Azure PowerShell para la máquina virtual de Azure. En primer lugar, debe instalar un certificado para el servicio en la nube de hospedaje de la máquina virtual. Vaya a [Configurar el acceso remoto seguro de PowerShell a Azure Virtual Machines](https://gallery.technet.microsoft.com/scriptcenter/Configures-Secure-Remote-b137f2fe) y descargue el archivo de script **InstallWinRMCertAzureVM.ps1** en el equipo local.

A continuación, instale Azure PowerShell si todavía no lo ha hecho. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

Después, abra un símbolo del sistema de Azure PowerShell y cambie la carpeta actual a la ubicación del archivo de script **InstallWinRMCertAzureVM.ps1** . Para ejecutar un script de Azure PowerShell, debe establecer la directiva de ejecución correcta. Ejecute el comando **Get-ExecutionPolicy** para determinar el nivel de directiva actual. Para obtener información sobre cómo establecer el nivel adecuado, vea [Set-ExecutionPolicy](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-5.1).

A continuación, rellene el nombre de su suscripción a Azure, el nombre del servicio en la nube y el nombre de la máquina virtual (quitando los caracteres < y >) y ejecute estos comandos.

```powershell
$subscr="<Name of your Azure subscription>"
$serviceName="<Name of the cloud service that contains the target virtual machine>"
$vmName="<Name of the target virtual machine>"
.\InstallWinRMCertAzureVM.ps1 -SubscriptionName $subscr -ServiceName $serviceName -Name $vmName
```

Puede obtener el nombre de suscripción correcto en la propiedad *SubscriptionName* de la pantalla del comando **Get-AzureSubscription**. Puede obtener el nombre del servicio en la nube de la máquina virtual en la columna *ServiceName* de la pantalla del comando **Get-AzureVM**.

Compruebe si tiene el certificado nuevo. Abra un complemento Certificados para el usuario actual y vaya a la carpeta **Entidades de certificación raíz de confianza\Certificados**. Debería ver un certificado con el nombre DNS del servicio en la nube en la columna Emitido para (ejemplo: cloudservice4testing.cloudapp.net).

A continuación, inicie una sesión remota de Azure PowerShell mediante el uso de estos comandos.

```powershell
$uri = Get-AzureWinRMUri -ServiceName $serviceName -Name $vmName
$creds = Get-Credential
Enter-PSSession -ConnectionUri $uri -Credential $creds
```

Después de escribir las credenciales de administrador válidas, debería ver algo parecido al siguiente símbolo del sistema de Azure PowerShell:

```powershell
[cloudservice4testing.cloudapp.net]: PS C:\Users\User1\Documents>
```

La primera parte de este indicador es el nombre del servicio en la nube que contiene la máquina virtual de destino, que podría ser diferente de "cloudservice4testing.cloudapp.net". Ahora puede emitir comandos de Azure PowerShell para este servicio en la nube, para investigar los problemas que se mencionaron y corregir la configuración.

### <a name="to-manually-correct-the-remote-desktop-services-listening-tcp-port"></a>Para corregir manualmente el puerto TCP de escucha de Servicios de Escritorio remoto
En la sesión remota de Azure PowerShell, ejecute este comando.

```powershell
Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"
```

La propiedad PortNumber muestra el número de puerto actual. Si es necesario, vuelva a poner el número de puerto del Escritorio remoto en su valor predeterminado (3389) con este comando.

```powershell
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 3389
```

Compruebe que el puerto cambió a 3389 con este comando.

```powershell
Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"
```

Salga de la sesión remota de Azure PowerShell con este comando.

```powershell
Exit-PSSession
```

Compruebe que el punto de conexión de Escritorio remoto para la máquina virtual de Azure también usa el puerto TCP 3398 como puerto interno. Reinicie la máquina virtual de Azure y vuelva a intentar la conexión a Escritorio remoto.

## <a name="additional-resources"></a>Recursos adicionales
[Cómo restablecer una contraseña o el servicio de Escritorio remoto para máquinas virtuales de Windows](./reset-rdp.md)

[Instalación y configuración de Azure PowerShell](/powershell/azure/)

[Solución de problemas de conexiones de Secure Shell (SSH) en una máquina virtual de Azure basada en Linux](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json)

[Solución de problemas de acceso a una aplicación que se ejecuta en una máquina virtual de Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json)
