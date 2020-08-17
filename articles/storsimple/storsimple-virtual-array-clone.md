---
title: Clonación de una copia de seguridad de StorSimple Virtual Array | Microsoft Docs
description: Aprenda a clonar una copia de seguridad y a recuperar un archivo de StorSimple Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: af6e979c-55e3-477c-b53e-a76a697f80c9
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/21/2016
ms.author: alkohli
ms.openlocfilehash: dd82f7dd821991bbcf57cc7d8105575e0baa4d12
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070687"
---
# <a name="clone-from-a-backup-of-your-storsimple-virtual-array"></a>Clonación a partir de una copia de seguridad de StorSimple Virtual Array

## <a name="overview"></a>Información general

En este artículo se describe paso a paso cómo clonar un conjunto de copias de seguridad de los recursos compartidos o de los volúmenes de Microsoft Azure StorSimple Virtual Array. La copia de seguridad clonada se utiliza para recuperar un archivo eliminado o perdido. El artículo también incluye los pasos que deben darse para realizar una recuperación a nivel de elemento en StorSimple Virtual Array configurado como servidor de archivos.

## <a name="clone-shares-from-a-backup-set"></a>Clonación de recursos compartidos de un conjunto de copias de seguridad

**Antes de intentar clonar los recursos compartidos, asegúrese de que tiene suficiente espacio en el dispositivo para completar la operación.** Siga estos pasos en [Azure Portal](https://portal.azure.com/) para realizar una clonación a partir de una copia de seguridad.

#### <a name="to-clone-a-share"></a>Para clonar un recurso compartido

1. Vaya a la hoja **Dispositivos**. Seleccione el dispositivo, haga clic en él y, después, en **Recursos compartidos**. Seleccione el recurso compartido que desea clonar y haga clic con el botón derecho en él para invocar el menú contextual. Seleccione **Clonar**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/cloneshare1.png)
2. En la hoja **Clonar**, haga clic en **Copia de seguridad > Seleccionar** y, después, realice las siguientes acciones: 
   
   a.    Filtrar una copia de seguridad por este dispositivo basándose en el intervalo de tiempo. Puede elegir entre **Últimos 7 días**, **Últimos 30 días** y **Último año**.
   
   b.    En la lista de copias de seguridad filtradas que se muestra, seleccione la copia de seguridad de la que se va a realizar la clonación.
   
   c.    Haga clic en **OK**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/cloneshare3.png)
3. En la hoja **Clonar**, haga clic en **Configuración de destino** y realice las siguientes acciones:
   
   a.    Proporcione un nombre de recurso compartido. El nombre del recurso compartido debe contener entre 3 y 127 caracteres.
   
   b.    Opcionalmente, proporcione la descripción del recurso compartido clonado.
   
   c.    No puede cambiar el tipo del recurso compartido al que va a restaurar. Los recursos compartidos en niveles se clonan “por niveles”, mientras que los recursos compartidos anclados localmente como “anclados localmente”.
   
   d.    La capacidad se establece con el mismo tamaño que el recurso compartido del que se realiza la clonación.
   
   e.    Asigne los administradores de este recurso compartido. Las propiedades del recurso compartido se pueden modificar a través del Explorador de archivos una vez finalizada la clonación.
   
   f.    Haga clic en **OK**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/cloneshare6.png)

4. Haga clic en **Clon** para iniciar un trabajo de clonación. Una vez completado el trabajo, se inicia la operación de clonación y se le envía una notificación. Para supervisar el progreso de la clonación, vaya a la hoja **Trabajos** y haga clic en el trabajo para ver sus detalles.
5. Una vez que la clonación se haya creado correctamente, vuelva a la hoja **Recursos compartidos** del dispositivo.
6. En la lista de recursos compartidos del dispositivo podrá ver el recurso compartido recién clonado. Los recursos compartidos en capas se clonarán “en capas”, mientras que los recursos compartidos anclados localmente se clonarán como “anclados localmente”.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/cloneshare10.png)

## <a name="clone-volumes-from-a-backup-set"></a>Clonación de volúmenes a partir de un conjunto de copias de seguridad

Para realizar la clonación de una copia de seguridad, en Azure Portal, tendrá que realizar pasos similares a los que se dan al clonar un recurso compartido. La operación de clonado clona la copia de seguridad a un volumen nuevo del mismo dispositivo virtual; no es posible clonar a otro dispositivo.

#### <a name="to-clone-a-volume"></a>Para clonar un volumen

1. Vaya a la hoja **Dispositivos**. Seleccione el dispositivo, haga clic en él y, después, en **Volúmenes**. Seleccione el volumen que desee clonar y haga clic con el botón derecho en él para abrir el menú contextual. Seleccione **Clonar**.
   
   ![Clonar un volumen](./media/storsimple-virtual-array-clone/clonevolume1.png)
2. En la hoja **Clonar**, haga clic en **Copia de seguridad** y, después, realice las siguientes acciones: 
   
   a.    Filtrar una copia de seguridad por este dispositivo basándose en el intervalo de tiempo. Puede elegir entre **Últimos 7 días**, **Últimos 30 días** y **Último año**. 
   
   b.    En la lista de copias de seguridad filtradas que se muestra, seleccione la copia de seguridad de la que se va a realizar la clonación.
   
   c.    Haga clic en **OK**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/clonevolume3.png)
