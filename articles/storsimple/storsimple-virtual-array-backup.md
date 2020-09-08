---
title: Tutorial de copia de seguridad de Microsoft Azure StorSimple Virtual Array | Microsoft Docs
description: Cree copias de seguridad programadas y manuales para Microsoft Azure StorSimple Virtual Array y realice una recuperación de nivel de elemento para restaurar un archivo eliminado en la matriz virtual.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: e3cdcd9e-33b1-424d-82aa-b369d934067e
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 41cb02e77fa77c5a7452c5e3e0d2080ee3d472d5
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183231"
---
# <a name="back-up-shares-or-volumes-on-your-storsimple-virtual-array"></a>Copia de seguridad de recursos compartidos o volúmenes de la matriz virtual de StorSimple

## <a name="overview"></a>Información general

La matriz virtual de StorSimple es un dispositivo virtual local de almacenamiento en nube híbrida que se puede configurar como servidor de archivos o servidor iSCSI. La matriz virtual permite al usuario crear copias de seguridad programadas y manuales de todos los recursos compartidos o volúmenes del dispositivo. Cuando se configura como servidor de archivos, también permite la recuperación a nivel de elemento. Este tutorial describe cómo crear copias de seguridad programadas y manuales, y realizar la recuperación a nivel de elemento para restaurar un archivo eliminado en una matriz virtual.

Este tutorial se aplica solo a instancias de StorSimple Virtual Array. Para obtener información sobre la serie 8000, vaya a [Creación de una copia de seguridad de un dispositivo de la serie 8000](storsimple-manage-backup-policies-u2.md)

## <a name="back-up-shares-and-volumes"></a>Crear copias de seguridad de los recursos compartidos y volúmenes

Las copias de seguridad proporcionan seguridad a partir de un momento específico y mejoran la capacidad de recuperación, al mismo tiempo que reducen los tiempos de restauración de recursos compartidos y volúmenes. Puede hacer una copia de seguridad de un recurso compartido o de un volumen de su dispositivo StorSimple de dos maneras: **Programada** o **Manual**. En las siguientes secciones se detallan cada uno de los métodos.

## <a name="change-the-backup-start-time"></a>Cambio de la hora de inicio de la copia de seguridad

> [!NOTE]
> En esta versión, se crean copias de seguridad programadas mediante una directiva predeterminada que se ejecuta diariamente a la hora especificada y hace una copia de seguridad de todos los recursos compartidos o volúmenes en el dispositivo. No es posible crear directivas personalizadas para copias de seguridad programadas en este momento.


