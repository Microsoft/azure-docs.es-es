---
title: Solución de problemas de Azure File Sync | Microsoft Docs
description: Solución de problemas comunes con Azure File Sync.
services: storage
author: jeffpatt24
ms.service: storage
ms.topic: article
ms.date: 01/31/2019
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: 26055727e308f8c05aece31746434d7e9a0a5abd
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65555942"
---
# <a name="troubleshoot-azure-file-sync"></a>Solucionar problemas de Azure File Sync
Use Azure File Sync para centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma Windows Server en una caché rápida de los recursos compartidos de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.

Este artículo está diseñado para ayudarle a solucionar problemas que podrían producirse con la implementación de Azure File Sync. Se describe cómo recopilar registros importantes del sistema si es necesario investigar los problemas en mayor profundidad. Si no encuentra una respuesta a su pregunta, póngase en contacto con nosotros mediante los siguientes canales (en orden incremental):

1. [Foro de Azure Storage](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata).
2. [UserVoice de Azure Files](https://feedback.azure.com/forums/217298-storage/category/180670-files).
3. Soporte técnico de Microsoft. Para crear una solicitud de soporte técnico, en Azure Portal, vaya a la pestaña **Ayuda**, seleccione el botón **Ayuda y soporte técnico** y elija **Nueva solicitud de soporte técnico**.

## <a name="im-having-an-issue-with-azure-file-sync-on-my-server-sync-cloud-tiering-etc-should-i-remove-and-recreate-my-server-endpoint"></a>Tengo un problema con Azure File Sync en mi servidor (sincronización, niveles en la nube, etc.). ¿Debería quitar y volver a crear el punto de conexión del servidor?
[!INCLUDE [storage-sync-files-remove-server-endpoint](../../../includes/storage-sync-files-remove-server-endpoint.md)]

## <a name="agent-installation-and-server-registration"></a>Instalación del agente y registro del servidor
<a id="agent-installation-failures"></a>**Solución de problemas de errores de instalación del agente**  
Si la instalación del agente de Azure File Sync no se produce correctamente, ejecute el siguiente comando desde un símbolo del sistema con privilegios elevados para activar el registro durante la instalación del agente:

```
StorageSyncAgent.msi /l*v AFSInstaller.log
```

Si la instalación produce error, revise el archivo installer.log para determinar la causa.

<a id="agent-installation-on-DC"></a>**Se produce un error al instalar el agente en el controlador de dominio de Active Directory**  
Si intenta instalar el agente de sincronización en un controlador de dominio de Active Directory donde el propietario del rol PDC está en un sistema operativo Windows Server 2008 R2 o inferior, puede tener un problema por el que el agente de sincronización no se pueda instalar.

Para resolverlo, transfiera el rol PDC a otro controlador de dominio que ejecute Windows Server 2012 R2 o una versión más reciente y, luego, instale el agente de sincronización.

<a id="server-registration-missing"></a>**El servidor no aparece en los servidores registrados de Azure Portal**  
Si un servidor no aparece en los **servidores registrados** de un servicio de sincronización de almacenamiento:
1. Inicie sesión en el servidor que desea registrar.
2. Abra el Explorador de archivos y, a continuación, vaya al directorio de instalación del agente de sincronización de almacenamiento (la ubicación predeterminada es C:\Program Files\Azure\StorageSyncAgent). 
3. Ejecute ServerRegistration.exe y complete el asistente para registrar el servidor en un servicio de sincronización de almacenamiento.

<a id="server-already-registered"></a>**El registro de servidor muestra un mensaje durante la instalación del agente de Azure File Sync: "El servidor ya está registrado"** 

![Captura de pantalla del cuadro de diálogo Registro de servidor con el mensaje que indica que el servidor ya está registrado](media/storage-sync-files-troubleshoot/server-registration-1.png)

Este mensaje aparece si el servidor se registró anteriormente en un servicio de sincronización de almacenamiento. Para anular el registro del servidor del servicio de sincronización de almacenamiento actual y registrarlo en un servicio de sincronización de almacenamiento nuevo, complete los pasos que se describen en [Anular el registro de un servidor de Azure File Sync](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service).

Si el servidor no aparece en la lista de **servidores registrados** en el servicio de sincronización de almacenamiento, ejecute los siguientes comandos de PowerShell en el servidor cuyo registro desea eliminar:

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Reset-StorageSyncServer
```

> [!Note]  
> Si el servidor forma parte de un clúster, puede utilizar el parámetro opcional *Reset-StorageSyncServer -CleanClusterRegistration* para quitar también el registro del clúster.

<a id="web-site-not-trusted"></a>**Al registrar un servidor, obtengo numerosas respuestas de tipo "el sitio web no es de confianza". ¿Por qué?**  
Este problema se produce cuando la directiva **Seguridad mejorada de Internet Explorer** está habilitada durante el registro de servidor. Para más información sobre cómo deshabilitar de manera apropiada la directiva **Seguridad mejorada de Internet Explorer**, consulte la sección [Preparación de los servidores de Windows Server para su uso con Azure File Sync](storage-sync-files-deployment-guide.md#prepare-windows-server-to-use-with-azure-file-sync) del artículo [Cómo implementar Azure File Sync](storage-sync-files-deployment-guide.md).

## <a name="sync-group-management"></a>Administración de grupos de sincronización
<a id="cloud-endpoint-using-share"></a>**Falla la creación de puntos de conexión de nube, con este error: "El recurso compartido de archivos de Azure especificado ya está siendo usado por un punto de conexión de nube diferente"**  
Este error se produce si el recurso compartido de archivos de Azure ya está siendo utilizado por otro punto de conexión de nube. 

Si ve este mensaje y el recurso compartido de archivos de Azure no está siendo utilizado actualmente por un punto de conexión de nube, complete los siguientes pasos para borrar los metadatos de Azure File Sync en el recurso compartido de archivos de Azure:

> [!Warning]  
> Si se eliminan los metadatos en un recurso compartido de archivos de Azure que está siendo utilizado por un punto de conexión de nube, las operaciones de Azure File Sync producirán un error. 

1. Vaya al recurso compartido de archivos de Azure en Azure Portal.  
2. Haga clic con el botón derecho en el recurso compartido de archivos de Azure y seleccione **Editar metadatos**.
3. Haga clic con el botón derecho en **SyncService** y, a continuación, seleccione **Eliminar**.

<a id="cloud-endpoint-authfailed"></a>**Falla la creación de puntos de conexión de nube, con este error: "AuthorizationFailed"**  
Este problema se produce si la cuenta de usuario no tiene derechos suficientes para crear un punto de conexión de nube. 

Para crear un punto de conexión de nube, la cuenta de usuario debe tener los siguientes permisos de autorización de Microsoft:  
* Lectura: Obtener definición de rol
* Escritura: Crear o actualizar una definición de rol personalizada
* Lectura: Obtener asignación de roles
* Escritura: Crear asignación de roles

Los siguientes roles integrados tienen los permisos necesarios de autorización de Microsoft:  
* Propietario
* Administrador de acceso de usuario

Para determinar si su rol de la cuenta de usuario tiene los permisos necesarios:  
1. En Azure Portal, seleccione **Grupos de recursos**.
2. Seleccione el grupo de recursos donde se encuentra la cuenta de almacenamiento y seleccione en **Control de acceso (IAM)**.
3. Seleccione la pestaña **Asignaciones de roles**.
4. Seleccione el **rol** (por ejemplo, propietario o colaborador) para su cuenta de usuario.
5. En la lista **Proveedor de recursos**, seleccione **Autorización de Microsoft**. 
    * **Asignación de roles** debe tener permisos de **lectura** y **escritura**.
    * **Definición de roles** debe tener permisos de **lectura** y **escritura**.

<a id="server-endpoint-createjobfailed"></a>**Falla la creación de puntos de conexión de servidor, con este error: "MgmtServerJobFailed" (código de error: -2134375898)**  
Este problema se produce si la ruta de acceso del punto de conexión de servidor se encuentra en el volumen del sistema y los niveles de la nube están habilitados. Los niveles de nube no se admiten en el volumen del sistema. Para crear un punto de conexión de servidor en el volumen del sistema, deshabilite los niveles de la nube al crear el punto de conexión de servidor.

<a id="server-endpoint-deletejobexpired"></a>**Falla la eliminación de puntos de conexión de servidor, con este error: "MgmtServerJobExpired"**                
Este problema se produce si el servidor está sin conexión o no tiene conectividad de red. Si el servidor ya no está disponible, anule el registro del servidor en el portal, lo que eliminará los puntos de conexión de servidor. Para eliminar los puntos de conexión de servidor, siga los pasos que se describen en [Anular el registro de un servidor de Azure File Sync](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service).

<a id="server-endpoint-provisioningfailed"></a>**No se puede abrir la página de propiedades de puntos de conexión del servidor o actualizar la directiva de niveles en la nube**  
Este problema puede producirse si hay un error en una operación de administración en el punto de conexión del servidor. Si la página de propiedades de puntos de conexión del servidor no se abre en Azure Portal, actualizar el punto de conexión del servidor con comandos de PowerShell desde el servidor puede solucionar este problema. 

```powershell
# Get the server endpoint id based on the server endpoint DisplayName property
Get-AzStorageSyncServerEndpoint `
    -ResourceGroupName myrgname `
    -StorageSyncServiceName storagesvcname `
    -SyncGroupName mysyncgroup | `
Tee-Object -Variable serverEndpoint

# Update the free space percent policy for the server endpoint
Set-AzStorageSyncServerEndpoint `
    -InputObject $serverEndpoint
    -CloudTiering `
    -VolumeFreeSpacePercent 60
```
<a id="server-endpoint-noactivity"></a>**El estado del punto de conexión del servidor es "Sin actividad" o "Pendiente" y el estado del servidor en la hoja de servidores registrados es "Aparece sin conexión"**  

Este problema puede producirse si no se está ejecutando el proceso del monitor de sincronización de almacenamiento o el servidor no puede comunicarse con el servicio de Azure File Sync debido a un servidor proxy o firewall.

Para resolver este problema, siga estos pasos:

1. Abra el administrador de tareas en el servidor y compruebe que se está ejecutando el proceso del monitor de sincronización de almacenamiento (AzureStorageSyncMonitor.exe). Si no se está ejecutando el proceso, primero pruebe a reiniciar el servidor. Si no se resuelve el problema al reiniciar el servidor, actualice a la [versión del agente](https://docs.microsoft.com/azure/storage/files/storage-files-release-notes) de Azure File Sync más reciente.
2. Compruebe que la configuración del firewall y del proxy está configurada correctamente:
    - Si el servidor está detrás de un firewall, compruebe que se permite el puerto 443 de salida. Si el firewall restringe el tráfico a dominios concretos, confirme que se puede acceder a los dominios enumerados en la [documentación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-firewall-and-proxy#firewall) del firewall.
    - Si el servidor está detrás de un proxy, configure los valores del proxy aplicables a toda la máquina o específicos de la aplicación mediante los pasos que se describen en la [documentación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-firewall-and-proxy#proxy) del proxy.

<a id="endpoint-noactivity-sync"></a>**El estado de mantenimiento del punto de conexión del servidor es "Sin actividad" y el estado del servidor en la hoja de servidores registrados es "En línea"**  

Si el estado de mantenimiento del punto de conexión del servidor es "Sin actividad", esto significa que el punto de conexión del servidor no ha registrado la actividad de sincronización en las últimas dos horas.

Es posible que un punto de conexión del servidor no registre la actividad de sincronización debido a los siguientes motivos:

- La versión del agente 4.3.0.0 o anterior está instalada y el servidor tiene una sesión de sincronización VSS (SnapshotSync) activa. Cuando una sesión de sincronización de VSS está activa en un punto de conexión del servidor, otros puntos de conexión del servidor en el mismo volumen no pueden iniciar ninguna sesión de sincronización hasta que finalice la sesión de sincronización de VSS anterior. Para resolver este problema, instale la versión del agente 5.0.2.0 o una versión más reciente que admita la sincronización de varios puntos de conexión de servidor en un volumen cuando la sesión de sincronización de VSS está activa.

    Para comprobar la actividad de sincronización actual de un servidor, consulte [¿Cómo se puede supervisar el progreso de una sesión de sincronización actual?](#how-do-i-monitor-the-progress-of-a-current-sync-session).

- El servidor alcanzó el número máximo de sesiones de sincronización simultáneas. 
    - Versión del agente 4.x y posterior: el límite varía en función de los recursos del sistema disponibles.
    - Versión del agente 3.x: dos sesiones sincronizadas activas por procesador o un máximo de ocho sesiones sincronizadas activas por servidor.

> [!Note]  
> Si el estado del servidor en la hoja de servidores registrados es "Aparece sin conexión", realice los pasos detallados en la sección [El estado del punto de conexión del servidor es "Sin actividad" o "Pendiente" y el estado del servidor en la hoja de servidores registrados es "Aparece sin conexión"](#server-endpoint-noactivity).

## <a name="sync"></a>Sync
<a id="afs-change-detection"></a>**Si creé un archivo directamente en el recurso compartido de archivos de Azure mediante SMB o el portal, ¿cuánto tiempo tarda el archivo en sincronizarse con los servidores del grupo de sincronización?**  
[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="serverendpoint-pending"></a>**El estado del punto de conexión del servidor está en estado pendiente durante varias horas**  
Este problema puede producirse si crea un punto de conexión de nube y usa un recurso compartido de archivos de Azure que contiene los datos. El trabajo de enumeración de cambios que analiza en busca de cambios en el recurso compartido de archivos de Azure debe completarse antes de que los archivos puedan sincronizarse entre la nube y los puntos de conexión del servidor. El tiempo para completar el trabajo depende del tamaño del espacio de nombres en el recurso compartido de archivos de Azure. El estado del punto de conexión del servidor debe actualizarse una vez que se complete el trabajo de enumeración de cambios.

### <a id="broken-sync"></a>¿Cómo superviso el estado de sincronización?
# <a name="portaltabportal1"></a>[Portal](#tab/portal1)
Dentro de cada grupo de sincronización, puede explorar en profundidad los puntos de conexión individuales del servidor para ver el estado de las últimas sesiones de sincronización completadas. Una columna verde de estado y un valor 0 en archivos que no se sincronizan indican que la sincronización está funcionando como se esperaba. Si este no es el caso, consulte a continuación una lista de errores comunes de sincronización y cómo tratar los archivos que no se están sincronizando. 

![Captura de pantalla de Azure Portal](media/storage-sync-files-troubleshoot/portal-sync-health.png)

# <a name="servertabserver"></a>[Servidor](#tab/server)
Vaya a registros de telemetría del servidor, que pueden encontrarse en el Visor de eventos en `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry`. El evento 9102 corresponde a una sesión de sincronización completada; para el último estado de sincronización, busque el evento más reciente con identificador 9102. SyncDirection indica si esta sesión fue una carga o descarga. Si HResult es 0, la sesión de sincronización se realizó correctamente. Un valor de HResult distinto de cero significa que se produjo un error durante la sincronización. Consulte a continuación para obtener una lista de errores comunes. Si PerItemErrorCount es mayor que 0, significa que algunos archivos o carpetas no se sincronizaron correctamente. Es posible tener un valor de HResult de 0 pero un valor de PerItemErrorCount mayor que 0.

Este es un ejemplo de carga correcta. Por brevedad, solo se enumeran a continuación algunos de los valores contenidos en cada evento 9102. 

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: 0, 
SyncFileCount: 2, SyncDirectoryCount: 0,
AppliedFileCount: 2, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0,
TransferredFiles: 2, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

Por el contrario, una carga incorrecta podría parecerse a esto:

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: -2134364065,
SyncFileCount: 0, SyncDirectoryCount: 0, 
AppliedFileCount: 0, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0, 
TransferredFiles: 0, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

A veces, las sesiones de sincronización pueden producir un error general o tienen un valor de PerItemErrorCount distinto de cero, pero aún así progresan y algunos archivos se sincronizan correctamente. Esto puede verse en los campos Applied* (AppliedFileCount, AppliedDirCount, AppliedTombstoneCount y AppliedSizeBytes), que indican qué parte de la sesión finalizará correctamente. Si ve varias sesiones de sincronización seguidas que tienen un error pero que tienen un número cada vez mayor de campos Applied*, debe dar tiempo para que se vuelva a intentar la sincronización antes de abrir una incidencia de soporte técnico.

---

### <a name="how-do-i-monitor-the-progress-of-a-current-sync-session"></a>¿Cómo se puede supervisar el progreso de una sesión de sincronización actual?
# <a name="portaltabportal1"></a>[Portal](#tab/portal1)
Dentro de su grupo de sincronización, vaya al punto de conexión del servidor en cuestión y consulte la sección Actividad de sincronización para ver el número de archivos cargados o descargados en la sesión de sincronización actual. Tenga en cuenta que este estado se retrasará unos 5 minutos, y si la sesión de sincronización es lo suficientemente pequeña como para completarse dentro de este período, es posible que no se notifique de ello en el portal. 

# <a name="servertabserver"></a>[Servidor](#tab/server)
Busque el evento 9302 más reciente en el registro de telemetría en el servidor (en el Visor de eventos, vaya a Registros de aplicaciones y servicios\Microsoft\FileSync\Agent\Telemetry). Este evento indica el estado de la sesión de sincronización actual. TotalItemCount indica cuántos archivos se van a sincronizar, AppliedItemCount indica el número de archivos que se han sincronizado hasta ahora y PerItemErrorCount indica el número de archivos que no se han sincronizado (consulte más abajo cómo tratar esto).

```
Replica Sync Progress. 
ServerEndpointName: <CI>sename</CI>, SyncGroupName: <CI>sgname</CI>, ReplicaName: <CI>rname</CI>, 
SyncDirection: Upload, CorrelationId: {AB4BA07D-5B5C-461D-AAE6-4ED724762B65}. 
AppliedItemCount: 172473, TotalItemCount: 624196. AppliedBytes: 51473711577, 
TotalBytes: 293363829906. 
AreTotalCountsFinal: true. 
PerItemErrorCount: 1006.
```
---

### <a name="how-do-i-know-if-my-servers-are-in-sync-with-each-other"></a>¿Cómo se puede saber si mis servidores están sincronizados entre sí?
# <a name="portaltabportal1"></a>[Portal](#tab/portal1)
Para cada servidor de un determinado grupo de sincronización, asegúrese de que:
- Las marcas de tiempo para el último intento de sincronización, tanto para la carga como para la descarga, son recientes.
- El estado está en color verde para la carga y descarga.
- El campo Actividad de sincronización muestra muy pocos o ningún archivo que quede por sincronizar.
- El campo Archivos que no se están sincronizando es 0 tanto para la carga como para la descarga.

# <a name="servertabserver"></a>[Servidor](#tab/server)
Examine las sesiones de sincronización completadas, que están marcadas por los eventos 9102 en el registro de eventos de telemetría para cada servidor (en el Visor de eventos, vaya a `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry`). 

1. En un servidor determinado, desea asegurarse de que las últimas sesiones de carga y descarga se hayan completado correctamente. Para ello, compruebe que los valores de HResult y PerItemErrorCount son 0 tanto para la carga como para la descarga (el campo SyncDirection indica si una sesión determinada es una sesión de carga o descarga). Tenga en cuenta que si no ve una sesión de sincronización completada recientemente, es probable que esté en curso una sesión de sincronización, lo que es de esperar si acaba de agregar o modificar una gran cantidad de datos.
2. Cuando un servidor está completamente actualizado con la nube y no tiene cambios para sincronizar en ninguna dirección, verá sesiones de sincronización vacías. Estos se indican mediante eventos de carga y descarga en los que todos los campos Sync* (SyncFileCount, SyncDirCount, SyncTombstoneCount y SyncSizeBytes) son cero, lo que significa que no había nada que sincronizar. Tenga en cuenta que es posible que estas sesiones de sincronización vacías no se produzcan en servidores de alta velocidad, ya que siempre hay algo nuevo que sincronizar. Si no hay ninguna actividad de sincronización, deben ocurrir cada 30 minutos. 
3. Si todos los servidores están actualizados con la nube, lo que significa que sus sesiones de carga y descarga recientes son sesiones de sincronización vacías, puede decir con razonable certeza que el sistema en su conjunto está sincronizado. 
    
Tenga en cuenta que si ha realizado cambios directamente en el recurso compartido de archivos de Azure, Azure File Sync no detectará este cambio hasta que se ejecute la enumeración de cambios, lo que ocurre una vez cada 24 horas. Es posible que un servidor diga que está actualizado con la nube cuando de hecho le faltan los cambios recientes realizados directamente en el recurso compartido de archivos de Azure. 

---

### <a name="how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing"></a>¿Cómo puedo ver si hay archivos o carpetas específicos que no se están sincronizando?
Si el valor de PerItemErrorCount en el servidor o el recuento de Archivos que no se están sincronizando en el portal es superior a 0 para una sesión de sincronización determinada, significa que algunos elementos no se sincronizan. Los archivos y las carpetas pueden tener características que impiden su sincronización. Estas características pueden ser persistentes y requieren una acción explícita para reanudar la sincronización, por ejemplo, quitar los caracteres no admitidos del nombre de archivo o carpeta. También pueden ser transitorias, lo que significa que el archivo o la carpeta reanudarán automáticamente la sincronización; por ejemplo, archivos con identificadores abiertos reanudarán automáticamente la sincronización cuando se cierre el archivo. Cuando el motor Azure File Sync detecta tal problema, se produce un registro de errores que se puede analizar para enumerar los elementos que no se están sincronizando correctamente.

Para ver estos errores, ejecute el script de PowerShell **FileSyncErrorsReport.ps1** (ubicado en el directorio de instalación del agente de Azure File Sync) para identificar los archivos que no se sincronizaron debido a identificadores abiertos, caracteres no admitidos u otros problemas. El campo ItemPath indica la ubicación del archivo en relación con el directorio de sincronización raíz. Consulte la lista de errores de sincronización más abajo para conocer los pasos de corrección.

#### <a name="troubleshooting-per-filedirectory-sync-errors"></a>Solución de errores de sincronización de archivo o directorio
**Registro de ItemResults: errores de sincronización por elemento**  

| HRESULT | HRESULT (decimal) | Cadena de error | Problema | Corrección |
|---------|-------------------|--------------|-------|-------------|
| 0x80c80207 | -2134375929 | ECS_E_SYNC_CONSTRAINT_CONFLICT | Un cambio de archivo o de directorio no se puede sincronizar todavía porque una carpeta dependiente aún no se ha sincronizado. Este elemento se sincronizará después de sincronizar los cambios dependientes. | No es necesaria ninguna acción. |
| 0x7b | 123 | ERROR_INVALID_NAME | El nombre de directorio o archivo no es válido. | Cambie el nombre del archivo o directorio en cuestión. Consulte [Tratamiento de caracteres no admitidos](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#handling-unsupported-characters) para más información. |
| 0x8007007b | -2147024773 | STIERR_INVALID_DEVICE_NAME | El nombre de directorio o archivo no es válido. | Cambie el nombre del archivo o directorio en cuestión. Consulte [Tratamiento de caracteres no admitidos](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#handling-unsupported-characters) para más información. |
| 0x80c80018 | -2134376424 | ECS_E_SYNC_FILE_IN_USE | No se puede sincronizar un archivo porque está en uso. El archivo se sincronizará cuando ya no esté en uso. | No es necesaria ninguna acción. Azure File Sync crea una instantánea VSS temporal una vez al día en el servidor para sincronizar archivos que tienen identificadores abiertos. |
| 0x80c8031d | -2134375651 | ECS_E_CONCURRENCY_CHECK_FAILED | Un archivo ha cambiado, pero el cambio aún no lo ha detectado la sincronización. La sincronización se recuperará después de detectar este cambio. | No es necesaria ninguna acción. |
| 0x80c8603e | -2134351810 | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED | El archivo no se puede sincronizar porque se ha alcanzado el límite de recursos compartidos de archivos de Azure. | Para resolver este problema, consulte la sección [Se ha alcanzado el límite de almacenamiento del recurso compartido de archivos de Azure](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#-2134351810) en la Guía de solución de problemas. |
| 0x80070005 | -2147024891 | E_ACCESSDENIED | Este error puede producirse por las razones siguientes: archivo está cifrado con una solución no admitida (por ejemplo, EFS de NTFS), archivo tiene una eliminación pendiente de estado o el archivo se encuentra en una carpeta de replicación de solo lectura DFS-R | Si el archivo está cifrado con una solución no admitida, descifrar el archivo y use una solución de cifrado admitidos. Para obtener una lista de soluciones de soporte técnico, consulte la sección [Soluciones de cifrado](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#encryption-solutions) en la Guía de planeamiento. Si el archivo se encuentra en un estado de eliminación pendiente, el archivo se eliminará una vez que se hayan cerrado todos identificadores de archivos abiertos. Si el archivo se encuentra en una carpeta de replicación de solo lectura DFS-R, Azure File Sync no admite los puntos de conexión de servidor en las carpetas de solo lectura de la replicación DFS-R. Consulte [Guía de planificación](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#distributed-file-system-dfs) para obtener más información.
| 0x20 | 32 | ERROR_SHARING_VIOLATION | No se puede sincronizar un archivo porque está en uso. El archivo se sincronizará cuando ya no esté en uso. | No es necesaria ninguna acción. |
| 0x80c80017 | -2134376425 | ECS_E_SYNC_OPLOCK_BROKEN | Se modificó un archivo durante la sincronización, por lo que debe sincronizarse de nuevo. | No es necesaria ninguna acción. |

#### <a name="handling-unsupported-characters"></a>Tratamiento de caracteres no admitidos
Si el script de PowerShell **FileSyncErrorsReport.ps1** muestra errores debido a caracteres no admitidos (códigos de error 0x7b y 0x8007007b), debe quitar los caracteres erróneos de los nombres de los archivos afectados o cambiarles el nombre. PowerShell probablemente imprimirá estos caracteres como signos de interrogación o rectángulos vacíos, ya que la mayoría de estos caracteres no tienen codificación visual estándar. Puede usar la [herramienta de evaluación](storage-sync-files-planning.md#evaluation-tool) para identificar los caracteres que no son compatibles.

La siguiente tabla contiene todos los caracteres Unicode que Azure File Sync aún no admite.

| Juego de caracteres | Número de caracteres |
|---------------|-----------------|
| <ul><li>0x0000009D ( cosc operating system command)</li><li>0x00000090 (dcs device control string)</li><li>0x0000008F (ss3 single shift three)</li><li>0x00000081 (high octet preset)</li><li>0x0000007F (del delete)</li><li>0x0000008D (ri reverse line feed)</li></ul> | 6 |
| 0x0000FDD0 - 0x0000FDEF (Arabic presentation forms-a) | 32 |
| 0x0000FFF0 - 0x0000FFFF (specials) | 16 |
| <ul><li>0x0001FFFE - 0x0001FFFF = 2 (noncharacter)</li><li>0x0002FFFE - 0x0002FFFF = 2 (noncharacter)</li><li>0x0003FFFE - 0x0003FFFF = 2 (noncharacter)</li><li>0x0004FFFE - 0x0004FFFF = 2 (noncharacter)</li><li>0x0005FFFE - 0x0005FFFF = 2 (noncharacter)</li><li>0x0006FFFE - 0x0006FFFF = 2 (noncharacter)</li><li>0x0007FFFE - 0x0007FFFF = 2 (noncharacter)</li><li>0x0008FFFE - 0x0008FFFF = 2 (noncharacter)</li><li>0x0009FFFE - 0x0009FFFF = 2 (noncharacter)</li><li>0x000AFFFE - 0x000AFFFF = 2 (noncharacter)</li><li>0x000BFFFE - 0x000BFFFF = 2 (noncharacter)</li><li>0x000CFFFE - 0x000CFFFF = 2 (noncharacter)</li><li>0x000DFFFE - 0x000DFFFF = 2 (noncharacter)</li><li>0x000EFFFE - 0x000EFFFF = 2 (undefined)</li><li>0x000FFFFE - 0x000FFFFF = 2 (supplementary private use area)</li></ul> | 30 |
| 0x0010FFFE, 0x0010FFFF | 2 |

### <a name="common-sync-errors"></a>Errores de sincronización comunes
<a id="-2147023673"></a>**Se canceló la sesión de sincronización.**  

| | |
|-|-|
| **HRESULT** | 0x800704c7 |
| **HRESULT (decimal)** | -2147023673 | 
| **Cadena de error** | ERROR_CANCELLED |
| **Se requiere una corrección** | No |

Las sesiones de sincronización pueden producir un error por varias razones, incluido el reinicio o actualización del servidor o instantáneas de VSS, entre otras. Aunque este error parece que requiere seguimiento, es seguro pasar por alto este error a menos que persista durante un período de varias horas.

<a id="-2147012889"></a>**No se pudo establecer ninguna conexión con el servicio.**    

| | |
|-|-|
| **HRESULT** | 0x80072EE7 |
| **HRESULT (decimal)** | -2147012889 | 
| **Cadena de error** | WININET_E_NAME_NOT_RESOLVED |
| **Se requiere una corrección** | Sí |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134376372"></a>**La solicitud del usuario fue limitada por el servicio.**  

| | |
|-|-|
| **HRESULT** | 0x80c8004c |
| **HRESULT (decimal)** | -2134376372 |
| **Cadena de error** | ECS_E_USER_REQUEST_THROTTLED |
| **Se requiere una corrección** | No |

No se requiere ninguna acción; el servidor lo intentará de nuevo. Si este error persiste durante más de un par de horas, cree una solicitud de soporte técnico.

<a id="-2134364043"></a>**Sincronización está bloqueada hasta que la detección de cambios complete después de la restauración**  

| | |
|-|-|
| **HRESULT** | 0x80c83075 |
| **HRESULT (decimal)** | -2134364043 |
| **Cadena de error** | ECS_E_SYNC_BLOCKED_ON_CHANGE_DETECTION_POST_RESTORE |
| **Se requiere una corrección** | No |

No es necesaria ninguna acción. Cuando comparte un archivo o se restaura (punto de conexión en la nube) mediante Azure Backup, sincronización está bloqueada hasta que se complete la detección de cambios en el recurso compartido de archivos de Azure. Detección de cambios se ejecuta inmediatamente una vez completada la restauración y la duración se basa en el número de archivos en el recurso compartido de archivos.

<a id="-2134364065"></a>**La sincronización no puede acceder al recurso compartido de archivos de Azure especificado en el punto de conexión de nube.**  

| | |
|-|-|
| **HRESULT** | 0x80c8305f |
| **HRESULT (decimal)** | -2134364065 |
| **Cadena de error** | ECS_E_CANNOT_ACCESS_EXTERNAL_STORAGE_ACCOUNT |
| **Se requiere una corrección** | Sí |

Este error se produce porque el agente de Azure File Sync no puede acceder al recurso compartido de archivos de Azure, lo que puede deberse a que el recurso compartido de archivos de Azure o la cuenta de almacenamiento que lo hospeda ya no existe. Puede solucionar este error siguiendo estos pasos:

1. [Compruebe que la cuenta de almacenamiento existe.](#troubleshoot-storage-account)
2. [Compruebe que la cuenta de almacenamiento no contiene ninguna regla de red.](#troubleshoot-network-rules)
3. [Asegúrese de que el recurso compartido de archivos de Azure existe.](#troubleshoot-azure-file-share)
4. [Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento.](#troubleshoot-rbac)

<a id="-2134364064"></a><a id="cannot-resolve-storage"></a>**No se pudo resolver el nombre de cuenta de almacenamiento usado.**  

| | |
|-|-|
| **HRESULT** | 0x80C83060 |
| **HRESULT (decimal)** | -2134364064 |
| **Cadena de error** | ECS_E_STORAGE_ACCOUNT_NAME_UNRESOLVED |
| **Se requiere una corrección** | Sí |

1. Compruebe que puede resolver el nombre DNS de almacenamiento desde el servidor.

    ```powershell
    Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 443
    ```
2. [Compruebe que la cuenta de almacenamiento existe.](#troubleshoot-storage-account)
3. [Compruebe que la cuenta de almacenamiento no contiene ninguna regla de red.](#troubleshoot-network-rules)

<a id="-1906441138"></a>**Error de sincronización debido a un problema con la base de datos de sincronización.**  

| | |
|-|-|
| **HRESULT** | 0x8e5e044e |
| **HRESULT (decimal)** | -1906441138 |
| **Cadena de error** | JET_errWriteConflict |
| **Se requiere una corrección** | Sí |

Este error se produce cuando hay un problema con la base de datos interna utilizada por Azure File Sync. Cuando este problema ocurre, cree una solicitud de soporte técnico y nos pondremos en contacto con usted para ayudarle a resolver este problema.

<a id="-2134364053"></a>**No se admite la versión del agente de Azure File Sync instalada en el servidor.**  

| | |
|-|-|
| **HRESULT** | 0x80C8306B |
| **HRESULT (decimal)** | -2134364053 |
| **Cadena de error** | ECS_E_AGENT_VERSION_BLOCKED |
| **Se requiere una corrección** | Sí |

Se producirá un error si la versión del agente de Azure File Sync instalada en el servidor no se admite. Para resolver este problema, [actualícelo]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#upgrade-paths) a una [versión del agente admitida]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#supported-versions).

<a id="-2134351810"></a>**Se ha alcanzado el límite de almacenamiento del recurso compartido de archivos de Azure.**  

| | |
|-|-|
| **HRESULT** | 0x80c8603e |
| **HRESULT (decimal)** | -2134351810 |
| **Cadena de error** | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED |
| **Se requiere una corrección** | Sí |

Este error se produce cuando se ha alcanzado el límite de almacenamiento de los recursos compartidos de archivos de Azure, lo que puede ocurrir si se aplica una cuota para un recurso compartido de archivos de Azure o si el uso excede los límites de algún recurso compartido. Para más información, consulte los [límites actuales para un recurso compartido de archivos de Azure](storage-files-scale-targets.md).

1. Vaya al grupo de sincronización dentro del Servicio de sincronización de almacenamiento.
2. Seleccione el punto de conexión de nube en el grupo de sincronización.
3. Observe el nombre del recurso compartido de archivos de Azure en el panel abierto.
4. Seleccione la cuenta de almacenamiento vinculada. Si se produce un error en este vínculo, significa que se ha quitado la cuenta de almacenamiento de referencia.

    ![Captura de pantalla que muestra el panel de detalles del punto de conexión de nube con un vínculo a la cuenta de almacenamiento.](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

5. Seleccione **Archivos** para ver la lista de recursos compartidos de archivos.
6. Haga clic en los tres puntos al final de la fila para el recurso compartido de archivos de Azure al que hace referencia el punto de conexión de nube.
7. Compruebe que el **uso** es menor que la **cuota**. Tenga en cuenta que a menos que se haya especificado una cuota alternativa, la cuota coincidirá con el [tamaño máximo del recurso compartido de archivos de Azure](storage-files-scale-targets.md).

    ![Captura de pantalla de las propiedades de los recursos compartidos de archivos de Azure.](media/storage-sync-files-troubleshoot/file-share-limit-reached-1.png)

Si el recurso compartido está lleno y no se ha establecido una cuota, una forma posible de solucionar este problema es hacer que cada subcarpeta del punto de conexión del servidor actual se convierta en su propio punto de conexión del servidor en sus propios grupos de sincronización independientes. De esta manera, cada subcarpeta se sincronizará con recursos compartidos de archivos individuales de Azure.

<a id="-2134351824"></a>**No se puede encontrar el recurso compartido de archivos de Azure.**  

| | |
|-|-|
| **HRESULT** | 0x80c86030 |
| **HRESULT (decimal)** | -2134351824 |
| **Cadena de error** | ECS_E_AZURE_FILE_SHARE_NOT_FOUND |
| **Se requiere una corrección** | Sí |

Este error se produce cuando el recurso compartido de archivos de Azure no está accesible. Para solucionar los problemas:

1. [Compruebe que la cuenta de almacenamiento existe.](#troubleshoot-storage-account)
2. [Asegúrese de que el recurso compartido de archivos de Azure existe.](#troubleshoot-azure-file-share)

Si se ha eliminado el recurso compartido de archivos de Azure, tiene que crear un nuevo recurso compartido de archivos y, después, volver a crear el grupo de sincronización. 

<a id="-2134364042"></a>**La sincronización está en pausa mientras esta suscripción a Azure esté suspendida.**  

| | |
|-|-|
| **HRESULT** | 0x80C83076 |
| **HRESULT (decimal)** | -2134364042 |
| **Cadena de error** | ECS_E_SYNC_BLOCKED_ON_SUSPENDED_SUBSCRIPTION |
| **Se requiere una corrección** | Sí |

Este error se produce cuando se suspende la suscripción a Azure. Se puede volver a habilitar la sincronización cuando se restaure la suscripción a Azure. Consulte [¿Por qué está deshabilitada mi suscripción a Azure y cómo reactivarla?](../../billing/billing-subscription-become-disable.md) para más información.

<a id="-2134364052"></a>**La cuenta de almacenamiento tiene configurado un firewall o redes virtuales.**  

| | |
|-|-|
| **HRESULT** | 0x80c8306c |
| **HRESULT (decimal)** | -2134364052 |
| **Cadena de error** | ECS_E_MGMT_STORAGEACLSNOTSUPPORTED |
| **Se requiere una corrección** | Sí |

Este error se produce cuando no se puede acceder al recurso compartido de archivos de Azure debido a un firewall de la cuenta de almacenamiento o porque la cuenta de almacenamiento pertenece a una red virtual. Azure File Sync todavía no admite esta característica. Para solucionar los problemas:

1. [Compruebe que la cuenta de almacenamiento existe.](#troubleshoot-storage-account)
2. [Compruebe que la cuenta de almacenamiento no contiene ninguna regla de red.](#troubleshoot-network-rules)

Quite estas reglas para corregir este problema. 

<a id="-2134375911"></a>**Error de sincronización debido a un problema con la base de datos de sincronización.**  

| | |
|-|-|
| **HRESULT** | 0x80c80219 |
| **HRESULT (decimal)** | -2134375911 |
| **Cadena de error** | ECS_E_SYNC_METADATA_WRITE_LOCK_TIMEOUT |
| **Se requiere una corrección** | No |

Este error suele resolverse solo y puede producirse si hay:

* Un gran número de cambios de archivos en todos los servidores del grupo de sincronización.
* Un gran número de errores en archivos y directorios individuales.

Si este error persiste durante más de unas horas, el usuario debe crear una solicitud de soporte técnico y nos pondremos en contacto con él para ayudarle a resolver este problema.

<a id="-2146762487"></a>**El servidor no pudo establecer una conexión segura. El servicio en la nube ha recibido un certificado inesperado.**  

| | |
|-|-|
| **HRESULT** | 0x800b0109 |
| **HRESULT (decimal)** | -2146762487 |
| **Cadena de error** | CERT_E_UNTRUSTEDROOT |
| **Se requiere una corrección** | Sí |

Este error puede ocurrir si su organización usa un proxy con terminación SSL o si una entidad malintencionada intercepta el tráfico entre el servidor y el servicio Azure File Sync. Si está seguro de que esto es lo previsto (porque la organización está utilizando un proxy de terminación SSL), puede omitir la comprobación del certificado con una anulación del registro.

1. Cree el valor de registro SkipVerifyingPinnedRootCertificate.

    ```powershell
    New-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Azure\StorageSync -Name SkipVerifyingPinnedRootCertificate -PropertyType DWORD -Value 1
    ```

2. Reinicie el servicio de sincronización en el servidor registrado.

    ```powershell
    Restart-Service -Name FileSyncSvc -Force
    ```

Al establecer este valor de registro, el agente Azure File Sync aceptará cualquier certificado SSL de confianza local cuando transfiera datos entre el servidor y el servicio en la nube.

<a id="-2147012894"></a>**No se pudo establecer ninguna conexión con el servicio.**  

| | |
|-|-|
| **HRESULT** | 0x80072EE2 |
| **HRESULT (decimal)** | -2147012894 |
| **Cadena de error** | WININET_E_TIMEOUT |
| **Se requiere una corrección** | Sí |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134375680"></a>**Error de sincronización debido a un problema con la autenticación.**  

| | |
|-|-|
| **HRESULT** | 0x80c80300 |
| **HRESULT (decimal)** | -2134375680 |
| **Cadena de error** | ECS_E_SERVER_CREDENTIAL_NEEDED |
| **Se requiere una corrección** | Sí |

Este error puede deberse a:

- La hora del servidor es incorrecta.
- No se ha podido eliminar el punto de conexión del servidor.
- Ha expirado el certificado usado para la autenticación. 
    Para comprobar si el certificado ha expirado, realice los pasos siguientes:  
    1. Abra el complemento MMC de certificados, seleccione la cuenta de equipo y vaya a Certificados (equipo local)\Personal\Certificates.
    2. Compruebe si ha expirado el certificado de autenticación del cliente.

Si la hora del servidor es correcta, realice los siguientes pasos para resolver el problema:

1. Compruebe si se ha instalado la versión 4.0.1.0 o posterior de Azure File Sync.
2. Ejecute los comandos de PowerShell siguientes en el servidor:

    ```powershell
    Reset-AzStorageSyncServerCertificate -ResourceGroupName <string> -StorageSyncServiceName <string>
    ```

<a id="-1906441711"></a><a id="-2134375654"></a><a id="doesnt-have-enough-free-space"></a>**El volumen donde se encuentra el punto de conexión de servidor tiene poco espacio en disco.**  

| | |
|-|-|
| **HRESULT** | 0x8e5e0211 |
| **HRESULT (decimal)** | -1906441711 |
| **Cadena de error** | JET_errLogDiskFull |
| **Se requiere una corrección** | Sí |
| | |
| **HRESULT** | 0x80c8031a |
| **HRESULT (decimal)** | -2134375654 |
| **Cadena de error** | ECS_E_NOT_ENOUGH_LOCAL_STORAGE |
| **Se requiere una corrección** | Sí |

Este error se produce porque el volumen se ha llenado. Este error suele producirse porque los archivos que se encuentran fuera del punto de conexión del servidor consumen espacio en el volumen. Para liberar espacio en el volumen, agregue puntos de conexión adicionales al servidor, mueve los archivos a un volumen diferente o aumente el tamaño del volumen en el que se encuentra el punto de conexión del servidor.

<a id="-2134364145"></a><a id="replica-not-ready"></a>**El servicio aún no está listo para sincronizarse con este punto de conexión del servidor.**  

| | |
|-|-|
| **HRESULT** | 0x80c8300f |
| **HRESULT (decimal)** | -2134364145 |
| **Cadena de error** | ECS_E_REPLICA_NOT_READY |
| **Se requiere una corrección** | No |

Este error se produce porque hay cambios en el recurso compartido de archivos Azure directamente y la detección de cambios está en curso. Se iniciará la sincronización cuando se complete la detección de cambios.

[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="-2134375877"></a><a id="-2134375908"></a><a id="-2134375853"></a>**Error de sincronización debido a problemas con varios archivos individuales.**  

| | |
|-|-|
| **HRESULT** | 0x80c8023b |
| **HRESULT (decimal)** | -2134364145 |
| **Cadena de error** | ECS_E_SYNC_METADATA_KNOWLEDGE_SOFT_LIMIT_REACHED |
| **Se requiere una corrección** | Sí |
| | |
| **HRESULT** | 0x80c8021c |
| **HRESULT (decimal)** | -2134375908 |
| **Cadena de error** | ECS_E_SYNC_METADATA_KNOWLEDGE_LIMIT_REACHED |
| **Se requiere una corrección** | Sí |
| | |
| **HRESULT** | 0x80c80253 |
| **HRESULT (decimal)** | -2134375853 |
| **Cadena de error** | ECS_E_TOO_MANY_PER_ITEM_ERRORS |
| **Se requiere una corrección** | Sí |

En los casos en los que hay muchos errores de sincronización por archivo, las sesiones de sincronización pueden empezar a producir errores. <!-- To troubleshoot this state, see [Troubleshooting per file/directory sync errors]().-->

> [!NOTE]
> Azure File Sync crea una instantánea VSS temporal una vez al día en el servidor para sincronizar archivos que tienen identificadores abiertos.

<a id="-2134376423"></a>**Error de sincronización debido a un problema con la ruta de acceso del punto de conexión de servidor.**  

| | |
|-|-|
| **HRESULT** | 0x80c80019 |
| **HRESULT (decimal)** | -2134376423 |
| **Cadena de error** | ECS_E_SYNC_INVALID_PATH |
| **Se requiere una corrección** | Sí |

Asegúrese de que la ruta de acceso existe, está en un volumen NTFS local y no es un punto de repetición de análisis ni un punto de conexión de servidor existente.

<a id="-2134375817"></a>**Se produjo un error en la sincronización porque la versión del controlador del filtro no es compatible con la versión del agente**.  

| | |
|-|-|
| **HRESULT** | 0x80C80277 |
| **HRESULT (decimal)** | -2134375817 |
| **Cadena de error** | ECS_E_INCOMPATIBLE_FILTER_VERSION |
| **Se requiere una corrección** | Sí |

Este error se debe a que la versión del controlador del filtro de la nube por niveles (StorageSync.sys) cargada no es compatible con el servicio del agente de sincronización de almacenamiento (FileSyncSvc). Si se actualizó el agente de Azure File Sync, reinicie el servidor para completar la instalación. Si el error persiste, desinstale al agente, reinicie el servidor y vuelva a instalar al agente de Azure File Sync.

<a id="-2134376373"></a>**El servicio no está disponible en este momento.**  

| | |
|-|-|
| **HRESULT** | 0x80c8004b |
| **HRESULT (decimal)** | -2134376373 |
| **Cadena de error** | ECS_E_SERVICE_UNAVAILABLE |
| **Se requiere una corrección** | No |

Este error se produce porque el servicio Azure File Sync no está disponible. Este error se resolverá automáticamente cuando el servicio Azure File Sync vuelva a estar disponible.

<a id="-2134375922"></a>**Error de sincronización debido a un problema transitorio con la base de datos de sincronización.**  

| | |
|-|-|
| **HRESULT** | 0x80c8020e |
| **HRESULT (decimal)** | -2134375922 |
| **Cadena de error** | ECS_E_SYNC_METADATA_WRITE_LEASE_LOST |
| **Se requiere una corrección** | No |

Este error se produce debido a un problema interno con la base de datos de sincronización. Este error se resolverá automáticamente cuando se vuelva a intentar la sincronización de Azure File Sync. Si este error continúa durante un período de tiempo prolongado, el usuario debe crear una solicitud de soporte técnico y nos pondremos en contacto con él para ayudarle a resolver este problema.

### <a name="common-troubleshooting-steps"></a>Pasos comunes de solución de problemas
<a id="troubleshoot-storage-account"></a>**Compruebe que la cuenta de almacenamiento existe.**  
# <a name="portaltabazure-portal"></a>[Portal](#tab/azure-portal)
1. Vaya al grupo de sincronización dentro del Servicio de sincronización de almacenamiento.
2. Seleccione el punto de conexión de nube en el grupo de sincronización.
3. Observe el nombre del recurso compartido de archivos de Azure en el panel abierto.
4. Seleccione la cuenta de almacenamiento vinculada. Si se produce un error en este vínculo, significa que se ha quitado la cuenta de almacenamiento de referencia.
    ![Captura de pantalla que muestra el panel de detalles del punto de conexión de nube con un vínculo a la cuenta de almacenamiento.](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
# Variables for you to populate based on your configuration
$region = "<Az_Region>"
$resourceGroup = "<RG_Name>"
$syncService = "<storage-sync-service>"
$syncGroup = "<sync-group>"

# Log into the Azure account
Connect-AzAccount

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = [System.String[]]@()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the " + `
        " selected Azure Region or the region is mistyped.")
}

