---
title: Configuración de MPIO en un host de Linux de StorSimple
description: Conozca los pasos necesarios para configurar E/S de múltiples rutas (MPIO) en el servidor host de StorSimple Linux (Centos 6.6).
author: alkohli
ms.assetid: ca289eed-12b7-4e2e-9117-adf7e2034f2f
ms.service: storsimple
ms.topic: how-to
ms.date: 06/12/2019
ms.author: alkohli
ms.openlocfilehash: 75ccfe7a8e62e519b1df89792211433260a6abf6
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294720"
---
# <a name="configure-mpio-on-a-storsimple-host-running-centos"></a>Configuración de MPIO en un host de StorSimple que ejecuta CentOS
Este artículo explica los pasos necesarios para configurar E/S de múltiples rutas (MPIO) en el servidor host de Centos 6.6. El servidor host está conectado al dispositivo de Microsoft Azure StorSimple para una alta disponibilidad a través de los iniciadores iSCSI. Describe detalladamente la detección automática de dispositivos de múltiples rutas de acceso y el programa de instalación específico solo para los volúmenes de StorSimple.

Este procedimiento se aplica a todos los modelos de dispositivos de la serie 8000 de StorSimple.

> [!NOTE]
> No se puede usar este procedimiento para StorSimple Cloud Appliance. Para obtener más información, consulte cómo configurar los servidores host para su instancia de Cloud Appliance.


## <a name="about-multipathing"></a>Acerca de múltiples rutas
La característica de múltiples rutas permite configurar varias rutas de acceso de E/S entre un servidor host y un dispositivo de almacenamiento. Estas rutas de acceso de E/S son conexiones físicas de SAN que pueden incluir cables independientes, conmutadores, interfaces de red y controladores. La característica de múltiples rutas agrega las rutas de acceso de E/S para configurar un nuevo dispositivo asociado a todas las rutas de acceso agregadas.

El propósito de las múltiples rutas tiene dos vertientes:

* **Alta disponibilidad**: proporciona una ruta alternativa si se produce un error en cualquier elemento de la ruta de acceso de E/S (por ejemplo, un cable, conmutador, interfaz de red o controlador).
* **Equilibrio de carga**: según la configuración de su dispositivo de almacenamiento, puede mejorar el rendimiento mediante la detección de cargas en las rutas de acceso de E/S y volver a equilibrar las cargas dinámicamente.

### <a name="about-multipathing-components"></a>Acerca de los componentes de múltiples rutas
La característica de múltiples rutas en Linux consta de componentes del kernel y de componentes del espacio de usuario, como se indica más abajo.

* **Kernel**: el componente principal es el *asignador de dispositivos* que redistribuye la E/S y admite la conmutación por error para las rutas de acceso y los grupos de la ruta de acceso.

* **Espacio de usuario**: son las *herramientas de múltiples rutas* que administran los dispositivos de múltiples rutas, para lo que se indicando al módulo de múltiples rutas del asignador de dispositivos lo que debe hacer. Las herramientas son las siguientes:
   
   * **Multipath**: muestra y configura los dispositivos con múltiples rutas.
   * **Multipathd**: demonio que ejecuta las múltiples rutas y supervisa las rutas de acceso.
   * **Devmap-name**: proporciona un nombre de dispositivo significativo a udev para devmaps.
   * **Kpartx**: asigna devmaps lineales a las particiones del dispositivo para realizar asignaciones de múltiples rutas que se puedan particionar.
   * **Multipath.conf**: archivo de configuración del demonio de múltiples rutas que se usa para sobrescribir la tabla de configuración integrada.

### <a name="about-the-multipathconf-configuration-file"></a>Acerca del archivo de configuración multipath.conf
El archivo de configuración `/etc/multipath.conf` permite que muchas de las características de múltiples rutas sean configurables por el usuario. El comando `multipath` y el demonio de kernel `multipathd` usan la información contenida en este archivo. El archivo se consulta únicamente durante la configuración de los dispositivos de múltiples rutas. Asegúrese de que todos los cambios se realizan antes de ejecutar el comando `multipath` . Si posteriormente modifica el archivo, deberá detener e iniciar multipathd de nuevo para que los cambios surtan efecto.

El archivo multipath.conf tiene cinco secciones:

- **Valores predeterminados en el nivel del sistema** *(defaults)* : los valores predeterminados de nivel de sistema se pueden invalidad.
- **Dispositivos restringidos** *(blacklist)* : puede especificar la lista de dispositivos que no deben controlarse mediante el asignador de dispositivos.
- **Excepciones a la lista negra** *(blacklist_exceptions)* : puede identificar dispositivos específicos para que se traten como dispositivos de múltiples rutas, aunque aparezcan en la lista negra.
- **Configuración específica del controlador de almacenamiento** *(devices)* : puede especificar valores de configuración que se aplicarán a los dispositivos con información de proveedor y producto.
- **Configuración específica de dispositivo** *(multipaths)* : estas sección se puede usar para ajustar la configuración de cada LUN.

## <a name="configure-multipathing-on-storsimple-connected-to-linux-host"></a>Configuración de múltiples rutas en StorSimple conectado al host Linux
Un dispositivo de StorSimple conectado a un host Linux puede configurarse para alta disponibilidad y equilibrio de carga. Por ejemplo, si el host Linux tiene dos interfaces conectadas a la SAN y el dispositivo tiene dos interfaces conectadas a la SAN de tal forma que estas interfaces estén en la misma subred, habrá 4 rutas de acceso disponibles. Sin embargo, si cada interfaz DATA en la interfaz del dispositivo y de host están en una subred IP diferente (y no enrutable), solo estarán disponibles dos rutas de acceso. Puede configurar múltiples rutas para que detecten automáticamente todas las rutas de acceso disponibles, elegir un algoritmo de equilibrio de carga para esas rutas de acceso, aplicar la configuración específica para los volúmenes únicamente de StorSimple y, después, habilitar y comprobar la característica de múltiples rutas.

Con el siguiente procedimiento explicamos cómo configurar las múltiples rutas cuando un dispositivo StorSimple con dos interfaces de red está conectado a un host con dos interfaces de red.

## <a name="prerequisites"></a>Requisitos previos
En esta sección se detallan los requisitos previos de configuración para el servidor CentOS y el dispositivo de StorSimple.

### <a name="on-centos-host"></a>En el host CentOS
1. Asegúrese de que el host CentOS tiene dos interfaces de red habilitadas. Escriba:
   
    `ifconfig`
   
    En el ejemplo siguiente se muestra la salida cuando dos interfaces de red (`eth0` y `eth1`) están presentes en el host.
   
    ```output
    [root@centosSS ~]# ifconfig
    eth0  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:41  
        inet addr:10.126.162.65  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3341/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3341/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:36536 errors:0 dropped:0 overruns:0 frame:0
        TX packets:6312 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:13994127 (13.3 MiB)  TX bytes:645654 (630.5 KiB)

    eth1  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:42  
        inet addr:10.126.162.66  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3342/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3342/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:25962 errors:0 dropped:0 overruns:0 frame:0
        TX packets:11 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:2597350 (2.4 MiB)  TX bytes:754 (754.0 b)

    loLink encap:Local Loopback  
        inet addr:127.0.0.1  Mask:255.0.0.0
        inet6 addr: ::1/128 Scope:Host
        UP LOOPBACK RUNNING  MTU:65536  Metric:1
        RX packets:12 errors:0 dropped:0 overruns:0 frame:0
        TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:0
        RX bytes:720 (720.0 b)  TX bytes:720 (720.0 b)
    ```
1. Instale *iSCSI-initiator-utils* en el servidor CentOS. Realice los pasos siguientes para instalar *iSCSI-initiator-utils*.
   
   1. Inicie sesión como `root` en el host CentOS.
   1. Instale *iSCSI-initiator-utils*. Escriba:
      
       `yum install iscsi-initiator-utils`
   1. Cuando *iSCSI-Initiator-utils* esté correctamente instalado, inicie el servicio iSCSI. Escriba:
      
       `service iscsid start`
      
       En ocasiones, `iscsid` puede no iniciarse y, en este caso, ser necesaria la opción `--force`.
   1. Para asegurarse de que el iniciador iSCSI está habilitado durante el tiempo de arranque, use el comando `chkconfig` para habilitar el servicio.
      
       `chkconfig iscsi on`
   1. Para comprobar que ha instalado correctamente, ejecute el comando:
      
       `chkconfig --list | grep iscsi`
      
       A continuación se muestra una salida de ejemplo.
      
        ```output
        iscsi   0:off   1:off   2:on3:on4:on5:on6:off
        iscsid  0:off   1:off   2:on3:on4:on5:on6:off
        ```
      
       En el ejemplo anterior, puede ver que el entorno iSCSI se ejecutará en tiempo de arranque en los niveles de ejecución 2, 3, 4 y 5.
