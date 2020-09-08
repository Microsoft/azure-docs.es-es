---
title: Configuración de un laboratorio de piratería ética con Azure Lab Services | Microsoft Docs
description: Aprenda a configurar un laboratorio con Azure Lab Services para enseñar técnicas de piratería ética.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 5134a7db824bad69f42a4051319479f712051446
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297593"
---
# <a name="set-up-a-lab-to-teach-ethical-hacking-class"></a>Configuración de un laboratorio para impartir una clase de piratería ética 
En este artículo se muestra cómo configurar una clase que se centra en la parte referida al análisis forense de la piratería ética. Las pruebas de penetración son una práctica que usa la comunidad de piratería ética y que se producen cuando alguien intenta obtener acceso al sistema o a la red para mostrar los puntos vulnerables que un atacante malintencionado podría aprovechar. 

En una clase de piratería ética, los alumnos pueden aprender técnicas modernas para protegerse frente a los puntos vulnerables. Cada alumno obtiene una máquina virtual host de Windows Server que tiene dos máquinas virtuales anidadas: una máquina virtual con una imagen de [Metasploitable3](https://github.com/rapid7/metasploitable3) y otra con una imagen de [Kali Linux](https://www.kali.org/). La máquina virtual Metasploitable se usa para la explotación, mientras que la máquina virtual Kali proporciona acceso a las herramientas necesarias para ejecutar tareas de análisis forense.

Este artículo se divide en dos secciones principales. En la primera sección se explica cómo crear el laboratorio educativo. En la segunda se explica cómo crear la máquina de plantilla con la virtualización anidada habilitada y con las herramientas e imágenes necesarias. En este caso, una imagen de Metasploitable y una imagen de Kali Linux en una máquina con Hyper-V habilitado para hospedar las imágenes.

## <a name="lab-configuration"></a>Configuración del laboratorio
Para configurar este laboratorio, para empezar necesita una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Una vez que obtenga una suscripción de Azure, puede crear una nueva cuenta de laboratorio en Azure Lab Services o usar una existente. Consulte el siguiente tutorial sobre la creación de una nueva cuenta de laboratorio: [Tutorial de configuración de una cuenta de laboratorio](tutorial-setup-lab-account.md).

Siga [este tutorial](tutorial-setup-classroom-lab.md) para crear un laboratorio y aplique la configuración siguiente:

| Tamaño de la máquina virtual | Imagen |
| -------------------- | ----- | 
| Mediano (virtualización anidada) | Windows Server 2019 Datacenter |

## <a name="template-machine"></a>Máquina de plantilla 

Una vez creada la máquina de plantilla, inicie la máquina y conéctese a ella para completar las tres tareas principales siguientes. 
 
1. Configure la máquina para la virtualización anidada. Esto permite habilitar todas las características de Windows adecuadas, como Hyper-V, y configura las redes para que las imágenes de Hyper-V puedan comunicarse entre sí y con Internet.
2. Configure la imagen de [Kali](https://www.kali.org/) Linux. Kali es una distribución de Linux que incluye herramientas para pruebas de penetración y auditoría de seguridad.
3. Configure la imagen de Metasploitable. En este ejemplo, se utilizará la imagen [Metasploitable3](https://github.com/rapid7/metasploitable3). Esta imagen se crea a propósito para tener puntos vulnerables de seguridad.

Un script que automatiza las tareas descritas anteriormente está disponible en [Scripts de piratería ética de los servicios de laboratorio](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/EthicalHacking).

### <a name="prepare-template-machine-for-nested-virtualization"></a>Preparación de una máquina de plantilla para virtualización anidada
Siga las instrucciones de [este artículo](how-to-enable-nested-virtualization-template-vm.md) para preparar la máquina virtual de plantilla para la virtualización anidada. 

### <a name="set-up-a-nested-virtual-machine-with-kali-linux-image"></a>Configuración de una máquina virtual anidada con una imagen de Kali Linux
Kali es una distribución de Linux que incluye herramientas para pruebas de penetración y auditoría de seguridad.

1. Descargue la imagen de [https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/).  
    1. Descargue **Kali Linux Hyper-V 64 bit** para Hyper-V.
    1. Extraiga el archivo .7z.  Si aún no tiene 7 zip, descárguelo de [https://www.7-zip.org/download.html](https://www.7-zip.org/download.html). Recuerde la ubicación de la carpeta extraída, ya que la necesitará más adelante.
2. Abra el **administrador de Hyper-V** desde Herramientas administrativas.
1. Seleccione **Acción** y, a continuación, seleccione **Importar máquina virtual**. 
1. En la página **Buscar carpeta** del Asistente para **importar máquinas virtuales**, elija la ubicación de la carpeta extraída que contiene la imagen de Kali Linux.

    ![Cuadro de diálogo Buscar carpeta](./media/class-type-ethical-hacking/locate-folder.png)
1. En la página **Seleccionar máquina virtual**, seleccione la imagen de Kali Linux.  En este caso, la imagen es **kali-linux-2019.3-hyperv**.

    ![Seleccionar imagen de Kali](./media/class-type-ethical-hacking/select-kali-image.png)
1.  En la página **Elegir tipo de importación**, seleccione **Copiar la máquina virtual (crear un identificador único nuevo)** .

    ![Elegir tipo de importación](./media/class-type-ethical-hacking/choose-import-type.png)
1. Acepte los valores predeterminados de las páginas **Elegir carpetas para archivos de máquina virtual** y **Elegir carpetas para almacenar discos duros virtuales** y, a continuación, seleccione **Siguiente**.
1. En la página **Conectar red**, elija **LabServicesSwitch** que creó anteriormente en la sección **Preparación de una plantilla para la virtualización anidada** de este artículo y, a continuación, seleccione **Siguiente**.

    ![Página Conectar red](./media/class-type-ethical-hacking/connect-network.png)
1. Seleccione **Finalizar** en la página **Resumen**. Espere hasta que se completen las operaciones de copia e importación. La máquina virtual Linux de Kali estará ya disponible en Hyper-V.
1. En el **administrador de Hyper-V**, elija **Acción** -> **Iniciar** y, a continuación, elija **Acción** -> **Conectar** para conectarse a la máquina virtual.  
12. El nombre de usuario predeterminado es `root` y la contraseña es `toor`. 

    > [!NOTE]
    > Si necesita desbloquear la imagen, presione la tecla CTRL y arrastre el mouse hacia arriba.

## <a name="set-up-a-nested-vm-with-metasploitable-image"></a>Configuración de una máquina virtual anidada con una imagen de Metasploitable  
La imagen de Rapid7 Metasploitable es una imagen configurada de forma intencionada con puntos vulnerables de seguridad. Usará esta imagen para probar y encontrar problemas. Las instrucciones siguientes le muestran cómo usar una imagen de Metasploitable creada previamente. Sin embargo, si se necesita una versión más reciente de la imagen de Metasploitable, consulte [https://github.com/rapid7/metasploitable3](https://github.com/rapid7/metasploitable3).

1. Vaya a [https://information.rapid7.com/download-metasploitable-2017.html](https://information.rapid7.com/download-metasploitable-2017.html). Rellene el formulario para descargar la imagen y seleccione el botón **Enviar**.
1. Seleccione el botón **Download Metasploitable Now** (Descargar Metasploitable ahora).
1. Cuando se descargue, extraiga el archivo zip y recuerde la ubicación.
1. Convierta el archivo vmdk extraído en un archivo vhdx para que pueda usarlo con Hyper-V. Para ello, abra PowerShell con privilegios administrativos y vaya a la carpeta donde reside el archivo vmdk y siga estas instrucciones:
    1. Descargue [Microsoft Virtual Machine Converter](https://download.microsoft.com/download/9/1/E/91E9F42C-3F1F-4AD9-92B7-8DD65DA3B0C2/mvmc_setup.msi) y ejecute el archivo mvmc_setup.msi cuando se le solicite.
    1. Importe el módulo de PowerShell.  La ubicación predeterminada en la que se instala el módulo es C:\Archivos de Programa\Microsoft Virtual Machine Converter\

        ```powershell
        Import-Module 'C:\Program Files\Microsoft Virtual Machine Converter\MvmcCmdlet.psd1'
        ```
    1. Convierta el archivo vmdk en un archivo vhd que pueda usar Hyper-V. Es posible que esta operación tarde varios minutos.
    
        ```powershell
        ConvertTo-MvmcVirtualHardDisk -SourceLiteralPath .\Metasploitable.vmdk -DestinationLiteralPath .\Metasploitable.vhdx -VhdType DynamicHardDisk -VhdFormat vhdx
        ```
    1. Copie el archivo metasploitable.vhdx recién creado en C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks\. 
1. Cree una nueva máquina virtual de Hyper-V.
    1. Abra el **administrador de Hyper-V**.
    1. Elija **Acción** -> **Nuevo** -> **Máquina virtual**.
    1. En la página **Antes de comenzar** del **Asistente para nueva máquina virtual**, haga clic en **Siguiente**.
    1. En la página **Especificar nombre y ubicación**, escriba **Metasploitable** como **nombre** y seleccione **Siguiente**.

        ![Asistente para nueva imagen de máquina virtual](./media/class-type-ethical-hacking/new-vm-wizard-1.png)
    1. En la página **Especificar generación**, acepte los valores predeterminados y seleccione **Siguiente**.
    1. En la página **Asignar memoria**, escriba **512 MB** para la **memoria de inicio** y seleccione **Siguiente**. 

        ![Pagina Asignar memoria](./media/class-type-ethical-hacking/assign-memory-page.png)
    1. En la página **Configurar funciones de red**, deje la conexión como **No conectada**. A continuación, configurará el adaptador de red.
    1. En la página **Conectar disco duro virtual**, seleccione **Usar un disco duro virtual existente**. Vaya a la ubicación del archivo **metasploitable.vhdx** creado en el paso anterior y seleccione **Siguiente**. 

        ![Página Conectar disco duro virtual](./media/class-type-ethical-hacking/connect-virtual-network-disk.png)
    1. En la página **Finalización del Asistente para crear nueva máquina virtual**, seleccione **Finalizar**.
    1. Una vez creada la máquina virtual, selecciónela en el administrador de Hyper-V. No encienda la máquina todavía.  
    1. Elija **Acción** -> **Configuración**.
    1. En el cuadro de diálogo **Settings for Metasploitable** (Configuración de Metasploitable), seleccione **Agregar hardware**. 
    1. Seleccione **Adaptador de red heredado** y seleccione **Agregar**.

        ![Página Adaptador de red](./media/class-type-ethical-hacking/network-adapter-page.png)
    1. En la página **Adaptador de red heredado**, seleccione **LabServicesSwitch** para la opción **Conmutador virtual** y seleccione **Aceptar**. LabServicesSwitch se creó al preparar la máquina de plantilla para Hyper-V en la plantilla en la sección **Preparación de una plantilla para la virtualización anidada**.

        ![Página Adaptador de red heredado](./media/class-type-ethical-hacking/legacy-network-adapter-page.png)
    1. La imagen de Metasploitable ya está lista para su uso. En el **administrador de Hyper-V**, elija **Acción** -> **Iniciar** y, a continuación, elija **Acción** -> **Conectar** para conectarse a la máquina virtual.  El nombre de usuario predeterminado es **msfadmin** y la contraseña es **msfadmin**. 


La plantilla ya está actualizada y tiene las imágenes necesarias para realizar una clase de pruebas de penetración de piratería ética, una imagen con herramientas para realizar las pruebas de penetración y otra imagen con puntos vulnerables de seguridad para detectar. Ahora se puede publicar la imagen de plantilla en la clase. Seleccione el botón **Publicar** en la página de la plantilla para publicarla en el laboratorio.
  

## <a name="cost"></a>Coste  
Si desea calcular el costo de este laboratorio, puede usar el ejemplo siguiente: 
 
Para una clase de 25 alumnos con 20 horas de clase programadas y 10 horas de cuota para deberes o tareas, el precio del laboratorio sería: 

25 alumnos * (20 + 10) horas * 55 unidades de laboratorio * 0,01 USD por hora = 412,50 USD. 

Para más información sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusión
En este artículo se explican los pasos necesarios para crear un laboratorio para clases de piratería ética. Incluye los pasos para configurar la virtualización anidada para la creación de dos máquinas virtuales dentro de la máquina virtual del host para realizar pruebas de penetración.

## <a name="next-steps"></a>Pasos siguientes
Los pasos siguientes son comunes a la configuración de cualquier laboratorio:

- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users) 

