---
title: Aprovisionamiento de la matriz virtual de StorSimple en VMware
description: En este segundo tutorial de la serie de implementación de matrices virtuales de StorSimple, se trata el aprovisionamiento de un dispositivo virtual en VMware.
author: alkohli
ms.assetid: 0425b2a9-d36f-433d-8131-ee0cacef95f8
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9810a34021aa039354aad24f84aff373229c0190
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021484"
---
# <a name="deploy-storsimple-virtual-array---provision-in-vmware"></a>Implementar una matriz virtual de StorSimple: Aprovisionamiento en VMware
![Diagrama que muestra los pasos necesarios para implementar una matriz virtual. La segunda parte del segundo paso está etiquetada como Aprovisionamiento en VMware y está resaltada.](./media/storsimple-virtual-array-deploy2-provision-vmware/vmware4.png)

## <a name="overview"></a>Información general

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

En este tutorial se describe cómo aprovisionar y conectarse a una instancia de StorSimple Virtual Array en un sistema host que ejecuta VMware ESXi 5.0, 5.5, 6.0 o 6.5. Este artículo se aplica a la implementación de instancias de StorSimple Virtual Array en Azure Portal, así como en la nube de administración pública de Microsoft Azure.

Para aprovisionar un dispositivo virtual y conectarse a él se necesitan privilegios de administrador. El aprovisionamiento y la instalación inicial pueden tardar unos 10 minutos en completarse.

## <a name="provisioning-prerequisites"></a>Requisitos previos de aprovisionamiento
Estos son los requisitos previos para aprovisionar un dispositivo virtual en un sistema host que usa VMware ESXi 5.0, 5.5, 6.0 o 6.5.

### <a name="for-the-storsimple-device-manager-service"></a>Para el servicio StorSimple Device Manager
Antes de comenzar, asegúrese de que:

* Ha completado todos los pasos de [Preparar el portal para la matriz virtual de StorSimple](storsimple-virtual-array-deploy1-portal-prep.md).
* Ha descargado la imagen del dispositivo virtual para VMware desde el Portal de Azure. Para obtener más información, consulte el **Paso 3: Descargar la imagen del dispositivo virtual** de [Deploy StorSimple Virtual Array - Prepare the portal](storsimple-virtual-array-deploy1-portal-prep.md) (Implementación de StorSimple Virtual Array: preparar el portal).

### <a name="for-the-storsimple-virtual-device"></a>Para el dispositivo virtual StorSimple
Antes de implementar un dispositivo virtual, asegúrese de que:

* Tiene acceso a un sistema host que ejecuta Hyper-V (2008 R2 o superior) que pueda utilizarse para aprovisionar un dispositivo.
* El sistema host es capaz de utilizar los recursos siguientes para aprovisionar el dispositivo virtual:

  * Un mínimo de 4 núcleos.
  * Al menos 8 GB de RAM. Si desea configurar la matriz virtual como servidor de archivos, 8 GB admite menos de 2 millones de archivos. Necesita 16 GB de RAM para admitir 2-4 millones de archivos.
  * Una interfaz de red.
  * Un disco virtual de 500 GB para datos del sistema.

### <a name="for-the-network-in-datacenter"></a>Para la red en el centro de datos
Antes de comenzar, asegúrese de que:

* Ha revisado los requisitos de red para implementar un dispositivo virtual StorSimple y ha configurado la red del centro de datos según los requisitos. 

## <a name="step-by-step-provisioning"></a>Aprovisionamiento paso a paso
Para aprovisionar un dispositivo virtual y conectarse a él, es preciso que realice los pasos siguientes:

1. Asegúrese de que el sistema host tiene recursos suficientes para cumplir los requisitos mínimos del dispositivo virtual.
2. Aprovisione un dispositivo virtual en el hipervisor.
3. Inicie el dispositivo virtual y obtenga la dirección IP.

## <a name="step-1-ensure-host-system-meets-minimum-virtual-device-requirements"></a>Paso 1: Asegurarse de que el sistema host cumple con los requisitos mínimos del dispositivo virtual
Para crear un dispositivo virtual, necesitará:

* Acceso a un sistema host que use VMware ESXi Server 5.0, 5.5, 6.0 o 6.5.
* Cliente VMware vSphere en el sistema para administrar el host ESXi.

  * Un mínimo de 4 núcleos.
  * Al menos 8 GB de RAM. Si desea configurar la matriz virtual como servidor de archivos, 8 GB admite menos de 2 millones de archivos. Necesita 16 GB de RAM para admitir 2-4 millones de archivos.
  * Una interfaz de red conectada a la red capaz de enrutar el tráfico a Internet. El ancho de banda mínimo de Internet debe ser de 5 Mbps para permitir el funcionamiento óptimo del dispositivo.
  * Un disco virtual de 500 GB para datos.

## <a name="step-2-provision-a-virtual-device-in-hypervisor"></a>Paso 2: Aprovisionar un dispositivo virtual en el hipervisor
Realice los pasos siguientes para aprovisionar un dispositivo virtual en el hipervisor.

1. Copie la imagen del dispositivo virtual en el sistema. Esta imagen virtual la descargó a través de Azure Portal.

   1. Asegúrese de que ha descargado el archivo de imagen más reciente. Si ha descargado la imagen anterior, descárguelo nuevo para asegurarse de que se trata de la imagen más reciente. La última imagen tiene dos archivos (en lugar de uno).
   2. Anote la ubicación en la que copió la imagen, ya que la va a utilizar más adelante en el procedimiento.

2. Inicie sesión en el servidor de ESXi mediante el cliente de vSphere. Para crear una máquina virtual es preciso tener privilegios de administrador.

   ![Captura de pantalla de la página de inicio de sesión del cliente de vSphere. Los cuadros IP address (Dirección IP), User name (Nombre de usuario) y Password (Contraseña) contienen valores y el botón Login (Inicio de sesión) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image1.png)
3. En el cliente de vSphere, en la sección de inventario en el panel izquierdo, seleccione el servidor ESXi.

   ![Captura de pantalla de la página principal del cliente de vSphere. En la sección Inventory (Inventario), el servidor ESXi está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image2.png)
4. Cargue el VMDK al servidor de ESXi. Desplácese hasta la pestaña **Configuration** (Configuración) del panel derecho. En **Hardware**, seleccione **Storage** (Almacenamiento).

   ![Captura de pantalla que muestra la pestaña Configuration (Configuración) del cliente de vSphere. En la sección Hardware, Storage (Almacenamiento) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image3.png)
5. En el panel derecho, bajo **Datastores**(Almacenes de datos), seleccione el almacén de datos donde quiere cargar el VMDK. El almacén de datos debe tener espacio suficiente para los discos de datos y el sistema operativo.

   ![Captura de pantalla que muestra la página Storage (Almacenamiento) del cliente de vSphere. La pestaña Datastores (Almacenes de datos) está abierta y contiene una lista de almacenes de datos. Un almacén de datos está seleccionado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image4.png)
6. Haga clic con el botón derecho y seleccione **Browse Datastore**(Examinar almacén de datos).

   ![Captura de pantalla que muestra el menú contextual del almacén de datos seleccionado. El elemento del almacén de datos Browse (Examinar) está seleccionado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image5.png)
7. Aparece la ventana **Datastore Browser** (Explorador del almacén de datos).

   ![Captura de pantalla de un explorador de almacén de datos. Se pueden ver las carpetas del almacén de datos.](./media/storsimple-virtual-array-deploy2-provision-vmware/image6.png)
8. En la barra de herramientas, haga clic en el icono :::image type="icon" source="./media/storsimple-virtual-array-deploy2-provision-vmware/image7.png"::: para crear una nueva carpeta. Especifique el nombre de la carpeta y tome nota. Utilizará este nombre de carpeta más adelante al crear una máquina virtual (práctica recomendada). Haga clic en **OK**.

   ![Captura de pantalla de un explorador de almacén de datos con el nuevo icono de carpeta resaltado. Un cuadro de diálogo se ha rellenado con un nombre de carpeta y el botón OK (Aceptar) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image8.png)
9. La nueva carpeta aparece en el panel izquierdo del **Datastore Browser**(Explorador del almacén de datos).

   ![Captura de pantalla de un explorador de almacén de datos con la nueva carpeta visible en la jerarquía de carpetas.](./media/storsimple-virtual-array-deploy2-provision-vmware/image9.png)