1. Instale *device-mapper-multipath*. Escriba:
   
    `yum install device-mapper-multipath`
   
    Se iniciará la instalación. Cuando se le pida confirmación, escriba **Y** .

### <a name="on-storsimple-device"></a>En el dispositivo de StorSimple
El dispositivo de StorSimple debe disponer de:

* Un mínimo de dos interfaces habilitadas para iSCSI. Para comprobar que hay dos interfaces habilitadas para iSCSI en el dispositivo StorSimple, realice los pasos siguientes en el Portal de Azure clásico para el dispositivo StorSimple:
  
  1. Inicie sesión en el Portal clásico para el dispositivo StorSimple.
  1. Seleccione el servicio StorSimple Manager, haga clic en **Dispositivos** y elija el dispositivo StorSimple. Haga clic en **Configurar** y compruebe la configuración de la interfaz de red. A continuación se muestra una captura de pantalla con dos interfaces de red habilitadas para iSCSI. En este caso DATA 2 y DATA 3, las dos interfaces de 10 GbE están habilitadas para iSCSI.
     
      ![Configuración de MPIO StorSimple DATA 2](./media/storsimple-configure-mpio-on-linux/IC761347.png)
     
      ![Configuración de MPIO StorSimple DATA 3](./media/storsimple-configure-mpio-on-linux/IC761348.png)
     
      En la página **Configurar**
     
     1. Asegúrese de que ambas interfaces de red están habilitadas para iSCSI. El campo **iSCSI habilitado** debe establecerse en **Sí**.
     1. Asegúrese de que las interfaces de red tienen la misma velocidad; ambas deben ser 1 GbE o 10 GbE.
     1. Anote las direcciones IPv4 de las interfaces habilitadas para iSCSI y guárdelas para su uso posterior en el host.
* Las interfaces de iSCSI en el dispositivo de StorSimple deben ser accesibles desde el servidor CentOS.
      Para comprobarlo, debe proporcionar las direcciones IP de las interfaces de red habilitadas para iSCSI de StorSimple en el servidor host. A continuación se muestran los comandos usados y la salida correspondiente con DATA2 (10.126.162.25) y DATA3 (10.126.162.26):
  
    ```console
    [root@centosSS ~]# iscsiadm -m discovery -t sendtargets -p 10.126.162.25:3260
    10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
    10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
    ```

### <a name="hardware-configuration"></a>Configuración de hardware
Se recomienda conectar las dos interfaces de red iSCSI en rutas de acceso independientes para redundancia. La siguiente ilustración muestra la configuración de hardware recomendada para múltiples rutas de alta disponibilidad y equilibrio de carga para el servidor CentOS y el dispositivo StorSimple.

![Configuración de hardware MPIO para StorSimple en un host de Linux](./media/storsimple-configure-mpio-on-linux/MPIOHardwareConfigurationStorSimpleToLinuxHost2M.png)

Como se muestra en la ilustración anterior:

* El dispositivo StorSimple está en una configuración activa-pasiva con dos controladores.
* Dos conmutadores SAN están conectados a los controladores del dispositivo.
* En el dispositivo StorSimple se habilitan dos iniciadores iSCSI.
* Dos interfaces de red están habilitadas en el host CentOS.

La configuración anterior brindará 4 rutas diferentes entre el dispositivo y el host si las interfaces de host y los datos sean enrutables.

> [!IMPORTANT]
> * Recomendamos no combinar interfaces de red de 1 GbE y de 10 GbE para las múltiples rutas. Cuando se usan dos interfaces de red, ambas deben ser de idéntico tipo.
> * En el dispositivo StorSimple, DATA0, DATA1, DATA4 y DATA5 son interfaces de 1 GbE mientras que DATA2 y DATA3 son interfaces de red de 10 GbE.|
> 
> 

## <a name="configuration-steps"></a>Pasos de configuración
Los pasos de configuración de múltiples rutas implican la configuración de las rutas de acceso disponibles para la detección automática, mediante la especificación del algoritmo de equilibrio de carga que se va a usar, la habilitación de múltiples rutas y finalmente la comprobación de la configuración. En las siguientes secciones se detallan cada uno de estos pasos.

### <a name="step-1-configure-multipathing-for-automatic-discovery"></a>Paso 1: Configuración de múltiples rutas para la detección automática
Se pueden detectar y configurar automáticamente los dispositivos compatibles con múltiples rutas.

1. Inicialice el archivo `/etc/multipath.conf` . Escriba:
   
     `mpathconf --enable`
   
    El comando anterior crea un archivo `sample/etc/multipath.conf` .
