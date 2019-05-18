---
title: Instalación de un servidor de destino maestro de Linux para la conmutación por recuperación a un sitio local | Microsoft Docs
description: Aprenda a configurar un servidor de destino maestro de Linux para la conmutación por recuperación a un sitio local durante la recuperación ante desastres de máquinas virtuales de VMware en Azure con Azure Site Recovery.
author: mayurigupta13
services: site-recovery
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/06/2019
ms.author: mayg
ms.openlocfilehash: bcfeca34eb11caaddac06971fe7f825a142586a2
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602075"
---
# <a name="install-a-linux-master-target-server-for-failback"></a>Instalación de un servidor de destino maestro de Linux para la conmutación por recuperación
Después de conmutar por error las máquinas virtuales a Azure, puede conmutarlas por recuperación en el sitio local. Para ello, debe volver a proteger la máquina virtual de Azure en el sitio local. Para realizar este proceso, necesitará un servidor de destino maestro local que reciba el tráfico. 

Si la máquina virtual protegida es una máquina virtual Windows, necesitará un destino maestro de Windows. Para una máquina virtual Linux, se necesita un destino maestro de Linux. Lea los pasos siguientes para aprender a crear e instalar un destino maestro de Linux.

> [!IMPORTANT]
> A partir de la versión 9.10.0 del servidor de destino maestro, el servidor de destino maestro más reciente solo puede instalarse en un servidor Ubuntu 16.04. Las nuevas instalaciones no están permitidas en servidores CentOS6.6. Pero puede seguir actualizando los servidores de destino maestros anteriores con la versión 9.10.0.
> El servidor de destino maestro no se admite en LVM.

## <a name="overview"></a>Información general
En este artículo se proporcionan instrucciones para instalar un destino maestro Linux.

Publique cualquier comentario o pregunta que tenga al final del artículo, o bien en el [foro de Azure Recovery Services](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).

## <a name="prerequisites"></a>Requisitos previos

* Para elegir el host en el que se va a implementar el destino maestro, determine si la conmutación por recuperación va a ser en una máquina virtual local existente o en una nueva máquina virtual. 
    * En el caso de una máquina virtual existente, el host del destino maestro debe tener acceso a los almacenes de datos de la máquina virtual.
    * Si la máquina virtual local no existe (en caso de recuperación de ubicación alternativa), se creará la máquina virtual de conmutación por recuperación en el mismo host que el destino maestro. Puede elegir cualquier host ESXi para instalar el destino maestro.
* El destino maestro debe estar en una red que pueda comunicarse con el servidor de procesos y el servidor de configuración.
* La versión del destino maestro debe ser igual o anterior a las versiones del servidor de procesos y el servidor de configuración. Por ejemplo, si la versión del servidor de configuración es 9.4, la versión del destino maestro puede ser 9.4 o 9.3, pero no 9.5.
* El destino maestro solo puede ser una máquina virtual de VMware y no un servidor físico.

## <a name="sizing-guidelines-for-creating-master-target-server"></a>Directrices de ajuste de tamaño para la creación del servidor de destino maestro

Cree el destino maestro conforme a las siguientes directrices de ajuste de tamaño:
- **RAM**: 6 GB o más
- **Tamaño de disco del SO**: 100 GB o más (para instalar el sistema operativo)
- **Additional disk size for retention drive** (Tamaño adicional de disco para la unidad de retención): 1 TB
- **CPU cores** (Núcleos de CPU): 4 núcleos o más

Se admiten los siguientes kernels de Ubuntu.


|Serie de kernel  |Admite hasta la versión  |
|---------|---------|
|4.4.      |4.4.0-81-generic         |
|4.8      |4.8.0-56-generic         |
|4.10     |4.10.0-24-generic        |


## <a name="deploy-the-master-target-server"></a>Implementación del servidor de destino maestro

### <a name="install-ubuntu-16042-minimal"></a>Instalación de Ubuntu 16.04.2 mínima

Siga los siguientes pasos para instalar el sistema operativo Ubuntu 16.04.2 de 64 bits.

