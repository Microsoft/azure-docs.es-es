---
title: Solución de problemas de Azure Files en Windows | Microsoft Docs
description: Solución de problemas de Azure Files en Windows. Vea problemas comunes relacionados con Azure Files al conectarse desde clientes Windows y las posibles soluciones.
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 08/31/2019
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: 3bd059e59bebe9ae1ecc8f2f00dd63f873e08944
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269376"
---
# <a name="troubleshoot-azure-files-problems-in-windows"></a>Solucione problemas de Azure Files en Windows

En este artículo se enumeran los problemas habituales relacionados con Microsoft Azure Files cuando se conecta desde clientes Windows. También se proporcionan posibles causas de estos problemas y sus resoluciones. Además de los pasos de solución de problemas de este artículo, también puede usar [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows)  para asegurarse de que el entorno de cliente Windows cumpla los requisitos previos. AzFileDiagnostics automatiza la detección de la mayoría de los síntomas que se mencionan en este artículo y le ayuda a configurar su entorno para obtener un rendimiento óptimo. Esta información también se puede encontrar en el [Solucionador de problemas de recursos compartidos de Azure Files](https://support.microsoft.com/help/4022301/troubleshooter-for-azure-files-shares), que proporciona los pasos necesarios para ayudarle con problemas relativos a la conexión, asignación o montaje de recursos compartidos de Azure Files.

<a id="error5"></a>
## <a name="error-5-when-you-mount-an-azure-file-share"></a>Error 5 al montar un recurso compartido de archivos de Azure

Al intentar montar un recurso compartido de archivos, puede recibir el siguiente error:

- Error de sistema 5. Acceso denegado.

### <a name="cause-1-unencrypted-communication-channel"></a>Causa 1: Canal de comunicación sin cifrar

Por seguridad, se bloquean las conexiones a recursos compartidos de archivos de Azure si no el canal de comunicación no está cifrado y si el intento de conexión no se realiza desde el mismo centro de datos donde residen los recursos compartidos de archivos de Azure. Las conexiones no cifradas del mismo centro de datos también se pueden bloquear si la configuración de [transferencia segura requerida](https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer) está habilitada en la cuenta de almacenamiento. Se proporciona un canal de comunicación cifrado solamente si el sistema operativo cliente del usuario admite el cifrado SMB.

Windows 8, Windows Server 2012 y versiones posteriores de cada sistema negocian solicitudes que incluyen SMB 3.0, que es compatible con el cifrado.

### <a name="solution-for-cause-1"></a>Solución para la causa 1

1. Conéctese desde un cliente que admita el cifrado SMB (Windows 8, Windows Server 2012 o versiones posteriores) o bien conéctese desde una máquina virtual que se encuentre en el mismo centro de datos que la cuenta de Azure Storage que se utilice para el recurso compartido de archivos de Azure.
2. Compruebe que la configuración de la [transferencia segura requerida](https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer) esté deshabilitada en la cuenta de almacenamiento si el cliente no admite el cifrado SMB.

### <a name="cause-2-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Causa 2: Las reglas de firewall o de red virtuales están habilitadas en la cuenta de almacenamiento 

Si las reglas de red virtual (VNET) y de firewall están configuradas en la cuenta de almacenamiento, se denegará el acceso al tráfico de red a menos que la dirección IP del cliente o la red virtual tengan permitido el acceso.

### <a name="solution-for-cause-2"></a>Solución para la causa 2

Compruebe que las reglas de firewall y de red virtual están configuradas correctamente en la cuenta de almacenamiento. Para probar si las reglas de firewall o de red virtual están causando el problema, cambie temporalmente la configuración de la cuenta de almacenamiento para **Permitir el acceso desde todas las redes**. Para más información, consulte [Configuración de redes virtuales y firewalls de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-network-security).

### <a name="cause-3-share-level-permissions-are-incorrect-when-using-identity-based-authentication"></a>Causa 3: Los permisos de nivel de recurso compartido no son correctos cuando se usa la autenticación basada en identidades

Si los usuarios acceden al recurso compartido de archivos de Azure mediante la autenticación de Active Directory (AD) o Azure Active Directory Domain Services (Azure AD DS), el acceso al recurso compartido de archivos generará el error "Acceso denegado" si los permisos de nivel de recurso compartido son incorrectos. 

### <a name="solution-for-cause-3"></a>Solución para la causa 3

Para actualizar los permisos de nivel de recurso compartido, consulte [Asignar permisos de acceso a una identidad](https://docs.microsoft.com/azure/storage/files/storage-files-identity-auth-active-directory-domain-service-enable#2-assign-access-permissions-to-an-identity).

<a id="error53-67-87"></a>
## <a name="error-53-error-67-or-error-87-when-you-mount-or-unmount-an-azure-file-share"></a>Error 53, Error 67 o Error 87 al montar o desmontar un recurso compartido de archivos de Azure

Cuando intenta montar un recurso compartido de archivos desde un entorno local o un centro de datos diferente, es posible que observe los errores siguientes:

- Error de sistema 53. No se ha encontrado la ruta de acceso de la red.
- Error de sistema 67. No se encuentra el nombre de red especificado.
- Error de sistema 87. El parámetro no es correcto.

### <a name="cause-1-port-445-is-blocked"></a>Causa 1: El puerto 445 está bloqueado

Los errores del sistema 53 o 67 pueden producirse cuando se bloquea la comunicación de salida del puerto 445 a un centro de datos de Azure Files. Para ver el resumen de los ISP que permiten o deniegan el acceso desde el puerto 445, visite [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx).

Para comprobar si el firewall o el ISP está bloqueando el puerto 445, use la herramienta [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) o el cmdlet `Test-NetConnection`. 

Para usar el cmdlet `Test-NetConnection`, debe tener instalado el módulo Azure PowerShell. Para obtener más información, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). No olvide reemplazar `<your-storage-account-name>` y `<your-resource-group-name>` por los nombres correspondientes de su cuenta de almacenamiento.

   
```azurepowershell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"

# This command requires you to be logged into your Azure account, run Login-AzAccount if you haven't
# already logged in.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName

# The ComputerName, or host, is <storage-account>.file.core.windows.net for Azure Public Regions.
# $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as sovereign clouds
# or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
Test-NetConnection -ComputerName ([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) -Port 445
```
    
Si la conexión se realizó correctamente, verá la siguiente salida:
    
  
```azurepowershell
ComputerName     : <your-storage-account-name>
RemoteAddress    : <storage-account-ip-address>
RemotePort       : 445
InterfaceAlias   : <your-network-interface>
SourceAddress    : <your-ip-address>
TcpTestSucceeded : True
```
 

> [!Note]  
> El comando anterior devuelve la dirección IP actual de la cuenta de almacenamiento. No se garantiza que esta dirección IP permanezca igual, podría cambiar en cualquier momento. No codifique de forma rígida esta dirección IP en los scripts o en una configuración de firewall.

### <a name="solution-for-cause-1"></a>Solución para la causa 1

#### <a name="solution-1---use-azure-file-sync"></a>Solución 1: use Azure File Sync
Azure File Sync puede transformar su instancia de Windows Server local en una caché rápida de su recurso compartido de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Azure File Sync funciona a través del puerto 443 y, por tanto, puede utilizarse como una solución alternativa para acceder a Azure Files desde clientes con el puerto 445 bloqueado. [Obtenga información sobre cómo configurar Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-sync-files-extend-servers).

#### <a name="solution-2---use-vpn"></a>Solución 2: use VPN
Al establecer una conexión VPN a su cuenta de almacenamiento específica, el tráfico pasará a través de un túnel seguro en lugar de hacerlo a través de Internet. Siga las [instrucciones de configuración de la VPN](storage-files-configure-p2s-vpn-windows.md) para acceder a Azure Files desde Windows.

#### <a name="solution-3---unblock-port-445-with-help-of-your-ispit-admin"></a>Solución 3: desbloquee el puerto 445 con la ayuda de su ISP o administrador de TI
Colabore con su departamento de TI o ISP para abrir el puerto 445, de modo que acepte las conexiones salientes a los [intervalos de IP de Azure](https://www.microsoft.com/download/details.aspx?id=41653).

#### <a name="solution-4---use-rest-api-based-tools-like-storage-explorerpowershell"></a>Solución 4: use las herramientas basadas en la API de REST como el Explorador de Storage o Powershell
Azure Files también admite REST además de SMB. El acceso a REST funciona a través del puerto 443 (tcp estándar). Existen diversas herramientas que se escriben mediante la API de REST y que ofrecen una experiencia de interfaz de usuario mejorada. Una de ellas es el [Explorador de Storage](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows). [Descargue e instale el Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) y conéctese a su recurso compartido de archivos respaldado por Azure Files. También puede utilizar [PowerShell](https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-powershell), que también utiliza la API de REST.

### <a name="cause-2-ntlmv1-is-enabled"></a>Causa 2: NTLMv1 está habilitado

También se producen los errores del sistema 53 u 87 si se ha habilitado la comunicación NTLMv1 en el cliente. Azure Files solo admite la autenticación NTLMv2. Si se habilita NTLMv1, el cliente será menos seguro. Por lo tanto, se bloquea la comunicación con Azure Files. 

Para determinar si esta es la causa del error, averigüe si la siguiente subclave del Registro está establecida en el valor 3:

**HKLM\SYSTEM\CurrentControlSet\Control\Lsa &gt; LmCompatibilityLevel**

Para obtener más información, consulte el tema [LmCompatibilityLevel](https://technet.microsoft.com/library/cc960646.aspx) en TechNet.

### <a name="solution-for-cause-2"></a>Solución para la causa 2

Revierta el valor **LmCompatibilityLevel** al predeterminado, 3, en la siguiente subclave del Registro:

  **HKLM\SYSTEM\CurrentControlSet\Control\Lsa**

<a id="error1816"></a>
## <a name="error-1816---not-enough-quota-is-available-to-process-this-command"></a>Error 1816: cuota insuficiente para procesar este comando

### <a name="cause"></a>Causa

El error 1816 se produce cuando se alcanza el límite superior de identificadores abiertos simultáneos que se permiten para un archivo o directorio en el recurso compartido de archivos de Azure. Para más información, consulte [Objetivos de escalabilidad de Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-scale-targets#azure-files-scale-targets).

### <a name="solution"></a>Solución

Reduzca el número de identificadores abiertos simultáneos cerrando algunos de ellos y vuelva a intentarlo. Para más información, consulte [Lista de comprobación de rendimiento y escalabilidad de Microsoft Azure Storage](../common/storage-performance-checklist.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Para ver los identificadores abiertos de un recurso compartido de archivos, un directorio o un archivo, use el cmdlet de PowerShell [Get-AzStorageFileHandle](https://docs.microsoft.com/powershell/module/az.storage/get-azstoragefilehandle).  

Para cerrar los identificadores abiertos de un recurso compartido de archivos, un directorio o un archivo, use el cmdlet de PowerShell [Close-AzStorageFileHandle](https://docs.microsoft.com/powershell/module/az.storage/close-azstoragefilehandle).

> [!Note]  
> Los cmdlets Get-AzStorageFileHandle y Close-AzStorageFileHandle se incluyen en la versión 2.4 o posteriores del módulo Az de PowerShell. Para instalar el módulo Az de PowerShell más reciente, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps).

<a id="noaaccessfailureportal"></a>
## <a name="error-no-access-when-you-try-to-access-or-delete-an-azure-file-share"></a>Error "sin acceso" al intentar acceder a un recurso compartido de archivos de Azure o eliminarlo  
Al intentar acceder a un recurso compartido de archivos de Azure en el portal o eliminarlo, puede que reciba el siguiente error:

Sin acceso  
Código de error: 403 

### <a name="cause-1-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Causa 1: Las reglas de firewall o de red virtuales están habilitadas en la cuenta de almacenamiento

### <a name="solution-for-cause-1"></a>Solución para la causa 1

Compruebe que las reglas de firewall y de red virtual están configuradas correctamente en la cuenta de almacenamiento. Para probar si las reglas de firewall o de red virtual están causando el problema, cambie temporalmente la configuración de la cuenta de almacenamiento para **Permitir el acceso desde todas las redes**. Para más información, consulte [Configuración de redes virtuales y firewalls de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-network-security).

### <a name="cause-2-your-user-account-does-not-have-access-to-the-storage-account"></a>Causa 2: Su cuenta de usuario no tiene acceso a la cuenta de almacenamiento

### <a name="solution-for-cause-2"></a>Solución para la causa 2

Vaya a la cuenta de almacenamiento donde se encuentra el recurso compartido de archivos de Azure, haga clic en **Control de acceso (IAM)** y compruebe que su cuenta de usuario tiene acceso a la cuenta de almacenamiento. Para más información, consulte [Protección de su cuenta de almacenamiento con el control de acceso basado en rol (RBAC)](https://docs.microsoft.com/azure/storage/blobs/security-recommendations#data-protection).

<a id="open-handles"></a>
## <a name="unable-to-delete-a-file-or-directory-in-an-azure-file-share"></a>No se puede eliminar un archivo o un directorio en un recurso compartido de archivos de Azure
Al intentar eliminar un archivo, puede que reciba el siguiente mensaje de error:

El recurso especificado está marcado para su eliminación por parte de un cliente SMB.

### <a name="cause"></a>Causa
Este problema suele producirse si el archivo o el directorio tiene un identificador abierto. 

### <a name="solution"></a>Solución

Si los clientes SMB cerraron todos los identificadores abiertos y el problema sigue ocurriendo, realice lo siguiente:

- Use el cmdlet de PowerShell [Get-AzStorageFileHandle](https://docs.microsoft.com/powershell/module/az.storage/get-azstoragefilehandle) para ver los identificadores abiertos.

- Use el cmdlet de PowerShell [Close-AzStorageFileHandle](https://docs.microsoft.com/powershell/module/az.storage/close-azstoragefilehandle) para cerrar los identificadores abiertos. 

> [!Note]  
> Los cmdlets Get-AzStorageFileHandle y Close-AzStorageFileHandle se incluyen en la versión 2.4 o posteriores del módulo Az de PowerShell. Para instalar el módulo Az de PowerShell más reciente, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps).

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-windows"></a>Lentitud al copiar archivos a y desde Azure Files en Windows

Puede que experimente un rendimiento lento cuando intente transferir archivos al servicio Azure Files.

- Si no tiene un requisito mínimo de tamaño de E/S específico, se recomienda que use 1 MiB como tamaño de E/S para disfrutar de un rendimiento óptimo.
-   Si conoce el tamaño final de un archivo que está ampliando mediante operaciones de escritura y el software no presenta problemas de compatibilidad cuando la cola no escrita del archivo contiene ceros, establezca el tamaño de archivo con antelación en lugar de hacer que cada escritura sea una escritura de ampliación.
-   Utilice el método de copia correcto:
    -   Use [AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) para todas las transferencias entre dos recursos compartidos de archivos.
    -   Utilice [Robocopy](/azure/storage/files/storage-files-deployment-guide#robocopy) entre recursos compartidos de archivos y un equipo local.

### <a name="considerations-for-windows-81-or-windows-server-2012-r2"></a>Consideraciones para Windows 8.1 o Windows Server 2012 R2

En el caso de los clientes que ejecutan Windows 8.1 o Windows Server 2012 R2, asegúrese de que la revisión [KB3114025](https://support.microsoft.com/help/3114025) esté instalada. Esta revisión mejora el rendimiento de la creación y el cierre de identificadores.

Puede ejecutar el siguiente script para comprobar si se ha instalado la revisión:

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

Si la revisión está instalada, se muestra el siguiente resultado:

`HKEY_Local_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies {96c345ef-3cac-477b-8fcd-bea1a564241c} REG_DWORD 0x1`

> [!Note]
> Las imágenes de Windows Server 2012 R2 de Azure Marketplace tienen la revisión KB3114025 instalada de manera predeterminada desde diciembre de 2015.

<a id="shareismissing"></a>
## <a name="no-folder-with-a-drive-letter-in-my-computer-or-this-pc"></a>No hay ninguna carpeta con una letra de unidad en "Mi PC" o "Este equipo"

Si asigna un recurso compartido de archivos de Azure como administrador mediante net use, el recurso compartido parece que falta.

### <a name="cause"></a>Causa

De manera predeterminada, el Explorador de archivos de Windows no se ejecuta como administrador. Si se ejecuta net use desde un símbolo del sistema de administrador, la unidad de red se asigna como administrador. Dado que las unidades asignadas son específicas para los usuarios, la cuenta de usuario con la que se haya iniciado sesión no mostrará las unidades si estas se han montado con una cuenta de usuario distinta.

### <a name="solution"></a>Solución
Monte el recurso compartido desde una línea de comandos que no sea de administrador. Como alternativa, puede seguir las instrucciones de [este tema de TechNet](https://technet.microsoft.com/library/ee844140.aspx) para configurar el valor del Registro **EnableLinkedConnections**.

<a id="netuse"></a>
## <a name="net-use-command-fails-if-the-storage-account-contains-a-forward-slash"></a>Se produce un error en el comando net use si la cuenta de almacenamiento contiene una barra diagonal

### <a name="cause"></a>Causa

El comando net use interpreta la barra diagonal (/) como una opción de línea de comandos. Si el nombre de la cuenta de usuario comienza por una barra diagonal, se produce un error en la asignación de unidad.

### <a name="solution"></a>Solución

Puede utilizar cualquiera de los siguientes pasos para solucionar el problema:

- Ejecute el siguiente comando de PowerShell:

  `New-SmbMapping -LocalPath y: -RemotePath \\server\share -UserName accountName -Password "password can contain / and \ etc"`

  Desde un archivo por lotes, puede ejecutar el comando de este modo:

  `Echo new-smbMapping ... | powershell -command –`

- Encierre la clave entre comillas dobles para solucionar el problema, a menos que la barra diagonal sea el primer carácter. En ese caso, use el modo interactivo y escriba la contraseña por separado o vuelva a generar las claves para obtener una que no empiece por barra diagonal.

<a id="cannotaccess"></a>
## <a name="application-or-service-cannot-access-a-mounted-azure-files-drive"></a>La aplicación o el servicio no pueden acceder a una unidad montada de Azure Files

### <a name="cause"></a>Causa

Las unidades se montan para usuarios individuales. Si su aplicación o servicio se ejecuta con una cuenta de usuario diferente a la que montó la unidad, la aplicación no verá la unidad.

### <a name="solution"></a>Solución

Pruebe una de estas soluciones:

-   Monte la unidad desde la misma cuenta de usuario que contiene la aplicación. Puede usar una herramienta como PsExec.
- Pase el nombre y la clave de la cuenta de almacenamiento en los parámetros de nombre de usuario y contraseña del comando net use.
- Use el comando cmdkey para agregar las credenciales en el Administrador de credenciales. Haga esta operación desde una línea de comandos en el contexto de la cuenta de servicio, mediante un inicio de sesión interactivo o mediante `runas`.
  
  `cmdkey /add:<storage-account-name>.file.core.windows.net /user:AZURE\<storage-account-name> /pass:<storage-account-key>`
- Asigne el recurso compartido directamente sin usar una letra de unidad asignada. Algunas aplicaciones pueden no volver a conectarse a la letra de unidad correctamente, por lo que resultará más confiable usar la ruta de acceso UNC completa. 

  `net use * \\storage-account-name.file.core.windows.net\share`

Después de seguir estas instrucciones, podría recibir el mensaje de error siguiente cuando ejecute net use para la cuenta de servicio de red o del sistema: "Error de sistema 1312. Una sesión de inicio especificada no existe. Es posible que haya finalizado." En este caso, asegúrese de que el nombre de usuario pasado a net use incluya la información de dominio (por ejemplo, "[nombre de la cuenta de almacenamiento].file.core.windows.net").

<a id="doesnotsupportencryption"></a>
## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>Error "El archivo se va a copiar a un destino que no es compatible con el cifrado"

Cuando se copia un archivo a través de la red, el archivo se descifra en el equipo de origen, se transmite en texto no cifrado y se vuelve a cifrar en el destino. Sin embargo, es posible que vea el siguiente error al intentar copiar un archivo cifrado: "El archivo se va a copiar a un destino que no es compatible con el cifrado".

### <a name="cause"></a>Causa
Este problema puede producirse si está usando Sistema de cifrado de archivos (EFS). Es posible copiar archivos cifrados por BitLocker en Azure Files. En cambio, Azure Files no admite NTFS EFS.

### <a name="workaround"></a>Solución alternativa
Para copiar un archivo a través de la red, primero debe descifrarlo. Utilice uno de los métodos siguientes:

- Use el comando **copy /d**. Permite que los archivos cifrados se guarden como archivos descifrados en el destino.
- Establezca la siguiente clave del Registro:
  - Ruta de acceso = HKLM\Software\Policies\Microsoft\Windows\System
  - Tipo de valor = DWORD
  - Nombre = CopyFileAllowDecryptedRemoteDestination
  - Valor = 1

Tenga en cuenta que la configuración de la clave del Registro afecta a todas las operaciones de copia llevadas a cabo en recursos compartidos de red.

## <a name="slow-enumeration-of-files-and-folders"></a>Enumeración lenta de archivos y carpetas

### <a name="cause"></a>Causa

Este problema puede producirse si hay no hay suficiente memoria caché en el equipo cliente para directorios grandes.

### <a name="solution"></a>Solución

Para resolver este problema, ajuste el valor de registro **DirectoryCacheEntrySizeMax** para permitir almacenar en caché listados de directorios más grandes en el equipo cliente:

- Ubicación: HKLM\System\CCS\Services\Lanmanworkstation\Parameters
- Nombre del valor: DirectoryCacheEntrySizeMax 
- Tipo de valor: DWORD
 
 
Por ejemplo, puede establecerlo en 0x100000 y ver si el rendimiento mejora.

## <a name="error-aaddstenantnotfound-in-enabling-azure-active-directory-domain-service-azure-ad-ds-authentication-for-azure-files-unable-to-locate-active-tenants-with-tenant-id-aad-tenant-id"></a>Error AadDsTenantNotFound al habilitar la autenticación de Azure Active Directory Domain Service (Azure AD DS) para Azure Files "No se pueden encontrar inquilinos activos con el id. de inquilino aad-tenant-id"

### <a name="cause"></a>Causa

El error AadDsTenantNotFound se produce al intentar [habilitar la autenticación de Azure Active Directory Domain Services (Azure AD DS) en Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md) en una cuenta de almacenamiento donde [Azure AD Domain Service (Azure AD DS)](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-overview) no se ha creado en el inquilino de Azure AD de la suscripción asociada.  

### <a name="solution"></a>Solución

Habilite Azure AD DS en el inquilino de Azure AD de la suscripción donde se ha implementado la cuenta de almacenamiento. Necesita privilegios de administrador del inquilino de Azure AD para crear un dominio administrado. Si no es el administrador del inquilino de Azure AD, póngase en contacto con el administrador y siga las instrucciones paso a paso para [habilitar Azure Active Directory Domain Services mediante Azure Portal](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-getting-started).

[!INCLUDE [storage-files-condition-headers](../../../includes/storage-files-condition-headers.md)]

## <a name="error-system-error-1359-has-occurred-an-internal-error-received-over-smb-access-to-file-shares-with-azure-active-directory-domain-service-azure-ad-ds-authentication-enabled"></a>El error "Error de sistema 1359. Error interno" recibido a través del acceso de SMB a los recursos compartidos de archivos con la autenticación de Azure Active Directory Domain Service (Azure AD DS) habilitada

### <a name="cause"></a>Causa

El error "Error de sistema 1359. Error interno" se produce al intentar conectarse al recurso compartido de archivos con la autenticación de Azure AD DS habilitada en una instancia de Azure AD DS con un nombre DNS de dominio que empieza con un carácter numérico. Por ejemplo, si el nombre DNS del dominio de Azure AD DS es "1domain", recibe este error al intentar montar el recurso compartido de archivos con las credenciales de Azure AD. 

### <a name="solution"></a>Solución

Actualmente, puede considerar la posibilidad de volver a implementar la instancia de Azure AD DS con un nombre DNS de dominio nuevo que se aplique con las reglas siguientes:
- Los nombres no pueden empezar con un carácter numérico.
- Los nombres deben tener entre 3 y 63 caracteres de longitud.

## <a name="unable-to-mount-azure-files-with-ad-credentials"></a>No se puede montar Azure Files con las credenciales de AD 

### <a name="self-diagnostics-steps"></a>Pasos del diagnóstico automático
En primer lugar, asegúrese de que ha seguido los cuatro pasos para [habilitar la autenticación de Azure Files AD](https://docs.microsoft.com/azure/storage/files/storage-files-identity-auth-active-directory-enable).

En segundo lugar, pruebe a [montar un recurso compartido de archivos de Azure con la clave de la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-windows). Si se produce un error al montar, descargue [AzFileDiagnostics.ps1](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) para ayudarle a validar el cliente que ejecuta el entorno, detectar la configuración de cliente incompatible que podría provocar un error de acceso en Azure Files, proporcionar instrucciones preceptivas sobre la solución autónoma de problemas y recopilar los seguimientos de diagnóstico.

En tercer lugar, puede ejecutar el cmdlet Debug-AzStorageAccountAuth para realizar un conjunto de comprobaciones básicas en la configuración de AD con el usuario de AD que ha iniciado sesión. Este cmdlet se admite en las [versiones posteriores a AzFilesHybrid v 0.1.2](https://github.com/Azure-Samples/azure-files-samples/releases). Este cmdlet se debe ejecutar con un usuario de AD que tenga permisos de propietario en la cuenta de almacenamiento de destino.  
```PowerShell
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```
El cmdlet realiza estas comprobaciones de forma secuencial y proporciona instrucciones que seguir cuando aparezcan errores:
1. CheckADObjectPasswordIsCorrect: asegúrese de que la contraseña configurada en la identidad de AD que representa la cuenta de almacenamiento coincide con la de la clave kerb1 o kerb2 de la cuenta de almacenamiento. Si la contraseña es incorrecta, puede ejecutar [Update-AzStorageAccountADObjectPassword](https://docs.microsoft.com/azure/storage/files/storage-files-identity-ad-ds-update-password) para restablecer la contraseña. 
2. CheckADObject: confirme que hay un objeto en Active Directory que representa la cuenta de almacenamiento y que tiene el SPN correcto (nombre de entidad de seguridad de servicio). Si SPN no se configura correctamente, ejecute el cmdlet Set-AD devuelto en el cmdlet debug para configurar SPN.
3. CheckDomainJoined: compruebe que la máquina cliente está unida por el dominio a AD. Si el equipo no está unido a un dominio deAD, consulte este [artículo](https://docs.microsoft.com/windows-server/identity/ad-fs/deployment/join-a-computer-to-a-domain#:~:text=To%20join%20a%20computer%20to%20a%20domain&text=Navigate%20to%20System%20and%20Security,join%2C%20and%20then%20click%20OK) para obtener instrucciones sobre la unión a un dominio.
4. CheckPort445Connectivity: compruebe que el puerto 445 está abierto para la conexión SMB. Si el puerto requerido no está abierto, consulte la herramienta de solución de problemas [AzFileDiagnostics.ps1](https://gallery.technet.microsoft.com/Troubleshooting-tool-for-a9fa1fe5) para ver los problemas de conectividad con Azure Files.
5. CheckSidHasAadUser: compruebe que el usuario de AD que ha iniciado sesión está sincronizado con Azure AD. Si desea buscar si un usuario de AD específico está sincronizado con Azure AD, puede especificar el nombre de usuario y el dominio en los parámetros de entrada. 
6. CheckGetKerberosTicket: intente obtener un vale de Kerberos para conectarse a la cuenta de almacenamiento. Si no hay un token de Kerberos válido, ejecute el cmdlet klist get cifs/storage-account-name.file.core.windows.net y examine el código de error para la causa principal del error de recuperación de vales.
7. CheckStorageAccountDomainJoined: Compruebe si se ha habilitado la autenticación de AD y si se han rellenado las propiedades de AD de la cuenta. Si no es así, consulte la instrucción [aquí](https://docs.microsoft.com/azure/storage/files/storage-files-identity-ad-ds-enable) para habilitar la autenticación de AD DS en Azure Files. 

## <a name="unable-to-configure-directoryfile-level-permissions-windows-acls-with-windows-file-explorer"></a>No se pueden configurar los permisos de nivel de directorio/archivo (ACL de Windows) con el Explorador de archivos de Windows.

### <a name="symptom"></a>Síntoma

Puede experimentar cualquiera de los síntomas que se describen a continuación al intentar configurar las ACL de Windows con el Explorador de archivos en un recurso compartido de archivos montado:
- Después de hacer clic en Editar permiso en la pestaña Seguridad, el Asistente para permisos no se carga. 
- Al intentar seleccionar un nuevo usuario o grupo, la ubicación del dominio no muestra el dominio de AD DS correcto. 

### <a name="solution"></a>Solución

Se recomienda usar la [herramienta icacls](https://docs.microsoft.com/windows-server/administration/windows-commands/icacls) para configurar los permisos de nivel de directorio o archivo como solución alternativa. 

## <a name="errors-when-running-join-azstorageaccountforauth-cmdlet"></a>Errores al ejecutar el cmdlet Join-AzStorageAccountForAuth

### <a name="error-the-directory-service-was-unable-to-allocate-a-relative-identifier"></a>Error: "El servicio de directorio no puede asignar un identificador relativo".

Este error puede producirse si un controlador de dominio que contiene el rol FSMO del maestro de RID no está disponible o se quitó del dominio y se restauró a partir de una copia de seguridad.  Confirme que todos los controladores de dominio están en ejecución y se encuentran disponibles.

### <a name="error-cannot-bind-positional-parameters-because-no-names-were-given"></a>Error: "No se pueden enlazar los parámetros de posición porque no se proporcionaron nombres".

Lo más probable es que este error se haya desencadenado debido a un error de sintaxis en el comando Join-AzStorageAccountforAuth.  Compruebe si hay errores ortográficos o de sintaxis en el comando y compruebe que está instalada la última versión del módulo AzFilesHybrid (https://github.com/Azure-Samples/azure-files-samples/releases) ).  

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.
Si sigue necesitando ayuda, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.