StorSimple Virtual Array tiene una directiva de copia de seguridad predeterminada que se inicia a una hora determinada del día (22:30) y hace una copia de seguridad de todos los recursos compartidos o volúmenes del dispositivo una vez al día. Puede cambiar la hora a la que se inicia la copia de seguridad, pero no puede cambiar la frecuencia y la retención (que especifica el número de copias de seguridad para conservar). En estas copias de seguridad, se copia todo el dispositivo virtual. Esto puede afectar potencialmente al rendimiento del dispositivo y a las cargas de trabajo implementadas en el dispositivo. Por lo tanto, es aconsejable que programe que las copias de seguridad se realicen en horas de poca actividad.

 Para cambiar la hora de inicio predeterminada de las copias de seguridad, siga estos pasos en [Azure Portal](https://portal.azure.com/).

#### <a name="to-change-the-start-time-for-the-default-backup-policy"></a>Para cambiar la hora de inicio para la directiva de copia de seguridad predeterminada

1. Vaya a **Dispositivos**. Se mostrará la lista de dispositivos registrados en el servicio StorSimple Device Manager. 
   
    ![navegar a dispositivos](./media/storsimple-virtual-array-backup/changebuschedule1.png)

2. Seleccione y haga clic en el dispositivo. Se mostrará la hoja **Configuración**. Vaya a **Administrar > Directivas de copia de seguridad**.
   
    ![seleccionar un dispositivo](./media/storsimple-virtual-array-backup/changebuschedule2.png)

3. En la hoja **Directivas de copia de seguridad**, la hora de inicio predeterminada es 22:30. La nueva hora de inicio de la programación diaria se puede especificar en la zona horaria del dispositivo.
   
    ![navegar a las directivas de copia de seguridad](./media/storsimple-virtual-array-backup/changebuschedule5.png)

4. Haga clic en **Save**(Guardar).

### <a name="take-a-manual-backup"></a>Creación de una copia de seguridad manual

Además de las copias de seguridad programadas, es posible realizar una copia de seguridad manual (a petición) del dispositivo en cualquier momento.

#### <a name="to-create-a-manual-backup"></a>Creación de una copia de seguridad manual

1. Vaya a **Dispositivos**. Seleccione el dispositivo y haga clic con el botón derecho en **...** en el extremo derecho de la fila seleccionada. En el menú contextual, seleccione **Hacer copia de seguridad**.
   
    ![navegar para realizar copias de seguridad](./media/storsimple-virtual-array-backup/takebackup1m.png)

2. En la hoja **Hacer copia de seguridad**, haga clic en **Hacer copia de seguridad**. Así se realizará una copia de seguridad de todos los recursos compartidos del servidor de archivos o de todos los volúmenes de un servidor iSCSI. 
   
    ![inicio de la copia de seguridad](./media/storsimple-virtual-array-backup/takebackup2m.png)
   
    Se inicia una copia de seguridad a petición y verá que se ha iniciado un trabajo de copia de seguridad.
   
    ![inicio de la copia de seguridad](./media/storsimple-virtual-array-backup/takebackup3m.png) 
   
    Una vez que el trabajo se haya completado correctamente, volverá a recibir una notificación. Después, se inicia el proceso de copia de seguridad.
   
    ![trabajo de copia de seguridad creado](./media/storsimple-virtual-array-backup/takebackup4m.png)

3. Para realizar un seguimiento del progreso de las copias de seguridad y examinar los detalles del trabajo, haga clic en la notificación. Esto le llevará a **Detalles del trabajo**.
   
     ![detalles del trabajo de copia de seguridad](./media/storsimple-virtual-array-backup/takebackup5m.png)

4. Una vez que se haya completado la copia de seguridad, vaya a **Administración > Catálogo de copias de seguridad**. Verá una instantánea en la nube de todos los recursos compartidos (o volúmenes) del dispositivo.
   
    ![Copia de seguridad completada](./media/storsimple-virtual-array-backup/takebackup19m.png) 

## <a name="view-existing-backups"></a>Ver copias de seguridad existentes
Para ver las copias de seguridad existentes, siga estos pasos en Azure Portal.

#### <a name="to-view-existing-backups"></a>Para ver copias de seguridad existentes

1. Vaya a la hoja **Dispositivos**. Seleccione y haga clic en el dispositivo. En la hoja **Configuración**, vaya a **Administración > Catálogo de copias de seguridad**.
   
    ![Navegar al catálogo de copias de seguridad](./media/storsimple-virtual-array-backup/viewbackups1.png)
2. Especifique los criterios siguientes que se van a usar para los filtros:
   
   - **Intervalo de tiempo**: puede ser **Última hora**, **Últimas 24 horas**, **Últimos 7 días**, **Últimos 30 días**, **Último año** y **Fecha personalizada**.
    
   - **Dispositivos**: selecciónelos en la lista de servidores de archivos o servidores iSCSI registrados en el servicio StorSimple Device Manager.
   
   - **Iniciado**: pueden ser **programados** automáticamente (por una directiva de copia de seguridad) o iniciados **manualmente** (por usted).
   
     ![Filtrar copias de seguridad](./media/storsimple-virtual-array-backup/viewbackups2.png)

3. Haga clic en **Aplicar**. La lista filtrada de las copias de seguridad se muestra en la hoja **Catálogo de copias de seguridad**. Tenga en cuenta que no se pueden mostrar más de cien elementos de copia de seguridad en un momento dado.
   
    ![Catálogo de copias de seguridad actualizado](./media/storsimple-virtual-array-backup/viewbackups3.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [administración de la matriz virtual de StorSimple](storsimple-ova-web-ui-admin.md).

