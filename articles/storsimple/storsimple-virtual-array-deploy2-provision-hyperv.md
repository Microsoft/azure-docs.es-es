---
title: Aprovisionamiento de StorSimple Virtual Array en Hyper-V | Microsoft Docs
description: En este segundo tutorial de implementación de StorSimple Virtual Array, se trata el aprovisionamiento de una matriz virtual en Hyper-V.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 4354963c-e09d-41ac-9c8b-f21abeae9913
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d6dfc95820e911781ffa88e2207601703f165839
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070586"
---
# <a name="deploy-storsimple-virtual-array---provision-in-hyper-v"></a>Implementar una matriz virtual de StorSimple: Aprovisionamiento en Hyper-V
![Diagrama que muestra los pasos necesarios para implementar una matriz virtual. La primera parte del segundo paso está etiquetada Provision on Hyper-V y está resaltada.](./media/storsimple-virtual-array-deploy2-provision-hyperv/hyperv4.png)

## <a name="overview"></a>Información general

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

En este tutorial se describe cómo aprovisionar StorSimple Virtual Array en un sistema host que ejecuta Hyper-V en Windows Server 2012 R2, Windows Server 2012 o Windows Server 2008 R2 y conectarse a él. Este artículo se aplica a la implementación de instancias de StorSimple Virtual Array en Azure Portal, así como en la nube de administración pública de Microsoft Azure.

Para aprovisionar y configurar una matriz virtual se necesitan privilegios de administrador. El aprovisionamiento y la instalación inicial pueden tardar unos 10 minutos en completarse.

## <a name="provisioning-prerequisites"></a>Requisitos previos de aprovisionamiento
Aquí encontrará los requisitos previos para aprovisionar una matriz virtual en un sistema host que ejecuta Hyper-V en Windows Server 2012 R2, Windows Server 2012 o Windows Server 2008 R2.

### <a name="for-the-storsimple-device-manager-service"></a>Para el servicio StorSimple Device Manager
Antes de comenzar, asegúrese de que:

* Ha completado todos los pasos de [Preparar el portal para la matriz virtual de StorSimple](storsimple-virtual-array-deploy1-portal-prep.md).
* Ha descargado la imagen de matriz virtual para Hyper-V de Azure Portal. Para obtener más información, consulte el **Paso 3: Descargar la imagen de la matriz virtual** de la guía [Preparación del portal para StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md).

  > [!IMPORTANT]
  > El software que se ejecuta en StorSimple Virtual Array solo puede usarse junto con el servicio StorSimple Device Manager.
  >
  >

### <a name="for-the-storsimple-virtual-array"></a>Para StorSimple Virtual Array
Antes de implementar una matriz virtual, asegúrese de que:

* Tiene acceso a un sistema host que ejecuta Hyper-V en Windows Server 2008 R2 o superior que puede utilizarse para aprovisionar un dispositivo.
* El sistema host es capaz de dedicar los recursos siguientes para aprovisionar su dispositivo virtual:

  * Un mínimo de 4 núcleos.
  * Al menos 8 GB de RAM. Si desea configurar la matriz virtual como servidor de archivos, 8 GB admite menos de 2 millones de archivos. Necesita 16 GB de RAM para admitir 2-4 millones de archivos.
  * Una interfaz de red.
  * Un disco virtual de 500 GB para datos.