3. En la hoja **Clonar**, haga clic en **Configuración de volumen de destino** y realice las siguientes acciones:
   
   a. El nombre del dispositivo se rellena automáticamente.
   
   b. Especifique un nombre de volumen para el **volumen clonado**. El nombre del volumen debe contener de 3 a 127 caracteres.
   
   c. El tipo de volumen se establece automáticamente en el volumen original. Los volúmenes en capas se clonan “en capas”, mientras que los volúmenes anclados localmente se clonan como “anclados localmente”.
   
   d. En **Hosts conectados**, haga clic en **Seleccionar**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/clonevolume4.png)
4. En la hoja **Hosts conectados**, seleccione un ACR existente o agregue un ACR nuevo. Para agregar un ACR nuevo, será preciso especificar un nombre de ACR y el IQN del host. Haga clic en **Seleccionar**.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/clonevolume5.png)
5. Haga clic en **Clonar** para iniciar un trabajo de clonación.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/clonevolume6.png)  
6. Después de cree el trabajo de clonación, se iniciará la clonación. Una vez creado el clon, se muestra en la hoja Volúmenes del dispositivo. Tenga en cuenta que los volúmenes en capas se clonan “en capas”, mientras que los volúmenes anclados localmente se clonan como “anclados localmente”.
   
   ![Clonar una copia de seguridad](./media/storsimple-virtual-array-clone/clonevolume8.png)
7. Cuando el volumen se muestre online en la lista de volúmenes, estará disponible para el uso. En el host de iniciador iSCSI, actualice la lista de destinos en la ventana de propiedades del iniciador iSCSI. Un nuevo destino que contiene el nombre del volumen clonado debe aparecer como "inactivo" en la columna de estado.
8. Seleccione el destino y haga clic en **Conectar**. Cuando el iniciador se conecta al destino, el estado debe cambiar a **Conectado**.
9. En la ventana **Administración de discos**, aparecerán los volúmenes montados, tal como se muestra en la siguiente ilustración. Haga clic con el botón derecho en el volumen detectado (haga clic en el nombre del disco) y, a continuación, haga clic en **Conectado**.

> [!IMPORTANT]
> Si se produce un error en el trabajo de clonación al intentar clonar un volumen o un recurso compartido desde un conjunto de copia de seguridad, todavía se puede crear en el portal un recurso compartido o un volumen de destino. Es importante eliminar este recurso compartido o este volumen de destino del portal para minimizar los problemas que deriven de este elemento en el futuro.
> 
> 

## <a name="item-level-recovery-ilr"></a>Recuperación a nivel de elemento (ILR)

Esta versión incluye la recuperación a nivel de elemento (ILR) en StorSimple Virtual Array configurado como un servidor de archivos. Gracias a esta característica, podrá realizar una recuperación pormenorizada de los archivos y carpetas mediante una copia de seguridad en la nube de todos los recursos compartidos en el dispositivo StorSimple. Puede recuperar los archivos eliminados de las copias de seguridad recientes mediante un modelo de autoservicio.

Cada recurso compartido tiene una carpeta con formato *.backups* que contiene las copias de seguridad más recientes. Puede navegar a la copia de seguridad que desee, copiar los archivos y carpetas pertinentes de la copia de seguridad y restaurarlas. Gracias a esta característica no es necesario llamar a los administradores para restaurar archivos a través de copias de seguridad.

1. Al realizar la ILR, las copias de seguridad se pueden ver a través del Explorador de archivos. Haga clic en el recurso compartido específico que quiera ver en la copia de seguridad. Verá una carpeta con formato *.backups* creada en el recurso compartido que almacena todas las copias de seguridad. Expanda la carpeta con formato *.backups* para ver las copias de seguridad. La carpeta muestra la vista seccionada de toda la jerarquía de copias de seguridad. Normalmente, esta vista se crea a petición y en tan solo un par de segundos.
   
   Las cinco últimas copias de seguridad se muestran de esta forma y pueden utilizarse para realizar una recuperación a nivel de elemento. Las cinco últimas copias de seguridad incluyen tanto las copias de seguridad programadas predeterminadas como las manuales.
   
   * **Copias de seguridad programadas** denominadas &lt;nombreDeDispositivo&gt; DailySchedule-AAAAMMDD-HHMMSS-UTC.
   * **Copias de seguridad manuales** denominadas Ad-hoc-AAAAMMDD-HHMMSS-UTC.
     
     ![Captura de pantalla del Explorador de archivos que muestra la carpeta .backups. Dentro de esa carpeta, se selecciona la carpeta de copias de seguridad manuales.](./media/storsimple-virtual-array-clone/image14.png)

2. Identifique la copia de seguridad que contenga la versión más reciente del archivo eliminado. Aunque el nombre de la carpeta contiene una marca de tiempo UTC en cada uno de los casos anteriores, la hora en que se creó la carpeta es la hora real del dispositivo cuando comenzó la copia de seguridad. Use la marca de tiempo de la carpeta para localizar e identificar las copias de seguridad.

3. Busque la carpeta o el archivo que quiera restaurar en la copia de seguridad que identificó en el paso anterior. Tenga en cuenta que solo podrá ver los archivos o carpetas para los que tenga permiso. Si no puede acceder a determinados archivos o carpetas, póngase en contacto con un administrador de recursos compartidos. El administrador puede utilizar el Explorador de archivos para editar los permisos de los recursos compartidos y proporcionarle acceso al archivo o carpeta específicos. Es recomendable que el administrador de recursos compartidos sea un grupo de usuarios, en lugar de un usuario individual.

4. Copie el archivo o la carpeta en el recurso compartido adecuado del servidor de archivos de StorSimple.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo [administrar la matriz virtual de StorSimple mediante la interfaz de usuario web local](storsimple-ova-web-ui-admin.md).