# Check to ensure resource group exists
$resourceGroups = [System.String[]]@()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    throw [System.Exception]::new("The provided resource group $resourceGroup does not exist.")
}

# Check to make sure the provided Storage Sync Service
# exists.
$syncServices = [System.String[]]@()

Get-AzStorageSyncService -ResourceGroupName $resourceGroup | ForEach-Object {
    $syncServices += $_.StorageSyncServiceName
}

if ($syncServices -notcontains $syncService) {
    throw [System.Exception]::new("The provided Storage Sync Service $syncService does not exist.")
}

# Check to make sure the provided Sync Group exists
$syncGroups = [System.String[]]@()

Get-AzStorageSyncGroup -ResourceGroupName $resourceGroup -StorageSyncServiceName $syncService | ForEach-Object {
    $syncGroups += $_.SyncGroupName
}

if ($syncGroups -notcontains $syncGroup) {
    throw [System.Exception]::new("The provided sync group $syncGroup does not exist.")
}

# Get reference to cloud endpoint
$cloudEndpoint = Get-AzStorageSyncCloudEndpoint `
    -ResourceGroupName $resourceGroup `
    -StorageSyncServiceName $syncService `
    -SyncGroupName $syncGroup

# Get reference to storage account
$storageAccount = Get-AzStorageAccount | Where-Object { 
    $_.Id -eq $cloudEndpoint.StorageAccountResourceId
}

if ($storageAccount -eq $null) {
    throw [System.Exception]::new("The storage account referenced in the cloud endpoint does not exist.")
}
```
---