### <a name="for-the-network-in-the-datacenter"></a>Para la red del centro de datos
Antes de comenzar, revise los requisitos de red para implementar StorSimple Virtual Array y configurar correctamente la red del centro de datos. Para más información, consulte [Requisitos del sistema de la matriz virtual de StorSimple](storsimple-ova-system-requirements.md#networking-requirements).

## <a name="step-by-step-provisioning"></a>Aprovisionamiento paso a paso
Para aprovisionar una matriz virtual y conectarse a ella, es preciso que realice los pasos siguientes:

1. Asegúrese de que el sistema host tiene recursos suficientes para cumplir los requisitos mínimos de la matriz virtual.
2. Aprovisione una matriz virtual en el hipervisor.
3. Inicie la matriz virtual y obtenga la dirección IP.

En las siguientes secciones se explican cada uno de estos pasos.

## <a name="step-1-ensure-that-the-host-system-meets-minimum-virtual-array-requirements"></a>Paso 1: Asegúrese de que el sistema host cumple los requisitos mínimos de la matriz virtual
Para crear una matriz virtual, se necesita:

* El rol de Hyper-V instalado en Windows Server 2012 R2, Windows Server 2012 o Windows Server 2008 R2 SP1.
* Administrador de Hyper-V de Microsoft en un cliente Microsoft Windows conectado al host.

Asegúrese de que el hardware subyacente (sistema host) en el que crea la matriz virtual pueda dedicar los siguientes recursos a la matriz virtual:

* Un mínimo de 4 núcleos.
* Al menos 8 GB de RAM. Si desea configurar la matriz virtual como servidor de archivos, 8 GB admite menos de 2 millones de archivos. Necesita 16 GB de RAM para admitir 2-4 millones de archivos.
* Una interfaz de red.
* Un disco virtual de 500 GB para datos del sistema.

## <a name="step-2-provision-a-virtual-array-in-hypervisor"></a>Paso 2: Aprovisione una matriz virtual en el hipervisor
Realice los pasos siguientes para aprovisionar un dispositivo en el hipervisor.

#### <a name="to-provision-a-virtual-array"></a>Para aprovisionar una matriz virtual
1. En el host de Windows Server, copie la imagen de matriz virtual en una unidad local. Esta imagen (VHD o VHDX) la descargó a través de Azure Portal. Anote la ubicación en la que copió la imagen, ya que la va a utilizar más adelante en el procedimiento.
2. Abra el **Administrador del servidor**. En la esquina superior derecha, haga clic en **Herramientas** y seleccione **Administrador de Hyper-V**.

   ![Captura de pantalla del Administrador del servidor que muestra un menú Herramientas expandido con el elemento Administrador de Hyper-V resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image1.png)  

   Si está ejecutando Windows Server 2008 R2, abra el Administrador de Hyper-V. En el Administrador de servidores, haga clic en **Roles > Hyper-V > Administrador de Hyper-V**.
3. En el **Administrador de Hyper-V**, en el panel de ámbito, haga clic con el botón derecho en el nodo del sistema para abrir el menú contextual y luego haga clic en **Nuevo** > **Máquina virtual**.

   ![Captura de pantalla del Administrador de Hyper-V que muestra el menú contextual de un nodo del sistema, con los elementos Nuevo y Máquina virtual resaltados.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image2.png)
4. En la página **Antes de comenzar** del Asistente para nueva máquina virtual, haga clic en **Siguiente**.
5. En la página **Especificar nombre y ubicación**, escriba el **nombre** de la matriz virtual. Haga clic en **Next**.

   ![Captura de pantalla de la página Especificar nombre y ubicación que muestra el cuadro Nombre rellenado y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image4.png)
6. En la página **Especificar generación**, elija el tipo de imagen del dispositivo y haga clic en **Siguiente**. Esta página no aparece si está utilizando Windows Server 2008 R2.

   * Elija **Generación 2** si descargó una imagen .vhdx para Windows Server 2012 o versiones posteriores.
   * Elija **Generación 1** si descargó una imagen .vhd para Windows Server 2008 R2 o versiones posteriores.

   ![Captura de pantalla de la página Especificar generación que muestra la opción Generación 2 seleccionada y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image5.png)
7. En la página **Asignar memoria**, especifique una **memoria de inicio** de al menos **8192 MB**, no habilite la memoria dinámica y luego haga clic en **Siguiente**.

   ![Captura de pantalla de la página Asignar memoria que muestra un valor de 8192 en el cuadro Memoria de inicio. El botón Siguiente está resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image6.png)  
8. En la página **Configurar funciones de red**, especifique el conmutador virtual que está conectado a Internet y luego haga clic en **Siguiente**.

   ![Captura de pantalla de la página Configurar funciones de red que muestra el elemento Conmutador virtual de Hyper-V seleccionado en el cuadro Conexión y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image7.png)
9. En la página **Conectar disco duro virtual**, elija **Utilizar un disco duro virtual existente**, especifique la ubicación de la imagen de matriz virtual (.vhdx o .vhd) y haga clic en **Siguiente**.

   ![Captura de pantalla de la página Conectar disco duro virtual con Usar un disco duro virtual existente seleccionado, una ubicación indicada y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image8m.png)
10. Revise la sección **Resumen** y luego haga clic en **Finalizar** para crear la máquina virtual.

    ![Captura de pantalla de la página Finalización del Asistente para crear nueva máquina virtual que muestra una descripción de una máquina virtual. El botón Finalizar está resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image9.png)
11. Para cumplir los requisitos mínimos, se necesitan cuatro núcleos. Para agregar cuatro procesadores virtuales, seleccione el sistema de host en la ventana **Administrador de Hyper-V**. En el panel derecho, en la lista de **máquinas virtuales**, busque la máquina virtual que acaba de crear. Seleccione el nombre del equipo, haga clic con el botón derecho y seleccione **Configuración**.

    ![Captura de pantalla de la ventana Administrador de Hyper-V. Una máquina está seleccionada en la lista, y el elemento Configuración está resaltado en el menú contextual de esa máquina.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image10.png)
12. En la página **Configuración**, en el panel izquierdo, haga clic en **Procesador**. En el panel derecho, establezca **Número de procesadores virtuales** en 4 (o más). Haga clic en **Aplicar**.

    ![Captura de pantalla de la página de configuración con el elemento Procesador resaltado, el Número de procesadores virtuales establecido en cuatro y el botón Aplicar resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image11.png)
13. Para cumplir los requisitos mínimos, también debe agregar un disco virtual de datos de 500 GB. En la página **Configuración** :

    1. En el panel izquierdo, seleccione **Controlador SCSI**.
    2. En el panel derecho, seleccione **Unidad de disco duro** y haga clic en **Agregar**.

    ![Captura de pantalla de la página de configuración con el elemento Controlador SCSI seleccionado, el elemento Unidad de disco duro seleccionado y el botón Agregar resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image12.png)
14. En la página **Unidad de disco duro**, seleccione la opción **Disco duro virtual** y haga clic en **Nuevo**. Se inicia el **Asistente para crear discos rígidos virtuales**.

    ![Captura de pantalla de la página de configuración con el elemento Unidad de disco duro seleccionado, la opción Disco duro virtual seleccionada y el botón Nuevo resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image13.png)
15. En la página **Antes de comenzar** del Asistente para crear nuevo disco virtual, haga clic en **Siguiente**.
16. En la página **Elegir formato de disco**, acepte la opción predeterminada de formato **VHDX**. Haga clic en **Next**. Esta pantalla no aparece si se ejecuta Windows Server 2008 R2.

    ![Captura de pantalla de la página Elegir formato de disco con la opción VHDX seleccionada y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image15.png)
17. En la página **Elegir tipo de disco**, establezca el tipo de disco duro virtual como **Expansión dinámica** (recomendado). Un disco de **tamaño fijo** funcionaría, pero puede que necesite esperar mucho tiempo. Se recomienda que no utilice la opción **Diferenciación** . Haga clic en **Next**. En Windows Server 2012 R2 y Windows Server 2012, la opción predeterminada es **Expansión dinámica**, mientras que en Windows Server 2008 R2, la opción predeterminada es **Tamaño fijo**.

    ![Captura de pantalla de la página Elegir tipo de disco con la opción Expansión dinámica seleccionada y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image16.png)
18. En la página **Especificar nombre y ubicación**, proporcione un **nombre**, así como una **ubicación** (puede dirigirse a una) para el disco de datos. Haga clic en **Next**.

    ![Captura de pantalla de la página Especificar nombre y ubicación que muestra los valores en los cuadros Nombre y Ubicación. El botón Siguiente está resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image17.png)
19. En la página **Configurar disco**, seleccione la opción **Crear un nuevo disco duro virtual en blanco** y especifique el tamaño como **500 GB** (o más). Aunque el requisito mínimo son 500 GB, siempre puede aprovisionar un disco más grande. Tenga en cuenta que no se puede expandir o reducir el disco una vez aprovisionado. Para obtener más información sobre el tamaño del disco para aprovisionar, revise la sección de ajuste de tamaño en el documento de [procedimientos recomendados](storsimple-ova-best-practices.md). Haga clic en **Next**.

    ![Captura de pantalla de la página Configurar disco con la opción Crear un nuevo disco duro virtual en blanco seleccionada, el tamaño establecido en 500 y el botón Siguiente resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image18.png)
20. En la página **Resumen**, revise los detalles del disco de datos virtual y, si está satisfecho, haga clic en **Finalizar** para crear el disco. El asistente se cierra y se agrega un disco duro virtual al equipo.

    ![Captura de pantalla de la página Finalización del Asistente para crear discos rígidos virtuales que muestra una descripción de un disco duro virtual. El botón Finalizar está resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image19.png)
21. Vuelva a la página **Configuración**. Haga clic en **Aceptar** para cerrar la página **Configuración** y volver a la ventana Administrador de Hyper-V.

    ![Captura de pantalla de la página de configuración. El botón Aceptar está resaltado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image20.png)

## <a name="step-3-start-the-virtual-array-and-get-the-ip"></a>Paso 3: Inicie la matriz virtual y obtenga la dirección IP
Realice los pasos siguientes para iniciar la matriz virtual y conectarse a ella.

#### <a name="to-start-the-virtual-array"></a>Para iniciar la matriz virtual
1. Inicie la matriz virtual.

   ![Captura de pantalla de la ventana Administrador de Hyper-V. La nueva matriz está seleccionada en la lista, y el elemento Iniciar está resaltado en el menú contextual de esa matriz.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image21.png)
2. Cuando el dispositivo se esté ejecutando, seleccione el dispositivo, haga clic con el botón derecho y seleccione **Conectar**.

   ![Captura de pantalla de la ventana Administrador de Hyper-V. La nueva matriz está seleccionada, su estado es En ejecución, y el elemento Conectar está resaltado en el menú contextual.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image22.png)
3. Es posible que tenga que esperar entre 5 y 10 minutos para que el dispositivo esté listo. En la consola se muestra un mensaje de estado para indicar el progreso. Cuando el dispositivo esté listo, vaya a **Acción**. Presione `Ctrl + Alt + Delete` para iniciar sesión en la matriz virtual. El usuario predeterminado es *StorSimpleAdmin* y la contraseña predeterminada es *Password1*.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere que muestra caracteres indeterminados en el cuadro de inicio de sesión.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image23.png)
4. Por motivos de seguridad, la contraseña del administrador de dispositivos expira en el primer inicio de sesión, por lo que se le pedirá que la cambie.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere. El texto de la página indica que se debe cambiar la contraseña.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image24.png)

   Escriba una contraseña que contenga al menos 8 caracteres. La contraseña debe cumplir al menos 3 de los siguientes 4 requisitos: caracteres en mayúsculas, minúsculas, numéricos y especiales. Vuelva a escribir la contraseña para confirmarla. Se le notifica que la contraseña ha cambiado.

   ![Captura de pantalla de la pestaña Consola del cliente de vSphere. El texto de la página indica que la contraseña ha cambiado.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image25.png)