10. Haga clic en el icono de carga :::image type="icon" source="./media/storsimple-virtual-array-deploy2-provision-vmware/image10.png"::: y seleccione **Upload file**(Cargar archivo).

    ![Captura de pantalla que muestra el menú contextual del icono Upload (Cargar). El elemento Upload File (Cargar archivo) está seleccionado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image11.png)
11. Examine y seleccione los archivos VMDK que descargó. Hay dos archivos. Seleccionar un archivo para cargar.

    ![Captura de pantalla de un cuadro de diálogo que muestra carpetas y dos archivos VMDK. Uno de los archivos está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image12m.png)
12. Haga clic en **Abrir**. Se inicia la carga del archivo VMDK en el almacén de datos especificado. El archivo puede tardar varios minutos en cargarse.
13. Una vez finalizada la carga, se ve el archivo en el almacén de datos de la carpeta que creó.

    ![Captura de pantalla de un explorador de almacén de datos. La nueva carpeta está resaltada en la jerarquía de carpetas y el archivo cargado se puede ver en esa carpeta.](./media/storsimple-virtual-array-deploy2-provision-vmware/image14.png)

    Ahora, cargar el segundo archivo VMDK en el mismo almacén de datos.
14. Vuelva a la ventana de cliente de vSphere. Con el servidor ESXi seleccionado, haga clic con el botón secundario y seleccione **Nueva máquina virtual**.

    ![Captura de pantalla del menú contextual de un servidor ESXi. El elemento New virtual machine (Nueva máquina virtual) está seleccionado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image15.png)
15. Aparece una ventana **Crear nueva máquina virtual** . En la página **Configuration** (Configuración), seleccione la opción **Custom** (Personalizado). Haga clic en **Next**.
    ![Captura de pantalla de la página Configuration (Configuración) de la ventana Create new virtual machine (Crear nueva máquina virtual). La opción Custom (Personalizar) está seleccionada y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image16.png)
16. En la página **Nombre y ubicación** , especifique el nombre de la máquina virtual. Este nombre debe coincidir con el nombre de carpeta (práctica recomendada) que especificó anteriormente en el paso 8.

    ![Captura de pantalla de la página Name and location (Nombre y ubicación) de la ventana Create new virtual machine (Crear nueva máquina virtual). El cuadro Name (Name) está rellenado y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image17.png)
17. En la página **Almacenamiento** , seleccione un almacén de datos que desee utilizar para aprovisionar la máquina virtual.

    ![Captura de pantalla de la página Storage (Almacenamiento) de la ventana Create new virtual machine (Crear nueva máquina virtual). Hay un almacén de datos seleccionado y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image18.png)
18. En la página **Virtual Machine Version** (Versión de la máquina virtual), seleccione **Virtual Machine Version: 8** (Versión de la máquina virtual: 8).

    ![Captura de pantalla de la página Virtual Machine Version (Versión de la máquina virtual). La opción Virtual machine versión 8 (Versión de la máquina virtual 8) está seleccionada y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image19.png)
19. En la página **Guest Operating System** (Sistema operativo invitado), seleccione **Windows** como **Guest Operating System** (Sistema operativo invitado). Para **Version** (Versión), en la lista desplegable, seleccione **Microsoft Windows Server 2012 (64 bits)** .

    ![Captura de pantalla de la página Guest Operating System (Sistema operativo invitado) con Windows seleccionado, la versión establecida en Microsoft Windows Server 2012 (64 bits) y el botón Next (Siguiente) resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image20.png)
20. En la página **CPU**, ajuste **Number of virtual sockets** (Número de sockets virtuales) y **Number of cores per virtual socket** (Número de núcleos por socket virtual) para que **Total number of cores** (Número total de núcleos) sea de 4 (o más). Haga clic en **Next**.

    ![Captura de pantalla de la página CPU que muestra un socket virtual, cuatro núcleos por socket virtual y cuatro núcleos totales. El botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image21.png)
21. En la página **Memoria** , especifique 8 GB (o más) de RAM. Haga clic en **Next**.

    ![Captura de pantalla de la página Memory (Memoria). El tamaño de la memoria está rellenado con un valor de 8 GB.](./media/storsimple-virtual-array-deploy2-provision-vmware/image22.png)
22. En la página **Red** , especifique el número de interfaces de red. El requisito mínimo es una interfaz de red.

    ![Captura de pantalla de la página Network (Red). El número de interfaces de red está establecido en uno y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image23.png)