1. Inicie el servicio de múltiples rutas. Escriba:
   
    `service multipathd start`
   
    Verá la salida siguiente:
   
    `Starting multipathd daemon:`
1. Habilite la detección automática de múltiples rutas. Escriba:
   
    `mpathconf --find_multipaths y`
   
    Esto modificará la sección de valores predeterminados de `multipath.conf` , tal como se muestra a continuación:
   
    ```config
    defaults {
    find_multipaths yes
    user_friendly_names yes
    path_grouping_policy multibus
    }
    ```

### <a name="step-2-configure-multipathing-for-storsimple-volumes"></a>Paso 2: Configuración de múltiples rutas para volúmenes de StorSimple
De forma predeterminada, todos los dispositivos se encuentran en la lista negra del archivo multipath.conf y se omitirán. Deberá crear excepciones de la lista negra para permitir las múltiples rutas para volúmenes de dispositivos StorSimple.

1. Edite el archivo `/etc/mulitpath.conf` . Escriba:
   
    `vi /etc/multipath.conf`
1. Busque la sección de blacklist_exceptions en el archivo multipath.conf. El dispositivo StorSimple debe mostrarse como una excepción de la lista negra de esta sección. Puede quitar el comentario de las líneas pertinentes en este archivo para modificarlo, tal como se muestra a continuación (use solo el modelo específico del dispositivo que esté usando):
   
    ```config
    blacklist_exceptions {
        device {
                    vendor  "MSFT"
                    product "STORSIMPLE 8100*"
        }
        device {
                    vendor  "MSFT"
                    product "STORSIMPLE 8600*"
        }
    }
    ```

### <a name="step-3-configure-round-robin-multipathing"></a>Paso 3: Configuración de múltiples rutas por round-robin
Este algoritmo de equilibrio de carga usa todas las múltiples rutas disponibles en el controlador activo de forma equilibrada y por round-robin.

1. Edite el archivo `/etc/multipath.conf` . Escriba:
   
    `vi /etc/multipath.conf`
1. En la sección `defaults`, establezca `path_grouping_policy` en `multibus`. `path_grouping_policy` especifica la directiva de agrupación de rutas de acceso predeterminada se aplique a múltiples rutas no especificadas. La sección de valores predeterminados será como se muestra a continuación.
   
    ```config
    defaults {
            user_friendly_names yes
            path_grouping_policy multibus
    }
    ```

> [!NOTE]
> Los valores más comunes de `path_grouping_policy` incluyen:
> 
> * conmutación por error = 1 ruta de acceso por grupo de prioridad
> * multibus = todas las rutas de acceso válidas en un grupo de prioridad
> 
> 

### <a name="step-4-enable-multipathing"></a>Paso 4: Habilitación de múltiples rutas
1. Reinicie el demonio `multipathd` . Escriba:
   
    `service multipathd restart`
1. La salida será como se muestra a continuación:
   
    ```output
    [root@centosSS ~]# service multipathd start
    Starting multipathd daemon:  [OK]
    ```

### <a name="step-5-verify-multipathing"></a>Paso 5: Comprobación de múltiples rutas
1. En primer lugar asegúrese de que la conexión iSCSI se establece con el dispositivo StorSimple como sigue:
   
   a. Detecte su dispositivo StorSimple. Escriba:
      
    `iscsiadm -m discovery -t sendtargets -p  <IP address of network interface on the device>:<iSCSI port on StorSimple device>`
    
    La salida cuando la dirección IP de DATA0 es 10.126.162.25 y el puerto 3260 está abierto en el dispositivo StorSimple para el tráfico iSCSI saliente es como se muestra a continuación:
    
    ```output
    10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    ```

    Copie el IQN del dispositivo StorSimple, `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`, desde la salida anterior.

   b. Conecte con el dispositivo mediante el IQN de destino. El dispositivo StorSimple aquí es el destino de iSCSI. Escriba:

      `iscsiadm -m node --login -T <IQN of iSCSI target>`

    En el ejemplo siguiente se muestra la salida con un IQN de destino de `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`. La salida indica que ha conectado correctamente a las dos interfaces de red habilitadas para iSCSI en el dispositivo.

    ```output
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    ```

    Si ve solo una interfaz de host y dos rutas de acceso, tendrá que habilitar ambas interfaces de host para iSCSI. Puede seguir las [instrucciones detalladas en la documentación de Linux](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/5/html/Online_Storage_Reconfiguration_Guide/iscsioffloadmain.html).