5. Tras cambiar la contraseña correctamente, se puede reiniciar la matriz virtual. Espere a que el dispositivo se inicie.

   ![Captura de pantalla de la página principal de StorSimpleAdmin. El texto de la página pide al usuario que espere el servicio de notificación de eventos del sistema.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image26.png)

    La consola de Windows PowerShell del dispositivo se muestra junto a una barra de progreso.

   ![Captura de pantalla que muestra una ventana de consola con una barra de progreso. El texto de la ventana indica que la configuración inicial está en curso y pide al usuario que espere.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image27.png)
6. Los pasos 6 a 8 solo se aplican cuando se arranca en un entorno sin DHCP. Si se encuentra en un entorno de DHCP, omita estos pasos y vaya al paso 9. Si ha arrancado el dispositivo en un entorno sin DHCP, verá la siguiente pantalla.

   ![Captura de pantalla que muestra una ventana de la consola con el texto que describe el dispositivo. El símbolo del sistema dice "Controller" y aparece listo para la entrada.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image28m.png)

    A continuación, configure la red.
7. Utilice el comando `Get-HcsIpAddress` para enumerar las interfaces de red habilitadas en la matriz virtual. Si el dispositivo tiene una única interfaz de red habilitada, el nombre predeterminado asignado a esta interfaz es `Ethernet`.

   ![Captura de pantalla que muestra una ventana de la consola con la salida del comando Get-HcsIpAddress. "Ethernet" aparece como nombre del dispositivo.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image29m.png)
