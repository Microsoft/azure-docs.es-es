---
title: Migración de datos del dispositivo de la serie 5000-7000 de StorSimple a la serie 8000 | Microsoft Docs
description: Obtenga información sobre cómo migrar datos en el dispositivo de la serie 5000-7000 de StorSimple a la serie 8000 y cuáles son los requisitos previos para el proceso de migración.
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/23/2018
ms.author: alkohli
ms.openlocfilehash: 0fc18c6d67935889b0ba0c306dc326eca3b888f5
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184302"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-8000-series-device"></a>Migración de datos del dispositivo de la serie 5000-7000 de StorSimple a la serie 8000

> [!IMPORTANT]
> - El 31 de julio de 2019, finalizará el estado de compatibilidad de la serie StorSimple 5000/7000. Se recomienda que los clientes de la serie StorSimple 5000/7000 migren a una de las alternativas descritas en el documento.
> - La migración es actualmente una operación asistida. Si tiene la intención de migrar datos de su dispositivo StorSimple de la serie 5000-7000 a un dispositivo de la serie 8000, necesita programar la migración con Soporte técnico de Microsoft. Soporte técnico de Microsoft habilitará entonces la suscripción para la migración. Para más información, consulte cómo [abrir una incidencia de soporte técnico](storsimple-8000-contact-microsoft-support.md).
> - Después de archivar la solicitud de servicio, puede tardar un par de semanas en ejecutar el plan de migración e iniciar la migración.
> - Antes de ponerse en contacto con el servicio de Soporte técnico de Microsoft, no olvide revisar y completar los [requisitos previos de migración](#migration-prerequisites) que se indican en el artículo.

## <a name="overview"></a>Información general

En este artículo se presenta la característica de migración que permite a los clientes de la serie 5000-7000 de StorSimple migrar sus datos a un dispositivo físico de la serie 8000 de StorSimple o a un dispositivo en la nube 8010/8020. Este artículo también se vincula a un tutorial paso a paso descargable de los pasos necesarios para migrar datos desde un dispositivo antiguo de la serie 5000-7000 a un dispositivo físico o en la nube de la serie 8000.

Este artículo se aplica tanto al dispositivo local de la serie 8000 como StorSimple Cloud Appliance.


## <a name="migration-feature-versus-host-side-migration"></a>Característica de migración en comparación con la migración en el lado del host

Puede mover los datos mediante la característica de migración o bien al realizar una migración en el lado del host. En esta sección se describen los detalles de cada método incluidas las ventajas e inconvenientes. Utilice esta información para averiguar qué método desea utilizar para migrar los datos.

La característica de migración simula un proceso de recuperación ante desastres de la serie 7000/5000 a la serie 8000. Esta característica le permite migrar los datos del formato de la serie 5000/7000 al formato de la serie 8000 en Azure. El proceso de migración se inicia mediante la herramienta de migración de StorSimple. La herramienta inicia la descarga y la conversión de los metadatos de copia de seguridad en el dispositivo de la serie 8000 y después utiliza la última copia de seguridad para exponer los volúmenes en el dispositivo.

| Ventajas                                                                                                                                     |Desventajas                                                                                                                                                              |
|-------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| El proceso de migración conserva el historial de copias de seguridad que se realizaron en la serie 5000/7000.                                               | Cuando los usuarios intentan acceder a los datos, esta migración descargará los datos desde Azure, lo que ocasiona costes de descarga de datos.                                     |
| No se migra ningún dato en el lado del host.                                                                                                     | El proceso necesita un tiempo de inactividad entre el inicio de la copia de seguridad y la última copia de seguridad que aparece en la serie 8000 (se puede calcular mediante la herramienta de migración). |
| Este proceso conserva todas las directivas, plantillas de ancho de banda, cifrado y otras configuraciones en los dispositivos de la serie 8000.                      | El acceso de los usuarios solo devolverá los datos a los que accedan los usuarios y no rehidratará todo el conjunto de datos.                                                  |
| Este proceso requiere tiempo adicional para convertir todas las copias de seguridad más antiguas de Azure, lo que se hace de manera asíncrona sin afectar a la producción. | La migración solo se puede realizar a un nivel de configuración en la nube.  Los volúmenes individuales en una configuración en la nube no pueden migrarse de manera independiente.                       |

Una migración del lado del host permite configurar de forma independiente la serie 8000 y copiar los datos del dispositivo de la serie 5000/7000 al dispositivo de la serie 8000. Esto equivale a migrar datos de un dispositivo de almacenamiento a otro. Se utiliza una variedad de herramientas tales como Diskboss o robocopy para copiar los datos.

| Ventajas                                                                                                                      |Desventajas                                                                                                                                                                                                      |
|---------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| La migración puede ser abordada de manera gradual por volumen.                                               | Las copias de seguridad anteriores (realizadas en la serie 5000/7000) no estarán disponibles en el dispositivo de la serie 8000.                                                                                                       |
| Permite consolidar los datos en una única cuenta de almacenamiento en Azure.                                                       | La primera copia de seguridad en la nube en la serie 8000 tardará más tiempo, ya que hay que copiar todos los datos de la serie 8000 en Azure.                                                                     |
| Después de una migración correcta, todos los datos son locales en el dispositivo. No hay latencias al acceder a los datos. | El consumo de almacenamiento de Azure aumentará hasta que se eliminen los datos del dispositivo 5000/7000.                                                                                                        |
|                                                                                                                           | Si el dispositivo de la serie 7000/5000 tiene una gran cantidad de datos, durante la migración, estos datos deben descargarse de Azure, lo que ocasionará costos y latencias relacionados con la descarga de datos de Azure. |

Este artículo se centra únicamente en la característica de migración de dispositivos de la serie 5000/7000 a la serie 8000. Para más información sobre la migración del lado del host, consulte el documento [Migration from other storage devices](https://download.microsoft.com/download/9/4/A/94AB8165-CCC4-430B-801B-9FD40C8DA340/Migrating%20Data%20to%20StorSimple%20Volumes_09-02-15.pdf) (Migración desde otros dispositivos de almacenamiento).

## <a name="migration-prerequisites"></a>Requisitos previos de la migración

Estos son los requisitos previos de migración para su dispositivo antiguo de la serie 5000 o 7000 y el dispositivo StorSimple de la serie 8000.

> [!IMPORTANT]
> Revise y complete los requisitos previos de migración antes de presentar una solicitud de servicio a Soporte técnico de Microsoft.

### <a name="for-the-50007000-series-device-source"></a>Para el dispositivo de la serie 5000/7000 (origen)

Antes de comenzar la migración, asegúrese de que:

* Tiene su dispositivo origen de la serie 5000 o 7000; el dispositivo puede estar activo o inactivo.

    > [!IMPORTANT]
    > Le recomendamos que tenga acceso en serie a este dispositivo durante todo el proceso de migración. En caso de que haya algún problema con el dispositivo, el acceso en serie puede ayudar a resolverlo.

* El dispositivo origen de la serie 5000 o 7000 ejecuta la versión de software v2.1.1.518 o posterior. Las versiones anteriores no se admiten.
* Para comprobar la versión que la serie 5000 o 7000 ejecuta, mire en la esquina superior derecha de su interfaz de usuario web. Debería mostrar la versión de software que está ejecutando el dispositivo. Para la migración, la serie 5000 o 7000 debería estar ejecutando la versión v2.1.1.518.

    ![Comprobación de la versión de software en los dispositivos antiguos](media/storsimple-8000-migrate-from-5000-7000/check-version-legacy-device1.png)

    * Si el dispositivo activo no ejecuta la versión v2.1.1.518 o posterior, actualice el sistema a la versión mínima requerida. Para obtener instrucciones detalladas, consulte el artículo sobre [cómo actualizar el sistema a la versión v2.1.1.518](http://onlinehelp.storsimple.com/111_Appliance/6_System_Upgrade_Guides/Current_(v2.1.1)/000_Software_Patch_Upgrade_Guide_v2.1.1.518).
    * Si está ejecutando la versión v2.1.1.518, vaya a la interfaz de usuario web para ver si hay alguna notificación de errores en la restauración del registro. Si se produce un error en la restauración del registro, ejecute la restauración del registro. Para más información, vaya a la sección sobre [cómo ejecutar la restauración del registro](http://onlinehelp.storsimple.com/111_Appliance/2_User_Guides/1_Current_(v2.1.1)/1_Web_UI_User_Guide_WIP/2_Configuration/4_Cloud_Accounts/1_Cloud_Credentials#Restoring_Backup_Registry).
    * Si tiene un dispositivo inactivo que no estaba ejecutando la versión v2.1.1.518, realice una conmutación por error a un dispositivo de reemplazo que ejecute la versión v2.1.1.518. Para obtener instrucciones detalladas, consulte la recuperación ante desastres de su dispositivo StorSimple de la serie 5000/7000.
    * Realice una copia de seguridad de los datos de su dispositivo realizando una instantánea en la nube.
    * Compruebe si hay otros trabajos de copia de seguridad activos que se estén ejecutando en el dispositivo de origen. Esto incluye los trabajos en el host de la consola de Protección de datos de StorSimple. Espere a que finalicen los trabajos actuales.


### <a name="for-the-8000-series-physical-device-target"></a>Para el dispositivo físico de la serie 8000 (destino)

Antes de comenzar la migración, asegúrese de que:

* El dispositivo de la serie 8000 de destino está registrado y en funcionamiento. Para más información, consulte cómo [implementar el dispositivo StorSimple con el servicio StorSimple Manager](storsimple-8000-deployment-walkthrough-u2.md).
* El dispositivo de la serie 8000 tiene instalada la actualización 5 de la serie 8000 de StorSimple y ejecuta la versión 6.3.9600.17845 o posterior. Si el dispositivo no tiene instaladas las últimas actualizaciones, deberá instalarlas antes de poder realizar la migración. Para obtener más información, consulte cómo [instalar la última actualización en el dispositivo de la serie 8000](storsimple-8000-install-update-5.md).
* La suscripción a Azure está habilitada para la migración. Si la suscripción no está habilitada, póngase en contacto con el soporte técnico de Microsoft para habilitarla para la migración.

### <a name="for-the-80108020-cloud-appliance-target"></a>Para el dispositivo en la nube 8010/8020 (destino)

Antes de comenzar la migración, asegúrese de que:

* El dispositivo de destino en la nube está registrado y en funcionamiento. Para más información, consulte el artículo [Implementación y administración de una instancia de StorSimple Cloud Appliance](storsimple-8000-cloud-appliance-u2.md).
* El dispositivo en la nube ejecuta la actualización 5 del software de StorSimple serie 8000 versión 6.3.9600.17845. Si el dispositivo en la nube no ejecuta la actualización 5, cree un nuevo dispositivo en la nube con la actualización 5 antes de proceder con la migración. Para más información, consulte cómo [crear un dispositivo en la nube 8010/8020](storsimple-8000-cloud-appliance-u2.md).

### <a name="for-the-computer-running-storsimple-migration-tool"></a>Para el equipo que ejecuta la herramienta de migración de StorSimple

La herramienta de migración de StorSimple es una herramienta basada en interfaz de usuario que le permite migrar datos de un dispositivo StorSimple serie 5000/7000 a otro de la serie 8000. Para instalar la herramienta de migración de StorSimple, utilice un equipo que cumpla los requisitos siguientes.

El equipo tiene conectividad a Internet y:

* Se ejecuta el siguiente sistema operativo
    * Windows 10
    * Windows Server 2012 R2 (o superior) para instalar la herramienta de migración de StorSimple.
* Tiene .NET 4.5.2 instalado.
* Tiene un mínimo de 5 GB de espacio libre para instalar y usar la herramienta.

> [!TIP]
> Si su dispositivo StorSimple está conectado a un host de Windows Server, puede instalar la herramienta de migración en el equipo host de Windows Server.

#### <a name="to-install-storsimple-migration-tool"></a>Para instalar la herramienta de migración de StorSimple

Realice los pasos siguientes para instalar la herramienta de migración de StorSimple en el equipo.

1. Copie la carpeta _StorSimple8000SeriesMigrationTool_ en el equipo Windows. Asegúrese de que la unidad donde se copia el software tiene espacio suficiente.

    Abra el archivo de configuración de la herramienta _StorSimple8000SeriesMigrationTool.exe.config_ en la carpeta. Este es el fragmento de código del archivo.
    
    ```xml
        <add key="UserName" value="username@xyz.com" />
        <add key="SubscriptionName" value="YourSubscriptionName" />
        <add key="SubscriptionId" value="YourSubscriptionId" />
        <add key="TenantId" value="YourTenantId" />
        <add key="ResourceName" value="YourResourceName" />
        <add key="ResourceGroupName" value="YourResourceGroupName" />

    ```
2. Modifique los valores correspondientes a las claves y reemplácelos por:

    * `UserName`: nombre de usuario para iniciar sesión en Azure portal.
    * `SubscriptionName and SubscriptionId`: nombre e identificador de la suscripción a Azure. En la página de aterrizaje del Servicio StorSimple Device Manager, en **General**, haga clic en **Propiedades**. Copie el nombre y el identificador de la suscripción asociada a su servicio.
    * `ResourceName`: nombre del servicio StorSimple Device Manager en Azure Portal. También se muestra en Propiedades del servicio.
    * `ResourceGroup`: nombre del grupo de recursos asociado al Servicio de StorSimple Device Manager en Azure Portal. También se muestra en Propiedades del servicio.
    ![Comprobación de las propiedades del servicio para el dispositivo de destino](media/storsimple-8000-migrate-from-5000-7000/check-service-properties1.png)
    * `TenantId`: el identificador de inquilino de Azure Active Directory en Azure Portal. Inicie sesión en Microsoft Azure como administrador. En el portal de Microsoft Azure, haga clic en **Azure Active Directory**. En **Administrar**, haga clic en **Propiedades**. El identificador de inquilino se muestra en el cuadro **Id. de directorio**.
    ![Comprobación del identificador de inquilino para Azure Active Directory](media/storsimple-8000-migrate-from-5000-7000/check-tenantid-aad.png)

3.  Guarde los cambios realizados en el archivo de configuración.
4.  Ejecute el archivo _StorSimple8000SeriesMigrationTool.exe_ para iniciar la herramienta. Cuando se le pidan, proporcione las credenciales asociadas a su suscripción en Azure Portal. 
5.  Se muestra la interfaz de usuario de la herramienta de migración de StorSimple.
  

## <a name="next-steps"></a>Pasos siguientes
Descargue las instrucciones paso a paso sobre cómo [migrar datos desde un dispositivo StorSimple de la serie 5000-7000 a un dispositivo de la serie 8000](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b).