1. Se muestra un volumen al servidor CentOS desde el dispositivo StorSimple. Para más información, consulte [Paso 6: Crear un volumen](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume) mediante Azure Portal en el dispositivo StorSimple.

1. Compruebe las rutas de acceso disponibles. Escriba:

    `multipath -l`

      En el ejemplo siguiente se muestra la salida de dos interfaces de red en un dispositivo StorSimple conectado a una interfaz de red de host única con dos rutas de acceso disponibles.

    ```output
    mpathb (36486fd20cc081f8dcd3fccb992d45a68) dm-3 MSFT,STORSIMPLE 8100
    size=100G features='0' hwhandler='0' wp=rw
    `-+- policy='round-robin 0' prio=0 status=active
    |- 7:0:0:1 sdc 8:32 active undef running
    `- 6:0:0:1 sdd 8:48 active undef running
    ```

    En el ejemplo siguiente se muestra la salida de dos interfaces de red en un dispositivo StorSimple conectado a dos interfaces de red de host con cuatro rutas de acceso disponibles.

    ```output
    mpathb (36486fd27a23feba1b096226f11420f6b) dm-2 MSFT,STORSIMPLE 8100
    size=100G features='0' hwhandler='0' wp=rw
    `-+- policy='round-robin 0' prio=0 status=active
    |- 17:0:0:0 sdb 8:16 active undef running
    |- 15:0:0:0 sdd 8:48 active undef running
    |- 14:0:0:0 sdc 8:32 active undef running
    `- 16:0:0:0 sde 8:64 active undef running
    ```

    Una vez configuradas las rutas de acceso, consulte las instrucciones específicas de su sistema operativo del host (Centos 6.6) para montar y dar formato a este volumen.

## <a name="troubleshoot-multipathing"></a>Solución de problemas de múltiples rutas
En esta sección se proporcionan algunos consejos útiles si surge algún problema durante la configuración de múltiples rutas.

Q. No puedo ver que los cambios en el archivo `multipath.conf` surtan efecto.

A. Si ha realizado algún cambio en el archivo `multipath.conf` , tendrá que reiniciar el servicio de múltiples rutas. Escriba el siguiente comando:

`service multipathd restart`

Q. He habilitado dos interfaces de red en el dispositivo StorSimple y dos interfaces de red en el host. Al mostrar las rutas de acceso disponibles, veo solo dos rutas de acceso. Esperaba ver cuatro rutas de acceso disponibles.

A. Asegúrese de que las dos rutas de acceso se encuentran en la misma subred y son enrutables. Si las interfaces de red se encuentran en VLAN distintas y no son enrutables, verá solo dos rutas de acceso. Una manera de comprobarlo es asegurarse de que puede tener acceso a las interfaces de host desde una interfaz de red en el dispositivo StorSimple. Tendrá que [ponerse en contacto con el soporte técnico de Microsoft](storsimple-8000-contact-microsoft-support.md) ya que esta comprobación solo se puede realizar a través de una sesión de soporte técnico.

Q. Cuando muestro las rutas de acceso disponibles, no aparece ninguna salida.

A. Normalmente, si no se ve ninguna ruta de acceso, indica un problema con el demonio de múltiples rutas y es muy probable que cualquier problema resida en el archivo `multipath.conf`.

También sería conveniente comprobar que realmente puede ver algunos discos después de conectarse al destino, ya que ninguna respuesta de las listas de múltiples rutas también puede deberse a que no hay ningún disco.

* Use el comando siguiente para volver a examinar el bus SCSI:
  
    `$ rescan-scsi-bus.sh`(parte del paquete sg3_utils)
* Escriba los comandos siguientes:
  
    `$ dmesg | grep sd*`
     
     Or
  
    `$ fdisk -l`
  
    Se devolverán los detalles de los discos agregados recientemente.
* Para determinar si se trata de un disco de StorSimple, use los siguientes comandos:
  
    `cat /sys/block/<DISK>/device/model`
  
    Esto devolverá una cadena, que determinará si es un disco de StorSimple.

Una causa menos probable pero posible también podría ser iscsid pid desusado. Use el comando siguiente para cerrar las sesiones de iSCSI:

`iscsiadm -m node --logout -p <Target_IP>`

Repita este comando para todas las interfaces de red conectada en el destino iSCSI, que es el dispositivo StorSimple. Una vez cerrada la sesión de todas las sesiones de iSCSI, use el IQN de destino de iSCSI para restablecer la sesión de iSCSI. Escriba el siguiente comando:

`iscsiadm -m node --login -T <TARGET_IQN>`


Q. No sé con certeza si mi dispositivo está permitido.

A. Para comprobar si el dispositivo está permitido, use el siguiente comando interactivo para solucionar problemas:

```console
multipathd -k
multipathd> show devices
available block devices:
ram0 devnode blacklisted, unmonitored
ram1 devnode blacklisted, unmonitored
ram2 devnode blacklisted, unmonitored
ram3 devnode blacklisted, unmonitored
ram4 devnode blacklisted, unmonitored
ram5 devnode blacklisted, unmonitored
ram6 devnode blacklisted, unmonitored
ram7 devnode blacklisted, unmonitored
ram8 devnode blacklisted, unmonitored
ram9 devnode blacklisted, unmonitored
ram10 devnode blacklisted, unmonitored
ram11 devnode blacklisted, unmonitored
ram12 devnode blacklisted, unmonitored
ram13 devnode blacklisted, unmonitored
ram14 devnode blacklisted, unmonitored
ram15 devnode blacklisted, unmonitored
loop0 devnode blacklisted, unmonitored
loop1 devnode blacklisted, unmonitored
loop2 devnode blacklisted, unmonitored
loop3 devnode blacklisted, unmonitored
loop4 devnode blacklisted, unmonitored
loop5 devnode blacklisted, unmonitored
loop6 devnode blacklisted, unmonitored
loop7 devnode blacklisted, unmonitored
sr0 devnode blacklisted, unmonitored
sda devnode whitelisted, monitored
dm-0 devnode blacklisted, unmonitored
dm-1 devnode blacklisted, unmonitored
dm-2 devnode blacklisted, unmonitored
sdb devnode whitelisted, monitored
sdc devnode whitelisted, monitored
dm-3 devnode blacklisted, unmonitored
```


Para más información, consulte el artículo acerca de la [solución de problemas para múltiples rutas](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/dm_multipath/mpio_admin-troubleshoot).

## <a name="list-of-useful-commands"></a>Lista de comandos útiles
| Tipo | Get-Help | Descripción |
| --- | --- | --- |
| **iSCSI** |`service iscsid start` |Iniciar el servicio iSCSI |
| &nbsp; |`service iscsid stop` |Detener el servicio iSCSI |
| &nbsp; |`service iscsid restart` |Reiniciar el servicio iSCSI |
| &nbsp; |`iscsiadm -m discovery -t sendtargets -p <TARGET_IP>` |Descubrir destinos disponibles en la dirección especificada |
| &nbsp; |`iscsiadm -m node --login -T <TARGET_IQN>` |Iniciar sesión en el destino iSCSI |
| &nbsp; |`iscsiadm -m node --logout -p <Target_IP>` |Cerrar sesión en el destino iSCSI |
| &nbsp; |`cat /etc/iscsi/initiatorname.iscsi` |Imprimir el nombre del iniciador iSCSI |
| &nbsp; |`iscsiadm -m session -s <sessionid> -P 3` |Comprobar el estado de la sesión de iSCSI y el volumen detectado en el host |
| &nbsp; |`iscsi -m session` |Muestra todas las sesiones iSCSI establecidas entre el host y el dispositivo StorSimple |
|  | | |
| **Múltiples rutas** |`service multipathd start` |Iniciar el daemon de múltiples rutas |
| &nbsp; |`service multipathd stop` |Detener el daemon de múltiples rutas |
| &nbsp; |`service multipathd restart` |Reiniciar el daemon de múltiples rutas |
| &nbsp; |`chkconfig multipathd on` </br> O BIEN </br> `mpathconf -with_chkconfig y` |Habilitar el daemon de múltiples rutas al arrancar |
| &nbsp; |`multipathd -k` |Iniciar la consola interactiva para la solución de problemas |
| &nbsp; |`multipath -l` |Enumerar dispositivos y conexiones de múltiples rutas |
| &nbsp; |`mpathconf --enable` |Crear un archivo de ejemplo mulitpath.conf en `/etc/mulitpath.conf` |
|  | | |

## <a name="next-steps"></a>Pasos siguientes
Cuando está configurando MPIO en el host Linux, es posible que tenga que hacer referencia a los siguientes documentos de CentoS 6.6:

* [Configuración de MPIO en CentOS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/dm_multipath/index)
* [Guía de aprendizaje de Linux](http://linux-training.be/linuxsys.pdf)
