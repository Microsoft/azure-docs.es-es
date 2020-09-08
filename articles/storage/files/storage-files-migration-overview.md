---
title: Migración a recursos compartidos de archivos de Azure
description: Obtenga información sobre las migraciones a los recursos compartidos de archivos de Azure y busque la guía de migración.
author: fauhse
ms.service: storage
ms.topic: conceptual
ms.date: 3/18/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 4223e3bc572a689472dce136b60599034566b274
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654266"
---
# <a name="migrate-to-azure-file-shares"></a>Migración a recursos compartidos de archivos de Azure

En este artículo se tratan los aspectos básicos de una migración a los recursos compartidos de archivos de Azure.

Este artículo contiene los aspectos básicos de la migración y una tabla de las guías de migración. Estas guías le ayudan a trasladar los archivos a recursos compartidos de archivos de Azure. Las guías se organizan en función de dónde se encuentran los datos y el modelo de implementación (solo en la nube o híbrido) al que va a realizar el cambio.

## <a name="migration-basics"></a>Aspectos básicos de la migración

Azure tiene varios tipos de almacenamiento en la nube disponibles. Un aspecto fundamental de las migraciones de archivos a Azure es determinar qué opción de almacenamiento de Azure es la adecuada para los datos.

Los [recursos compartidos de archivos de Azure](storage-files-introduction.md) son excelentes para los datos de archivos de uso general. Estos datos incluyen cualquier contenido para el que use un recurso compartido de SMB o NFS local. Con [Azure File Sync](storage-sync-files-planning.md), puede copiar en caché el contenido de varios recursos compartidos de archivos de Azure en servidores que ejecuten Windows Server en el entorno local.

En el caso de una aplicación que se ejecute actualmente en un servidor local, el almacenamiento de archivos en un recurso compartido de archivos de Azure puede ser una buena opción. Puede trasladar la aplicación a Azure y usar los recursos compartidos de archivos de Azure como almacenamiento compartido. También puede considerar los [discos de Azure](../../virtual-machines/managed-disks-overview.md) para este escenario.

Algunas aplicaciones en la nube no dependen de SMB ni del acceso compartido o a datos locales de la máquina. Para estas, el almacenamiento de objetos como [blobs de Azure](../blobs/storage-blobs-overview.md) suele ser la mejor opción.

La clave de cualquier migración es capturar toda la fidelidad aplicable de los archivos al mover los archivos desde su ubicación de almacenamiento actual a Azure. El grado de fidelidad que admite la opción de almacenamiento de Azure y el nivel de exigencia de su escenario también le ayudan a elegir el almacenamiento de Azure adecuado. Normalmente, los datos de archivos de uso general dependen de los metadatos de archivo. Pero puede que no sea el caso de los datos de la aplicación.

Los dos componentes básicos de un archivo son los siguientes:

- **Flujo de datos** El flujo de datos de un archivo almacena el contenido del archivo.
- **Metadatos del archivo**: Los metadatos del archivo tienen estos subcomponentes:
   * Atributos de archivo, como solo lectura
   * Permisos del archivo, que se conocen como *permisos NTFS* o *ACL de archivos y carpetas*
   * Marcas de tiempo, en particular, las marcas de tiempo de creación y última modificación
   * Flujo de datos alternativo, que es un espacio para almacenar grandes cantidades de propiedades no estándar.

La fidelidad de los archivos en una migración se puede definir como la capacidad de:

- Almacenar toda la información del archivo aplicable en el origen.
- Transferir archivos con la herramienta de migración.
- Almacenar archivos en el almacenamiento de destino de la migración.

