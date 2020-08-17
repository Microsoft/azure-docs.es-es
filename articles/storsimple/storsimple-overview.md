---
title: Información general de la solución de la serie 8000 de StorSimple | Microsoft Docs
description: Describe el almacenamiento en capas de StorSimple, el dispositivo, el dispositivo virtual, los servicios y la administración de almacenamiento, y define términos clave usados en StorSimple.
services: storsimple
documentationcenter: NA
author: SharS
manager: timlt
editor: ''
ms.assetid: 7144d218-db21-4495-88fb-e3b24bbe45d1
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/10/2017
ms.author: timlt
ms.openlocfilehash: 64f91263154f7e65238acdcef9d97e5f2d09aad5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006031"
---
# <a name="storsimple-8000-series-a-hybrid-cloud-storage-solution"></a>Serie StorSimple 8000: una solución de almacenamiento en la nube híbrida

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Información general
Bienvenido a Microsoft Azure StorSimple, una solución de almacenamiento integrada que administra tareas de almacenamiento entre dispositivos locales y el almacenamiento en la nube de Microsoft Azure. StorSimple es una solución de red de área de almacenamiento (SAN) eficaz, rentable y fácilmente administrable que elimina muchos de los problemas y gastos asociados con el almacenamiento empresarial y la protección de datos. Usa un dispositivo el dispositivo StorSimple 8000 exclusivo, se integra con servicios en la nube y proporciona un conjunto de herramientas de administración para proporcionar una vista transparente de todo el almacenamiento empresarial, incluido el almacenamiento en la nube. (La información de implementación de StorSimple publicada en el sitio web de Microsoft Azure se aplica solo a los dispositivos StorSimple de la serie 8000. Si está utilizando un dispositivo StorSimple de la serie 5000 o 7000, vaya a [Ayuda de StorSimple](http://onlinehelp.storsimple.com/)).

StorSimple usa [niveles de almacenamiento](#automatic-storage-tiering) para administrar los datos almacenados en diversos medios de almacenamiento. El espacio de trabajo actual está almacenado localmente en unidades de estado sólido (SSD), los datos que se usan con menos frecuencia se almacenan en unidades de disco duro (HDD) y los datos de archivo se insertan en la nube. Además, StorSimple usa la desduplicación y la compresión para reducir la cantidad de almacenamiento que los datos consumen. Para obtener más información, vaya a [Desduplicación y compresión](#deduplication-and-compression). Para obtener definiciones de otros términos y conceptos clave utilizados en la documentación de la serie StorSimple 8000, vaya a [Terminología de StorSimple](#storsimple-terminology) al final de este artículo.

Además de la administración de almacenamiento, las características de protección de datos de StorSimple le permiten crear copias de seguridad a petición y programadas, así como almacenarlas localmente o en la nube. Las copias de seguridad se realizan en forma de instantáneas incrementales, lo que significa que se pueden crear y restaurar rápidamente. Las instantáneas en la nube pueden resultar esenciales en escenarios de recuperación ante desastres, ya que reemplazan a los sistemas de almacenamiento secundario (por ejemplo, la copia de seguridad en cinta) y le permiten restaurar los datos a su centro de datos o a sitios alternativos si es necesario.

![icono de vídeo](./media/storsimple-overview/video_icon.png) Vea el vídeo para acceder a una introducción rápida a Microsoft Azure StorSimple.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/StorSimple-Hybrid-Cloud-Storage-Solution/player]

## <a name="why-use-storsimple"></a>¿Por qué usar StorSimple?
En la tabla siguiente se describen algunas de las ventajas principales que proporciona Microsoft Azure StorSimple.

| Característica | Ventaja |
| --- | --- |
| Integración transparente |Usa el protocolo iSCSI para vincular de manera invisible instalaciones de almacenamiento de datos. Esto garantizar que los datos almacenados en la nube, en el centro de datos o en servidores remotos parezcan estar almacenados en una ubicación única. |
| Costos de almacenamiento reducidos |Asigna almacenamiento local o en la nube suficiente para cumplir las actuales exigencias y extiende el almacenamiento en la nube solo cuando es necesario. Disminuye aún más los requisitos de almacenamiento y los gastos al eliminar las versiones redundantes de los mismos datos (desduplicación) y mediante la compresión. |
| Administración simplificada del almacenamiento |Proporciona herramientas de administración del sistema para configurar y administrar datos almacenados localmente, en un servidor remoto y en la nube. Además, puede administrar copias de seguridad y restaurar funciones desde un complemento de Microsoft Management Console (MMC).|
| Mejor recuperación ante desastres y cumplimiento normativo |No requiere tiempo de recuperación extendido. En lugar de eso, restaura los datos según sea necesario. Esto significa que las operaciones normales pueden seguir con un mínimo de interrupción. Además, puede configurar directivas para especificar programaciones de copias de seguridad y retención de datos. |
| Movilidad de datos |Es posible tener acceso a los datos cargados a los servicios en la nube de Microsoft Azure desde otros sitios para fines de recuperación y migración. Además, puede usar StorSimple para configurar StorSimple Cloud Appliances en máquinas virtuales (VM) que se ejecutan en Microsoft Azure. Luego, las máquinas virtuales pueden utilizar dispositivos virtuales para obtener acceso a los datos almacenados para fines de prueba o recuperación. |
| Continuidad empresarial |Permite a los usuarios de la serie 5000-7000 de StorSimple migrar sus datos a un dispositivo StorSimple de la serie 8000. |
| Disponibilidad en el Portal de administración de Azure |StorSimple ya está disponible en el Portal de Azure Government. Para obtener más información, consulte [Implementación del dispositivo StorSimple local en el Portal de Government](storsimple-8000-deployment-walkthrough-gov-u2.md). |
| Disponibilidad y protección de datos |La serie 8000 de StorSimple admite el almacenamiento con redundancia de zona (ZRS), además del almacenamiento con redundancia local (LRS) y del almacenamiento con redundancia geográfica (GRS). Para obtener información detallada sobre ZRS, consulte [este artículo sobre las opciones de redundancia de Azure Storage](https://azure.microsoft.com/documentation/articles/storage-redundancy/) . |
| Compatibilidad para las aplicaciones críticas |StorSimple le permite identificar los volúmenes apropiados como anclados localmente, lo que a su vez garantiza que los datos que requieren las aplicaciones críticas no están organizados en capas en la nube. Los volúmenes anclados localmente no están sujetos a las latencias de la nube ni a los problemas de conectividad. Para más información sobre los volúmenes anclados localmente, consulte [Usar el servicio de Administrador de StorSimple para administrar volúmenes (Update 2)](storsimple-8000-manage-volumes-u2.md). |
| Baja latencia y alto rendimiento |Puede crear aplicaciones en la nube que se beneficien de las características de baja latencia y alto rendimiento del almacenamiento premium de Azure. Para más información sobre aplicaciones en la nube premium StorSimple, consulte [Implementar y administrar un dispositivo virtual StorSimple en Azure](storsimple-8000-cloud-appliance-u2.md). |


## <a name="storsimple-components"></a>Componentes StorSimple
La solución de Microsoft Azure StorSimple incluye los siguientes componentes:

* **Dispositivo de Microsoft Azure StorSimple** : una matriz de almacenamiento híbrida local que contiene SSD y HDD, en conjunto con controladores redundantes y funcionalidades de conmutación automática por error. Los controladores administran la organización en niveles del almacenamiento y ponen los datos actualmente utilizados (o activos) en el almacenamiento local (en el dispositivo o en servidores locales), mientras que mueven los datos usados con menos frecuencia a la nube.
* **StorSimple Cloud Appliance**: también conocido como Aplicación virtual StorSimple, se trata de una versión de software del dispositivo StorSimple que replica la arquitectura y la mayoría de las funcionalidades del dispositivo de almacenamiento híbrido físico. StorSimple Cloud Appliance se ejecuta en un solo nodo en una máquina virtual de Azure. Los dispositivos virtuales premium, que sacan partido del almacenamiento premium de Azure, están disponibles en Update 2 y posterior.
* **Servicio Administrador de dispositivos de StorSimple**: es una extensión de Azure Portal que le permite administrar un dispositivo StorSimple o StorSimple Cloud Appliance desde una única interfaz web. Puede utilizar el servicio Administrador de dispositivos de StorSimple para crear y administrar servicios, ver y administrar dispositivos, ver alertas, administrar volúmenes y ver y administrar las directivas de copia de seguridad y el catálogo de copias de seguridad.
* **Windows PowerShell para StorSimple** : una interfaz de línea de comandos que puede usar para administrar el dispositivo de StorSimple. Windows PowerShell para StorSimple tiene características que le permite registrar su dispositivo de StorSimple, configurar la interfaz de red en su dispositivo, instalar ciertos tipos de actualizaciones, solucionar problemas con su dispositivo mediante el acceso a la sesión de soporte y cambiar el estado del dispositivo. Puede tener acceso a Windows PowerShell para StorSimple si se conecta a la consola serial o si usa la comunicación remota de Windows PowerShell.
* **Cmdlets de Azure PowerShell StorSimple** : un conjunto de cmdlets de Windows PowerShell que permiten automatizar las tareas de nivel de servicio y de migración desde la línea de comandos. Para obtener más información sobre los cmdlets de Azure PowerShell, consulte la [referencia de los cmdlets](/powershell/module/servicemanagement/azure.service/?view=azuresmps-3.7.0#azure).
* **Snapshot Manager de StorSimple** : un complemento de MMC que usa grupos de volúmenes y el Servicio de instantáneas de volumen de Windows para generar copias de seguridad coherentes con la aplicación. Además, puede usar StorSimple Snapshot Manager para crear programaciones de copias de seguridad y clonar o restaurar volúmenes.
* **Adaptador de StorSimple para SharePoint** : es una herramienta que extiende de manera transparente el almacenamiento y la protección de datos de Microsoft Azure StorSimple a las granjas de SharePoint Server, a la vez que hace que sea posible ver y administrar el almacenamiento de StorSimple desde el Portal de administración central de SharePoint.

El siguiente diagrama proporciona una vista de alto nivel de la arquitectura y componentes de Microsoft Azure StorSimple.

![Arquitectura de StorSimple](./media/storsimple-overview/overview-big-picture.png)

Las secciones siguientes describen los componentes con más detalle y explican la manera en que la solución ordena los datos, asigna el almacenamiento y facilita la administración del almacenamiento y la protección de los datos. La última sección proporciona definiciones de algunos términos y conceptos importantes relacionados con los componentes de StorSimple y su administración.

## <a name="storsimple-device"></a>Dispositivo de StorSimple
El dispositivo de Microsoft Azure StorSimple es una matriz de almacenamiento híbrida local que proporciona almacenamiento principal y acceso iSCSI a los datos almacenados en él. Administra la comunicación con el almacenamiento en la nube y ayuda a garantizar la seguridad y confidencialidad de todos los datos que están almacenados en la solución de Microsoft Azure StorSimple.

El dispositivo de StorSimple incluye SSD y unidades de disco duro (HDD), además de compatibilidad para agrupación en clústeres y la conmutación automática por error. Contiene un procesador compartido, almacenamiento compartido y dos controladores reflejados. Cada controlador proporciona lo siguiente:

* Conexión a un equipo host
* Hasta seis puertos de red para conectar a la red de área local (LAN)
* Supervisión de hardware
* Memoria de acceso aleatorio no volátil (NVRAM), que conserva la información incluso cuando se interrumpe la energía
* Actualización compatible con clústeres para administrar actualizaciones de software en servidores en un clúster de conmutación por error para que las actualizaciones tengan un efecto mínimo o nulo en la disponibilidad de servicio
* Servicio de clúster, que funciona como un clúster back-end, proporciona alta disponibilidad y minimiza todos los efectos adversos que podrían producirse si un HDD o una SSD presentan errores o quedan sin conexión

En cualquier punto del tiempo solo hay un controlador activo. Si se produce un error en el controlador activo, el segundo controlador se activa de manera automática.

Para obtener más información, consulte [Componentes y estado de hardware de StorSimple](storsimple-8000-monitor-hardware-status.md).

## <a name="storsimple-cloud-appliance"></a>StorSimple Cloud Appliance
Puede usar StorSimple para crear una aplicación en la nube que replique la arquitectura y las funcionalidades del dispositivo de almacenamiento híbrido físico. StorSimple Cloud Appliance (también conocido como Aplicación virtual StorSimple) se ejecuta en un solo nodo en una máquina virtual de Azure. (Una aplicación virtual solo se puede crear en una máquina virtual de Azure. No puede crear uno en un dispositivo de StorSimple o en un servidor local).

La aplicación en la nube tiene las siguientes características:

* Se comporta como un dispositivo físico y puede ofrecer una interfaz iSCSI a las máquinas virtuales en la nube.
* Puede crear un número ilimitado de aplicaciones en la nube y activarlas y desactivarlas según sea necesario.
* Puede ayudar a simular entornos locales en escenarios de recuperación ante desastres, desarrollo y pruebas, además de facilitar la recuperación de nivel de elemento a partir de copias de seguridad.

StorSimple Cloud Appliance está disponible en dos modelos: el dispositivo 8010 (anteriormente conocido como el modelo 1100) y el 8020. El dispositivo 8010 tiene una capacidad máxima de 30 TB. El dispositivo 8020, que aprovecha las ventajas de almacenamiento premium de Azure, tiene una capacidad máxima de 64 TB. (En niveles locales, el almacenamiento premium de Azure almacena datos en SSD, mientras que el estándar los almacena en HDD). Tenga en cuenta que debe tener una cuenta de almacenamiento premium de Azure para usar el almacenamiento premium.

Para más información sobre StorSimple Cloud Appliance, consulte [Deploy and manage a StorSimple Cloud Appliance in Azure](storsimple-8000-cloud-appliance-u2.md) (Implementar y administrar StorSimple Cloud Appliance en Azure).

## <a name="storsimple-device-manager-service"></a>Servicio de administrador de dispositivos de StorSimple
Microsoft Azure StorSimple proporciona una interfaz de usuario basada en Web (el servicio Administrador de dispositivos de StorSimple) que le permite administrar de manera central el centro de datos y el almacenamiento en la nube. Puede usar el servicio de administrador de dispositivos de StorSimple para realizar las siguientes tareas:

* Ajustar la configuración del sistema para dispositivos de StorSimple.
* Configurar y administrar ajustes de seguridad para dispositivos de StorSimple.
* Configurar credenciales y propiedades de la nube.
* Configurar y administrar volúmenes en un servidor.
* Configurar grupos de volúmenes.
* Crear copias de seguridad y restaurar datos.
* Supervisar el rendimiento.
* Revisar la configuración del sistema e identificar posibles problemas.

Puede utilizar el servicio Administrador de dispositivos de StorSimple para realizar todas las tareas de administración, excepto las que requieren tiempo de inactividad del sistema, como la configuración inicial y la instalación de actualizaciones.

Para obtener más información, consulte [Uso del servicio de administrador de dispositivos de StorSimple para administrar su dispositivo StorSimple](storsimple-8000-manager-service-administration.md).

## <a name="windows-powershell-for-storsimple"></a>Windows PowerShell para StorSimple
Windows PowerShell para StorSimple proporciona una interfaz de línea de comandos que puede utilizar para crear y administrar el servicio de Microsoft Azure StorSimple y configurar y supervisar los dispositivos de StorSimple. Se trata de una interfaz de la línea de comandos basada en Windows PowerShell que incluye cmdlets dedicados para administrar su dispositivo de StorSimple. Windows PowerShell para StorSimple tiene características que le permiten:

* Registrar un dispositivo.
* Configurar la interfaz de red en un dispositivo.
* Instalar ciertos tipos de actualizaciones.
* Solucionar problemas del dispositivo mediante el acceso a la sesión de soporte.
* Cambiar el estado del dispositivo.

Puede tener acceso a Windows PowerShell para StorSimple desde una consola serial (en un equipo host conectado directamente al dispositivo) o de manera remota mediante el uso de la comunicación remota de Windows PowerShell. Observe que algunas tareas de Windows PowerShell para StorSimple, como el registro inicial del dispositivo, solo se pueden realizar en la consola serial.

Para obtener más información, consulte [Uso de Windows PowerShell para StorSimple para administrar su dispositivo](storsimple-8000-windows-powershell-administration.md).

## <a name="azure-powershell-storsimple-cmdlets"></a>Cmdlets de Azure PowerShell StorSimple
Los cmdlets de StorSimple de Azure PowerShell son un conjunto de cmdlets de Windows PowerShell que permiten automatizar las tareas migración y de nivel de servicio desde la línea de comandos. Para obtener más información sobre los cmdlets de Azure PowerShell, consulte la [referencia de los cmdlets](/powershell/module/servicemanagement/azure.service/?view=azuresmps-3.7.0).

## <a name="storsimple-snapshot-manager"></a>Snapshot Manager de StorSimple
StorSimple Snapshot Manager es un complemento de Microsoft Management Console (MMC) que se puede usar para crear copias de seguridad puntuales y coherentes de datos locales y en la nube. El complemento se ejecuta en un host basado en Windows Server. Puede utilizar StorSimple Snapshot Manager para:

* Configurar, crear copias de seguridad y eliminar volúmenes.
* Configurar grupos de volúmenes para asegurarse de que los datos con copia de seguridad son coherentes con la aplicación.
* Administrar directivas de copia de seguridad para que se creen copias de seguridad de los datos según una programación predeterminada y se almacenen en una ubicación designada (localmente o en la nube).
* Restaurar volúmenes y archivos individuales.

Las copias de seguridad se capturan como instantáneas, las que solo registran los cambios desde que se capturó la última instantánea y requieren mucho menos espacio de almacenamiento que las copias de seguridad completas. Puede crear programaciones de copia de seguridad o crear copias de seguridad inmediatas, según sea necesario. De manera adicional, puede usar StorSimple Snapshot Manager para establecer directivas de retención que controlan la cantidad de instantáneas que se guardarán. Si más adelante necesita restaurar datos desde una copia de seguridad, StorSimple Snapshot Manager le permite seleccionar desde el catálogo de instantáneas locales o en la nube. 

Si se produce un desastre o si debe restaurar datos por cualquier otro motivo, StorSimple Snapshot Manager los restaura de manera incremental según sea necesario. La restauración de los datos no requiere que apague todo el sistema mientras restaura un archivo, reemplaza equipo o mueve operaciones a otro sitio.

Para obtener más información, vaya a [Qué es el Administrador de instantáneas de StorSimple](storsimple-what-is-snapshot-manager.md)

## <a name="storsimple-adapter-for-sharepoint"></a>Adaptador de StorSimple para SharePoint
Microsoft Azure StorSimple incluye el adaptador de StorSimple para SharePoint, un componente opcional que extiende de manera transparente el almacenamiento de StorSimple y las características de protección de datos a las granjas de servidores de SharePoint. El adaptador funciona con un proveedor de Almacenamiento remoto de blobs (RBS) y la característica RBS de SQL Server, lo que le permite mover blobs a un servidor con copia de seguridad realizada por el sistema Microsoft Azure StorSimple. Microsoft Azure StorSimple luego almacena los datos de blob de manera local o en la nube, según el uso.

El adaptador de StorSimple para SharePoint se administra desde el Portal de administración central de SharePoint. Por consiguiente, la administración de SharePoint sigue siendo centralizada y todo el almacenamiento pareciera encontrarse en la granja de SharePoint.

Para obtener más información, consulte [Adaptador de StorSimple para SharePoint](storsimple-adapter-for-sharepoint.md). 

## <a name="storage-management-technologies"></a>Tecnologías de administración de almacenamiento
Además del dispositivo dedicado de StorSimple, el dispositivo virtual y otros componentes, Microsoft Azure StorSimple usa las siguientes tecnologías de software para proporcionar acceso rápido a los datos y disminuir el consumo de almacenamiento:

* [Organización automática del almacenamiento en niveles](#automatic-storage-tiering) 
* [Aprovisionamiento fino](#thin-provisioning) 
* [Desduplicación y compresión](#deduplication-and-compression) 

### <a name="automatic-storage-tiering"></a>Organización automática del almacenamiento en niveles
Microsoft Azure StorSimple ordena automáticamente los datos en niveles lógicos según su uso actual, su antigüedad y la relación con otros datos. Los datos más activos se almacenan de manera local, mientras que los datos menos activos y los datos inactivos se migran automáticamente a la nube. El siguiente diagrama ilustra este enfoque de almacenamiento.

![Niveles de almacenamiento de StorSimple](./media/storsimple-overview/hcs-data-services-storsimple-components-tiers.png)

Para habilitar el acceso rápido, StorSimple almacena datos muy activos (datos activos) en SSD en el dispositivo StorSimple. Almacena datos que se usan de manera ocasional (datos semiactivos) en discos duros en el dispositivo o en los servidores del centro de datos. Mueve a la nube datos inactivos, datos de copia de seguridad y datos retenidos para fines de archivo o de cumplimiento. 

> [!NOTE]
> En Update 2 o posterior, puede especificar un volumen anclado localmente, en cuyo caso los datos permanecen en el dispositivo local y no en capas en la nube. 


StorSimple ajusta y reordena las asignaciones de datos y almacenamiento a medida que cambian los patrones de uso. Por ejemplo, cierta información puede volverse menos activo con el tiempo. A medida que se vuelve menos activa, migra de SSD a HDD y, a continuación, a la nube. Si esos mismos datos vuelven a activarse, se migran de vuelta al dispositivo de almacenamiento.

El proceso de organización en niveles del almacenamiento se produce como sigue:

1. Un administrador del sistema configura una cuenta de almacenamiento en la nube de Microsoft Azure.
2. El administrador usa la consola en serie y el servicio Administrador de dispositivos de StorSimple (que se ejecuta en Azure Portal) para configurar el servidor del dispositivo y de los archivos, y así crear directivas de protección de datos y volúmenes. Las máquinas locales (como servidores de archivo) usan Interfaz estándar de equipos pequeños de Internet (iSCSI) para acceder al dispositivo StorSimple.
3. Inicialmente, StorSimple almacena los datos en el nivel de SSD rápido del dispositivo.
4. A medida que el nivel de SSD empieza a carecer de capacidad, StorSimple desduplica y comprime los bloques de datos más antiguos y los mueve al nivel de unidades de disco duro.
5. Si el nivel de unidades de disco duro comienza a carecer de capacidad, StorSimple cifra los bloques de datos más antiguos y los envía de forma segura a la cuenta de almacenamiento de Microsoft Azure a través de HTTPS.
6. Microsoft Azure crea varias réplicas de los datos en su centro de datos y en un centro de datos remoto, lo que garantiza que los datos puedan recuperarse si se produce un desastre.
7. Cuando el servidor de archivos solicita datos almacenados en la nube, StorSimple los devuelve sin problemas y almacena una copia en el nivel de SSD del dispositivo StorSimple.

#### <a name="how-storsimple-manages-cloud-data"></a>Cómo administra StorSimple los datos en la nube

StorSimple desduplica los datos del cliente a través de todas las instantáneas y los datos principales (datos escritos por los hosts). Aunque la desduplicación es excelente para la eficacia de almacenamiento, convierte la pregunta "¿Qué hay en la nube?" en complicada. Los datos de nivel principal y los datos de instantánea se superponen entre sí. Un único fragmento de datos en la nube podría ser usado como datos de nivel principal y también varias instantáneas pueden hacer referencia a él. Cada instantánea en la nube garantiza que una copia de todos los datos en un momento del tiempo está bloqueada en la nube hasta que se elimine esa instantánea.

Los datos solo se eliminan de la nube cuando no hay referencias a esos datos. Por ejemplo, si se toma una instantánea en la nube de todos los datos que están en el dispositivo de StorSimple y, a continuación, se eliminan algunos datos principales, se verían los _datos principales_ eliminados inmediatamente. Los _datos en la nube_ que incluyen los datos en niveles y las copias de seguridad, permanecen igual. Esto ocurre porque aún hay una instantánea haciendo referencias a los datos en la nube. Después de eliminar la instantánea en la nube (y cualquier otra instantánea que haga referencia a los mismos datos), se elimina el consumo en la nube. Antes de eliminar datos en la nube, comprobamos que ninguna instantánea hace referencia a esos datos. Este proceso se denomina _recolección de elementos no utilizados_ y es un servicio en segundo plano que se ejecuta en el dispositivo. La eliminación de datos en la nube no es inmediata ya que el servicio de recolección de elementos no utilizados comprueba cualquier otra referencia a esos datos antes de la eliminación. La velocidad de la recolección de elementos no utilizados depende del número total de instantáneas y los datos totales. Normalmente, los datos en la nube se limpian en menos de una semana.


### <a name="thin-provisioning"></a>Aprovisionamiento fino
El aprovisionamiento fino es una tecnología de virtualización en que el almacenamiento disponible parece superar los recursos físicos. En lugar de reservar almacenamiento suficiente por adelantado, StorSimple utiliza el aprovisionamiento fino para asignar solo el espacio suficiente para cumplir con los requisitos actuales. La naturaleza elástica del almacenamiento en la nube facilita este enfoque porque StorSimple puede aumentar o disminuir el almacenamiento en la nube para satisfacer con las exigencias cambiantes.

> [!NOTE]
> El aprovisionamiento de los volúmenes anclados localmente no puede ser fino. El almacenamiento asignado a un volumen sólo local se aprovisiona totalmente cuando se crea el volumen.


### <a name="deduplication-and-compression"></a>Desduplicación y compresión
Microsoft Azure StorSimple utiliza la desduplicación y la compresión de datos para reducir aún más los requisitos de almacenamiento.

La desduplicación disminuye la cantidad general de datos almacenados al eliminar la redundancia en el conjunto de datos almacenados. A medida que cambia la información, StorSimple omite los datos no modificados y solo captura los cambios. Además, StorSimple reduce la cantidad de datos almacenados al identificar y eliminar información innecesaria. 

> [!NOTE]
> Los datos de los volúmenes anclados localmente no se desduplican ni comprimen. Sin embargo, las copias de seguridad de volúmenes localmente anclados sí se desduplican y comprimen.


## <a name="storsimple-workload-summary"></a>Resumen de la carga de trabajo de StorSimple
A continuación, presentamos un resumen de las cargas de trabajo de StorSimple.

| Escenario | Carga de trabajo | Compatible | Restricciones | Versión |
| --- | --- | --- | --- | --- |
| Colaboración |Uso compartido de archivos |Sí | |Todas las versiones |
| Colaboración |Uso compartido de archivos distribuidos |Sí | |Todas las versiones |
| Colaboración |SharePoint |Sí* |Solo compatible con volúmenes localmente anclados |Actualización 2 y posterior |
| Archivado |Archivado de archivos sencillo |Sí | |Todas las versiones |
| Virtualización |Máquinas virtuales |Sí* |Solo compatible con volúmenes localmente anclados |Actualización 2 y posterior |
| Base de datos |SQL |Sí* |Solo compatible con volúmenes localmente anclados |Actualización 2 y posterior |
| Vigilancia de vídeo |Vigilancia de vídeo |Sí* |Compatible cuando el dispositivo StorSimple se emplea en exclusiva para esta carga de trabajo |Actualización 2 y posterior |
| Copia de seguridad |Copia de seguridad de destino principal |Sí* |Compatible cuando el dispositivo StorSimple se emplea en exclusiva para esta carga de trabajo |Actualización 3 y posteriores |
| Copia de seguridad |Copia de seguridad de destino secundario |Sí* |Compatible cuando el dispositivo StorSimple se emplea en exclusiva para esta carga de trabajo |Actualización 3 y posteriores |

*Sí&#42;. Se deben aplicar las restricciones y directrices de la solución.*

Las cargas de trabajo siguientes no son compatibles con dispositivos de la serie 8000 de StorSimple. Si se implementa en StorSimple, estas cargas de trabajo generarán una configuración no admitida.

* Imágenes médicas
* Exchange
* VDI
* Oracle
* SAP
* Macrodatos
* Distribución de contenido
* Arranque desde SCSI

A continuación, se muestra una lista de los componentes de infraestructura compatibles con StorSimple.

| Escenario | Carga de trabajo | Compatible | Restricciones | Versión |
| --- | --- | --- | --- | --- |
| General |ExpressRoute |Sí | |Todas las versiones |
| General |DataCore FC |Sí* |Compatible con DataCore SANsymphony |Todas las versiones |
| General |DFSR |Sí* |Solo compatible con volúmenes localmente anclados |Todas las versiones |
| General |Indización |Sí* |Para volúmenes en capas, solo se admite la indización de metadatos (sin datos).<br>Para volúmenes anclados localmente, se admite la indización completa. |Todas las versiones |
| General |Antivirus |Sí* |Para volúmenes en capas, se admite solo el examen al abrir y cerrar.<br> Para volúmenes anclados localmente, se admite el examen completo. |Todas las versiones |

*Sí&#42;. Se deben aplicar las restricciones y directrices de la solución.*

A continuación se muestra una lista de otro software que se usa con StorSimple para crear soluciones.

| Tipo de carga de trabajo | Software usado con StorSimple | Versiones compatibles|Vincular a la guía de soluciones| 
| --- | --- | --- | --- |
| Destino de copia de seguridad |Veeam |Veeam versión 9 y versiones posterior |[StorSimple como destino de copia de seguridad con Veeam](storsimple-configure-backup-target-veeam.md)|
| Destino de copia de seguridad |Veritas Backup Exec |Backup Exec 16 y versiones posteriores |[StorSimple como destino de copia de seguridad con Backup Exec](storsimple-configure-backup-target-using-backup-exec.md)|
| Destino de copia de seguridad |Veritas NetBackup |NetBackup 7.7.x y versiones posteriores  |[StorSimple como destino de copia de seguridad con NetBackup](storsimple-configure-backuptarget-netbackup.md)|
| Uso compartido de archivos global <br></br> Colaboración |Talon  |[StorSimple con Talon](https://www.talonstorage.com/products/archive/fast-deployment-azure-storsimple) | |

## <a name="storsimple-terminology"></a>Terminología de StorSimple
Antes de implementar la solución Microsoft Azure StorSimple, se recomienda que revise los siguientes términos y definiciones.

### <a name="key-terms-and-definitions"></a>Términos clave y definiciones
| Término (acrónimo o abreviatura) | Descripción |
| --- | --- |
| registros de control de acceso (ACR) |Un registro asociado a un volumen en el dispositivo de Microsoft Azure StorSimple que determina qué hosts pueden conectarse a él. La determinación está basada en el Nombre calificado iSCSI (IQN) de los hosts (contenidos en el ACR) que se conectan al dispositivo StorSimple. |
| AES-256 |Un algoritmo Estándar de cifrado avanzado (AES) de 256 bits para cifrar los datos cuando se desplaza hacia y desde la nube. |
| tamaño de unidad de asignación (AUS) |La menor cantidad de espacio en disco que se puede asignar para contener un archivo en los sistemas de archivos de Windows. Si un tamaño de archivo no es un múltiplo par del tamaño del clúster, debe utilizarse el espacio adicional para almacenar el archivo (hasta el siguiente múltiplo del tamaño del clúster), lo que genera una pérdida de espacio y la fragmentación del disco duro. <br>El AUS recomendado para volúmenes StorSimple de Azure es de 64 KB porque funciona bien con los algoritmos de desduplicación. |
| organización automática del almacenamiento en niveles |Mover automáticamente los datos menos activos de SSD a HDD a un nivel en la nube y, a continuación, habilitar la administración de todo el almacenamiento de una interfaz de usuario central. |
| catálogo de copias de seguridad |Un conjunto de copias de seguridad, normalmente relacionadas con el tipo de aplicación que se utilizó. Esta colección se muestra en la hoja Catálogo de copias de seguridad de la interfaz de usuario del servicio Administrador de dispositivos de StorSimple. |
| archivo del catálogo de copias de seguridad |Un archivo que contiene una lista de instantáneas disponibles almacenadas actualmente en la base de datos de copias de seguridad de StorSimple Snapshot Manager. |
| directiva de copia de seguridad |Una selección de volúmenes, tipo de copia de seguridad y un calendario que le permite crear copias de seguridad en un programa definido. |
| objetos grandes binarios (BLOB) |Una colección de datos binarios almacenados como una sola entidad en un sistema de administración de base de datos. Los BLOB suelen ser imágenes, audio u otros objetos multimedia, aunque a veces el código binario ejecutable se almacena como un BLOB. |
| Protocolo de autenticación por desafío mutuo (CHAP) |Un protocolo utilizado para autenticar al mismo nivel de una conexión, basada en el mismo nivel comparten una contraseña o secreto. CHAP puede ser unidireccional o mutuo. Con el CHAP unidireccional el destino autentica un iniciador. El CHAP mutuo requiere que el destino autentique el iniciador y, a continuación, que el iniciador autentique al destino. |
| clon |Una copia duplicada de un volumen. |
| Nube como nivel (CaaT) |Almacenamiento integrado en la nube como un nivel en la arquitectura de almacenamiento de modo que todo el almacenamiento parece ser parte de una red de almacenamiento empresarial. |
| proveedor de servicios de nube (CSP) |Un proveedor de servicios de informática en la nube. |
| instantánea de la nube |Una copia de punto en el tiempo de los datos del volumen que se almacenan en la nube. Una instantánea de nube es equivalente a una instantánea replicada en un sistema de almacenamiento externo. Las instantáneas en la nube son especialmente útiles en escenarios de recuperación ante desastres. |
| clave de cifrado de almacenamiento en la nube |Una contraseña o clave utilizada por su dispositivo StorSimple para tener acceso a los datos cifrados enviados por el dispositivo a la nube. |
| actualización de clústeres |Administrar actualizaciones de software en servidores en un clúster de conmutación por error para que las actualizaciones tengan un efecto mínimo o nulo en la disponibilidad de servicio. |
| ruta de datos |Una colección de unidades funcionales que realizan operaciones interconectadas de procesamiento de datos. |
| desactivar |Una acción permanente que interrumpe la conexión entre el dispositivo StorSimple y el servicio de nube asociada. Las instantáneas en la nube del dispositivo permanecen después de este proceso y se pueden clonar o usar para la recuperación de desastres. |
| creación de reflejo de disco |Replicación de volúmenes de disco lógicos en distintas unidades en tiempo real para garantizar la disponibilidad continua. |
| creación de reflejo de disco dinámico |Replicación de volúmenes de discos lógicos en discos dinámicos. |
| discos dinámicos |Un formato de volumen de disco que utiliza el Administrador de discos lógicos (LDM) para almacenar y administrar datos en varios discos físicos. Los discos dinámicos se pueden ampliar para proporcionar más espacio libre. |
| Gabinete Extended Bunch of Disks (EBOD) |Un gabinete secundario del dispositivo StorSimple de Microsoft Azure que contiene discos duros adicionales para almacenamiento adicional. |
| aprovisionamiento grueso |Un aprovisionamiento de almacenamiento convencional mediante el que se asigna espacio de almacenamiento en base a necesidades previstas (y por lo general más allá de la necesidad actual). Consulte también *Aprovisionamiento fino*. |
| unidad de disco duro (HDD) |Una unidad que usa placas giratorias para almacenar datos. |
| almacenamiento de nube híbrida |Una arquitectura de almacenamiento que usa los recursos locales y remotos, incluidos el almacenamiento en la nube. |
| Internet Small Computer System Interface (iSCSI) |Un estándar de red de almacenamiento basado en el Protocolo de Internet (IP) para vincular el equipo de almacenamiento de datos o funciones. |
| iniciador iSCSI |Un componente de software que permite a un equipo host que ejecuta Windows conectarse a una red de almacenamiento externo basada en iSCSI. |
| Nombre calificado iSCSI (IQN) |Un nombre único que identifica un destino o iniciador iSCSI. |
| destino iSCSI |Un componente de software que proporciona subsistemas de disco iSCCSI centralizados en redes de área de almacenamiento. |
| archivo de datos |Un enfoque de almacenamiento en el que los datos de archivo están accesibles todo el tiempo (no se almacena fuera del sitio en la cinta, por ejemplo). Microsoft Azure StorSimple usa el archivo vivo. |
| Volumen anclado localmente |un volumen que reside en el dispositivo y nunca se almacena en capas en la nube. |
| instantánea local |Una copia de punto en el tiempo de los datos del volumen que se almacenan en el dispositivo StorSimple de Microsoft Azure. |
| Microsoft Azure StorSimple |Una solución eficaz que consta de un dispositivo de almacenamiento del centro de datos y el software que permite a las organizaciones de TI aprovechar el almacenamiento en la nube como si fuese un almacenamiento de centro de datos. StorSimple simplifica la administración y protección de datos al mismo tiempo que reduce los costos. La solución consolida el almacenamiento principal, archivo, copia de seguridad y recuperación ante desastres (DR) a través de la integración sin problemas con la nube. Mediante la combinación de administración de datos de almacenamiento SAN y de nube en una plataforma empresarial, los dispositivos StorSimple habilitan la velocidad, la sencillez y confiabilidad para todas las necesidades relacionadas con el almacenamiento. |
| Módulo de alimentación y refrigeración (PCM) |Componentes de hardware del dispositivo StorSimple que constan de fuentes de alimentación y el ventilador de refrigeración, por este motivo el nombre Módulo de alimentación y refrigeración. El gabinete principal del dispositivo tiene dos PCM de 764 W, mientras que el gabinete EBOD tiene dos PCM de 580 W. |
| gabinete principal |El gabinete principal del dispositivo StorSimple que contiene los controladores de la plataforma de la aplicación. |
| objetivo de tiempo de recuperación (RTO) |La cantidad máxima de tiempo que debe emplearse antes de que un proceso o sistema empresarial esté completamente restaurado tras un desastre. |
| SCSI conectado en serie (SAS) |Un tipo de unidad de disco duro (HDD). |
| clave de cifrado de datos de servicio |Una clave a disposición de cualquier nuevo dispositivo StorSimple que se registra con el servicio Administrador de dispositivos de StorSimple. Los datos de configuración transferidos entre el servicio Administrador de dispositivos de StorSimple y el dispositivo se cifran con una clave pública y, a continuación, se pueden descifrar únicamente en el dispositivo mediante una clave privada. La clave de cifrado de datos de servicio permite al servicio obtener esta clave privada para el descifrado. |
| clave de registro del servicio |Una clave que le ayuda a registrar el dispositivo StorSimple con el servicio Administrador de dispositivos de StorSimple, para que así aparezca en Azure Portal para diversas acciones administrativas. |
| Small Computer System Interface (SCSI) |Un conjunto de estándares para conectar físicamente equipos y pasar datos entre ellos. |
| unidad de estado sólido (SSD) |Un disco que no contiene piezas móviles; por ejemplo, una unidad flash. |
| Cuenta de almacenamiento |Un conjunto de credenciales de acceso vinculado a la cuenta de almacenamiento para un proveedor de servicios de nube determinado. |
| Adaptador de StorSimple para SharePoint |Un componente de Microsoft Azure StorSimple que extiende de manera transparente el almacenamiento de StorSimple y la protección de datos a las granjas de servidores de SharePoint. |
| Servicio de administrador de dispositivos de StorSimple |Una extensión de Azure Portal que le permite administrar Azure StorSimple en local y en dispositivos virtuales. |
| Snapshot Manager de StorSimple |Un complemento de Microsoft Management Console (MMC) para administrar las operaciones de copia de seguridad y restauración en Microsoft Azure StorSimple. |
| realizar copia de seguridad |Una característica que permite al usuario realizar una copia de seguridad interactiva de un volumen. Es una forma alternativa de realizar una copia de seguridad manual de un volumen en lugar de realizar una copia de seguridad automatizada a través de una directiva definida. |
| Aprovisionamiento fino |Un método para optimizar la eficacia con la que se utiliza el espacio de almacenamiento disponible en sistemas de almacenamiento. En el aprovisionamiento fino, el almacenamiento se asigna entre varios usuarios según el espacio mínimo requerido por cada usuario en un momento dado. Consulte también *Aprovisionamiento grueso*. |
| organización en niveles |Organizar datos en agrupaciones lógicas basadas en el uso actual, la antigüedad y la relación con otros datos. StorSimple organiza automáticamente los datos en niveles. |
| volumen |Áreas de almacenamiento lógico presentadas en forma de unidades de disco. Los volúmenes StorSimple corresponden a los volúmenes montados por el host, incluidos aquellos detectados mediante el uso de iSCSI y un dispositivo StorSimple. |
| contenedor de volúmenes |Un grupo de volúmenes y la configuración que se aplica a ellos. Todos los volúmenes en el dispositivo StorSimple se agrupan en contenedores de volúmenes. La configuración del contenedor de volumen incluye cuentas de almacenamiento, la configuración de cifrado para los datos enviados a la nube con las claves de cifrado asociada y el ancho de banda consumido en operaciones relacionadas con la nube. |
| grupo de volúmenes |En StorSimple Snapshot Manager, un grupo de volúmenes es un conjunto de volúmenes configurados para facilitar el proceso de copia de seguridad. |
| Volume Shadow Copy Service (VSS) |Un servicio del sistema operativo de Windows Server que facilita la consistencia de aplicaciones, ya que se comunica con las aplicaciones para VSS para coordinar la creación de instantáneas incrementales. VSS garantiza que las aplicaciones están temporalmente inactivas cuando se realizan las instantáneas. |
| Windows PowerShell para StorSimple |Una interfaz de línea de comandos basada en Windows PowerShell utilizada para controlar y administrar un dispositivo StorSimple. Además de conservar algunas de las capacidades básicas de Windows PowerShell, esta interfaz tiene cmdlets adicionales dedicados orientados a administrar el dispositivo StorSimple. |

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información acerca de la [Seguridad de StorSimple](storsimple-8000-security.md).

