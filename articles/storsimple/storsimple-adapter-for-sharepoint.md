---
title: Instalación del adaptador de StorSimple para SharePoint | Microsoft Docs
description: Describe cómo instalar y configurar o quitar el adaptador de StorSimple para SharePoint en una granja de servidores de SharePoint.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 36c20b75-f2e5-4184-a6b5-9c5e618f79b2
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/06/2017
ms.author: alkohli
ms.openlocfilehash: a33e01d68a31585e216faa51ac8af193d86c6123
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90053283"
---
# <a name="install-and-configure-the-storsimple-adapter-for-sharepoint"></a>Instalar y configurar el adaptador de StorSimple para SharePoint
## <a name="overview"></a>Información general
El adaptador de StorSimple para SharePoint es un componente que le permite ofrecer almacenamiento flexible de Microsoft Azure StorSimple y protección de datos para granjas de servidores de SharePoint. Puede usar el adaptador para mover el contenido de objetos binarios grandes (BLOB) de las bases de datos de contenido de SQL Server al dispositivo híbrido de almacenamiento en la nube de Microsoft Azure StorSimple.

El adaptador de StorSimple para SharePoint funciona como un proveedor de almacenamiento remoto de blobs (RBS) y utiliza la característica de almacenamiento remoto de blobs de SQL Server para almacenar contenido de SharePoint sin estructurar (en forma de blobs) en un servidor de archivos que está respaldado por un dispositivo StorSimple.

> [!NOTE]
> El adaptador de StorSimple para SharePoint admite el almacenamiento remoto de blobs (RBS) de SharePoint Server 2010. No admite el almacenamiento externo de blobs (EBS) de SharePoint Server 2010.


* Para descargar el adaptador de StorSimple para SharePoint, vaya a [Adaptador de StorSimple para SharePoint][1] en el Centro de descarga de Microsoft.
* Para obtener información sobre el planeamiento de RBS y sus limitaciones, vaya a [Optar por usar RBS en SharePoint 2013][2] o [Planeamiento de RBS (SharePoint Server 2010)][3].