Para asegurarse de que la migración se realiza de manera fluida, identifique [la mejor herramienta de copia para sus necesidades](#migration-toolbox) y haga coincidir un destino de almacenamiento con el origen.

Teniendo en cuenta la información anterior, queda claro que el almacenamiento de destino para los archivos de uso general en Azure son los [recursos compartidos de archivos de Azure](storage-files-introduction.md).

A diferencia del almacenamiento de objetos en blobs de Azure, un recurso compartido de archivos de Azure puede almacenar de forma nativa metadatos de archivo. Los recursos compartidos de archivos de Azure también conservan la jerarquía de archivos y carpetas, los atributos y los permisos. Los permisos NTFS se pueden almacenar en archivos y carpetas, ya que son locales.

Un usuario de Active Directory, que es su controlador de dominio local, puede tener acceso de forma nativa a un recurso compartido de archivos de Azure. También puede tenerlo un usuario de Azure Active Directory Domain Services (Azure AD DS). Ambos usan su identidad actual para obtener acceso en función de los permisos de recurso compartido, así como de las ACL de archivos y carpetas. Este comportamiento es similar al de un usuario que se conecta a un recurso compartido de archivos local.

El flujo de datos alternativo es el aspecto principal de la fidelidad de los archivos que actualmente no se puede almacenar en un archivo de un recurso compartido de archivos de Azure. Se conserva localmente cuando se usa Azure File Sync.

Obtenga más información sobre la [autenticación de Azure AD](storage-files-identity-auth-active-directory-enable.md) y la [autenticación de Azure AD DS](storage-files-identity-auth-active-directory-domain-service-enable.md) de los recursos compartidos de archivos de Azure.

## <a name="migration-guides"></a>Guías de migración

En la tabla siguiente se enumeran las guías de migración detalladas.

Cómo debe usar la tabla:

1. Busque la fila del sistema de origen en el que están almacenados los archivos.

1. Elija uno de los destinos siguientes:

   - Una implementación híbrida con Azure File Sync para copiar en caché el contenido de los recursos compartidos de archivos de Azure de forma local
   - Recursos compartidos de archivos de Azure en la nube

   Seleccione la columna de destino que coincida con su elección.

1. Dentro de la intersección de origen y destino, una celda de la tabla muestra los escenarios de migración disponibles. Seleccione uno para vincularlo directamente a la guía de migración detallada.

Un escenario sin un vínculo aún no tiene ninguna guía de migración publicada. Compruebe de vez en cuando en esta tabla si hay actualizaciones. Las nuevas guías se publicarán en cuanto estén disponibles.

| Source | Destino: </br>Implementación híbrida | Destino: </br>Implementación solo en la nube |
|:---|:--|:--|
| | Combinación de herramientas:| Combinación de herramientas: |
| Windows Server 2012 R2 y versiones posteriores | <ul><li>[Azure File Sync](storage-sync-files-deployment-guide.md)</li><li>[Azure File Sync y Azure Data Box](storage-sync-offline-data-transfer.md)</li><li>[Azure File Sync y archivos previamente inicializados en la nube](storage-sync-offline-data-transfer.md#azure-file-sync-and-pre-seeded-files-in-the-cloud)</li><li>Azure File Sync y servicio de migración de almacenamiento</li></ul> | <ul><li>Azure File Sync</li><li>Azure File Sync y Data Box</li><li>Azure File Sync y servicio de migración de almacenamiento</li><li>RoboCopy</li></ul> |
| Windows Server 2012 y versiones anteriores | <ul><li>Azure File Sync y Data Box</li><li>Azure File Sync y servicio de migración de almacenamiento</li></ul> | <ul><li>Azure File Sync y servicio de migración de almacenamiento</li><li>RoboCopy</li></ul> |
| Almacenamiento conectado a la red (NAS) | <ul><li>[Azure File Sync y RoboCopy](storage-files-migration-nas-hybrid.md)</li></ul> | <ul><li>RoboCopy</li></ul> |
| Linux o Samba | <ul><li>[Azure File Sync y RoboCopy](storage-files-migration-linux-hybrid.md)</li></ul> | <ul><li>RoboCopy</li></ul> |
| Dispositivo de nube Microsoft Azure StorSimple 8100 o dispositivo de nube StorSimple 8600 | <ul><li>[Azure File Sync y dispositivo de nube StorSimple 8020](storage-files-migration-storsimple-8000.md)</li></ul> | |
| Dispositivo de nube StorSimple 1200 | <ul><li>[Azure File Sync](storage-files-migration-storsimple-1200.md)</li></ul> | |
| | | |

## <a name="migration-toolbox"></a>Cuadro de herramientas de migración

### <a name="file-copy-tools"></a>Herramientas de copia de archivos

Existen varias herramientas de copia de archivos disponibles de Microsoft y otras empresas. Para seleccionar la herramienta adecuada para el escenario de migración, debe tener en cuenta estas preguntas fundamentales:

* ¿La herramienta admite las ubicaciones de origen y destino para la copia de archivos?

* ¿La herramienta admite la ruta de acceso de red o los protocolos disponibles (por ejemplo, REST, SMB o NFS) entre las ubicaciones de almacenamiento de origen y de destino?

* ¿La herramienta conserva la fidelidad de los archivos necesaria que es compatible con las ubicaciones de origen y destino?

    En algunos casos, el almacenamiento de destino no admite la misma fidelidad que el origen. Si el almacenamiento de destino es suficiente para sus necesidades, la herramienta solo debe hacer coincidir las funcionalidades de fidelidad de los archivos de destino.

* ¿La herramienta cuenta con características que le permiten adaptarse a la estrategia de migración?

    Por ejemplo, tenga en cuenta si la herramienta le permite minimizar el tiempo de inactividad.
    
    Cuando una herramienta admite una opción para reflejar un origen en un destino, a menudo puede ejecutarla varias veces en el mismo origen y destino mientras el origen permanece accesible.

    La primera vez que se ejecuta la herramienta, copia la mayor parte de los datos. Puede que esta primera ejecución tarde un rato. A menudo tarda más tiempo del que le gustaría para dejar sin conexión el origen de datos de sus procesos empresariales.

    Si crea un reflejo de un origen en un destino (por ejemplo, con **robocopy /MIR**), puede volver a ejecutar la herramienta en ese mismo origen y destino. La ejecución es mucho más rápida porque solo necesita transportar los cambios del origen que se producen después de la ejecución anterior. Volver a ejecutar una herramienta de copia de esta manera puede reducir significativamente el tiempo de inactividad.

En la tabla siguiente se clasifican las herramientas de Microsoft y su idoneidad actual para los recursos compartidos de archivos de Azure:

| Recomendado | Herramienta | Compatibilidad con los recursos compartidos de archivos de Azure | Preservación de la fidelidad de los archivos |
| :-: | :-- | :---- | :---- |
|![Sí, recomendado](media/storage-files-migration-overview/circle-green-checkmark.png)| RoboCopy | Compatible. Los recursos compartidos de archivos de Azure se pueden montar como unidades de red. | Fidelidad completa* |
|![Sí, recomendado](media/storage-files-migration-overview/circle-green-checkmark.png)| Azure File Sync | Integrado de forma nativa en recursos compartidos de archivos de Azure. | Fidelidad completa* |
|![Sí, recomendado](media/storage-files-migration-overview/circle-green-checkmark.png)| Servicio de migración de almacenamiento | Indirectamente compatible. Los recursos compartidos de archivos de Azure se pueden montar como unidades de red en servidores de destino de SMS. | Fidelidad completa* |
|![Sí, recomendado](media/storage-files-migration-overview/circle-green-checkmark.png)| AzCopy, versión 10.4 o posteriores| Compatible. | Fidelidad completa* |
|![No es totalmente recomendable.](media/storage-files-migration-overview/triangle-yellow-exclamation.png)| Data Box | Compatible. | No copia los metadatos. [Data Box se puede usar con Azure File Sync](storage-sync-offline-data-transfer.md). |
|![No es totalmente recomendable.](media/storage-files-migration-overview/triangle-yellow-exclamation.png)| Explorador de Azure Storage, versión 1.14 | Compatible. | No copia las ACL. Admite marcas de tiempo.  |
|![No recomendado](media/storage-files-migration-overview/circle-red-x.png)| Azure Data Factory | Compatible. | No copia los metadatos. |
|||||

*\* Fidelidad total: cumple o supera las funcionalidades de los recursos compartidos de archivos de Azure.*

### <a name="migration-helper-tools"></a>Herramientas del asistente para la migración

En esta sección se describen las herramientas que le ayudan a planear y ejecutar las migraciones.

#### <a name="robocopy-from-microsoft-corporation"></a>RoboCopy de Microsoft Corporation

RoboCopy es una de las herramientas más aplicables a las migraciones de archivos. Se incluye como parte de Windows. La [documentación principal de RoboCopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) es un recurso útil para las numerosas opciones de esta herramienta.

#### <a name="treesize-from-jam-software-gmbh"></a>TreeSize de JAM Software GmbH

Azure File Sync se escala principalmente con el número de elementos (archivos y carpetas) y no con la cantidad de almacenamiento total. La herramienta TreeSize le permite determinar el número de elementos de los volúmenes de Windows Server.

Puede usar la herramienta para crear una perspectiva antes de una [implementación de Azure File Sync](storage-sync-files-deployment-guide.md). También puede usarla cuando se active la nube por niveles después de la implementación. En ese escenario, verá el número de elementos y los directorios que usan más la memoria caché del servidor.

La versión probada de la herramienta es la 4.4.1. Es compatible con los archivos de niveles en la nube. La herramienta no producirá la coincidencia de los archivos en niveles durante su funcionamiento normal.

## <a name="next-steps"></a>Pasos siguientes

1. Cree un plan para la implementación de los recursos compartidos de archivos de Azure (solo en la nube o híbridos) que quieras.
1. Revise la lista de guías de migración disponibles para encontrar la guía detallada que coincide con el origen y la implementación de los recursos compartidos de archivos de Azure.

A continuación, le incluimos más información sobre las tecnologías de Azure Files mencionadas en este artículo:

* [¿Qué es Azure Files](storage-files-introduction.md)
* [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
* [Información general de nube por niveles ](storage-sync-cloud-tiering.md)