1.   Vaya a la [vínculo de descarga](http://old-releases.ubuntu.com/releases/16.04.2/ubuntu-16.04.2-server-amd64.iso), elija el reflejo más cercano y descargar un ISO de 64 bits mínima de Ubuntu 16.04.2.
Coloque una imagen ISO de 64 bits mínima de Ubuntu 16.04.2 en la unidad de DVD e inicie el sistema.

1.  Seleccione el **inglés** como idioma preferido y, a continuación, seleccione **Entrar**.
    
    ![Seleccionar un idioma](./media/vmware-azure-install-linux-master-target/image1.png)
1. Seleccione **Install Ubuntu Server** (Instalar servidor Ubuntu) y, a continuación, seleccione **Entrar**.

    ![Selección de instalación de servidor Ubuntu](./media/vmware-azure-install-linux-master-target/image2.png)

1.  Seleccione el **inglés** como idioma preferido y, a continuación, seleccione **Entrar**.

    ![Selección de inglés como idioma preferido](./media/vmware-azure-install-linux-master-target/image3.png)

1. Seleccione la opción adecuada de la lista de opciones **Time Zone** (Zona horaria) y, a continuación, seleccione **Entrar**.

    ![Selección de la zona horaria correcta](./media/vmware-azure-install-linux-master-target/image4.png)

1. Seleccione **No** (opción predeterminada) y, a continuación, seleccione **Entrar**.

     ![Configuración del teclado](./media/vmware-azure-install-linux-master-target/image5.png)
1. Seleccione **inglés (Estados Unidos)** como el país o región de origen para el teclado y, a continuación, seleccione **ENTRAR**.

1. Seleccione **English (US)** [Inglés (EE. UU.)] como distribución del teclado y, luego, seleccione **Entrar**.

1. Escriba el nombre de host del servidor en el cuadro **Hostname** (Nombre de host) y, luego, seleccione **Continue** (Continuar).

1. Para crear una cuenta de usuario, escriba el nombre de usuario y, luego, seleccione **Continue** (Continuar).

      ![Crear una cuenta de usuario](./media/vmware-azure-install-linux-master-target/image9.png)

1. Escriba la contraseña de la nueva cuenta de usuario y luego seleccione **Continue** (Continuar).

1.  Confirme la contraseña del nuevo usuario y, luego, seleccione **Continue** (Continuar).

    ![Confirmación de las contraseñas](./media/vmware-azure-install-linux-master-target/image11.png)

1.  En la siguiente selección que permite cifrar su directorio, seleccione **No** (opción predeterminada) y, luego, seleccione **Entrar**.

1. Si la zona horaria que se muestra es correcta, seleccione **Sí** (opción predeterminada) y, luego, seleccione **Entrar**. Para volver a configurar la zona horaria, seleccione **No**.

1. En las opciones de método de partición, seleccione **Guided - use entire disk** (Guiado: usar todo el disco) y, luego, seleccione **Entrar**.

     ![Selección de la opción de método de partición](./media/vmware-azure-install-linux-master-target/image14.png)

1.  Seleccione el disco adecuado en las opciones de **Select disk to partition** (Seleccionar disco para la partición) y, luego, seleccione **Entrar**.

    ![Selección del disco](./media/vmware-azure-install-linux-master-target/image15.png)

1.  Seleccione **Sí** para escribir los cambios en el disco y, luego, seleccione **Entrar**.

    ![Selección de la opción predeterminada](./media/vmware-azure-install-linux-master-target/image16-ubuntu.png)

1.  En la selección de configuración del proxy, seleccione la opción predeterminada, seleccione **Continue** (Continuar) y, luego, seleccione **Entrar**.
     
     ![Selección de forma de administrar las actualizaciones](./media/vmware-azure-install-linux-master-target/image17-ubuntu.png)

1.  Seleccione la opción **No automatic updates** (No aplicar actualizaciones de forma automática) en las opciones de administración de las actualizaciones de su sistema y, luego, seleccione **Entrar**.

     ![Selección de forma de administrar las actualizaciones](./media/vmware-azure-install-linux-master-target/image18-ubuntu.png)

    > [!WARNING]
    > Dado que el servidor de destino maestro de Azure Site Recovery exige una versión muy concreta de Ubuntu, tiene que asegurarse de que las actualizaciones del kernel se deshabiliten para la máquina virtual. Si están habilitadas, las actualizaciones normales hacen que el servidor de destino maestro no funcione correctamente. Asegúrese de seleccionar la opción **No automatic updates** (Sin actualizaciones automáticas).

1.  Seleccione las opciones predeterminadas. Si quiere usar OpenSSH para la conexión SSH, seleccione la opción **OpenSSH server** (Servidor de OpenSSH) y luego **Continue** (Continuar).

    ![Selección de software](./media/vmware-azure-install-linux-master-target/image19-ubuntu.png)

1. En la selección de instalación del cargador de arranque GRUB, seleccione **Yes** (Sí) y, luego, seleccione **Entrar**.
     
    ![Instalador de arranque de GRUB](./media/vmware-azure-install-linux-master-target/image20.png)


1. Seleccione el dispositivo adecuado para la instalación del cargador de arranque (preferiblemente **/dev/sda**) y, luego, seleccione **Entrar**.
     
    ![Seleccione el dispositivo adecuado.](./media/vmware-azure-install-linux-master-target/image21.png)

1. Seleccione **Continue** (Continuar) y, luego, seleccione **Entrar** para finalizar la instalación.

    ![Fin de la instalación](./media/vmware-azure-install-linux-master-target/image22.png)

1. Una vez finalizada la instalación, inicie sesión en la máquina virtual con las credenciales del nuevo usuario. (Vea el **paso 10** para más información).

1. Siga los pasos que se indican en la siguiente captura de pantalla para establecer la contraseña del usuario raíz. Luego inicie sesión como usuario raíz.

    ![Establecimiento de la contraseña del usuario raíz](./media/vmware-azure-install-linux-master-target/image23.png)


### <a name="configure-the-machine-as-a-master-target-server"></a>Configuración de la máquina como servidor de destino maestro

Para obtener el identificador de cada disco duro SCSI de una máquina virtual Linux, debe habilitar el parámetro **disk.EnableUUID = TRUE**. Para habilitar este parámetro, siga estos pasos:

1. Apague la máquina virtual.

2. Haga clic con el botón derecho en la entrada de la máquina virtual del panel izquierdo y luego seleccione **Editar configuración**.

3. Seleccione la pestaña **Opciones**.

4. En el panel izquierdo, seleccione **Opciones avanzadas** > **General** y luego el botón **Parámetros de configuración** de la parte inferior derecha de la pantalla.

    ![Abrir parámetros de configuración](./media/vmware-azure-install-linux-master-target/image24-ubuntu.png) 

    La opción **Configuration Parameters** (Parámetros de configuración) no está disponible cuando la máquina está en ejecución. Para activar esta pestaña, apague la máquina virtual.

5. Vea si ya existe una fila con **disk.EnableUUID**.

   - Si el valor existe y está establecido en **False**, cámbielo a **True**. (Los valores no distinguen mayúsculas de minúsculas).

   - Si el valor existe y está establecido en **True**, seleccione **Cancelar**.

   - Si el valor no existe, seleccione **Agregar fila**.

   - En la columna de nombre, agregue **disk.EnableUUID** y luego establezca el valor en **TRUE**.

     ![Comprobación de si el disk.EnableUUID existe](./media/vmware-azure-install-linux-master-target/image25.png)

#### <a name="disable-kernel-upgrades"></a>Deshabilitación de actualizaciones de kernel

El servidor de destino maestro de Azure Site Recovery exige una versión concreta de Ubuntu, así que asegúrese de que las actualizaciones del kernel se deshabiliten para la máquina virtual. Si están habilitadas, pueden hacer que el servidor de destino maestro no funcione correctamente.

#### <a name="download-and-install-additional-packages"></a>Descarga e instalación de paquetes adicionales

> [!NOTE]
> Asegúrese de que tiene conectividad a Internet para descargar e instalar paquetes adicionales. Si no tiene conectividad a Internet, tiene que encontrar manualmente estos paquetes Deb e instalarlos.

 `apt-get install -y multipath-tools lsscsi python-pyasn1 lvm2 kpartx`

### <a name="get-the-installer-for-setup"></a>Obtención del instalador para la configuración

Si el destino maestro tiene conectividad a Internet, puede usar los pasos siguientes para descargar el instalador. De lo contrario, puede copiar el instalador del servidor de procesos y luego instalarlo.

#### <a name="download-the-master-target-installation-packages"></a>Descarga de los paquetes de instalación del destino maestro

[Descargue los bits más recientes de instalación del destino maestro de Linux](https://aka.ms/latestlinuxmobsvc).

Para descargarlos mediante Linux, escriba lo siguiente:

`wget https://aka.ms/latestlinuxmobsvc -O latestlinuxmobsvc.tar.gz`

> [!WARNING]
> Asegúrese de que descarga y descomprime el instalador en el directorio principal. Si lo descomprime en **/usr/Local**, se produce un error en la instalación.


#### <a name="access-the-installer-from-the-process-server"></a>Acceso al instalador desde el servidor de procesos

1. En el servidor de procesos, vaya a **C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc\repository**.

2. Copie el archivo necesario del instalador del servidor de procesos y guárdelo como **latestlinuxmobsvc.tar.gz** en el directorio principal.


### <a name="apply-custom-configuration-changes"></a>Aplicación de cambios en la configuración personalizada

Para aplicar cambios de configuración personalizados, siga estos pasos:


1. Ejecute el siguiente comando para descomprimir el archivo binario.

    `tar -zxvf latestlinuxmobsvc.tar.gz`

    ![Captura de pantalla del comando que se va a ejecutar](./media/vmware-azure-install-linux-master-target/image16.png)

2. Ejecute el comando siguiente para conceder permiso.

    `chmod 755 ./ApplyCustomChanges.sh`


3. Ejecute el siguiente comando para ejecutar el script.
    
    `./ApplyCustomChanges.sh`

> [!NOTE]
> Ejecute el script sólo una vez en el servidor. A continuación, cierre el servidor. Tras agregar un disco, reinicie el servidor como se explica en la siguiente sección.

### <a name="add-a-retention-disk-to-the-linux-master-target-virtual-machine"></a>Adición de un disco de retención a la máquina virtual de destino maestro de Linux

Use los pasos siguientes para crear un disco de retención:

1. Conecte un nuevo disco de 1 TB a la máquina virtual de destino maestro Linux y luego iníciela.

2. Use el comando **multipath -ll** para conocer el identificador de múltiples rutas del disco de retención: **multipath -ll**

    ![Identificador de múltiples rutas](./media/vmware-azure-install-linux-master-target/image27.png)

3. Formatee la unidad y, a continuación, cree un sistema de archivos en la nueva unidad: **mkfs.ext4 /dev/mapper/<identificador de múltiples rutas del disco de retención>**.
    
    ![Sistema de archivos](./media/vmware-azure-install-linux-master-target/image23-centos.png)

4. Después de crear el sistema de archivos, monte el disco de retención.

    ```
    mkdir /mnt/retention
    mount /dev/mapper/<Retention disk's multipath id> /mnt/retention
    ```

5. Cree la entrada **fstab** para montar la unidad de retención cada vez que se inicie el sistema.
    
    `vi /etc/fstab`
    
    Seleccione **Insertar** para comenzar a editar el archivo. Cree una nueva línea y luego inserte el siguiente texto. Edite el identificador de múltiples rutas del disco basándose en el identificador de múltiples rutas resaltado del comando anterior.

    **/dev/mapper/\<identificador de múltiples rutas de discos de retención >/mnt/retention ext4 rw 0 0**

    Seleccione **Esc** y luego escriba **:wq** (escribir y salir) para cerrar la ventana del editor.

### <a name="install-the-master-target"></a>Instalación del destino maestro

> [!IMPORTANT]
> La versión del servidor de destino maestro debe ser igual o anterior a las versiones del servidor de procesos y el servidor de configuración. Si no se cumple esta condición, la reprotección se realiza correctamente, pero se produce un error en la replicación.


> [!NOTE]
> Antes de instalar el servidor de destino maestro, compruebe que el archivo **/etc/hosts** de la máquina virtual contiene entradas que asignan el nombre de host local a las direcciones IP asociadas a todos los adaptadores de red.

1. Copie la frase de contraseña de **C:\ProgramData\Microsoft Azure Site Recovery\private\connection.passphrase** en el servidor de configuración. Luego guarde como **passphrase.txt** en el mismo directorio local al ejecutar el comando siguiente:

    `echo <passphrase> >passphrase.txt`

    Ejemplo: 

       `echo itUx70I47uxDuUVY >passphrase.txt`
    

2. Anote la dirección IP del servidor de configuración. Ejecute el siguiente comando para instalar el servidor de destino maestro y registre el servidor con el servidor de configuración.

    ```
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <ConfigurationServer IP Address> -P passphrase.txt
    ```

    Ejemplo: 
    
    ```
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i 104.40.75.37 -P passphrase.txt
    ```

Espere a que finalice el script. Si el destino maestro se registra correctamente, aparece en la página **Infraestructura de Site Recovery** del portal.


#### <a name="install-the-master-target-by-using-interactive-installation"></a>Instalación del destino maestro mediante la instalación interactiva

1. Ejecute el siguiente comando para instalar el destino maestro. Para el rol de agente, elija **destino maestro**.

    ```
    ./install
    ```

2. Elija la ubicación predeterminada para la instalación y luego seleccione **Entrar** para continuar.

    ![Elección de una ubicación predeterminada para la instalación del destino maestro](./media/vmware-azure-install-linux-master-target/image17.png)

Una vez finalizada la instalación, registre el servidor de configuración mediante la línea de comandos.

1. Anote la dirección IP del servidor de configuración. La necesitará en el paso siguiente.

2. Ejecute el siguiente comando para instalar el servidor de destino maestro y registre el servidor con el servidor de configuración.

    ```
    ./install -q -d /usr/local/ASR -r MT -v VmWare
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <ConfigurationServer IP Address> -P passphrase.txt
    ```
    Ejemplo: 

    ```
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i 104.40.75.37 -P passphrase.txt
    ```

     Espere a que finalice el script. Si el destino maestro se ha registrado correctamente, aparece en la página **Infraestructura de Site Recovery** del portal.


### <a name="install-vmware-tools--open-vm-tools-on-the-master-target-server"></a>Instale las herramientas de VMware o de open-vm-tools en el servidor de destino maestro.

Debe instalar las herramientas de VMware o de open-vm-tools en el destino maestro para poder detectar los almacenes de datos. Si las herramientas no están instaladas, la pantalla de reprotección no aparece en los almacenes de datos. Después de la instalación de las herramientas de VMware, tiene que reiniciar.

### <a name="upgrade-the-master-target-server"></a>Actualización del servidor de destino maestro

Ejecute al programa de instalación. Detecta automáticamente que el agente está instalado en el destino maestro. Para actualizar, seleccione **Y**.  Una vez finalizada la instalación, compruebe la versión del destino maestro instalado mediante el siguiente comando:

`cat /usr/local/.vx_version`


Verá que el campo **Versión** indica el número de versión del destino maestro.

## <a name="common-issues"></a>Problemas comunes

* Asegúrese de no activar Storage vMotion en ningún componente de administración como, por ejemplo, un destino maestro. Si el destino maestro se mueve después de un reprotección correcta, no se pueden desasociar los discos de máquina virtual (VMDK). En este caso se produce un error en la conmutación por recuperación.

* La máquina de destino maestro no debe contener ninguna instantánea en la máquina virtual. Si hay instantáneas, se produce un error en la conmutación por recuperación.

* Debido a ciertas configuraciones de NIC personalizadas, la interfaz de red está deshabilitada durante el inicio, por lo que el agente del destino maestro no se puede inicializar. Asegúrese de que se han definido correctamente las siguientes propiedades. Compruebe estas propiedades en /etc/sysconfig/network-scripts/ifcfg-eth* del archivo de la tarjeta Ethernet.
    * BOOTPROTO=dhcp
    * ONBOOT=yes


## <a name="next-steps"></a>Pasos siguientes
Una vez que han terminado la instalación y el registro del destino maestro, puede verlo en la sección **Destino maestro** de **Infraestructura de Site Recovery**, en la información general del servidor de configuración.

Ahora ya puede continuar con la [reprotección](vmware-azure-reprotect.md), seguido de la conmutación por recuperación.

