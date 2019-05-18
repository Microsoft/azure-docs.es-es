---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 05/10/2019
ms.author: anavin
ms.custom: include file
ms.openlocfilehash: 5aeb0e01192c0635def8eef0c73aa2d14b7921e2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546769"
---
## <a name="os-config"></a>Incorporación de direcciones IP a un sistema operativo de la máquina virtual

Conéctese e inicie sesión en una máquina virtual que creó con múltiples direcciones IP privadas. Debe agregar manualmente todas las direcciones IP privadas (incluida la principal) que ha agregado a la máquina virtual. Complete los pasos siguientes para el sistema operativo de su máquina virtual.

### <a name="windows"></a> Windows

1. En un símbolo del sistema, escriba *ipconfig /all*.  Solo verá la dirección IP privada *principal* (por medio de DHCP).
2. Escriba *ncpa.cpl* en el símbolo del sistema para abrir la ventana **Conexiones de red**.
3. Abra las propiedades del adaptador correspondiente: **Conexión de área local**.
4. Haga doble clic en el Protocolo de Internet versión 4 (IPv4).
5. Seleccione **Usar la siguiente dirección IP** y escriba los valores siguientes:

    * **Dirección IP**: Escriba el *principal* dirección IP privada
    * **Máscara de subred**: Conjunto basado en la subred. Por ejemplo, si la subred es una subred /24, la máscara de subred es 255.255.255.0.
    * **Puerta de enlace predeterminada**: La primera dirección IP en la subred. Si la subred es 10.0.0.1/24, la dirección IP de la puerta de enlace es 10.0.0.0/24.
    * Seleccione **Usar las siguientes direcciones de servidor DNS** y escriba los valores siguientes:
        * **Servidor DNS preferido**: Si no utiliza su propio servidor DNS, escriba 168.63.129.16.  Si usa su propio servidor DNS, escriba la dirección IP de su servidor.
    * Seleccione el botón **Avanzadas** y agregue más direcciones IP. Agregue cada una de las direcciones IP privadas secundarias, que agregó a la interfaz de red de Azure en un paso anterior, a la interfaz de red de Windows que se asigna a la dirección IP principal asignada a la interfaz de red de Azure.

        No asigne manualmente la dirección IP pública asignada a una máquina virtual de Azure en el sistema operativo de la máquina virtual. Al establecer manualmente la dirección IP privada en el sistema operativo, asegúrese de que sea la misma que la asignada a la [interfaz de red](../articles/virtual-network/virtual-network-network-interface-addresses.md#change-ip-address-settings) de Azure; de lo contrario, perderá la conectividad a la máquina virtual. Más información sobre la configuración de la [dirección IP privada](../articles/virtual-network/virtual-network-network-interface-addresses.md#private). Nunca debe asignar una dirección IP pública de Azure dentro del sistema operativo.

    * Haga clic en **Aceptar** para cerrar la configuración de TCP/IP y en **Aceptar** otra vez para cerrar la configuración del adaptador. Se restablece la conexión RDP.

6. En un símbolo del sistema, escriba *ipconfig /all*. Se muestran todas las direcciones IP que agregó y DHCP está desactivado.
7. Configure Windows para usar la dirección IP privada de la configuración IP principal de Azure como dirección IP principal en Windows. Para más información, consulte [Sin acceso a Internet desde una máquina virtual Windows de Azure que tiene varias direcciones IP](https://support.microsoft.com/help/4040882/no-internet-access-from-azure-windows-vm-that-has-multiple-ip-addresse). 

### <a name="validation-windows"></a>Validación (Windows)

Para asegurarse de que puede conectarse a internet desde su configuración de IP secundaria a través de la dirección IP pública asociada, cuando la haya agregado correctamente con los pasos anteriores, use el siguiente comando:

```bash
ping -S 10.0.0.5 hotmail.com
```
>[!NOTE]
>Para las configuraciones de IP secundarias, solo se puede hacer ping a Internet si la configuración tiene una dirección IP pública asociada. Para configuraciones de IP principales, no se requiere una dirección IP pública para hacer ping a Internet.

### <a name="linux-ubuntu-1416"></a>Linux (Ubuntu 14/16)
Se recomienda mirar la versión más reciente de la documentación de la distribución de Linux. 

1. Abra una ventana del terminal.
2. Asegúrese de ser el usuario raíz. Si no lo es, escriba el siguiente comando:

    ```bash
    sudo -i
    ```

3. Actualice el archivo de configuración de la interfaz de red (suponiendo que sea "eth0").

   * Mantenga el elemento de línea existente para dhcp. La dirección IP principal permanece configurada que estaba.
   * Agregue una configuración para una dirección IP estática adicional con los siguientes comandos:

       ```bash
       cd /etc/network/interfaces.d/
       ls
       ```

     Debería ver un archivo .cfg.
4. Abra el archivo. Debería ver las siguientes líneas al final del archivo:

    ```bash
    auto eth0
    iface eth0 inet dhcp
    ```

5. Agregue las líneas siguientes después de las que existen en este archivo:

    ```bash
    iface eth0 inet static
    address <your private IP address here>
    netmask <your subnet mask>
    ```

6. Guarde el archivo mediante el comando siguiente:

    ```bash
    :wq
    ```

7. Restablezca la interfaz de red con el comando siguiente:

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

    > [!IMPORTANT]
    > Si usa una conexión remota, ejecute ifdown e ifup en la misma línea.
    >

8. Compruebe que la dirección IP se agregue a la interfaz de red con el comando siguiente:

    ```bash
    ip addr list eth0
    ```

    Debería ver la dirección IP que agregó en la lista.

### <a name="linux-red-hat-centos-and-others"></a>Linux (Red Hat, CentOS y otros)

1. Abra una ventana del terminal.
2. Asegúrese de ser el usuario raíz. Si no lo es, escriba el siguiente comando:

    ```bash
    sudo -i
    ```

3. Escriba la contraseña y siga las instrucciones que aparezcan. Una vez que sea el usuario raíz, vaya a la carpeta de scripts de red con el comando siguiente:

    ```bash
    cd /etc/sysconfig/network-scripts
    ```

4. Muestra una lista de los archivos de ifcfg relacionados con el siguiente comando:

    ```bash
    ls ifcfg-*
    ```

    Debería ver *ifcfg-eth0* como uno de los archivos.

5. Para agregar una dirección IP, cree un archivo de configuración, tal y como se muestra a continuación. Tenga en cuenta que debe crearse un archivo para cada configuración de IP.

    ```bash
    touch ifcfg-eth0:0
    ```

6. Abra el archivo *ifcfg-eth0:0* con el comando siguiente:

    ```bash
    vi ifcfg-eth0:0
    ```

7. Agregue contenido al archivo, *eth0:0* en este caso, con el siguiente comando. Asegúrese de actualizar la información en función de su dirección IP.

    ```bash
    DEVICE=eth0:0
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.101.101
    NETMASK=255.255.255.0
    ```

8. Guarde el archivo mediante el comando siguiente:

    ```bash
    :wq
    ```

9. Reinicie los servicios de red y asegúrese de que los cambios sean correctos con los comandos siguientes:

    ```bash
    /etc/init.d/network restart
    ifconfig
    ```

    Debería ver la dirección IP que agregó, *eth0:0*, en la lista que se devuelve.

### <a name="validation-linux"></a>Validación (Linux)

Para asegurarse de que puede conectarse a internet desde su configuración de IP secundaria a través de la dirección IP pública asociada, use el siguiente comando:

```bash
ping -I 10.0.0.5 hotmail.com
```
>[!NOTE]
>Para las configuraciones de IP secundarias, solo se puede hacer ping a Internet si la configuración tiene una dirección IP pública asociada. Para configuraciones de IP principales, no se requiere una dirección IP pública para hacer ping a Internet.

Para máquinas virtuales Linux, al intentar validar la conectividad saliente de una NIC secundaria, debe agregar las rutas adecuadas. Existen distintas formas de hacerlo. Consulte la documentación correspondiente a su distribución de Linux. El siguiente es un método para lograr esto:

```bash
echo 150 custom >> /etc/iproute2/rt_tables 

ip rule add from 10.0.0.5 lookup custom
ip route add default via 10.0.0.1 dev eth2 table custom

```
- No olvide reemplazar:
    - **10.0.0.5** con la dirección IP privada que tiene una dirección IP pública asociada
    - **10.0.0.1** con su puerta de enlace predeterminada
    - **eth2** con el nombre de la NIC secundaria
