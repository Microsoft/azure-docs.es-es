---
title: StorSimple Snapshot Manager y volúmenes | Microsoft Docs
description: Describe cómo usar el complemento MMC de Snapshot Manager de StorSimple para ver y administrar volúmenes y configurar copias de seguridad.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 78896323-e57c-431e-bbe2-0cbde1cf43a2
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 04/18/2016
ms.author: alkohli
ms.openlocfilehash: 79a239def70f0455f63c18d86397ec8d927f244c
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90054914"
---
# <a name="use-storsimple-snapshot-manager-to-view-and-manage-volumes"></a>Uso de Administrador de instantáneas StorSimple para ver y administrar volúmenes
## <a name="overview"></a>Información general
Puede usar el nodo **Volúmenes** de StorSimple Snapshot Manager (en el panel **Ámbito**) para seleccionar volúmenes y ver información sobre ellos. Los volúmenes se presentan como unidades que corresponden a los volúmenes montados por el host. El nodo **Volúmenes** muestra los volúmenes locales y los tipos de volúmenes admitidos por StorSimple, incluidos los volúmenes detectados mediante iSCSI y un dispositivo. 

Para obtener más información sobre los volúmenes admitidos, vaya a [Compatibilidad con varios tipos de volumen](storsimple-what-is-snapshot-manager.md#support-for-multiple-volume-types).

![Lista de volúmenes en el panel Resultados](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Volume_node.png)

El nodo **Volúmenes** también permite volver a examinar o eliminar volúmenes después de que los detecte Snapshot Manager de StorSimple. 

Este tutorial explica cómo puede montar, inicializar y dar formato a volúmenes y usar luego Administrador de instantáneas StorSimple para:

* Ver información acerca de volúmenes 
* Eliminar volúmenes
* Volver a examinar volúmenes 
* Configurar un volumen básico y realizar una copia de seguridad
* Configurar un volumen reflejado dinámico y realizar una copia de seguridad

> [!NOTE]
> Todas las acciones del nodo **Volumen** también están disponibles en el panel **Acciones**.
> 
> 

## <a name="mount-volumes"></a>Montaje de volúmenes
Utilice el procedimiento siguiente para montar, inicializar y dar formato a volúmenes de StorSimple. Este procedimiento usa Administración de discos, una utilidad del sistema para administrar discos duros y los volúmenes o particiones correspondientes. Para obtener más información sobre Administración de discos, vaya a [Administración de discos](https://technet.microsoft.com/library/cc770943.aspx) en el sitio web de Microsoft TechNet.

#### <a name="to-mount-volumes"></a>Para montar volúmenes
1. En el equipo host, inicie el iniciador iSCSI de Microsoft.
2. Proporcione una de las direcciones IP de interfaz como el portal de destino o la dirección IP de detección y conéctese al dispositivo. Una vez conectado el dispositivo, los volúmenes estarán accesibles para el sistema Windows. Para obtener más información sobre el uso del iniciador iSCSI de Microsoft, vaya a la sección "Conexión a un dispositivo de destino iSCSI" en [Instalación y configuración del iniciador iSCSI de Microsoft][1].
3. Utilice cualquiera de las siguientes opciones para iniciar Administración de discos:
   
   * Escriba Diskmgmt.msc en el cuadro **Ejecutar** .
   * Inicie el Administrador del servidor, expanda el nodo **Almacenamiento** y, luego, seleccione **Administración de discos**.
   * Inicie **Herramientas administrativas**, expanda el nodo **Administración de equipos** y, luego, seleccione **Administración de discos**. 
     
     > [!NOTE]
     > Debe usar privilegios de administrador para ejecutar Administración de discos.
     > 
     > 
4. Tome los volúmenes en línea:
   
   1. En Administración de discos, haga clic con el botón derecho en cualquier volumen marcado **Sin conexión**.
   2. Haga clic en **Reactivar disco**. El disco debería estar marcado como **En línea** después de que se vuelva a activar el disco.
5. Inicialice el volumen:
   
   1. Haga clic con el botón derecho en los volúmenes detectados.
   2. En el menú, seleccione **Inicializar disco**.
   3. En el cuadro de diálogo **Inicializar disco**, seleccione los discos que desea inicializar y, luego, haga clic en **Aceptar**.
6. Dé formato a volúmenes simples:
   
   1. Haga clic con el botón derecho en un volumen al que desee dar formato.
   2. En el menú, seleccione **Nuevo volumen simple**.
   3. Utilice al Asistente para nuevo volumen simple para dar formato al volumen:
      
      * Especifique el tamaño del volumen.
      * Proporcione una letra de unidad.
      * Seleccione el sistema de archivos NTFS.
      * Especifique un tamaño de unidad de asignación de 64 KB.
      * Realice un formateo rápido.
7. Dé formato a volúmenes de varias particiones. Para obtener instrucciones, vaya a la sección "Particiones y volúmenes" en [Implementación de Administración de discos](https://msdn.microsoft.com/library/dd163556.aspx).

## <a name="view-information-about-your-volumes"></a>Visualización de información acerca de los volúmenes
Utilice el procedimiento siguiente para ver información acerca de volúmenes locales y de Azure StorSimple.

#### <a name="to-view-volume-information"></a>Para ver la información de volumen
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple. 
2. En el panel **Ámbito**, haga clic en el nodo **Volúmenes**. Aparecerá una lista de los volúmenes locales y montados, incluidos todos los volúmenes de Azure StorSimple, en el panel **Resultados** . Las columnas del panel **Resultados** son configurables. (Haga clic con el botón derecho en el nodo **Volúmenes**, seleccione **Ver** y, después, seleccione **Agregar o quitar columnas**).
   
    ![Configurar las columnas](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_View_volumes.png)
   
   | Columna Resultados | Descripción |
   |:--- |:--- |
   |  Nombre |La columna **Nombre** contiene la letra de unidad asignada a cada volumen detectado. |
   |  Dispositivo |La columna **Dispositivo** contiene la dirección IP del dispositivo conectado al equipo host. |
   |  Nombre del volumen del dispositivo |La columna **Nombre del volumen del dispositivo** contiene el nombre del volumen del dispositivo al que pertenece el volumen seleccionado. Este es el nombre del volumen definido en Azure Portal para ese volumen específico. |
   |  Rutas de acceso |La columna **Rutas de acceso** muestra la ruta de acceso al volumen. Es la letra de unidad o el punto de montaje en el que el volumen es accesible en el equipo host. |

## <a name="delete-a-volume"></a>Eliminar un volumen
Utilice el procedimiento siguiente para eliminar un volumen en Administrador de instantáneas StorSimple.

> [!NOTE]
> No se puede eliminar un volumen si forma parte de un grupo de volúmenes. (La opción Eliminar no está disponible para los volúmenes que son miembros de un grupo de volúmenes). Debe eliminar el grupo de volúmenes completo para eliminar el volumen.

#### <a name="to-delete-a-volume"></a>Para eliminar un volumen
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic en el nodo **Volúmenes**. 
3. En el panel **Resultados** , haga clic con el botón derecho en el volumen que quiera eliminar.
4. En el menú, haga clic en **Eliminar**. 
   
    ![Eliminar un volumen](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Delete_volume.png) 
5. Aparece el cuadro de diálogo **Eliminar volumen** . Escriba **Confirmar** en el cuadro de texto y, luego, haga clic en **Aceptar**.
6. De forma predeterminada, Administrador de instantáneas StorSimple realiza una copia de seguridad de un volumen antes de eliminarlo. Esta precaución puede protegerle frente a pérdidas de datos si la eliminación es accidental. Snapshot Manager de StorSimple muestra un mensaje de progreso de una **Instantánea automática** mientras hace una copia de seguridad del volumen. 
   
    ![Mensaje de instantánea automático](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Automatic_snap.png) 

## <a name="rescan-volumes"></a>Volver a examinar volúmenes
Use el procedimiento siguiente para volver a examinar los volúmenes conectados con Administrador de instantáneas StorSimple.

#### <a name="to-rescan-the-volumes"></a>Para volver a examinar los volúmenes
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic con el botón derecho en **Volúmenes** y, luego, haga clic en **Volver a examinar volúmenes**.
   
    ![Volver a examinar volúmenes](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Rescan_volumes.png)
   
    Este procedimiento sincroniza la lista de volúmenes con Administrador de instantáneas StorSimple. Los cambios, como nuevos volúmenes o volúmenes eliminados, se reflejarán en los resultados.

## <a name="configure-and-back-up-a-basic-volume"></a>Configuración y copia de seguridad de un volumen básico
Utilice el siguiente procedimiento para configurar una copia de seguridad de un volumen básico e iniciar inmediatamente una copia de seguridad o crear una directiva de copias de seguridad programadas.

### <a name="prerequisites"></a>Prerrequisitos
Antes de empezar:

* Asegúrese de que el equipo host y el dispositivo StorSimple están configurados correctamente. Para obtener más información, vaya a [Implementación del dispositivo StorSimple local](storsimple-deployment-walkthrough-u2.md).
* Instalación y configuración de Administrador de instantáneas StorSimple Para obtener más información, vaya a [Implementación de StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md).

#### <a name="to-configure-backup-of-a-basic-volume"></a>Para configurar la copia de seguridad de un volumen básico
1. Cree un volumen básico en el dispositivo StorSimple.
2. Monte, inicialice y dé formato al volumen, tal como se describe en [Montaje de volúmenes](#mount-volumes). 
3. Haga clic en el icono Administrador de instantáneas StorSimple en su escritorio. Aparecerá el cuadro de diálogo Administrador de instantáneas StorSimple. 
4. En el panel **Ámbito**, haga clic con el botón derecho en el nodo **Volúmenes** y, luego, seleccione **Volver a examinar volúmenes**. Cuando termine el examen, debe aparecer una lista de volúmenes en el panel **Resultados** . 
5. En el panel **Resultados**, haga clic con el botón derecho en el volumen y, luego, seleccione **Crear grupo de volúmenes**. 
   
    ![Crear grupo de volúmenes](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Create_volume_group.png) 
6. En el cuadro de diálogo **Crear grupo de volúmenes**, escriba un nombre para el grupo de volúmenes, asígnele volúmenes y, luego, haga clic en **Aceptar**.
7. En el panel **Ámbito**, expanda el nodo **Grupos de volúmenes**. El nuevo grupo de volúmenes debe aparecer bajo el nodo **Grupos de volúmenes** . 
8. Haga clic con el botón derecho en el nombre del grupo de volúmenes.
   
   * Para iniciar un trabajo de copia de seguridad (a petición) interactivo, haga clic en **Realizar copia de seguridad**. 
   * Para programar una copia de seguridad automática, haga clic en **Crear directiva de copia de seguridad**. En la página **General** , seleccione un grupo de volúmenes de la lista. En la página **Programación** , escriba los detalles de la programación. Cuando haya terminado, haga clic en **Aceptar**. 
9. Para confirmar que se ha iniciado el trabajo de copia de seguridad, en el panel **Ámbito** expanda el nodo **Trabajos** y, luego, haga clic en el nodo **En ejecución**. Aparece la lista de trabajos actualmente en ejecución en el panel **Resultados** . 

## <a name="configure-and-back-up-a-dynamic-mirrored-volume"></a>Configuración y copia de seguridad de un volumen reflejado dinámico
Complete los pasos siguientes para configurar la copia de seguridad de un volumen reflejado dinámico:

* Paso 1: Uso de Administración de discos para crear un volumen reflejado dinámico. 
* Paso 2: Uso de Snapshot Manager de StorSimple para configurar la copia de seguridad.

### <a name="prerequisites"></a>Prerrequisitos
Antes de empezar:

* Asegúrese de que el equipo host y el dispositivo StorSimple están configurados correctamente. Para obtener más información, vaya a [Implementación del dispositivo StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).
* Instalación y configuración de Administrador de instantáneas StorSimple Para obtener más información, vaya a [Implementación de StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md).
* Configure dos volúmenes en el dispositivo StorSimple. (En los ejemplos, los volúmenes disponibles son **Disco 1** y **Disco 2**). 

### <a name="step-1-use-disk-management-to-create-a-dynamic-mirrored-volume"></a>Paso 1: Uso de Administración de discos para crear un volumen reflejado dinámico
Administración de discos es una utilidad del sistema para administrar discos duros y los volúmenes o particiones que contienen. Para obtener más información sobre Administración de discos, vaya a [Administración de discos](https://technet.microsoft.com/library/cc770943.aspx) en el sitio web de Microsoft TechNet.

#### <a name="to-create-a-dynamic-mirrored-volume"></a>Para crear un volumen reflejado dinámico
1. Utilice cualquiera de las siguientes opciones para iniciar Administración de discos: 
   
   * Abra el cuadro **Ejecutar**, escriba **Diskmgmt.msc** y presione Entrar.
   * Inicie el Administrador del servidor, expanda el nodo **Almacenamiento** y, luego, seleccione **Administración de discos**. 
   * Inicie **Herramientas administrativas**, expanda el nodo **Administración de equipos** y, luego, seleccione **Administración de discos**. 
2. Asegúrese de que dispone de dos volúmenes en el dispositivo StorSimple. (En el ejemplo, los volúmenes disponibles son **Disco 1** y **Disco 2**). 
3. En la ventana Administración de discos, en la columna derecha de la parte inferior, haga clic con el botón derecho en **Disco 1** y seleccione **Nuevo volumen reflejado**. 
   
    ![Nuevo volumen reflejado](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_New_mirrored_volume.png) 
4. En la página **Nuevo volumen reflejado** del asistente, haga clic en **Siguiente**.
5. En la página **Seleccionar discos**, seleccione **Disco 2** en el panel **Seleccionados**, haga clic en **Agregar** y, luego, en **Siguiente**. 
6. En la página **Asignar letra de unidad o ruta de acceso**, acepte los valores predeterminados y, luego, haga clic en **Siguiente**. 
7. En la página **Formatear volumen**, en el cuadro **Tamaño de unidad de asignación**, seleccione **64 KB**. Active la casilla **Dar formato rápido** y, luego, haga clic en **Siguiente**. 
8. En la página **Finalización del Asistente para nuevo volumen reflejado**, revise la configuración y, luego, haga clic en **Finalizar**. 
9. Aparece un mensaje para indicar que el disco básico se convertirá en un disco dinámico. Haga clic en **Sí**.
   
    ![Mensaje de conversión de disco dinámico](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Disk_management_msg.png) 
10. En Administración de discos, compruebe que los discos 1 y 2 se muestren como volúmenes reflejados dinámicos. (**Dinámicos** debe aparecer en la columna de estado y el color de la barra de capacidad debe cambiar a rojo, lo que indica un volumen reflejado). 
    
    ![Discos dinámicos reflejados de Administración de discos](./media/storsimple-snapshot-manager-manage-volumes/HCS_SSM_Verify_dynamic_disks_2.png) 

### <a name="step-2-use-storsimple-snapshot-manager-to-configure-backup"></a>Paso 2: Uso de Snapshot Manager de StorSimple para configurar la copia de seguridad
Utilice el siguiente procedimiento para configurar un volumen reflejado dinámico e iniciar inmediatamente una copia de seguridad o crear una directiva de copias de seguridad programadas.

#### <a name="to-configure-backup-of-a-dynamic-mirrored-volume"></a>Para configurar la copia de seguridad de un volumen reflejado dinámico
1. Haga clic en el icono Administrador de instantáneas StorSimple en su escritorio. Aparecerá el cuadro de diálogo Administrador de instantáneas StorSimple. 
2. En el panel **Ámbito**, haga clic con el botón derecho en el nodo **Volúmenes** y seleccione **Volver a examinar volúmenes**. Cuando termine el examen, debe aparecer una lista de volúmenes en el panel **Resultados** . El volumen reflejado dinámico se muestra como un único volumen. 
3. En el panel **Resultados**, haga clic con el botón derecho en el volumen reflejado dinámico y, luego, haga clic en **Crear grupo de volúmenes**. 
4. En el cuadro de diálogo **Crear grupo de volúmenes**, escriba un nombre para el grupo de volúmenes, asigne el volumen reflejado dinámico a este grupo y, después, haga clic en **Aceptar**. 
5. En el panel **Ámbito**, expanda el nodo **Grupos de volúmenes**. El nuevo grupo de volúmenes debe aparecer bajo el nodo **Grupos de volúmenes**. 
6. Haga clic con el botón derecho en el nombre del grupo de volúmenes. 
   
   * Para iniciar un trabajo de copia de seguridad (a petición) interactivo, haga clic en **Realizar copia de seguridad**. 
   * Para programar una copia de seguridad automática, haga clic en **Crear directiva de copia de seguridad**. En la página **General** , seleccione el grupo de volúmenes de la lista. En la página **Programación** , escriba los detalles de la programación. Cuando haya terminado, haga clic en **Aceptar**. 
7. Puede supervisar el trabajo de copia de seguridad mientras se ejecuta. En el panel **Ámbito**, expanda el nodo **Trabajos** y, después, haga clic en **En ejecución**. Aparecerán los detalles del trabajo en el panel **Resultados**. Cuando finalice el trabajo de copia de seguridad, los detalles se transferirán a la lista de trabajos de **Últimas 24 horas**. 

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple para administrar la solución de StorSimple](storsimple-snapshot-manager-admin.md).
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple  para crear y administrar grupos de volúmenes](storsimple-snapshot-manager-manage-volume-groups.md).

<!--Reference links-->
[1]: https://msdn.microsoft.com/library/ee338480(v=ws.10).aspx
