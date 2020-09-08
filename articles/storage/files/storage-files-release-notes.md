---
title: Notas de la versión del agente de Azure File Sync | Microsoft Docs
description: Lea las notas de la versión del agente de Azure File Sync, que le permite centralizar los recursos compartidos de archivos de la organización en Azure Files.
services: storage
author: wmgries
ms.service: storage
ms.topic: conceptual
ms.date: 7/28/2020
ms.author: wgries
ms.subservice: files
ms.openlocfilehash: 00590c2b12e8600b2a650444b97b6558e3a52da1
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904945"
---
# <a name="release-notes-for-the-azure-file-sync-agent"></a>Notas de la versión del agente de Azure File Sync
Azure File Sync le permite centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Las instalaciones de Windows Server se transforman en una memoria caché rápida de los recursos compartidos de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a los datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.

En este artículo se tratan las notas de la versión de las versiones compatibles del agente de Azure File Sync.

## <a name="supported-versions"></a>Versiones compatibles
Las siguientes versiones son compatibles con Azure File Sync:

| Hito | Número de versión del agente | Fecha de la versión | Estado |
|----|----------------------|--------------|------------------|
| Versión V10.1: [KB4522411](https://support.microsoft.com/en-us/help/4522411)| 10.1.0.0 | 5 de junio de 2020 | Compatible |
| Paquete acumulativo de actualizaciones de mayo de 2020: [KB4522412](https://support.microsoft.com/help/4522412)| 10.0.2.0 | 19 de mayo de 2020 | Compatible |
| Versión V10: [KB4522409](https://support.microsoft.com/en-us/help/4522409)| 10.0.0.0 | 9 de abril de 2020 | Compatible |
| Paquete acumulativo de las actualizaciones de diciembre de 2019: [KB4522360](https://support.microsoft.com/help/4522360)| 9.1.0.0 | 12 de diciembre de 2019 | Compatible |
| Versión V9 - [KB4522359](https://support.microsoft.com/help/4522359)| 9.0.0.0 | 2 de diciembre de 2019 | Compatible |
| Versión V8 - [KB4511224](https://support.microsoft.com/help/4511224)| 8.0.0.0 | 8 de octubre de 2019 | Compatible |
| Paquete acumulativo de actualizaciones de julio de 2019: [KB4490497](https://support.microsoft.com/help/4490497)| 7.2.0.0 | 24 de julio de 2019 | Compatible: la versión del agente expirará el 1 de septiembre de 2020. |
| Paquete acumulativo de actualizaciones de julio de 2019: [KB4490496](https://support.microsoft.com/help/4490496)| 7.1.0.0 | 12 de julio de 2019 | Compatible: la versión del agente expirará el 1 de septiembre de 2020. |
| Versión V7: [KB4490495](https://support.microsoft.com/help/4490495)| 7.0.0.0 | 19 de junio de 2019 | Compatible: la versión del agente expirará el 1 de septiembre de 2020. |
| Versión V6 | 6.0.0.0 - 6.3.0.0 | N/D | No compatible: las versiones del agente expiraron el 21 de abril de 2020. |
| Versión V5 | 5.0.2.0 - 5.2.0.0 | N/D | No compatibles: las versiones del agente expiraron el 18 de marzo de 2020 |
| Versión V4 | 4.0.1.0-4.3.0.0 | N/D | No compatibles: las versiones del agente expiraron el 6 de noviembre de 2019 |
| Versión V3 | 3.1.0.0 - 3.4.0.0 | N/D | No compatibles: las versiones del agente expiraron el 19 de agosto de 2019 |
| Agentes anteriores a la disponibilidad general | 1.1.0.0 - 3.0.13.0 | N/D | No compatibles: las versiones del agente expiraron el 1 de octubre de 2018 |

### <a name="azure-file-sync-agent-update-policy"></a>Directiva de actualización del agente de Azure File Sync
[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## <a name="agent-version-10100"></a>Versión 10.1.0.0 del agente
Las notas siguientes corresponden a la versión 10.1.0.0 del agente de Azure File Sync, publicada el 5 de junio de 2020. Estas notas son adicionales a las notas de la versión enumeradas para las versiones 10.0.0.0 y 10.0.2.0.

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Compatibilidad con el punto de conexión privado de Azure
    - El tráfico de sincronización al servicio de sincronización de almacenamiento ahora se puede enviar a un punto de conexión privado. Esto permite la tunelización a través de una conexión de ExpressRoute o VPN. Para obtener más información, consulte [Configuración de puntos de conexión de red de Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-sync-files-networking-endpoints).
- La métrica Archivos sincronizados ahora muestra el progreso mientras se ejecuta una sincronización grande, en lugar de al final.
- Varias mejoras de confiabilidad para la instalación de agentes, la organización de nube por niveles, la sincronización y la telemetría

## <a name="agent-version-10020"></a>Versión del agente 10.0.2.0
Las notas de la versión siguientes corresponden a la versión 10.0.2.0 del agente de Azure File Sync publicada el 19 de mayo de 2020. Estas notas se suman a las notas de la versión enumeradas para la versión 10.0.0.0.

Esto se ha corregido en esta versión:  
- El agente de sincronización de almacenamiento (FileSyncSvc) se bloquea con frecuencia después de instalar el agente de Azure File Sync v10.

> [!Note]  
>Esta versión no lanzó como paquete piloto en los servidores que están configurados para actualizarse automáticamente cuando está disponible una nueva versión. Para instalar esta actualización, use Microsoft Update o el catálogo de Microsoft Update (consulte [KB4522412](https://support.microsoft.com/help/4522412) para obtener instrucciones sobre la instalación).

## <a name="agent-version-10000"></a>Versión 10.0.0.0 del agente
Las notas de la versión siguientes corresponden a la versión 10.0.0.0 del agente de Azure File Sync (publicada el 9 de abril de 2020).

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Progreso de sincronización mejorado en el portal
    - Con la versión V10 del agente, Azure Portal comenzará pronto a mostrar el tipo de sesión de sincronización que está en ejecución. Por ejemplo, descarga inicial, descarga normal, recuperación en segundo plano (casos rápidos de recuperación ante desastres) y similar. 

- Experiencia mejorada del portal de nube por niveles
    - Si tiene archivos que presentan errores de almacenamiento por niveles o recuperación, ahora puede ver los errores por niveles en las propiedades del punto de conexión de servidor.
    - Se encuentra disponible información adicional de la nube por niveles para un punto de conexión de servidor:
        - Tamaño de la caché local
        - Eficiencia del uso de la memoria caché
        - Detalles de la directiva de la nube por niveles: tamaño del volumen, espacio disponible actual o la hora del último acceso al archivo más antiguo en la caché local.
    - Estos cambios se incluirán en Azure Portal poco después de la versión inicial V10 del agente.

- Compatibilidad con la migración del servicio de sincronización de almacenamiento o la cuenta de almacenamiento a otro inquilino de Azure Active Directory (AAD)
    - Azure File Sync ahora admite la migración del servicio de sincronización de almacenamiento o de la cuenta de almacenamiento a un grupo de recursos, una suscripción o un inquilino de Azure AD diferentes.
    
- Mejoras varias de rendimiento y confiabilidad
    - La detección de cambios en el recurso compartido de archivos de Azure puede producir un error si las reglas de red virtual (VNET) y de firewall están configuradas en la cuenta de almacenamiento.
    - Consumo de memoria reducido asociado con la recuperación. 
    - Rendimiento mejorado al usar el cmdlet [Invoke-AzStorageSyncChangeDetection](https://docs.microsoft.com/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection).
    - Otras mejoras de confiabilidad varias. 
    
### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con la opción de implementación de Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y esto puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Para sincronizar inmediatamente los archivos que se modifican en el recurso compartido de archivos de Azure, se puede usar el cmdlet [Invoke-AzStorageSyncChangeDetection](https://docs.microsoft.com/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) de PowerShell para iniciar de forma manual la detección de cambios en el recurso compartido. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos, suscripción o inquilino de Azure AD diferentes. Después de mover el servicio de sincronización de almacenamiento o la cuenta de almacenamiento, debe dar acceso a la aplicación Microsoft.StorageSync a la cuenta de almacenamiento (consulte [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Al crear el punto de conexión de nube, el servicio de sincronización de almacenamiento y la cuenta de almacenamiento deben estar en el mismo inquilino de Azure AD. Una vez creado el punto de conexión de nube, el servicio de sincronización de almacenamiento y la cuenta de almacenamiento se pueden migrar a distintos inquilinos de Azure AD.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.

## <a name="agent-version-9100"></a>Versión del agente 9.1.0.0
Las notas siguientes corresponden a la versión 9.1.0.0 del agente de Azure File Sync (publicada el 12 de diciembre de 2019). Estas notas se suman a las notas de la versión enumeradas para la versión 9.0.0.0.

Esto se ha corregido en esta versión:  
- Se produce un error en la sincronización con uno de los siguientes errores después de actualizar a la versión 9.0 del agente de Azure File Sync:
    - 0x8e5e044e (JET_errWriteConflict)
    - 0x8e5e0450 (JET_errInvalidSesid)
    - 0x8e5e0442 (JET_errInstanceUnavailable)

## <a name="agent-version-9000"></a>Versión del agente 9.0.0.0
Las notas siguientes corresponden a la versión 9.0.0.0 del agente de Azure File Sync (publicada el 2 de diciembre de 2019).

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Compatibilidad con el Autoservicio de restauración
    - Los usuarios pueden ahora restaurar sus archivos mediante la característica de la versión anterior. Antes de la versión v9, no se admitía la característica de la versión anterior en los volúmenes que tenían habilitada la nube por niveles. Esta característica tiene que habilitarse por separado para cada volumen en el que existe un punto de conexión con nube por niveles habilitada. Para obtener más información, consulte:  
[Autoservicio de restauración a través de versiones anteriores y VSS (Servicio de instantáneas de volumen)](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#self-service-restore-through-previous-versions-and-vss-volume-shadow-copy-service). 
 
- Compatibilidad con tamaños de recurso compartido de archivos más grandes 
    - Azure File Sync admite hasta 64 TiB y 100 millones de archivos en un mismo espacio de nombres de sincronización.  
 
- Compatibilidad de desduplicación de datos en Server 2019 
    - La desduplicación de datos es ahora compatible con la nube por niveles habilitada en Windows Server 2019. Para admitir la desduplicación de datos en volúmenes con nube por niveles, es necesario instalar Windows Update [KB4520062](https://support.microsoft.com/help/4520062). 
 
- Tamaño mínimo de archivo mejorado para organizar un archivo en niveles 
    - El tamaño mínimo para organizar un archivo en niveles se basa ahora en el tamaño del clúster del sistema de archivos (el doble del tamaño del clúster del sistema de archivos). Por ejemplo, de forma predeterminada, el tamaño del clúster del sistema de archivos NTFS es 4 KB, el tamaño mínimo de archivo resultante para organizar un archivo en niveles es 8 KB. 
 
- Cmdlet de prueba de conectividad de red 
    - Como parte de la configuración de Azure File Sync, se tiene que establecer contacto con varios puntos de conexión de servicio. Cada uno tiene su propio nombre DNS que tiene que ser accesible para el servidor. Estas direcciones URL también son específicas de la región en la que se ha registrado un servidor. Una vez registrado un servidor, se puede usar el cmdlet de prueba de conectividad (PowerShell y la utilidad de registro de servidor) para probar las comunicaciones con todas las direcciones URL específicas de este servidor. Este cmdlet puede ayudar a solucionar problemas si una comunicación incompleta impide que el servidor funcione totalmente con Azure File Sync, y se puede usar para ajustar las configuraciones de proxy y firewall.  
 
        Para ejecutar la prueba de conectividad de red, ejecute los siguientes comandos de PowerShell: 
 
        Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"  
        Test-StorageSyncNetworkConnectivity
 
- Retirada de la mejora del punto de conexión del servidor cuando la nube por niveles está habilitada 
    - Como antes, la eliminación de un punto de conexión de servidor no produce la eliminación de archivos en el recurso compartido de archivos de Azure. Sin embargo, el comportamiento de los puntos de reanálisis en el servidor local ha cambiado. Los puntos de reanálisis (punteros a archivos que no son locales en el servidor) se eliminan ahora al eliminar un punto de conexión de servidor. Los archivos completamente almacenados en caché permanecerán en el servidor. Esta mejora se realizó para evitar los [archivos huérfanos en niveles](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint) al quitar un punto de conexión de servidor. Si se vuelve a crear el punto de conexión de servidor, los puntos de reanálisis de los archivos en niveles se volverán a crear en el servidor.  
 
- Mejoras de rendimiento y confiabilidad. 
    - Errores de recuperación reducidos. El tamaño de la recuperación ahora se ajusta automáticamente en función del ancho de banda de red. 
    - Rendimiento de descarga mejorado al agregar un nuevo servidor a un grupo de sincronización. 
    - Archivos reducidos que no se sincronizan debido a conflictos de restricción. 
    - Los archivos no se pueden segmentar o se recuperan de forma inesperada en ciertos escenarios si la ruta de acceso del punto de conexión del servidor es un punto de montaje del volumen.
    
### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con la opción de implementación de Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y esto puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso discrecional (DACL) de un descriptor de seguridad, si es mayor de 2 KB. (Este problema aplica únicamente si tiene más de unas 40 entradas de control de acceso [ACE] en un solo elemento).
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los archivos en niveles se volverán inaccesibles si no se recuperan antes de eliminar el punto de conexión de servidor. Para restaurar el acceso a los archivos, vuelva a crear el punto de conexión de servidor. Si han pasado 30 días desde que se eliminó el punto de conexión de servidor o si se ha eliminado el punto de conexión en la nube, los archivos en niveles que no se hayan recuperado serán inutilizables. Para más información, consulte [No se puede acceder a los archivos organizados en niveles de servicio en el servidor después de eliminar un punto de conexión de servidor](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint).
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Para sincronizar inmediatamente los archivos que se modifican en el recurso compartido de archivos de Azure, se puede usar el cmdlet [Invoke-AzStorageSyncChangeDetection](https://docs.microsoft.com/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) de PowerShell para iniciar de forma manual la detección de cambios en el recurso compartido. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos o suscripción diferentes dentro del inquilino de Azure AD existente. Si se mueve la cuenta de almacenamiento, debe dar acceso al servicio File Sync híbrido a la cuenta de almacenamiento (consulte el apartado [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Azure File Sync no permite mover la suscripción a un inquilino de Azure AD distinto.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.
- Es posible que los archivos no se puedan segmentar si pagefile.sys se encuentra en un volumen que tiene habilitada nube por niveles. El archivo pagefile.sys debe encontrarse en un volumen que tenga deshabilitada la nube por niveles.

## <a name="agent-version-8000"></a>Versión del agente 8.0.0.0
Las notas siguientes corresponden a la versión 8.0.0.0 del agente de Azure File Sync publicada el 8 de octubre de 2019.

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Mejoras en el rendimiento de restauración
    - Tiempos de recuperación más rápidos para la recuperación mediante Azure Backup. Los archivos restaurados se sincronizarán de nuevo con los servidores de Azure File Sync mucho más rápidamente. 
- Experiencia mejorada del portal de nube por niveles  
    - Si se produce un error al recuperar archivos con niveles, ahora puede ver los errores de recuperación en las propiedades del punto de conexión de servidor. Además, el estado del punto de conexión del servidor ahora mostrará un error y los pasos de mitigación si el controlador del filtro de nube por niveles no está cargado en el servidor.
- Instalación del agente más sencilla
    - El módulo Az\AzureRM de PowerShell ya no es necesario para registrar el servidor, de modo que la instalación es más sencilla y rápida.
- Mejoras varias de rendimiento y confiabilidad

### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con la opción de implementación de Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y esto puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso discrecional (DACL) de un descriptor de seguridad, si es mayor de 2 KB. (Este problema aplica únicamente si tiene más de unas 40 entradas de control de acceso [ACE] en un solo elemento).
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los archivos en niveles se volverán inaccesibles si no se recuperan antes de eliminar el punto de conexión de servidor. Para restaurar el acceso a los archivos, vuelva a crear el punto de conexión de servidor. Si han pasado 30 días desde que se eliminó el punto de conexión de servidor o si se ha eliminado el punto de conexión en la nube, los archivos en niveles que no se hayan recuperado serán inutilizables. Para más información, consulte [No se puede acceder a los archivos organizados en niveles de servicio en el servidor después de eliminar un punto de conexión de servidor](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint).
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Para sincronizar inmediatamente los archivos que se modifican en el recurso compartido de archivos de Azure, se puede usar el cmdlet [Invoke-AzStorageSyncChangeDetection](https://docs.microsoft.com/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) de PowerShell para iniciar de forma manual la detección de cambios en el recurso compartido. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos o suscripción diferentes dentro del inquilino de Azure AD existente. Si se mueve la cuenta de almacenamiento, debe dar acceso al servicio File Sync híbrido a la cuenta de almacenamiento (consulte el apartado [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Azure File Sync no permite mover la suscripción a un inquilino de Azure AD distinto.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.

## <a name="agent-version-7200"></a>Versión del agente 7.2.0.0
Las notas siguientes corresponden a la versión 7.2.0.0 del agente de Azure File Sync publicada el 24 de julio de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 7.0.0.0.

Lista de problemas corregidos en esta versión:  
- El agente de sincronización de almacenamiento (FileSyncSvc) se bloquea si la configuración del proxy es NULL.
- El punto de conexión del servidor iniciará BCDR (error 0x80c80257-ECS_E_BCDR_IN_PROGRESS) si hay varios puntos de conexión en el servidor con el mismo nombre.
- Se han realizado mejoras de confiabilidad de la nube por niveles.

## <a name="agent-version-7100"></a>Versión del agente 7.1.0.0
Las notas siguientes corresponden a la versión 7.1.0.0 del agente de Azure File Sync publicada el 12 de julio de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 7.0.0.0.

Lista de problemas corregidos en esta versión:  
- El acceso o exploración en una ubicación de punto de conexión de servidor a través de SMB son lentos en Windows Server 2012 R2. 
- Aumento del uso de CPU después de instalar el agente de Azure File Sync versión 6.
- Mejoras en la telemetría de la nube por niveles.
- Mejoras de confiabilidad varias para la sincronización y la nube por niveles.

## <a name="agent-version-7000"></a>Versión del agente 7.0.0.0
Las notas siguientes corresponden a la versión 7.0.0.0 del agente de Azure File Sync, publicada el 19 de junio de 2019.

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Compatibilidad con tamaños de recurso compartido de archivos más grandes
    - Con la versión preliminar de recursos compartidos de archivos de Azure más grandes, aumentamos también nuestros límites de compatibilidad en File Sync. En este primer paso, Azure File Sync admite hasta 25 TB y 50 millones de archivos en un mismo espacio de nombres de sincronización. Para solicitar la versión preliminar de recurso compartido de archivos de gran tamaño, rellene este formulario: https://aka.ms/azurefilesatscalesurvey. 
- Compatibilidad con la configuración del firewall y la red virtual en cuentas de almacenamiento
    - Ahora Azure File Sync admite la configuración del firewall y la red virtual en las cuentas de almacenamiento. Para configurar la implementación de modo que funcione con la configuración del firewall y la red virtual, consulte [Configuración de los ajustes de red virtual y del firewall](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=azure-portal#configure-firewall-and-virtual-network-settings).
- Cmdlet de PowerShell para sincronizar inmediatamente los archivos modificados en el recurso compartido de archivos de Azure
    - Para sincronizar inmediatamente los archivos que se modifican en el recurso compartido de archivos de Azure, se puede usar el cmdlet Invoke-AzStorageSyncChangeDetection de PowerShell para iniciar de forma manual la detección de cambios en el recurso compartido. Este cmdlet está pensado para escenarios en los que algún tipo de proceso automatizado está realizando cambios en el recurso compartido de archivos de Azure o en los que es el administrador el que efectúa los cambios (por ejemplo, al mover archivos y directorios al recurso compartido). En el caso de los cambios del usuario final, se recomienda instalar el agente de Azure File Sync en una VM de IaaS y hacer que los usuarios finales accedan al recurso compartido de archivos a través de ella. De este modo, todos los cambios se sincronizarán rápidamente con otros agentes sin necesidad de usar el cmdlet Invoke-AzStorageSyncChangeDetection. Para más información, consulte la documentación sobre [Invoke-AzStorageSyncChangeDetection](https://docs.microsoft.com/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection).
- Mejor experiencia del portal si detecta que hay archivos que no se están sincronizando
    - Si tiene archivos que no se pueden sincronizar, ahora puede distinguir los errores transitorios y persistentes en el portal. Los errores transitorios suelen resolverse por sí mismos sin necesidad de una acción del administrador. Por ejemplo, un archivo que está actualmente en uso no se sincronizará hasta que se cierre el identificador de archivos. En el caso de los errores persistentes, ahora se muestra el número de archivos afectados por cada error. El recuento de errores persistentes también se muestra en la columna de archivos no sincronizados de todos los puntos de conexión de servidor de un grupo de sincronización.
- Mejor restauración en el nivel de archivo de Azure Backup
    - Ahora, los archivos individuales restaurados con Azure Backup se detectan y se sincronizan más rápidamente con el punto de conexión de servidor.
- Mejor confiabilidad del cmdlet de recuperación de nube por niveles 
    - Ahora, el cmdlet Invoke-StorageSyncFileRecall permite a los clientes especificar el número de reintentos por archivo y el intervalo entre reintentos por archivo, como RoboCopy. Anteriormente, este cmdlet recuperaba todos los archivos en niveles de una ruta de acceso determinada en orden aleatorio. Con el nuevo parámetro -Order, este cmdlet recuperará primero los datos de acceso frecuente y cumplirá la directiva de nube por niveles (detenga la recuperación si se cumple la directiva de fecha o se alcanza el espacio libre del volumen; lo que suceda primero).
- Compatibilidad únicamente con TLS 1.2 (TLS 1.0 y 1.1 están deshabilitados)
    - Así pues, Azure File Sync admite el uso de TLS 1.2 solo en aquellos servidores que tengan las versiones de TLS 1.0 y 1.1 deshabilitadas. Antes de esta mejora, el registro del servidor generaba un error si TLS 1.0 y 1.1 estaban deshabilitados en el servidor.
- Mejoras varias de rendimiento y confiabilidad de la sincronización y la nube por niveles
    - Existen varias mejoras de confiabilidad y rendimiento en esta versión. Algunas de ellas están orientadas a mejorar la eficacia de la nube por niveles, y Azure File Sync funciona en general de mejor forma en situaciones en las que hay una programación de limitación de ancho de banda establecida.

### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con la opción de implementación de Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y esto puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso discrecional (DACL) de un descriptor de seguridad, si es mayor de 2 KB. (Este problema aplica únicamente si tiene más de unas 40 entradas de control de acceso [ACE] en un solo elemento).
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los archivos en niveles se volverán inaccesibles si no se recuperan antes de eliminar el punto de conexión de servidor. Para restaurar el acceso a los archivos, vuelva a crear el punto de conexión de servidor. Si han pasado 30 días desde que se eliminó el punto de conexión de servidor o si se ha eliminado el punto de conexión en la nube, los archivos en niveles que no se hayan recuperado serán inutilizables.
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos o suscripción diferentes dentro del inquilino de Azure AD existente. Si se mueve la cuenta de almacenamiento, debe dar acceso al servicio File Sync híbrido a la cuenta de almacenamiento (consulte el apartado [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Azure File Sync no permite mover la suscripción a un inquilino de Azure AD distinto.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.

## <a name="agent-version-6300"></a>Versión del agente 6.3.0.0
Las notas siguientes corresponden a la versión 6.3.0.0 del agente de Azure File Sync, publicada el 27 de junio de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 6.0.0.0.

Lista de problemas corregidos en esta versión:  
- El acceso o exploración en una ubicación de punto de conexión de servidor a través de SMB son lentos en Windows Server 2012 R2. 
- Aumento del uso de CPU después de instalar el agente de Azure File Sync versión 6.
- Mejoras en la telemetría de la nube por niveles.

## <a name="agent-version-6200"></a>Versión del agente 6.2.0.0
Las notas siguientes corresponden a la versión 6.2.0.0 del agente de Azure File Sync, publicada el 13 de junio de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 6.0.0.0.

Lista de problemas corregidos en esta versión:  
- Después de crear un punto de conexión del servidor, el uso de la CPU se dispara cuando una tarea de recuperación en segundo plano descarga archivos en el servidor.
- Las operaciones de sincronización y de nube por niveles pueden producir el error ECS_E_SERVER_CREDENTIAL_NEEDED debido a la expiración del token.
- Recuperar un archivo puede generar un error si la dirección URL para descargar el archivo contiene caracteres reservados. 

## <a name="agent-version-6100"></a>Versión del agente 6.1.0.0
Las notas siguientes corresponden a la versión 6.1.0.0 del agente de Azure File Sync, publicada el 6 de mayo de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 6.0.0.0.

Lista de problemas corregidos en esta versión:  
- Windows Admin Center no muestra la configuración del punto de conexión del servidor y la versión del agente en servidores que tienen instalada la versión 6.0 del agente de Azure File Sync.

## <a name="agent-version-6000"></a>Versión del agente 6.0.0.0
Las notas siguientes corresponden a la versión 6.0.0.0 del agente de Azure File Sync, publicada el 22 de abril de 2019.

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Compatibilidad con la actualización automática del agente
  - Hemos atendido sus comentarios y hemos agregado una característica de actualización automática del agente de servidor de Azure File Sync. Para más información, vea [Directiva de actualización del agente de Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#azure-file-sync-agent-update-policy).
- Compatibilidad con las ACL de recurso compartido de archivos de Azure
  - Azure File Sync siempre ha permitido la sincronización de ACL entre puntos de conexión de servidor, pero las ACL no se sincronizaban en el punto de conexión en la nube (recurso compartido de archivos de Azure). Esta versión incluye compatibilidad con la sincronización de ACL entre puntos de conexión de servidor y de nube.
- Sesiones de sincronización de carga y descarga en paralelo de un punto de conexión de servidor 
  - Ahora, los puntos de conexión de servidor admiten la carga y descarga de archivos al mismo tiempo. Se acabó eso de esperar a que una descarga se complete para poder cargar archivos en el recurso compartido de archivos de Azure. 
- Nuevos cmdlets de nube por niveles para obtener estados de volúmenes y niveles
  - Ahora, se pueden usar dos nuevos cmdlets de PowerShell locales de servidor para obtener información de recuperación de archivos y de nube por niveles. Estos cmdlets proporcionan información de registro de dos canales de eventos en el servidor:
    - Get-StorageSyncFileTieringResult muestra todos los archivos (y sus rutas de acceso) que aún no se han establecido en niveles, y por qué.
    - Get-StorageSyncFileRecallResult informa de todos los eventos de recuperación de archivos. Muestra todos los archivos recuperados (y sus rutas de acceso), así como si la recuperación ha sido correcta o no.
  - Ambos canales de eventos pueden almacenar de forma predeterminada hasta 1 MB cada uno, aunque la cantidad de archivos mostrada se puede aumentar incrementando el tamaño del canal de eventos.
- Compatibilidad con el modo FIPS
  - Ahora, Azure File Sync permite habilitar el modo FIPS en servidores que tienen instalado el agente de Azure File Sync.
    - Antes de habilitar el modo FIPS en el servidor, instale el agente de Azure File Sync y el [módulo PackageManagement](https://www.powershellgallery.com/packages/PackageManagement/1.1.7.2) en el servidor. Si FIPS ya está habilitado en el servidor, [descargue manualmente](/powershell/scripting/gallery/how-to/working-with-packages/manual-download) el [módulo PackageManagement](https://www.powershellgallery.com/packages/PackageManagement/1.1.7.2) en el servidor.
- Mejoras varias de confiabilidad de la sincronización y la nube por niveles

### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con la opción de implementación de Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso discrecional (DACL) de un descriptor de seguridad, si es mayor de 2 KB. (Este problema aplica únicamente si tiene más de unas 40 entradas de control de acceso [ACE] en un solo elemento).
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los archivos en niveles se volverán inaccesibles si no se recuperan antes de eliminar el punto de conexión de servidor. Para restaurar el acceso a los archivos, vuelva a crear el punto de conexión de servidor. Si han pasado 30 días desde que se eliminó el punto de conexión de servidor o si se ha eliminado el punto de conexión en la nube, los archivos en niveles que no se hayan recuperado serán inutilizables.
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos o suscripción diferentes dentro del inquilino de Azure AD existente. Si se mueve la cuenta de almacenamiento, debe dar acceso al servicio File Sync híbrido a la cuenta de almacenamiento (consulte el apartado [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Azure File Sync no permite mover la suscripción a un inquilino de Azure AD distinto.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.
- Al ver las propiedades de archivo desde un cliente de SMB, puede parecer que el atributo sin conexión se ha establecido incorrectamente debido a que SMB almacena en caché los metadatos del archivo.

## <a name="agent-version-5200"></a>Versión del agente 5.2.0.0
Las notas siguientes corresponden a la versión 5.2.0.0 del agente de Azure File Sync, publicada el 4 de abril de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 5.0.2.0.

Lista de problemas corregidos en esta versión:  
- Mejoras de confiabilidad de las características de transferencia de datos sin conexión y reanudación de transferencia de datos
- Mejoras en la telemetría de sincronización

## <a name="agent-version-5100"></a>Versión del agente 5.1.0.0
Las notas siguientes corresponden a la versión 5.1.0.0 del agente de Azure File Sync, publicada el 7 de marzo de 2019. Estas notas se suman a las notas de la versión enumeradas para la versión 5.0.2.0.

Lista de problemas corregidos en esta versión:  
- Los archivos pueden no sincronizarse con el error 0x80c8031d (ECS_E_CONCURRENCY_CHECK_FAILED) si la enumeración de cambios no tiene lugar en el servidor de archivos.
- Si una sesión de sincronización o un archivo reciben un error 0x80072f78 (WININET_E_INVALID_SERVER_RESPONSE), ahora la sincronización intentará realizar la operación de nuevo.
- Los archivos pueden no sincronizarse con el error 0x80c80203 (ECS_E_SYNC_INVALID_STAGED_FILE).
- Se puede consumir mucha memoria al recuperar archivos.
- Mejoras en la telemetría de la nube por niveles. 

## <a name="agent-version-5020"></a>Versión del agente 5.0.2.0
Las notas siguientes son para la versión 5.0.2.0 del agente de Azure File Sync (publicada el 12 de febrero de 2019).

### <a name="improvements-and-issues-that-are-fixed"></a>Mejoras y problemas que se han solucionado

- Compatibilidad con la nube de Azure Government
  - Hemos agregado compatibilidad de versión preliminar para la nube de Azure Government. Esto requiere una suscripción a la lista de permitidos y una descarga de agente especial de Microsoft. Para acceder a la versión preliminar, envíenos un correo electrónico directamente a [AzureFiles@microsoft.com](mailto:AzureFiles@microsoft.com).
- Compatibilidad con la desduplicación de datos
    - La desduplicación de datos ahora es totalmente compatible con la nube por niveles habilitada en Windows Server 2016 y Windows Server 2019. Habilitar la desduplicación en un volumen con la nube por niveles habilitada, le permite almacenar en caché más archivos en el entorno local sin necesidad de aprovisionar más almacenamiento.
- Compatibilidad con la transferencia de datos sin conexión (por ejemplo, a través de Data Box)
    - Migre fácilmente grandes cantidades de datos en Azure File Sync a través de cualquier medio que elija. Puede elegir Azure Data Box, AzCopy e incluso servicios de migración de terceros. No es necesario usar grandes cantidades de ancho de banda para trasladar los datos a Azure y, en el caso de Data Box, simplemente envíelos por correo electrónico. Para más información, consulte la [documentación sobre transferencia de datos sin conexión](https://aka.ms/AFS/OfflineDataTransfer).
- Rendimiento mejorado de la sincronización
    - Puede que los clientes con varios puntos de conexión de servidor en el mismo volumen hayan experimentado un rendimiento de sincronización lento antes de esta versión. Azure File Sync crea una instantánea VSS temporal una vez al día en el servidor para sincronizar archivos que tienen identificadores abiertos. Sync admite ya que varios puntos de conexión de servidor se sincronicen en un volumen si hay una sesión de sincronización de VSS activa. Ya no es necesario esperar a que una sesión de sincronización de VSS se termine para que pueda reanudarse la sincronización en otros puntos de conexión de servidor del volumen.
- Supervisión mejorada en el portal
    - Se han agregado gráficos en el portal del servicio de sincronización de almacenamiento para poder ver:
        - El número de archivos sincronizados
        - El tamaño de los datos transferidos
        - El número de archivos que no se están sincronizando
        - El tamaño de los datos que se recuperan
        - El estado de conectividad del servidor
    - Para más información, consulte [Supervisión de Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-sync-files-monitoring).
- Confiabilidad y escalabilidad mejoradas
    - El número máximo de objetos del sistema de archivos (archivos y directorios) en un directorio se ha aumentado a 1 000 000. El límite anterior era de 200 000.
    - La sincronización intentará reanudar la transferencia de datos en lugar de volver a transmitirlos si una transferencia se interrumpe debido a archivos de gran tamaño 

### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Para la instalación y las instrucciones de uso, consulte la sección [Herramienta de evaluación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#evaluation-cmdlet) en la Guía de planeación. 

### <a name="agent-installation-and-server-configuration"></a>Instalación del agente y configuración del servidor
Para más información sobre cómo instalar y configurar el agente de Azure File Sync con Windows Server, vea [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) y [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

- El paquete de instalación del agente debe instalarse con permisos elevados (administrador).
- El agente no es compatible con las opciones de implementación de Windows Server Core ni Nano Server.
- El agente solo se admite en Windows Server 2019, Windows Server 2016 y Windows Server 2012 R2.
- El agente requiere al menos 2 GB de memoria. Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
- El servicio del agente de sincronización de Storage (FileSyncSvc) no admite puntos de conexión de servidor ubicados en un volumen que tiene el directorio de información del volumen del sistema (SVI) comprimido. Esta configuración producirá resultados inesperados.
- El modo FIPS no se admite y debe deshabilitarse. 

### <a name="interoperability"></a>Interoperabilidad
- Las aplicaciones antivirus, de copia de seguridad y de otro tipo que accedan a archivos organizados en niveles pueden provocar recuperaciones no deseadas, salvo que respeten el atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Para más información, vea [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md).
- Los filtros de archivos del Administrador de recursos del servidor de archivos (FSRM) pueden provocar un gran número de errores de sincronización cuando los archivos están bloqueados debido al filtro de archivos.
- No se admite la ejecución de sysprep en un servidor que tenga instalado el agente de Azure File Sync y puede provocar resultados inesperados. El agente de Azure File Sync se debe instalar después de implementar la imagen del servidor y completar la instalación mínima de la herramienta de preparación del sistema.

### <a name="sync-limitations"></a>Limitaciones de la sincronización
Los siguientes elementos no se sincronizan, pero el resto del sistema funciona con normalidad:
- Archivos con caracteres no admitidos. Consulte la [Guía de solución de problemas](storage-sync-files-troubleshoot.md#handling-unsupported-characters) para obtener la lista de caracteres no admitidos.
- Archivos o directorios que terminan con un punto.
- Rutas de acceso de más de 2,048 caracteres.
- La parte de la lista de control de acceso discrecional (DACL) de un descriptor de seguridad, si es mayor de 2 KB. (Este problema aplica únicamente si tiene más de unas 40 entradas de control de acceso [ACE] en un solo elemento).
- La parte de la lista de control de acceso del sistema (SACL) de un descriptor de seguridad que se utiliza para la auditoría.
- Atributos extendidos.
- Flujos de datos alternativos.
- Puntos de repetición de análisis.
- Vínculos físicos.
- La compresión (si se establece en un archivo de servidor) no se conserva al sincronizarse los cambios con ese archivo desde otros puntos de conexión.
- Los archivos cifrados con EFS (u otros cifrados con modo de usuario) que evitan que el servicio lea los datos.

    > [!Note]  
    > Azure File Sync siempre cifra los datos en tránsito. Los datos siempre se cifran en reposo en Azure.
 
### <a name="server-endpoint"></a>Punto de conexión de servidor
- Un punto de conexión de servidor solo se puede crear en un volumen NTFS. En este momento Azure File Sync no admite ReFS, FAT, FAT32 ni otros sistemas de archivos.
- Los archivos en niveles se volverán inaccesibles si no se recuperan antes de eliminar el punto de conexión de servidor. Para restaurar el acceso a los archivos, vuelva a crear el punto de conexión de servidor. Si han pasado 30 días desde que se eliminó el punto de conexión de servidor o si se ha eliminado el punto de conexión en la nube, los archivos en niveles que no se hayan recuperado serán inutilizables.
- Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.
- Los clústeres de conmutación por error solo son compatibles con discos en clúster, pero no con volúmenes compartidos de clúster (CSV).
- No se puede anidar un punto de conexión de servidor. Puede coexistir en el mismo volumen en paralelo con otro punto de conexión.
- No almacene un archivo de paginación de aplicación o sistema operativo en una ubicación de punto de conexión de servidor.
- Si se cambia el nombre del servidor, no se actualiza en el portal.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
- Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión de nube una vez cada 24 horas. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización.
- El servicio de sincronización del almacenamiento o la cuenta de almacenamiento se pueden mover a un grupo de recursos o suscripción diferentes dentro del inquilino de Azure AD existente. Si se mueve la cuenta de almacenamiento, debe dar acceso al servicio File Sync híbrido a la cuenta de almacenamiento (consulte el apartado [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac)).

    > [!Note]  
    > Azure File Sync no permite mover la suscripción a un inquilino de Azure AD distinto.

### <a name="cloud-tiering"></a>Niveles de nube
- Si un archivo en niveles se copia en otra ubicación mediante el uso de Robocopy, el archivo resultante no estará en niveles. El atributo sin conexión podría estar establecido porque Robocopy incluye incorrectamente dicho atributo en las operaciones de copia.
- Al copiar los archivos mediante Robocopy, utilice la opción /MIR para conservar las marcas de tiempo de archivo. Esto garantizará que los archivos antiguos se organicen por niveles antes que los archivos a los que se ha accedido recientemente.
- Al ver las propiedades de archivo desde un cliente de SMB, puede parecer que el atributo sin conexión se ha establecido incorrectamente debido a que SMB almacena en caché los metadatos del archivo.
