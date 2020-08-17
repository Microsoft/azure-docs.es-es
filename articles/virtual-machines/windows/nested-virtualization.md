---
title: Habilitación de la virtualización anidada en Azure Virtual Machines
description: Habilitación de la virtualización anidada en Azure Virtual Machines
author: cynthn
ms.author: cynthn
ms.date: 10/09/2017
ms.topic: how-to
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.openlocfilehash: 03df7db13ebd3ebec407bb046cc735c835e01068
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074252"
---
# <a name="how-to-enable-nested-virtualization-in-an-azure-vm"></a>Habilitación de la virtualización anidada en una máquina virtual de Azure

La virtualización anidada es compatible con varias familias de máquinas virtuales de Azure. Esta funcionalidad proporciona gran flexibilidad para admitir escenarios como entornos de desarrollo, pruebas, aprendizaje y demostración.   

En este artículo se analiza la habilitación de Hyper-V en una máquina virtual de Azure y la configuración de la conectividad de Internet a esa máquina virtual invitada.

## <a name="create-a-nesting-capable-azure-vm"></a>Creación de una máquina virtual de Azure compatible con el anidamiento

Cree una nueva máquina virtual de Azure con Windows Server 2016. Para una lista completa de los tamaños de máquina virtual que admiten anidamiento, visite el artículo [Unidad de proceso de Azure (ACU)](../acu.md).

Recuerde elegir un tamaño de máquina virtual lo suficientemente grande para admitir las exigencias de una invitada. En este ejemplo, se usa una máquina virtual con tamaño D3_v3. 