8. Utilice el cmdlet `Set-HcsIpAddress` para configurar la red. Observe el ejemplo siguiente:

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

    ![Captura de pantalla que muestra una ventana de la consola con la salida del comando Get-Help Set-HcsIpAddress y el uso correcto del comando Set-HcsIpAddress.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image30.png)
9. Una vez que haya finalizado la instalación inicial y el dispositivo haya arrancado, verá el texto de titular del dispositivo. Anote la dirección IP y la dirección URL que se muestran en el texto del titular para administrar el dispositivo. Utilice esta dirección IP para conectarse a la interfaz de usuario web de la matriz virtual y complete la instalación local y el registro.

   ![Captura de pantalla que muestra una ventana de la consola con texto de titular del dispositivo. Ese texto incluye la dirección IP y la dirección URL del dispositivo.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image31m.png)
10. (Opcional) Realice este paso únicamente si va a implementar el dispositivo en Government Cloud. Ahora podrá habilitar el modo FIPS (Estándar federal de procesamiento de información) de los Estados Unidos en el dispositivo. El estándar FIPS 140 define algoritmos criptográficos aprobados para su uso por parte de los sistemas informáticos del Gobierno federal de los Estados Unidos a fin de garantizar la protección de los datos confidenciales.

    1. Para habilitar el modo FIPS, ejecute el siguiente cmdlet:

        `Enable-HcsFIPSMode`
    2. Reinicie el dispositivo una vez que haya habilitado el modo FIPS para que las validaciones criptográficas surtan efecto.

       > [!NOTE]
       > Puede habilitar o deshabilitar el modo FIPS en su dispositivo. No es posible alternar entre el modo FIPS y no FIPS en el dispositivo.
       >
       >