El resto de esta introducción describe brevemente el rol del adaptador de StorSimple para SharePoint y los límites de capacidad y rendimiento de SharePoint que debe tenerse en cuenta antes de instalar y configurar el adaptador. Después de revisar esta información, vaya a [Instalación del adaptador de StorSimple para SharePoint](#storsimple-adapter-for-sharepoint-installation) para comenzar a configurar el adaptador.

### <a name="storsimple-adapter-for-sharepoint-benefits"></a>Beneficios del adaptador de StorSimple para SharePoint
En un sitio de SharePoint, el contenido se almacena como datos BLOB no estructurados en una o más bases de datos de contenido. De forma predeterminada, estas bases de datos se hospedan en equipos que ejecutan SQL Server y se encuentran en la granja de servidores de SharePoint. Los BLOBs pueden aumentar rápidamente de tamaño, consumiendo grandes cantidades de almacenamiento local. Por este motivo, es posible que desee encontrar otra solución de almacenamiento más económica. SQL Server proporciona una tecnología denominada almacenamiento remoto de blobs (RBS) que le permite almacenar contenido BLOB en el sistema de archivos, fuera de la base de datos de SQL Server. Con RBS, los BLOBs pueden residir en el sistema de archivos del equipo que ejecuta SQL Server, o pueden almacenarse en el sistema de archivos de otro equipo del servidor.

RBS requiere el uso de un proveedor RBS, como el adaptador de StorSimple para SharePoint, para habilitar RBS en SharePoint. El adaptador de StorSimple para SharePoint funciona con RBS, lo que le permite mover los BLOBs a un servidor con copia de seguridad en el sistema Microsoft Azure StorSimple. Microsoft Azure StorSimple luego almacena los datos de blob de manera local o en la nube, según el uso. Los BLOBs que son muy activos (normalmente conocidos como datos de capa 1 o datos activos) residen localmente. Los datos menos activos y los datos de archivo residen en la nube. Después de habilitar RBS en una base de datos de contenido, cualquier contenido BLOB nuevo creado en SharePoint se almacena en el dispositivo StorSimple y no en la base de datos de contenido.

La implementación de RBS en Microsoft Azure StorSimple proporciona los siguientes beneficios:

* Al mover el contenido BLOB en un servidor independiente, puede reducir la carga de consultas en SQL Server, lo que puede mejorar la capacidad de respuesta de SQL Server. 
* StorSimple de Azure usa la desduplicación y la compresión para reducir el tamaño de los datos.
* StorSimple de Azure proporciona protección de datos en forma de instantáneas locales y en la nube. Además, si coloca la base de datos en el dispositivo StorSimple, puede realizar una copia de seguridad conjunta de la base de datos de contenido y los BLOBs con preparación para los bloqueos. (El traslado de la base de datos de contenido al dispositivo solo se admite para el dispositivo de la serie StorSimple 8000. Las series 5000 ó 7000 no admiten esta característica).
* StorSimple de Azure incorpora características de recuperación ante desastres, entre otras, la conmutación por error, la recuperación de archivos y volúmenes (incluida la recuperación de prueba), y la rápida restauración de los datos.
* Puede utilizar software de recuperación de datos, como Kroll Ontrack PowerControls, con las instantáneas de StorSimple de datos BLOB para realizar la recuperación a nivel de elemento de contenido de SharePoint. (Este software de recuperación de datos se compra por separado).
* El adaptador de StorSimple para SharePoint se conecta al portal de administración central de SharePoint, lo que le permite administrar toda la solución de SharePoint desde una ubicación central.

El traslado de contenido BLOB al sistema de archivos puede proporcionar otros ahorros de costos y beneficios. Por ejemplo, al utilizar RBS, puede reducirse la necesidad de costosos almacenamientos de capa 1 y, dado que reduce la base de datos de contenido, RBS puede reducir el número de bases de datos requeridas en la granja de servidores de SharePoint. Sin embargo, otros factores, como los límites de tamaño de bases de datos y la cantidad de contenido no RBS, también pueden afectar los requisitos de almacenamiento. Para obtener más información sobre los costes y las ventajas de utilizar RBS, vea [Planeamiento de RBS (SharePoint Foundation 2010)][4] y [Optar por usar RBS en SharePoint 2013][5].

### <a name="capacity-and-performance-limits"></a>Límites de capacidad y rendimiento
Antes de considerar la utilización de RBS en la solución de SharePoint, debe conocer los límites probados de rendimiento y capacidad de SharePoint Server 2010 y SharePoint Server 2013, y cómo estos límites se relacionan con el rendimiento aceptable. Para obtener más información, consulte [Restricciones y límites de software de SharePoint 2013](https://technet.microsoft.com/library/cc262787.aspx).

Revise lo siguiente antes de configurar RBS:

* Asegúrese de que el tamaño total del contenido (el tamaño de una base de datos de contenido más el tamaño de los BLOBs externalizados asociados) no supere el límite de tamaño de RBS admitido por SharePoint. Este límite es de 200 GB. 
  
    **Para medir el tamaño de la base de datos de contenido y del BLOB**
  
  1. Ejecute esta consulta en el WFE de Administración central. Inicie el Shell de administración de SharePoint y, a continuación, escriba el siguiente comando de Windows PowerShell para obtener el tamaño de las bases de datos de contenido:
     
     `Get-SPContentDatabase | Select-Object -ExpandProperty DiskSizeRequired`
     
      Este paso obtiene el tamaño de la base de datos de contenido en el disco.
  2. Ejecute una de las siguientes consultas de SQL en SQL Management Studio en el cuadro de SQL server de cada base de datos de contenido y agregue el resultado al número obtenido en el paso 1.
     
     En las bases de datos de contenido de SharePoint 2013, escriba:
     
     `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[DocStreams] WHERE [Content] IS NULL`
     
     En las bases de datos de contenido de SharePoint 2010, escriba:
     
     `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[AllDocs] WHERE [Content] IS NULL`
     
     Este paso obtiene el tamaño de los BLOB que se externalizaron.
* Recomendamos guardar todo el contenido BLOB y de bases de datos localmente en el dispositivo StorSimple. El dispositivo StorSimple es un clúster de dos nodos para alta disponibilidad. Al colocar las bases de datos de contenido y los BLOBs en el dispositivo StorSimple, se obtiene una alta disponibilidad.
  
    Utilice los procedimientos recomendados de migración tradicional de SQL Server para mover la base de datos de contenido al dispositivo StorSimple. Mueva la base de datos solo después de haber movido todo el contenido BLOB de la base de datos al recurso compartido de archivos mediante RBS. Si elige mover la base de datos de contenido al dispositivo StorSimple, recomendamos configurar el almacenamiento de la base de datos de contenido en el dispositivo como el volumen principal.
* En Microsoft Azure StorSimple, si se usan volúmenes en capas, no hay forma de garantizar que el contenido almacenado localmente en el dispositivo StorSimple no se nivele con el almacenamiento en la nube de Microsoft Azure. Por lo tanto, se recomienda usar volúmenes de StorSimple anclados localmente junto con RBS de SharePoint. Esto garantizará que todo el contenido de blob siga estando almacenado de manera local en el dispositivo StorSimple y no se transfiera a Microsoft Azure.
* Si no almacena las bases de datos de contenido en el dispositivo StorSimple, utilice los procedimientos recomendados de alta disponibilidad tradicionales de SQL Server compatibles con RBS. La agrupación en clústeres de SQL Server admite RBS, mientras que la creación de reflejos de SQL Server no es compatible. 

> [!WARNING]
> Si no tiene RBS habilitado, no es recomendable mover la base de datos de contenido al dispositivo StorSimple. Se trata de una configuración no probada.

## <a name="storsimple-adapter-for-sharepoint-installation"></a>Instalación del adaptador de StorSimple para SharePoint
Antes de instalar al adaptador de StorSimple para SharePoint, debe configurar el dispositivo StorSimple y asegurarse de que la granja de servidores de SharePoint y la creación de instancias de SQL Server cumplen todos los requisitos previos. En este tutorial se describen los requisitos de configuración, así como los procedimientos de instalación y actualización del adaptador de StorSimple para SharePoint.

## <a name="configure-prerequisites"></a>Configuración de los requisitos previos
Antes de instalar al adaptador de StorSimple para SharePoint, asegúrese de que el dispositivo StorSimple, la granja de servidores de SharePoint y la creación de instancias de SQL Server cumplen los siguientes requisitos previos.

### <a name="system-requirements"></a>Requisitos del sistema
El adaptador de StorSimple para SharePoint funciona con el siguiente hardware y software:

* Sistema operativo compatible: Windows Server 2008 R2 SP1, Windows Server 2012 o Windows Server 2012 R2.
* Versiones de SharePoint compatibles: SharePoint Server 2010 o SharePoint Server 2013
* Versiones de SQL Server compatibles: SQL Server 2008 Enterprise Edition, SQL Server 2008 R2 Enterprise Edition o SQL Server 2012 Enterprise Edition
* Dispositivos StorSimple compatibles: serie StorSimple 8000, serie StorSimple 7000 o serie StorSimple 5000.

### <a name="storsimple-device-configuration-prerequisites"></a>Requisitos previos de configuración del dispositivo StorSimple
El dispositivo StorSimple es un dispositivo de bloques y, como tal, requiere un servidor de archivos en el que se pueden hospedar los datos. Recomendamos utilizar un servidor independiente en lugar de un servidor existente de la granja de servidores de SharePoint. Este servidor de archivos debe estar en la misma red de área local (LAN) que el equipo con SQL Server que hospeda las bases de datos de contenido.

> [!TIP]
> * Si configura su granja de SharePoint para alta disponibilidad, también debe implementar el servidor de archivos para alta disponibilidad.
> * Si no almacena la base de datos de contenido en el dispositivo StorSimple, utilice los procedimientos recomendados de alta disponibilidad tradicionales compatibles con RB. La agrupación en clústeres de SQL Server admite RBS, mientras que la creación de reflejos de SQL Server no es compatible. 


Asegúrese de configurar correctamente su dispositivo StorSimple y de configurar los volúmenes apropiados para respaldar la implementación de SharePoint de modo que sean accesibles desde su equipo con SQL Server. Vaya a [Implementar el dispositivo StorSimple local](storsimple-8000-deployment-walkthrough-u2.md) si todavía no ha implementado ni configurado su dispositivo StorSimple. Anote la dirección IP del dispositivo StorSimple ya que la necesitará durante la instalación del adaptador de StorSimple para SharePoint.

Además, asegúrese de que el volumen que se utilizará para la externalización de BLOBs cumple los siguientes requisitos:

* El volumen debe formatearse con un tamaño de unidad de asignación de 64 KB.
* Los servidores front-end web (WFE) y de aplicaciones deben tener la capacidad de obtener acceso al volumen a través de la ruta de acceso UNC (convención de nomenclatura universal).
* La granja de servidores de SharePoint debe configurarse para escribir en el volumen.

> [!NOTE]
> Después de instalar y configurar el adaptador, toda la externalización de BLOBs debe pasar por el dispositivo StorSimple (el dispositivo presentará los volúmenes a SQL Server y administrará las capas de almacenamiento). No puede utilizar ningún otro destino para la externalización de BLOBs.


Si planea utilizar el Administrador de instantáneas StorSimple para tomar instantáneas de los datos de bases de datos y de BLOB, asegúrese de instalar el Administrador de instantáneas StorSimple en el servidor de bases de datos, de modo que pueda utilizar el servicio del objeto de escritura de SQL para implementar el servicio de instantáneas de volumen (VSS) de Windows.

> [!IMPORTANT]
> El Administrador de instantáneas StorSimple no admite el objeto de escritura del VSS de SharePoint y no puede tomar instantáneas coherentes con la aplicación de los datos de SharePoint. En un escenario de SharePoint, el Administrador de instantáneas StorSimple proporciona solo copias de seguridad preparadas para bloqueos.


## <a name="sharepoint-farm-configuration-prerequisites"></a>Requisitos previos de configuración de la granja de SharePoint
Asegúrese de que su granja de servidores de SharePoint está configurada correctamente, de la siguiente manera:

* Verifique que la granja de servidores de SharePoint esté en buen estado y compruebe lo siguiente:
* Todos los servidores de aplicaciones y WFE de SharePoint registrados en la granja están en funcionamiento y se les puede hacer ping desde el servidor en el que instalará el adaptador de StorSimple para SharePoint.
* El servicio de temporizador de SharePoint (SPTimerV3 o SPTimerV4) está en ejecución en cada servidor WFE y servidor de aplicaciones.
* El servicio de temporizador de SharePoint y el grupo de aplicaciones de IIS en que se ejecuta el sitio de administración central de SharePoint tienen privilegios administrativos.
* Asegúrese de que Internet Explorer Enhanced Security Context (IE ESC) está deshabilitado. Siga estos pasos para deshabilitar IE ESC:
  
  1. Cierre todas las instancias de Internet Explorer.
  2. Inicie el Administrador de servidores.
  3. En el panel izquierdo, haga clic en **Servidor local**.
  4. En el panel derecho, junto a **Configuración de seguridad mejorada de IE**, haga clic en **Activar**.
  5. En **Administradores**, haga clic en **Desactivar**.
  6. Haga clic en **OK**.

## <a name="remote-blob-storage-rbs-prerequisites"></a>Requisitos previos del almacenamiento remoto de blobs (RBS)
Asegúrese de utilizar una versión compatible de SQL Server. Solo las siguientes versiones son compatibles y pueden utilizarse con RBS:

* SQL Server 2008 Enterprise Edition
* SQL Server 2008 R2 Enterprise Edition
* SQL Server 2012 Enterprise Edition

Los BLOBS pueden externalizarse solo en los volúmenes que el dispositivo StorSimple presenta a SQL Server. No se admite ningún otro destino para la externalización de BLOBS.

Cuando se hayan completado todos los pasos de configuración de los requisitos previos, vaya a [Instalación del adaptador de StorSimple para SharePoint](#install-the-storsimple-adapter-for-sharepoint).

## <a name="install-the-storsimple-adapter-for-sharepoint"></a>Instalación del adaptador de StorSimple para SharePoint
Utilice los siguientes pasos para instalar el adaptador de StorSimple para SharePoint. Si va a reinstalar el software, consulte [Actualización o reinstalación del adaptador de StorSimple para SharePoint](#upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint). El tiempo necesario para la instalación depende del número total de bases de datos de SharePoint contenidas en la granja de servidores de SharePoint.

[!INCLUDE [storsimple-install-sharepoint-adapter](../../includes/storsimple-install-sharepoint-adapter.md)]

## <a name="configure-rbs"></a>Configuración de RBS
Después de instalar el adaptador de StorSimple para SharePoint, configure RBS según se describe en el procedimiento siguiente.

> [!TIP]
> El adaptador de StorSimple para SharePoint se conecta a la página Administración central de SharePoint, lo que permite habilitar o deshabilitar RBS en cada base de datos de contenido de la granja de SharePoint. Sin embargo, habilitar o deshabilitar RBS en la base de datos de contenido causa el restablecimiento de IIS, el cual, según la configuración de la granja, puede interrumpir momentáneamente la disponibilidad del front-end web (WFE) de SharePoint. (Factores como el uso de un equilibrador de carga front-end, la carga de trabajo actual del servidor, etc., pueden limitar o eliminar tal interrupción). Para proteger a los usuarios de las interrupciones, recomendamos habilitar o deshabilitar RBS solo durante una ventana de mantenimiento planificada.


[!INCLUDE [storsimple-sharepoint-adapter-configure-rbs](../../includes/storsimple-sharepoint-adapter-configure-rbs.md)]

## <a name="configure-garbage-collection"></a>Configuración de la recolección de elementos no utilizados
Cuando los objetos se eliminan de un sitio de SharePoint, no se eliminan automáticamente del volumen de almacenamiento de RBS. En cambio, un programa de mantenimiento asincrónico que se ejecuta en segundo plano elimina los BLOBS huérfanos del almacén de archivos. Los administradores del sistema pueden programar este proceso para que se ejecute periódicamente o pueden iniciarlo cuando lo estimen necesario.

Este programa de mantenimiento (Microsoft.Data.SqlRemoteBlobs.Maintainer.exe) se instala automáticamente en todos los servidores WFE y servidores de aplicaciones de SharePoint al habilitar RBS. El programa se instala en la siguiente ubicación: *unidad de arranque*:\Archivos de programa\Microsoft SQL Remote Blob Storage 10.50\Maintainer\

Para obtener información sobre cómo configurar y usar el programa de mantenimiento, vea [Mantener RBS en SharePoint Server 2013][8].

> [!IMPORTANT]
> El programa del mantenedor de RBS realiza un uso intensivo de recursos. Debe programarlo para que se ejecute solo durante periodos de poca actividad en la granja de SharePoint.


### <a name="delete-orphaned-blobs-immediately"></a>Elimine los BLOBS huérfanos de inmediato
Si necesita eliminar BLOBS huérfanos inmediatamente, puede utilizar las siguientes instrucciones. Tenga en cuenta que estas instrucciones son un ejemplo de cómo se puede hacer esto en un entorno de SharePoint 2013 con los siguientes componentes:

* El nombre de la base de datos de contenido es WSS_Content.
* El nombre de SQL Server es SHRPT13-SQL12\SHRPT13.
* El nombre de la aplicación web es SharePoint – 80.

[!INCLUDE [storsimple-sharepoint-adapter-garbage-collection](../../includes/storsimple-sharepoint-adapter-garbage-collection.md)]

## <a name="upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint"></a>Actualización o reinstalación del adaptador de StorSimple para SharePoint
Utilice el siguiente procedimiento para actualizar el servidor de SharePoint y luego reinstalar el adaptador StorSimple para SharePoint, o simplemente para actualizar o reinstalar el adaptador en una granja de servidores de SharePoint existente.

> [!IMPORTANT]
> Revise la siguiente información antes de intentar actualizar su software de SharePoint y/o actualizar o reinstalar el adaptador de StorSimple para SharePoint:
> 
> * Los archivos migrados previamente al almacenamiento externo mediante RBS no estarán disponibles hasta que finalice la reinstalación y se vuelva a habilitar la característica RBS. Para limitar el impacto en el usuario, realice cualquier actualización o reinstalación durante una ventana de mantenimiento programada.
> * El tiempo necesario para la actualización o reinstalación puede variar en función del número total de bases de datos de SharePoint contenidas en la granja de servidores de SharePoint.
> * Una vez completada la actualización o reinstalación, debe habilitar RBS para las bases de datos de contenido. Para obtener más información, vea [Configuración de RBS](#configure-rbs) .
> * Si va a configurar RBS para una granja de SharePoint que contiene un gran número de bases de datos (más de 200), podría agotarse el tiempo de espera de la página **Administración central de SharePoint** . Si esto sucede, actualice la página. Esto no afecta al proceso de configuración.


[!INCLUDE [storsimple-upgrade-sharepoint-adapter](../../includes/storsimple-upgrade-sharepoint-adapter.md)]

## <a name="storsimple-adapter-for-sharepoint-removal"></a>Eliminación del adaptador de StorSimple para SharePoint
En lo siguientes procedimientos se describe cómo devolver los BLOBs a las bases de datos de contenido de SQL Server y luego desinstalar el adaptador de StorSimple para SharePoint. 

> [!IMPORTANT]
> Tiene que devolver los BLOB a las bases de datos de contenido antes de desinstalar el software del adaptador.


### <a name="before-you-begin"></a>Antes de empezar
Recopile la siguiente información antes de devolver los datos a las bases de datos de contenido de SQL Server y comenzar el proceso de eliminación del adaptador:

* Los nombres de todas las bases de datos para las que RBS está habilitado
* La ruta de acceso de UNC del almacén de blobs configurado

### <a name="move-the-blobs-back-to-the-content-databases"></a>Devolver los blobs a las bases de datos de contenido
Antes de desinstalar el adaptador de StorSimple para el software de SharePoint, debe migrar todos los blobs que estaban externalizados a las bases de datos de contenido de SQL Server. Si intenta desinstalar el adaptador de StorSimple para SharePoint antes de devolver todos los blobs a las bases de datos de contenido, verá el siguiente mensaje de advertencia.

![Mensaje de advertencia](./media/storsimple-adapter-for-sharepoint/sasp1.png)

#### <a name="to-move-the-blobs-back-to-the-content-databases"></a>Para devolver los blobs a las bases de datos de contenido
1. Descargue cada objeto externalizado.
2. Abra la página **Administración central de SharePoint** y vaya a **Configuración del sistema**.
3. En **Azure StorSimple**, haga clic en **Configurar el adaptador de StorSimple**.
4. En la página **Configurar el adaptador de StorSimple**, haga clic en el botón **Deshabilitar** debajo de cada una de las bases de datos de contenido que quiere quitar del almacenamiento de blobs externo. 
5. Elimine los objetos de SharePoint y luego vuelva a cargarlos.

También puede usar el cmdlet de Microsoft `RBS Migrate()` PowerShell incluido con SharePoint. Para obtener más información, vea [migrar contenido dentro o fuera de RBS](https://technet.microsoft.com/library/ff628255.aspx).

Después de devolver los blobs a la base de datos de contenido, vaya al siguiente paso: [Desinstalar el adaptador](#uninstall-the-adapter).

### <a name="uninstall-the-adapter"></a>Desinstalar el adaptador
Cuando haya devuelto los blobs a las bases de datos de contenido de SQL Server, use una de las siguientes opciones para desinstalar el adaptador de StorSimple para SharePoint.

#### <a name="to-use-the-installation-program-to-uninstall-the-adapter"></a>Para usar el programa de instalación para desinstalar el adaptador
1. Use una cuenta con privilegios de administrador para iniciar sesión en el servidor front-end web (WFE).
2. Haga doble clic en el adaptador de StorSimple para el instalador de SharePoint. Se inicia el Asistente para la instalación.
   
    ![Asistente para la instalación](./media/storsimple-adapter-for-sharepoint/sasp2.png)
3. Haga clic en **Next**. Aparece la siguiente página.
   
    ![Página de eliminación del Asistente para la instalación](./media/storsimple-adapter-for-sharepoint/sasp3.png)
4. Haga clic en **Quitar** para seleccionar el proceso de eliminación. Aparece la siguiente página.
   
    ![Página de confirmación del Asistente para la instalación](./media/storsimple-adapter-for-sharepoint/sasp4.png)
5. Haga clic en **Quitar** para confirmar la eliminación. Aparece la siguiente página de progreso.
   
    ![Página de progreso del Asistente para la instalación](./media/storsimple-adapter-for-sharepoint/sasp5.png)
6. Cuando se completa la desinstalación, aparece la página de finalización. Haga clic en **Finalizar** para cerrar el Asistente para la instalación.

#### <a name="to-use-the-control-panel-to-uninstall-the-adapter"></a>Para usar el Panel de Control para desinstalar el adaptador
1. Abra el Panel de Control y luego haga clic en **Programas y características**.
2. Seleccione **Adaptador de StorSimple para SharePoint** y luego haga clic en **Desinstalar**.

## <a name="next-steps"></a>Pasos siguientes
[Obtenga más información sobre StorSimple](storsimple-overview.md).

<!--Reference links-->
[1]: https://www.microsoft.com/download/details.aspx?id=44073
[2]: https://technet.microsoft.com/library/ff628583(v=office.15).aspx
[3]: https://technet.microsoft.com/library/ff628583(v=office.14).aspx
[4]: https://technet.microsoft.com/library/ff628569(v=office.14).aspx
[5]: https://technet.microsoft.com/library/ff628583(v=office.15).aspx
[8]: https://technet.microsoft.com/library/ff943565.aspx