23. En la página **SCSI Controller** (Controlador SCSI), acepte el valor predeterminado **LSI Logic SAS**.

    ![Captura de pantalla de la página SCSI Controller (Controlador SCSI). La opción LSI Logic SAS está seleccionada y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image24.png)
24. En la página **Selecta a Disk** (Seleccionar un disco), elija **Use an existing virtual disk** (Utilizar un disco virtual existente). Haga clic en **Next**.

    ![Captura de pantalla de la página Select a disk (Seleccionar un disco), con la opción Use an existing virtual disk (Utilizar un disco virtual existente) seleccionada y el botón Next (Siguiente) resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image25.png)
25. En la página **Select Existing Disk** (Seleccionar disco existente), en **Disk File Path** (Ruta de acceso de archivo de disco), haga clic en **Browse** (Examinar). Se abrirá un cuadro de diálogo **Examinar almacenes de datos** . Desplácese hasta la ubicación en la que cargó el VMDK. Ahora ve solo un archivo en el almacén de datos, ya que se han combinado los dos archivos que cargó al principio. Seleccione el archivo y haga clic en **Aceptar**. Haga clic en **Next**.

    ![Captura de pantalla de la página Select existing disk (Seleccionar disco existente). El botón Browse (Examinar) está resaltado y un cuadro de diálogo contiene un archivo y un botón OK (Aceptar) resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image26.png)
26. En la página **Advanced Options** (Opciones avanzadas), acepte el valor predeterminado y haga clic en **Next** (Siguiente).

    ![Captura de pantalla de la página Advanced Options (Opciones avanzadas). El botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image27.png)
27. En la página **Listo para completarse** , revise toda la configuración asociada a la nueva máquina virtual. Marque **Editar la configuración de la máquina virtual antes de completarse**. Haga clic en **Continuar**.

    ![Captura de pantalla de la página Ready to Complete (Lista para finalizar) con el botón Continue (Continuar) resaltado. La opción Edit the virtual machine settings before completion (Editar la configuración de la máquina virtual antes de completarse) está marcada.](./media/storsimple-virtual-array-deploy2-provision-vmware/image28.png)
28. En la página **Virtual Machines Properties** (Propiedades de las máquinas virtuales), en la pestaña **Hardware**, busque el hardware del dispositivo. Seleccione **Nuevo disco duro**. Haga clic en **Agregar**.

    ![Captura de pantalla de la pestaña Hardware de la página Virtual Machines Properties (Propiedades de las máquinas virtuales). La opción New Hard Disk (Nuevo disco duro) está seleccionada en la lista de hardware. El botón Add (Agregar) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image29.png)
29. Verá una ventana **Agregar hardware**. En la página **Device Type** (Tipo de dispositivo), en **Choose the type of device you wish to add** (Elegir el tipo de dispositivo que desea agregar), seleccione **Hard Disk** (Unidad de disco duro) y haga clic en **Next** (Siguiente).

    ![Captura de pantalla de la página Device Type (Tipo de dispositivo) de la ventana Add hardware (Agregar hardware). El dispositivo de disco duro está seleccionado y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image30.png)
30. En la página **Select a Disk** (Seleccionar un disco), elija **Create a new virtual disk** (Crear un nuevo disco virtual). Haga clic en **Next**.

    ![Captura de pantalla de la página Select a Disk (Seleccionar un disco). La opción Create a new virtual disk (Crear un nuevo disco virtual) está seleccionada y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image31.png)
31. En la página **Create a Disk** (Crear un disco), cambie el valor de **Disk Size** (Tamaño de disco) a 500 GB (o más). Aunque el requisito mínimo son 500 GB, siempre puede aprovisionar un disco más grande. Tenga en cuenta que no se puede expandir o reducir el disco una vez aprovisionado. Para obtener más información sobre el tamaño del disco para aprovisionar, revise la sección de ajuste de tamaño en el documento de [procedimientos recomendados](storsimple-ova-best-practices.md). En **Disk Provisioning** (Aprovisionamiento de disco), seleccione **Thin Provision** (Aprovisionamiento fino). Haga clic en **Next**.

    ![Captura de pantalla de la página Create a disk (Crear un disco). El tamaño del disco está establecido en 500 GB, la opción Thin provision (Aprovisionamiento fino) está seleccionada y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image32.png)