Si el dispositivo no cumple los requisitos mínimos de configuración, verá el siguiente error en el texto del titular (se muestra a continuación). Modifique la configuración del dispositivo para que el equipo tenga los recursos adecuados para cumplir los requisitos mínimos. A continuación, puede reiniciar y conectarse al dispositivo. Consulte los requisitos mínimos de la configuración en el Paso 1: Asegúrese de que el sistema host cumple los requisitos mínimos de la matriz virtual.

![Captura de pantalla que muestra una ventana de la consola con texto de titular del dispositivo. Ese texto incluye un mensaje de error que proporciona una dirección URL para solucionar el problema.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image32.png)

Si encuentra cualquier otro error durante la configuración inicial mediante la interfaz de usuario web local, consulte los siguientes flujos de trabajo:

* Ejecute pruebas de diagnóstico para [solucionar problemas de configuración de la interfaz de usuario web](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors).
* [Genere el paquete de registro y vea los archivos del registro](storsimple-ova-web-ui-admin.md#generate-a-log-package).

## <a name="next-steps"></a>Pasos siguientes
* [Configurar la matriz virtual de StorSimple como servidor de archivos](storsimple-virtual-array-deploy3-fs-setup.md)
* [Configurar la matriz virtual de StorSimple como servidor iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md)