Puede ver la disponibilidad regional de las máquinas virtuales de las series Dv3 o Ev3 [aquí](https://azure.microsoft.com/regions/services/).

>[!NOTE]
>
>Para instrucciones detalladas sobre cómo crear una máquina virtual nueva, consulte [Creación y administración de máquinas virtuales Windows con el módulo de Azure PowerShell](./tutorial-manage-vm.md)
    
## <a name="connect-to-your-azure-vm"></a>Conexión a una máquina virtual de Azure

Cree una conexión a Escritorio remoto en la máquina virtual.

1. Haga clic en el botón **Conectar** en las propiedades de la máquina virtual. Se crea y se descarga un archivo de Protocolo de Escritorio remoto (archivo .rdp).

2. Para conectarse a la máquina virtual, abra el archivo RDP descargado. Cuando se le solicite, haga clic en **Conectar**. En un equipo Mac, necesita un cliente RDP como este [Cliente de Escritorio remoto](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) de Mac App Store.

3. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual y, a continuación, haga clic en **Aceptar**.

4. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Haga clic en **Sí** o **Conectar** para continuar con la conexión.

## <a name="enable-the-hyper-v-feature-on-the-azure-vm"></a>Habilitación de la característica Hyper-V en la máquina virtual de Azure
Puede configurar estos ajustes de manera manual o usar el script de PowerShell que se proporciona para automatizar la configuración.

### <a name="option-1-use-a-powershell-script-to-configure-nested-virtualization"></a>Opción 1: Usar un script de PowerShell para configurar la virtualización anidada
Hay un script de PowerShell para habilitar la virtualización anidada en un host de Windows Server 2016 disponible en [GitHub](https://github.com/charlieding/Virtualization-Documentation/tree/live/hyperv-tools/Nested). El script comprueba los requisitos previos y, luego, configura la virtualización anidada en la máquina virtual de Azure. Es necesario realizar un reinicio de la máquina virtual de Azure para completar la configuración. Este script puede funcionar en otros entornos, pero no está garantizado. Consulte la entrada de blog de Azure que incluye una demostración de vídeo en directo sobre la virtualización anidada en Azure. https://aka.ms/AzureNVblog.

### <a name="option-2-configure-nested-virtualization-manually"></a>Opción 2: Configurar manualmente la virtualización anidada

1. En la máquina virtual de Azure, abra PowerShell como administrador. 

2. Habilite la característica Hyper-V y las herramientas de administración.

    ```powershell
    Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
    ```

    >[!WARNING] 
    >
    >Con este comando se reinicia la máquina virtual de Azure. Perderá la conexión RDP durante el proceso de reinicio.
    
3. Una vez que la máquina virtual de Azure se reinicie, reconéctese a la VM con RDP.

## <a name="set-up-internet-connectivity-for-the-guest-virtual-machine"></a>Configuración de la conectividad de Internet de la máquina virtual invitada
Cree un adaptador de red virtual nuevo para la máquina virtual invitada y configure una puerta de enlace NAT para habilitar la conectividad de Internet.

### <a name="create-a-nat-virtual-network-switch"></a>Creación de un conmutador de red virtual NAT

1. En la máquina virtual de Azure, abra PowerShell como administrador.
   
2. Cree un conmutador interno.

    ```powershell
    New-VMSwitch -Name "InternalNAT" -SwitchType Internal
    ```

3. Consulte las propiedades del conmutador y anote el valor de ifIndex del adaptador nuevo.

    ```powershell
    Get-NetAdapter
    ```

    ![NetAdapter](./media/virtual-machines-nested-virtualization/get-netadapter.png)

    >[!NOTE] 
    >
    >Anote el valor de "ifIndex" del conmutador virtual que acaba de crear.
    
4. Cree una dirección IP para la puerta de enlace NAT.
    
Para configurar la puerta de enlace, necesita información sobre la red:    
  * IPAddress: la dirección IP de la puerta de enlace NAT especifica la dirección IPv4 o IPv6 que se va a usar como la dirección de puerta de enlace predeterminada de la subred de la red virtual. El formato genérico es a.b.c.1 (por ejemplo, "192.168.0.1"). Aunque no es obligatorio que la posición final sea .1, habitualmente lo es (según la longitud de prefijo). Típicamente, debe usar un espacio de direcciones de red privadas RFC 1918. 
  * PrefixLength: la longitud de prefijo de la subred define el tamaño de la subred local (máscara de subred). La longitud de prefijo de la subred será un valor entero entre 0 y 32. 0 asignaría todo Internet, 32 solo permitiría una dirección IP asignada. Los valores comunes van de 24 a 12 en función de cuántas direcciones IP se deben adjuntar a NAT. Un valor de PrefixLength común es 24, que es una máscara de subred de 255.255.255.0.
  * InterfaceIndex: **ifIndex** es el índice de interfaz del conmutador virtual que se creó en el paso anterior. 

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 13
    ```

### <a name="create-the-nat-network"></a>Creación de la red NAT

Para configurar la puerta de enlace, deberá proporcionar información sobre la red y la puerta de enlace NAT:
  * Name: el nombre de la red NAT. 
  * InternalIPInterfaceAddressPrefix: el prefijo de la subred NAT describe tanto el prefijo de la dirección IP de la puerta de enlace NAT como la longitud de prefijo de la subred NAT anteriores. El formato genérico será a.b.c.0/longitud de prefijo de la subred NAT. 

En PowerShell, cree una red NAT nueva.
```powershell
New-NetNat -Name "InternalNat" -InternalIPInterfaceAddressPrefix 192.168.0.0/24
```


## <a name="create-the-guest-virtual-machine"></a>Creación de la máquina virtual invitada

>[!IMPORTANT] 
>
>El agente invitado de Azure no se admite en máquinas virtuales anidadas y puede causar problemas en el host y en las máquinas virtuales anidadas. No instale al agente de Azure en máquinas virtuales anidadas y no use una imagen para crear las máquinas virtuales anidadas que ya tienen instalado el agente invitado de Azure.

1. Abra el Administrador de Hyper-V y cree una máquina virtual nueva. Configure la máquina virtual para usar la red interna nueva que creó.
    
    ![NetworkConfig](./media/virtual-machines-nested-virtualization/configure-networking.png)
    
2. Instale un sistema operativo en la máquina virtual invitada.
    
    >[!NOTE] 
    >
    >Necesita los medios de instalación de un sistema operativo para instalarlo en la máquina virtual. En este caso se usa Windows 10 Enterprise.

## <a name="assign-an-ip-address-to-the-guest-virtual-machine"></a>Asignación de una dirección IP a la máquina virtual invitada

Puede asignar una dirección IP a la máquina virtual invitada ya sea estableciendo manualmente una dirección IP estática en la máquina virtual invitada o configurando DHCP en la máquina virtual de Azure para asignar de manera dinámica la dirección IP.

###  <a name="option-1-configure-dhcp-to-dynamically-assign-an-ip-address-to-the-guest-virtual-machine"></a>Opción 1: Configurar DHCP para asignar dinámicamente una dirección IP a la máquina virtual invitada
Siga estos pasos para configurar DHCP en la máquina virtual host para la asignación dinámica de direcciones.

#### <a name="install-dhcp-server-on-the-azure-vm"></a>Instalación del servidor DHCP en la VM de Azure

1. Abra el Administrador del servidor. En el panel, haga clic en **Agregar roles y característica**. Aparecerá el Asistente para agregar roles y características.
  
2. En el asistente, haga clic en **Siguiente** hasta llegar a la página Roles del servidor.
  
3. Haga clic para activar la casilla **Servidor DHCP**, haga clic en **Agregar características** y en **Siguiente** hasta completar el asistente.
  
4. Haga clic en **Instalar**.

#### <a name="configure-a-new-dhcp-scope"></a>Configuración de un nuevo ámbito de DHCP

1. Abra el Administrador DHCP.
  
2. En el panel de navegación, expanda el nombre del servidor, haga clic con el botón derecho **IPv4** y, luego, haga clic en **Nuevo ámbito**. Aparece el asistente para agregar un nuevo ámbito. Haga clic en **Siguiente**.
  
3. Escriba un nombre y una descripción para el ámbito y haga clic en **Siguiente**.
  
4. Defina un intervalo IP para el servidor DHCP (por ejemplo, de 192.168.0.100 a 192.168.0.200).
  
5. Haga clic en **Siguiente** hasta la página Puerta de enlace predeterminada. Escriba la dirección IP que creó anteriormente (por ejemplo, 192.168.0.1) como la puerta de enlace predeterminada y después haga clic en **Agregar**.
  
6. Haga clic en **Siguiente** hasta completar el asistente, deje todos los valores predeterminados y haga clic en **Finalizar**.
    
### <a name="option-2-manually-set-a-static-ip-address-on-the-guest-virtual-machine"></a>Opción 2: Establecer manualmente una dirección IP estática en la máquina virtual invitada
Si no configuró DHCP para asignar de manera dinámica una dirección IP a la máquina virtual invitada, siga estos pasos para establecer una dirección IP estática.

1. En la máquina virtual de Azure, abra PowerShell como administrador.

2. Haga clic con el botón derecho en la máquina virtual invitada y haga clic en Conectar.

3. Inicie sesión en la máquina virtual invitada.

4. En la máquina virtual invitada, abra el Centro de redes y recursos compartidos.

5. Configure el adaptador de red para una dirección dentro del intervalo de la red NAT que creó en la sección anterior.

En este ejemplo usará una dirección dentro del intervalo 192.168.0.0/24.

## <a name="test-connectivity-in-guest-virtual-machine"></a>Prueba de la conectividad en la máquina virtual invitada

En la máquina virtual invitada, abra el explorador y navegue a una página web.
    ![GuestVM](./media/virtual-machines-nested-virtualization/guest-virtual-machine.png)

Para obtener instrucciones sobre cómo habilitar la conectividad transparente entre las máquinas virtuales invitadas y las máquinas virtuales de Azure, consulte [este documento](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).