32. En la página **Opciones avanzadas** , acepte el valor predeterminado.

    ![Captura de pantalla de la página Advanced Options (Opciones avanzadas). El nodo de dispositivo virtual está establecido en SCSI (0:0) y el botón Next (Siguiente) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image33.png)
33. En la página **Listo para completarse** , revise las opciones del disco. Haga clic en **Finalizar**

    ![Captura de pantalla de la página Ready to Complete (Lista para finalizar). Aparece un resumen de las opciones de disco y el botón Finish (Finalizar) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image34.png)
34. Vuelva a la página Propiedades de la máquina virtual. Se agregó un nuevo disco duro a la máquina virtual. Haga clic en **Finalizar**

    ![Captura de pantalla de la página Virtual Machine Properties (Propiedades de las máquinas virtuales). La lista de hardware contiene el nuevo disco duro y el botón Finish (Finalizar) está resaltado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image35.png)
35. Con la máquina virtual seleccionada en el panel derecho, desplácese hasta la pestaña **Resumen** . Revise la configuración de la máquina virtual.

    ![Captura de pantalla de la pestaña Summary (Resumen) del cliente de vSphere. La nueva máquina virtual está resaltada y se pueden ver sus recursos y propiedades generales.](./media/storsimple-virtual-array-deploy2-provision-vmware/image36.png)

Ahora la máquina virtual está aprovisionada. El paso siguiente es encender este equipo y obtener la dirección IP.

> [!NOTE]
> Se recomienda que no instale las herramientas de VMware en la matriz virtual (aprovisionada anteriormente). La instalación de las herramientas de VMware generará una configuración incompatible.

## <a name="step-3-start-the-virtual-device-and-get-the-ip"></a>Paso 3: Inicio del dispositivo virtual y obtención de la dirección IP
Realice los pasos siguientes para iniciar el dispositivo virtual y conectarse a él.

#### <a name="to-start-the-virtual-device"></a>Para iniciar el dispositivo virtual
1. Inicie el dispositivo virtual. En el panel izquierdo del Administrador de configuración de vSphere, seleccione el dispositivo y haga clic con el botón secundario para mostrar el menú contextual. Seleccione **Power** (Encendido) y, luego, seleccione **Power on** (Encender). Esto debe encender la máquina virtual. Puede ver el estado en el panel inferior **Tareas recientes** del cliente vSphere.

   ![Captura de pantalla del menú contextual del dispositivo. El elemento Power (Encendido) está seleccionado. Aparece un menú adyacente con el elemento Power On (Encender) seleccionado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image37.png)
2. Las tareas de instalación pueden tardar unos minutos en completarse. Una vez que el dispositivo se esté ejecutando, diríjase a la pestaña **Consola** . Presione Ctrl+Alt+Supr para iniciar sesión en el dispositivo. Como alternativa, puede colocar el cursor en la ventana de la consola y presionar Ctrl+Alt+Insert. El usuario predeterminado es *StorSimpleAdmin* y la contraseña predeterminada es *Password1*.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere. El cuadro Password (Contraseña) está vacío.](./media/storsimple-virtual-array-deploy2-provision-vmware/image38.png)
3. Por motivos de seguridad, la contraseña del administrador de dispositivos expira en el primer inicio de sesión, por lo que se le pedirá que la cambie.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere. El texto de la página indica que se debe cambiar la contraseña.](./media/storsimple-virtual-array-deploy2-provision-vmware/image39.png)
4. Escriba una contraseña que contenga al menos 8 caracteres. La contraseña debe contener 3 de 4 de estos requisitos: caracteres en mayúsculas, minúsculas, números y caracteres especiales. Vuelva a escribir la contraseña para confirmarla. Se le notificará que la contraseña ha cambiado.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere. El texto de la página indica que la contraseña se ha cambiado.](./media/storsimple-virtual-array-deploy2-provision-vmware/image40.png)
5. Tras cambiar la contraseña correctamente, puede que se reinicie el dispositivo virtual. Espere a que se complete el reinicio. La consola de Windows PowerShell del dispositivo puede mostrarse junto a una barra de progreso.

   ![Captura de pantalla que muestra una ventana de la consola con una barra de progreso. El texto de la ventana indica que la instalación inicial está en curso y pide al usuario que espere.](./media/storsimple-virtual-array-deploy2-provision-vmware/image41.png)