<a id="troubleshoot-network-rules"></a>**Compruebe que la cuenta de almacenamiento no contiene ninguna regla de red.**  
# <a name="portaltabazure-portal"></a>[Portal](#tab/azure-portal)
1. Una vez en la cuenta de almacenamiento, seleccione **Firewalls y redes virtuales** en el lado izquierdo de la cuenta de almacenamiento.
2. Dentro de la cuenta de almacenamiento, debe estar seleccionado el botón de radio **Permitir el acceso desde todas las redes**.
    ![Captura de pantalla que muestra el firewall de la cuenta de almacenamiento y las reglas de red deshabilitadas.](media/storage-sync-files-troubleshoot/file-share-inaccessible-2.png)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
if ($storageAccount.NetworkRuleSet.DefaultAction -ne 
    [Microsoft.Azure.Commands.Management.Storage.Models.PSNetWorkRuleDefaultActionEnum]::Allow) {
    throw [System.Exception]::new("The storage account referenced contains network " + `
        "rules which are not currently supported by Azure File Sync.")
}
```
---

<a id="troubleshoot-azure-file-share"></a>**Asegúrese de que el recurso compartido de archivos de Azure existe.**  
# <a name="portaltabazure-portal"></a>[Portal](#tab/azure-portal)
1. Haga clic en **Introducción** en la tabla de contenido para volver a la página de la cuenta de almacenamiento principal izquierda.
2. Seleccione **Archivos** para ver la lista de recursos compartidos de archivos.
3. Compruebe que el recurso compartido de archivos al que hace referencia el punto de conexión de nube aparece en la lista de recursos compartidos de archivos (debería haberlo observado en el paso 1 anterior).

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $cloudEndpoint.AzureFileShareName -and
    $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    throw [System.Exception]::new("The Azure file share referenced by the cloud endpoint does not exist")
}
```
---

<a id="troubleshoot-rbac"></a>**Asegúrese de que Azure File Sync tiene acceso a la cuenta de almacenamiento.**  
# <a name="portaltabazure-portal"></a>[Portal](#tab/azure-portal)
1. Haga clic en **Control de acceso (IAM)** en la tabla de contenido de la izquierda.
1. Haga clic en la pestaña **Asignaciones de roles** de la lista de los usuarios y las aplicaciones (*entidades de servicio*) que tienen acceso a su cuenta de almacenamiento.
1. Compruebe que **Hybrid File Sync Service** aparece en la lista con el rol **Lector y acceso a los datos**. 

    ![Captura de pantalla de la entidad de servicio del servicio de sincronización de archivos híbrida en la pestaña de control de acceso de la cuenta de almacenamiento](media/storage-sync-files-troubleshoot/file-share-inaccessible-3.png)

    Si no aparece **Hybrid File Sync Service** en la lista, realice los pasos siguientes:

    - Haga clic en **Agregar**.
    - En el campo **Rol**, seleccione **Lector y acceso a los datos**.
    - En el campo **Seleccionar**, escriba **Hybrid File Sync Service**, seleccione el rol y haga clic en **Guardar**.

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell    
$role = Get-AzRoleAssignment -Scope $storageAccount.Id | Where-Object { $_.DisplayName -eq "Hybrid File Sync Service" }

if ($role -eq $null) {
    throw [System.Exception]::new("The storage account does not have the Azure File Sync " + `
                "service principal authorized to access the data within the " + ` 
                "referenced Azure file share.")
}
```
---

### <a name="how-do-i-prevent-users-from-creating-files-containing-unsupported-characters-on-the-server"></a>¿Cómo evito que los usuarios creen archivos que contengan caracteres no admitidos en el servidor?
Puede utilizar los [filtros de archivos del Administrador de recursos del servidor de archivos (FSRM)](https://docs.microsoft.com/windows-server/storage/fsrm/file-screening-management) para bloquear la creación en el servidor de archivos con caracteres no admitidos en sus nombres. Puede que tenga que hacerlo mediante PowerShell ya que la mayoría de los caracteres no admitidos no se pueden imprimir y por lo tanto necesita convertir primero sus representaciones hexadecimales como caracteres.

En primer lugar va a crear un grupo de archivos FSRM mediante el [cmdlet New-FsrmFileGroup](https://docs.microsoft.com/powershell/module/fileserverresourcemanager/new-fsrmfilegroup). En este ejemplo se define que el grupo contiene solo dos de los caracteres no admitidos, pero puede incluir tantos caracteres como sea necesario en su grupo de archivos.

```powershell
New-FsrmFileGroup -Name "Unsupported characters" -IncludePattern @(("*"+[char]0x00000090+"*"),("*"+[char]0x0000008F+"*"))
```

Una vez definido un grupo de archivos FSRM, puede crear un filtro de archivos FSRM utilizando el cmdlet New-FsrmFileScreen.

```powershell
New-FsrmFileScreen -Path "E:\AFSdataset" -Description "Filter unsupported characters" -IncludeGroup "Unsupported characters"
```

> [!Important]  
> Tenga en cuenta que los filtros de archivos solo deben utilizarse para bloquear la creación de caracteres no compatibles con Azure File Sync. Si se utilizan filtros de archivos en otros escenarios, la sincronización intentará descargar continuamente los archivos del recurso compartido de archivos de Azure al servidor y se bloqueará debido al filtro de archivos, lo que resultará en una elevada salida de datos. 

## <a name="cloud-tiering"></a>Niveles de nube 
Los errores en la organización en niveles en la nube pueden producirse de dos formas:

- Los archivos pueden dar error al organizarse en niveles, lo que significa que Azure File Sync lo intenta sin éxito con un archivo en Azure Files.
- Los archivos pueden producir un error en la recuperación, lo que significa que el filtro del sistema de archivos de Azure File Sync (StorageSync.sys) da un error al descargar los datos cuando un usuario intenta tener acceso a un archivo que ha sido organizado en niveles.

Hay dos clases principales de errores que pueden producirse de alguno de esos modos:

- Errores de almacenamiento en la nube
    - *Problemas transitorios de disponibilidad del servicio de almacenamiento*. Para más información, consulte el [Acuerdo de Nivel de Servicio (SLA) para Azure Storage](https://azure.microsoft.com/support/legal/sla/storage/v1_2/).
    - *Recurso compartido de archivos de Azure inaccesible*. Este error normalmente ocurre al eliminarse el recurso compartido de archivos de Azure cuando sigue siendo un punto de conexión de nube en un grupo de sincronización.
    - *Cuenta de almacenamiento inaccesible*. Este error suele ocurrir cuando se elimina la cuenta de almacenamiento mientras todavía tiene un recurso compartido de archivos de Azure que es un punto de conexión en la nube en un grupo de sincronización. 
- Errores del servidor 
  - *No está cargado el filtro de sistema de archivos de Azure File Sync (StorageSync.sys)*. Para responder a solicitudes de organización por niveles o recuperación, debe haber cargado el filtro del sistema de archivos de Azure File Sync. El filtro puede no cargarse por varios motivos, pero el más común es que un administrador lo descargue manualmente. El filtro del sistema de archivos de Azure File Sync se debe cargar en todo momento para que este funcione de forma adecuada.
  - *El punto de reanálisis falta, está dañado o ha sufrido algún otro problema*. Un punto de reanálisis es una estructura de datos especial en un archivo que consta de dos partes:
    1. Una etiqueta de reanálisis, que indica al sistema operativo que el filtro del sistema de archivos de Azure File Sync (StorageSync.sys) puede necesitar realizar alguna acción de E/S en el archivo. 
    2. Repita el análisis de los datos, lo que indica al sistema de archivos que filtre el URI del archivo en el punto de conexión en la nube asociado (el recurso compartido de archivos de Azure). 
        
       La manera más común de que un punto de reanálisis pueda resultar dañado es que un administrador intente modificar la etiqueta o sus datos. 
  - *Problemas de conectividad de red*. Con el fin organizar en niveles un archivo o de recuperarlo, el servidor debe tener conectividad a Internet.

En las secciones siguientes se indica cómo solucionar problemas de los niveles en la nube y determinar si se trata de un problema de almacenamiento en la nube o del servidor.

<a id="monitor-tiering-activity"></a>**Supervisión de la actividad de organización en niveles en un servidor**  
Para supervisar la actividad de organización en niveles en un servidor, use el identificador de eventos 9003, 9016 y 9029 en el registro de eventos de telemetría (ubicados en Applications and Services\Microsoft\FileSync\Agent en el Visor de eventos).

- El identificador de evento 9003 proporciona una distribución de errores para un punto de conexión de servidor. Por ejemplo, el recuento total de errores, el código de error, entre otros. Tenga en cuenta que se registra un evento por código de error.
- El identificador de evento 9016 proporciona resultados de conversión en fantasma para un volumen. Por ejemplo, el porcentaje de espacio libre, el número de archivos convertidos en fantasma en la sesión, el número de archivos no convertidos en fantasma, etc.
- El id. de evento 9029 proporciona información de la sesión de conversión en fantasma para el punto de conexión de un servidor. Por ejemplo, el número de archivos que se han intentado en la sesión, el número de archivos organizados en niveles de la sesión, el número de archivos ya organizados en niveles, entre otros.

<a id="monitor-recall-activity"></a>**Supervisión de la actividad de recuperación en un servidor**  
Para supervisar la actividad de recuperación en un servidor, use los id. de evento 9005, 9006, 9009 y 9059 en el registro de eventos de telemetría (ubicado en Applications and Services\Microsoft\FileSync\Agent en el Visor de eventos).

- El identificador de evento 9005 proporciona confiabilidad de recuperación a un punto de conexión de servidor. Por ejemplo, el total de archivos únicos a los que se puede acceder, el total de archivos único con acceso erróneo, entre otros.
- El identificador de evento 9006 proporciona una distribución de errores de recuperación a un punto de conexión de servidor. Por ejemplo, el total de solicitudes erróneas o el código de error. Tenga en cuenta que se registra un evento por código de error.
- El id. de evento 9009 proporciona información de la sesión de recuperación para un punto de conexión de servidor. Por ejemplo, DurationSeconds, CountFilesRecallSucceeded, CountFilesRecallFailed, etc.
- El id. de evento 9059 proporciona una distribución de recuperación de aplicación para un punto de conexión de servidor. Por ejemplo, ShareId, Application Name y TotalEgressNetworkBytes.

<a id="files-fail-tiering"></a>**Solución de problemas de archivos que no se apilan**  
Si no se pueden apilar archivos en Azure Files:

1. En el Visor de eventos, revise los registros de eventos de telemetría, operativos y de diagnósticos, ubicados en Applications and Services\Microsoft\FileSync\Agent. 
   1. Compruebe que los archivos existen en el recurso compartido de archivos de Azure.

      > [!NOTE]
      > Un archivo debe sincronizarse con un recurso compartido de archivos de Azure antes de que se puedan apilar.

   2. Compruebe que el servidor tiene conectividad a Internet. 
   3. Compruebe que los controladores de filtro de Azure File Sync (StorageSync.sys y StorageSyncGuard.sys) se están ejecutando:
       - En un símbolo del sistema con privilegios elevados, ejecute `fltmc`. Compruebe que aparecen los controladores de filtro del sistema de archivos StorageSync.sys y StorageSyncGuard.sys.

> [!NOTE]
> El identificador de evento 9003 se registra una vez por hora en el registro de eventos de telemetría si el archivo no se clasifica en niveles (se registra un evento por cada código de error). Se deben utilizar los registros de eventos operativos y de diagnóstico si se necesita información adicional para diagnosticar un problema.

<a id="files-fail-recall"></a>**Solución de problemas de archivos que no se pueden recuperar**  
Si no se pueden recuperar archivos:
1. En el Visor de eventos, revise los registros de eventos de telemetría, operativos y de diagnósticos, ubicados en Applications and Services\Microsoft\FileSync\Agent.
    1. Compruebe que los archivos existen en el recurso compartido de archivos de Azure.
    2. Compruebe que el servidor tiene conectividad a Internet. 
    3. Abra el complemento MMC de Servicios y compruebe que el servicio del agente de sincronización de almacenamiento (FileSyncSvc) se está ejecutando.
    4. Compruebe que los controladores de filtro de Azure File Sync (StorageSync.sys y StorageSyncGuard.sys) se están ejecutando:
        - En un símbolo del sistema con privilegios elevados, ejecute `fltmc`. Compruebe que aparecen los controladores de filtro del sistema de archivos StorageSync.sys y StorageSyncGuard.sys.

> [!NOTE]
> El identificador de evento 9006 se registra una vez cada hora en el registro de eventos de telemetría si el archivo no puede recuperarse (se registra un evento por cada código de error). Se deben utilizar los registros de eventos operativos y de diagnóstico si se necesita información adicional para diagnosticar un problema.

<a id="files-unexpectedly-recalled"></a>**Solución de problemas de archivos que se recuperan de manera inesperada en un servidor**  
Las aplicaciones antivirus, de backup y de otro tipo que leen gran cantidad de archivos provocan recuperaciones no previstas salvo que respeten la omisión del atributo sin conexión y pasen por alto la lectura del contenido de esos archivos. Omitir archivos sin conexión para los productos que admiten esta opción ayuda a evitar recuperaciones imprevistas durante las operaciones, como análisis antivirus o trabajos de backup.

Consulte con el proveedor de software para aprender cómo configurar su solución para omitir la lectura de archivos sin conexión.

Las recuperaciones imprevistas también se podrían producir en otros escenarios, como al buscar archivos en el Explorador de archivos. Abrir una carpeta que tiene archivos de niveles en la nube en el Explorador de archivos en el servidor podría producir recuperaciones imprevistas. Esto es incluso más probable si una solución antivirus está habilitada en el servidor.

> [!NOTE]
>Use el id. de evento 9059 del registro de eventos de telemetría para determinar qué aplicaciones están causando las recuperaciones. Este evento proporciona la distribución de recuperación de la aplicación para un punto de conexión de servidor y se registra una vez cada hora.

## <a name="general-troubleshooting"></a>Solución general de problemas
Si tiene problemas con Azure File Sync en un servidor, empiece por los pasos siguientes:
1. Revise los registros de eventos de telemetría, operativos y de diagnóstico en el Visor de eventos.
    - Los problemas de sincronización, de niveles y de recuperación se registran en los registros de telemetría, operativos y de diagnóstico en Applications and Services\Microsoft\FileSync\Agent.
    - Los problemas relacionados con la administración de un servidor (por ejemplo, los valores de configuración) se registran en los registros de eventos operativos y de diagnósticos en Applications and Services\Microsoft\FileSync\Management.
2. Compruebe que el servicio Azure File Sync se ejecuta en el servidor:
    - Abra el complemento MMC de Servicios y compruebe que el servicio del agente de sincronización de almacenamiento (FileSyncSvc) se está ejecutando.
3. Compruebe que los controladores de filtro de Azure File Sync (StorageSync.sys y StorageSyncGuard.sys) se están ejecutando:
    - En un símbolo del sistema con privilegios elevados, ejecute `fltmc`. Compruebe que aparecen los controladores de filtro del sistema de archivos StorageSync.sys y StorageSyncGuard.sys.

Si no se resuelve el problema, ejecute la herramienta AFSDiag:
1. Cree un directorio donde se guardará la salida de AFSDiag (por ejemplo, C:\Output).
2. Abra una ventana de PowerShell con privilegios elevados y ejecute los comandos siguientes (presione Entrar después de cada comando):

    ```powershell
    cd "c:\Program Files\Azure\StorageSyncAgent"
    Import-Module .\afsdiag.ps1
    Debug-Afs c:\output # Note: Use the path created in step 1.
    ```

3. Para el nivel de seguimiento del modo de kernel de Azure File Sync, escriba **1** (a menos que se especifique lo contrario, para crear seguimientos más detallados) y presione Entrar.
4. Para el nivel de seguimiento del modo de usuario de Azure File Sync, escriba **1** (a menos que se especifique lo contrario, para crear seguimientos más detallados) y presione Entrar.
5. Reproduzca el problema. Cuando haya terminado, escriba **D**.
6. Un archivo .zip que contiene los archivos de seguimiento y registros se guarda en el directorio de salida que especificó.

## <a name="see-also"></a>Vea también
- [Supervisión de Azure File Sync](storage-sync-files-monitoring.md)
- [Preguntas más frecuentes de Azure Files](storage-files-faq.md)
- [Solucione problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
- [Solución de problemas de Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md)
