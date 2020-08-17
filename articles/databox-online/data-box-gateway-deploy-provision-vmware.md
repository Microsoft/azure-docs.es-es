---
title: Tutorial acerca del aprovisionamiento de Azure Data Box Gateway en VMware | Microsoft Docs
description: El segundo tutorial para implementar Azure Data Box Gateway implica el aprovisionamiento de un dispositivo virtual en VMware.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: alkohli
ms.openlocfilehash: 64a33b12da0e9b90eaeb7039f65b2493a839a0c4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87054122"
---
# <a name="tutorial-provision-azure-data-box-gateway-in-vmware"></a>Tutorial: Aprovisionamiento de Azure Data Box Gateway en VMware

## <a name="overview"></a>Información general

En este tutorial se describe cómo aprovisionar Data Box Gateway en un sistema host que usa VMware ESXi 6.0, 6.5 o 6.7. 

Para aprovisionar un dispositivo virtual y conectarse a él se necesitan privilegios de administrador. El aprovisionamiento y la instalación inicial pueden tardar unos 10 minutos en completarse. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Asegurarse de que el host cumple los requisitos mínimos del dispositivo
> * Aprovisionamiento de un dispositivo virtual en VMWare
> * Iniciar el dispositivo virtual y obtener la dirección IP

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


## <a name="prerequisites"></a>Requisitos previos

Estos son los requisitos previos para aprovisionar un dispositivo virtual en un sistema host que usa VMware ESXi 6.0, 6.5 o 6.7.

### <a name="for-the-data-box-gateway-resource"></a>Para el recurso Data Box Gateway

Antes de comenzar, asegúrese de que:

* Ha completado todos los pasos de [Preparación del portal para Data Box Gateway](data-box-gateway-deploy-prep.md).
* Ha descargado la imagen de dispositivo virtual para VMware desde Azure Portal como se describe en [Preparación del portal para Data Box Gateway](data-box-gateway-deploy-prep.md).

  > [!IMPORTANT]
  > El software que se ejecuta en Data Box Gateway solo puede usarse con el recurso Data Box Gateway.

### <a name="for-the-data-box-gateway-virtual-device"></a>Para el dispositivo virtual de Data Box Gateway

Antes de implementar un dispositivo virtual, asegúrese de que:

* Tiene acceso a un sistema host que use VMware (ESXi 6.0, 6.5 o 6.7) que pueda utilizarse para aprovisionar un dispositivo.
* El sistema host es capaz de utilizar los recursos siguientes para aprovisionar el dispositivo virtual:

  * Un mínimo de 4 núcleos.
  * Al menos 8 GB de RAM.
  * Una interfaz de red.
  * Un disco de sistema operativo de 250 GB.
  * Un disco virtual de 2 TB para datos del sistema.

### <a name="for-the-network-in-datacenter"></a>Para la red en el centro de datos

Antes de empezar:

- Revise los requisitos de red para implementar Data Box Gateway y configure la red del centro de datos según dichos requisitos. Para más información, consulte [Requisitos de red de Data Box Gateway](data-box-gateway-system-requirements.md#networking-port-requirements).
- Asegúrese de que el ancho de banda mínimo de Internet es de 20 Mbps para que el dispositivo funcione de forma óptima.

## <a name="check-the-host-system"></a>Comprobación del sistema host

Para crear un dispositivo virtual, necesita:

* Acceso a un sistema host que use VMware ESXi Server 6.0, 6.5 o 6.7. Que el sistema host pueda dedicar los recursos siguientes al dispositivo virtual:
 
  * Un mínimo de 4 procesadores virtuales.
  * Al menos 8 GB de RAM. 
  * Una interfaz de red conectada a la red capaz de enrutar el tráfico a Internet.
  * Un disco de sistema operativo de 250 GB.
  * Un disco virtual de 2 GB para datos.
* Cliente VMware vSphere en el sistema para administrar el host ESXi.


## <a name="provision-a-virtual-device-in-hypervisor"></a>Aprovisionar un dispositivo virtual en el hipervisor

Realice los pasos siguientes para aprovisionar un dispositivo virtual en el hipervisor.

1. Copie la imagen del dispositivo virtual en el sistema. Esta imagen virtual (dos archivos) la ha descargado a través de Azure Portal. Anote la ubicación en la que copió la imagen, ya que la va a utilizar más adelante en el procedimiento.

2. Inicie sesión en el servidor ESXi mediante un explorador en esta dirección URL: `https://<IP address of the ESXi server>`. Para crear una máquina virtual es preciso tener privilegios de administrador.

   ![Página de inicio de sesión](./media/data-box-gateway-deploy-provision-vmware/image1.png)
  
3. Cargue el VMDK al servidor de ESXi. En el panel Navegador, seleccione **Almacenamiento**.

   ![Captura de pantalla de una página en el sitio del servidor ESXi que muestra el panel del navegador con la opción Almacenamiento seleccionada.](./media/data-box-gateway-deploy-provision-vmware/image2.png)

4. En el panel derecho, bajo **Datastores**(Almacenes de datos), seleccione el almacén de datos donde quiere cargar el VMDK. 

    - El almacén de datos debe ser de tipo VMFS5. 
    - También debe tener suficiente espacio disponible para los discos del sistema operativo y de datos.
   
5. Haga clic con el botón derecho y seleccione **Browse Datastore**(Examinar almacén de datos).

   ![Examen de un almacén de datos](./media/data-box-gateway-deploy-provision-vmware/image3.png)

6. Aparece la ventana **Datastore Browser** (Explorador del almacén de datos).

   ![Explorador del almacén de datos](./media/data-box-gateway-deploy-provision-vmware/image4.png)

7. En la barra de herramientas, haga clic en el icono **Crear directorio** para crear una carpeta. Especifique el nombre de la carpeta y tome nota. Utilizará este nombre de carpeta más adelante al crear una máquina virtual (práctica recomendada). Haga clic en **Crear directorio**.

   ![Creación del directorio](./media/data-box-gateway-deploy-provision-vmware/image5.png)

8. La nueva carpeta aparece en el panel izquierdo del **Datastore Browser**(Explorador del almacén de datos). Haga clic en el icono **Cargar** y seleccione **Upload file** (Cargar archivo).

    ![Carga de archivo](./media/data-box-gateway-deploy-provision-vmware/image6.png)

9. Examine y seleccione los archivos VMDK que descargó. Hay dos archivos. Seleccionar un archivo para cargar.

    ![Seleccionar el archivo que se va a cargar](./media/data-box-gateway-deploy-provision-vmware/image7.png)

10. Haga clic en **Abrir**. Se inicia la carga del archivo VMDK en el almacén de datos especificado. El archivo puede tardar varios minutos en cargarse.
11. Una vez finalizada la carga, se ve el archivo en el almacén de datos de la carpeta que creó. Ahora, cargar el segundo archivo VMDK en el mismo almacén de datos. Una vez que se hayan cargado los archivos, ambos de combinarán para crear uno solo. A partir de ese momento verá un único archivo en el directorio.

    ![Dos archivos VMDK se combinan para formar un solo archivo](./media/data-box-gateway-deploy-provision-vmware/image8.png)

12. Vuelva a la ventana de cliente de vSphere. En el panel Navegador, seleccione **Virtual Machines**. En el panel derecho, haga clic en **Create/Register VM** (Crear o registrar máquina virtual).

    ![Crear o registrar una máquina virtual](./media/data-box-gateway-deploy-provision-vmware/image9.png)

13. Aparece una **nueva máquina virtual**. En Seleccionar tipo de creación, elija **Create a new virtual machine** (Crear una máquina virtual) y haga clic en **Next** (Siguiente).
    ![Página Seleccionar tipo de creación](./media/data-box-gateway-deploy-provision-vmware/image10.png)

14. En la página **Select a Name and OS Name and Location** (Seleccionar un nombre. un nombre de sistema operativo y una ubicación), especifique el **nombre** de la máquina virtual. Dicho nombre debe ser el mismo que el de la carpeta (procedimiento recomendado) que especificó en el paso 7. En **Guest OS family** (Familia del SO de invitado), seleccione Windows y en **Guest OS version** (Versión del SO de invitado), elija Microsoft Windows Server 2016 (64 bits). Haga clic en **Next**.

    ![Página Select a Name and OS Name and Location (Seleccionar un nombre. un nombre de sistema operativo y una ubicación)](./media/data-box-gateway-deploy-provision-vmware/image11.png)

15. En la página **Select storage** (Seleccionar almacenamiento), seleccione el almacén de datos que desee utilizar para aprovisionar la máquina virtual. Haga clic en **Next**.

    ![Página Select storage (Seleccionar almacenamiento)](./media/data-box-gateway-deploy-provision-vmware/image12.png)
16. En la página **Customize settings** (Personalizar configuración), en **CPU** seleccione 4, en **Memory** (Memoria) seleccione 8192 MB (o más) y en **Hard disk 1** (Disco duro 1), seleccione 2 TB (o más). Elija el **disco duro SCSI** que se agregará. En este caso, era LSI Logic SAS. **Los discos IDE estáticos no son compatibles.** El **disco duro 1** es el disco de datos virtual. Tenga en cuenta que una vez que el disco se aprovisiona su tamaño no se puede reducir. Si intenta hacer esto, se perderán todos los datos locales del dispositivo. 

    ![Página Customize settings (Personalizar configuración)](./media/data-box-gateway-deploy-provision-vmware/image13.png)

    En la misma página, haga clic en **Add hard disk** (Agregar disco duro) y, después, seleccione **Existing hard disk** (Disco duro existente). Seleccione el archivo VMDK en el almacén de datos. Esta acción agrega un disco del sistema operativo. 

     ![Página Customize settings (Personalizar configuración)](./media/data-box-gateway-deploy-provision-vmware/image14.png)

    Desplácese hacia abajo hasta que vea el **nuevo disco duro** y expándalo para ver su configuración. En **Virtual Device Node** (Nodo de dispositivo virtual), seleccione **IDE controller 0** (Controlador IDE 0).

     ![Página Customize settings (Personalizar configuración)](./media/data-box-gateway-deploy-provision-vmware/image15.png)

17. (Opcional) *Realice este paso solo si se ejecuta VMware ESXi Server 6.7*. En la página **Customize settings** (Personalizar configuración), haga clic en **VM options** (Opciones de máquina virtual). Vaya a **Boot options > Firmware** (Opciones de arranque > Firmware) y cámbiela a **BIOS**. De forma predeterminada, el valor se establece en EFI. Haga clic en **Next**.

    ![Página Customize settings (Personalizar configuración) si se ejecuta VMware ESXi Server 6.7](./media/data-box-gateway-deploy-provision-vmware/image15a.png)

18. En la página **Listo para completarse** , revise toda la configuración asociada a la nueva máquina virtual. Compruebe si el valor de CPU es 4, la memoria es 8192 MB, la interfaz de red es 1 y el disco duro 2 tiene el controlador IDE 0. Haga clic en **Finalizar**
   
    ![Página Listo para completarse](./media/data-box-gateway-deploy-provision-vmware/image16.png)
    ![Ready to Complete page](./media/data-box-gateway-deploy-provision-vmware/image17.png)

Ahora la máquina virtual está aprovisionada. Verá una notificación al efecto y la nueva máquina virtual se agregará a la lista de máquinas virtuales.

![Nueva máquina virtual agregada a la lista de máquinas virtuales](./media/data-box-gateway-deploy-provision-vmware/image17.png)

El siguiente paso es activar esta máquina virtual y obtener la dirección IP.

> [!NOTE]
> Se recomienda no instalar las herramientas de VMware en el dispositivo virtual (aprovisionado anteriormente). La instalación de las herramientas de VMware generará una configuración incompatible.

## <a name="start-the-virtual-device-and-get-the-ip"></a>Inicio del dispositivo virtual y obtención de la dirección IP

Realice los pasos siguientes para iniciar el dispositivo virtual y conectarse a él.

#### <a name="to-start-the-virtual-device"></a>Para iniciar el dispositivo virtual
1. Inicie el dispositivo virtual. En el panel derecho, seleccione el dispositivo en la lista de máquinas virtuales y haga clic con el botón derecho para abrir el menú contextual. Seleccione **Power** (Encendido) y, luego, seleccione **Power on** (Encender). Esto debe encender la máquina virtual. Su estado se puede ver en el panel inferior cliente web.

    ![Encender el dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image19.png)

2. Vuelva a seleccionar la máquina virtual. Haga clic con el botón derecho, seleccione **Console** (Consola) y, a continuación, seleccione **Open in a new window** (Abrir en ventana nueva).

    ![Abrir la consola del dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image20.png)

3. La consola de la máquina virtual se abre en una nueva ventana. 

    ![Consola del dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image21.png)

4. Una vez que el dispositivo se esté ejecutando, señale y haga clic en el cursor en la pestaña de la parte central superior de la ventana de la consola. Seleccione **Guest OS > Send keys > Ctrl+Alt+Delete** (SO invitado > Enviar teclas > Ctrl+Alt+Supr). Se desbloqueará la máquina virtual.

   ![Desbloquear el dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image22.png)

5. Especifique la contraseña para iniciar sesión en el equipo. La contraseña predeterminada es *Password1*.

   ![Escribir la contraseña del dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image23.png)

6. Los pasos 5 a 7 solo se aplican cuando se arranca en un entorno que no sea DHCP. Si se encuentra en un entorno DHCP, omita estos pasos y vaya al paso 8. Si ha arrancado el dispositivo en un entorno sin DHCP, verá un mensaje de confirmación al respecto: **Use el cmdlet Set-HcsIPAddress para configurar la red**. 
   
7. Para configurar la red, en el símbolo del sistema, utilice el comando `Get-HcsIpAddress` para enumerar las interfaces de red habilitadas en el dispositivo virtual. Si el dispositivo tiene una única interfaz de red habilitada, el nombre predeterminado asignado a esta interfaz es `Ethernet`.

8. Utilice el cmdlet `Set-HcsIpAddress` para configurar la red. A continuación se muestra un ejemplo:

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

9. Una vez que haya finalizado la instalación inicial y el dispositivo haya arrancado, verá el texto de titular del dispositivo. Anote la dirección IP y la dirección URL que se muestran en el texto del titular para administrar el dispositivo. Utilizará esta dirección IP para conectarse a la interfaz de usuario web del dispositivo virtual y completar la instalación local y la activación.

   ![Dirección URL de conexión y texto de banner del dispositivo virtual](./media/data-box-gateway-deploy-provision-vmware/image24.png)

Si el dispositivo no cumple los requisitos mínimos de configuración, verá un error en el texto del titular (se muestra a continuación). Debe modificar la configuración del dispositivo para que tenga los recursos adecuados para cumplir los requisitos mínimos. A continuación, puede reiniciar y conectarse al dispositivo. Consulte los requisitos mínimos de configuración indicados en [Asegurarse de que el sistema host cumple los requisitos mínimos del dispositivo virtual](#check-the-host-system).

Si encuentra cualquier otro error durante la configuración inicial mediante la interfaz de usuario web local, consulte los siguientes flujos de trabajo:

- [Ejecute pruebas de diagnóstico para solucionar problemas de configuración de la interfaz de usuario web](data-box-gateway-troubleshoot.md#run-diagnostics).
- [Genere el paquete de registro y vea los archivos del registro](data-box-gateway-troubleshoot.md#collect-support-package).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Data Box Gateway, como:

> [!div class="checklist"]
> * Asegurarse de que el host cumple los requisitos mínimos del dispositivo
> * Aprovisionamiento de un dispositivo virtual en VMWare
> * Iniciar el dispositivo virtual y obtener la dirección IP

Pase al siguiente tutorial para aprender a conectar, configurar y activar dispositivos virtuales.

* [Configuración y conexión a recursos compartidos en Data Box Gateway](data-box-gateway-deploy-connect-setup-activate.md)