6. Los pasos 6 a 8 solo se aplican cuando se arranca en un entorno sin DHCP. Si se encuentra en un entorno de DHCP, omita estos pasos y vaya al paso 9. Si ha arrancado el dispositivo en un entorno sin DHCP, verá la siguiente pantalla.

   ![Captura de pantalla que muestra una ventana de la consola con texto que describe el dispositivo. El símbolo del sistema dice "Controller" y aparece listo para la entrada.](./media/storsimple-virtual-array-deploy2-provision-vmware/image42m.png)

   A continuación, configure la red.
7. Utilice el comando `Get-HcsIpAddress` para enumerar las interfaces de red habilitadas en el dispositivo virtual. Si el dispositivo tiene una única interfaz de red habilitada, el nombre predeterminado asignado a esta interfaz es `Ethernet`.

   ![Captura de pantalla que muestra una ventana de la consola con la salida del comando Get-HcsIpAddress. "Ethernet" aparece como nombre del dispositivo.](./media/storsimple-virtual-array-deploy2-provision-vmware/image43m.png)
8. Utilice el cmdlet `Set-HcsIpAddress` para configurar la red. A continuación se muestra un ejemplo:

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

    ![Captura de pantalla que muestra una ventana de la consola con la salida del comando Get-Help Set-HcsIpAddress y el uso correcto del comando Set-HcsIpAddress.](./media/storsimple-virtual-array-deploy2-provision-vmware/image44.png)
9. Una vez que haya finalizado la instalación inicial y el dispositivo haya arrancado, verá el texto de titular del dispositivo. Anote la dirección IP y la dirección URL que se muestran en el texto del titular para administrar el dispositivo. Utilizará esta dirección IP para conectarse a la interfaz de usuario web del dispositivo virtual y completar la instalación local y el registro.

   ![Captura de pantalla que muestra una ventana de la consola con texto de titular del dispositivo. Ese texto incluye la dirección IP y la dirección URL del dispositivo.](./media/storsimple-virtual-array-deploy2-provision-vmware/image45.png)
10. (Opcional) Realice este paso únicamente si va a implementar el dispositivo en Government Cloud. Ahora podrá habilitar el modo FIPS (Estándar federal de procesamiento de información) de los Estados Unidos en el dispositivo. El estándar FIPS 140 define algoritmos criptográficos aprobados para su uso por parte de los sistemas informáticos del Gobierno federal de los Estados Unidos a fin de garantizar la protección de los datos confidenciales.

    1. Para habilitar el modo FIPS, ejecute el siguiente cmdlet:

        `Enable-HcsFIPSMode`
    2. Reinicie el dispositivo una vez que haya habilitado el modo FIPS para que las validaciones criptográficas surtan efecto.

       > [!NOTE]
       > Puede habilitar o deshabilitar el modo FIPS en su dispositivo. No es posible alternar entre el modo FIPS y no FIPS en el dispositivo.
       >
       >

Si el dispositivo no cumple los requisitos mínimos de configuración, verá un error en el texto del titular (se muestra a continuación). Debe modificar la configuración del dispositivo para que tenga los recursos adecuados para cumplir los requisitos mínimos. A continuación, puede reiniciar y conectarse al dispositivo. Consulte los requisitos mínimos de la configuración en el [Paso 1: Asegurarse de que el sistema host cumple los requisitos mínimos del dispositivo virtual](#step-1-ensure-host-system-meets-minimum-virtual-device-requirements).

![Captura de pantalla que muestra una ventana de la consola con texto de titular del dispositivo. Ese texto incluye un mensaje de error que proporciona una dirección URL para solucionar el problema.](./media/storsimple-virtual-array-deploy2-provision-vmware/image46.png)

Si encuentra cualquier otro error durante la configuración inicial mediante la interfaz de usuario web local, consulte los siguientes flujos de trabajo:

* Ejecute pruebas de diagnóstico para [solucionar problemas de configuración de la interfaz de usuario web](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors).
* [Genere el paquete de registro y vea los archivos del registro](storsimple-ova-web-ui-admin.md#generate-a-log-package).

## <a name="next-steps"></a>Pasos siguientes
* [Configurar la matriz virtual de StorSimple como servidor de archivos](storsimple-virtual-array-deploy3-fs-setup.md)
* [Configurar la matriz virtual de StorSimple como servidor iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md)
