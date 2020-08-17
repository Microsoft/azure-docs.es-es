---
title: Tutorial`:` Uso de Managed Service Identity en una máquina virtual Windows para tener acceso a Azure Storage con una credencial SAS en Azure AD
description: En este tutorial, se explica cómo se utilizan las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Storage utilizando las credenciales de SAS en lugar de una clave de acceso de la cuenta de almacenamiento.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/24/2019
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: e6ca5ec32e1f88572812b19cf08d4c6f9dc70af6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87018583"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage-via-a-sas-credential"></a>Tutorial: Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Storage utilizando una credencial de SAS

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Este tutorial muestra cómo usar una identidad asignada por el sistema para una máquina virtual Windows a fin de obtener una credencial de Firma de acceso compartido (SAS) del almacenamiento. En concreto, una [credencial SAS de servicio](/azure/storage/common/storage-dotnet-shared-access-signature-part-1?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures). 

Una credencial SAS de servicio permite conceder acceso limitado a los objetos de una cuenta de almacenamiento, durante un período de tiempo limitado y con un servicio específico (en nuestro caso, el servicio blob), sin exponer una clave de acceso a la cuenta. Puede usar una credencial SAS de la forma habitual al realizar operaciones de almacenamiento, por ejemplo, al usar el SDK de Storage. En este tutorial, mostramos cómo cargar y descargar un blob mediante PowerShell de Azure Storage. Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de almacenamiento
> * Conceder acceso a la máquina virtual a una SAS de cuenta de almacenamiento en Resource Manager 
> * Obtener un token de acceso mediante la identidad de la máquina virtual y utilizarlo para recuperar la credencial SAS desde Resource Manager 

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento 

Si aún no tiene una, creará ahora una cuenta de almacenamiento. También puede omitir este paso y conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso a la credencial de SAS de una cuenta de almacenamiento existente. 

1. Haga clic en el botón **+/Crear nuevo servicio** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Storage**, a continuación, en **Cuenta de almacenamiento** y se mostrará un nuevo panel "Crear cuenta de almacenamiento".
3. Escriba un nombre para la cuenta de almacenamiento, que utilizará más adelante.  
4. **Modelo de implementación** y **Clase de cuenta** debe establecerse en "Resource Manager" y "Uso General", respectivamente. 
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.
6. Haga clic en **Crear**.

    ![Creación de una nueva cuenta de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>Creación de un contenedor de blobs en la cuenta de almacenamiento

Más adelante se cargará y descargará un archivo a la nueva cuenta de almacenamiento. Dado que los archivos requieren almacenamiento de blobs, es necesario crear un contenedor de blobs en el que almacenar el archivo.

1. Vuelva a la cuenta de almacenamiento recién creada.
2. Haga clic en el vínculo **Contenedores** en el panel de la izquierda, en "Blob service".
3. Haga clic en **+ Contenedor** en la parte superior de la página y se abrirá un panel "Nuevo contenedor".
4. Asigne un nombre al contenedor, seleccione un nivel de acceso y, a continuación, haga clic en **Aceptar**. El nombre especificado se utilizará más adelante en el tutorial. 

    ![Creación de contenedores de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-a-storage-sas"></a>Concesión de acceso a la identidad administrada asignada por el sistema de la máquina virtual para usar una SAS de almacenamiento 

Azure Storage no admite la autenticación de Azure AD de forma nativa.  No obstante, puede usar una identidad administrada para recuperar una SAS de almacenamiento de Resource Manager y usar la SAS para acceder al almacenamiento.  En este paso, va a conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso a la SAS de la cuenta de almacenamiento.   

1. Vuelva a la cuenta de almacenamiento recién creada.   
2. Haga clic en el vínculo **Control de acceso (IAM)** en el panel izquierdo.  
3. Haga clic en **+ Agregar asignación de rol** en la parte superior de la página para agregar una asignación de roles nueva para la máquina virtual.
4. Establezca **Rol** en "Colaborador de la cuenta de almacenamiento", en el lado derecho de la página.  
5. En el menú desplegable siguiente, establezca **Asignar acceso a** en el recurso "Máquina virtual".  
6. A continuación, asegúrese de que la suscripción adecuada aparece en el menú desplegable **Suscripción** y después, establezca **Grupo de recursos** en "Todos los grupos de recursos".  
7. Por último, en **Seleccionar**, elija la máquina virtual Windows en el menú desplegable y haga clic en **Guardar**. 

    ![Texto alternativo de imagen](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>Obtención de un token de acceso utilizando la identidad de la máquina virtual y su uso para llamar a Azure Resource Manager 

En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente.

En esta parte tendrá que usar los cmdlets de PowerShell de Azure Resource Manager.  Si no lo tiene instalado, [descargue la versión más reciente](https://docs.microsoft.com/powershell/azure/) antes de continuar.

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Windows y, a continuación, desde la página **Información general**, haga clic en **Conectar** en la parte superior.
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la máquina virtual Windows. 
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra PowerShell en la sesión remota. 
4. Mediante el comando Invoke-WebRequest de Powershell, realice una solicitud al punto de conexión local de la identidad administrada de recursos de Azure y obtenga un token de acceso para Azure Resource Manager.

    ```powershell
       $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
    ```
    
    > [!NOTE]
    > El valor del parámetro "resource" debe coincidir exactamente con el que Azure AD espera. Al usar el id. de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI.
    
    A continuación, extraiga el elemento "Content", que se almacena como una cadena con formato de notación de objetos JavaScript (JSON) en el objeto $response. 
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```
    Luego, extraiga el token de acceso de la respuesta.
    
    ```powershell
    $ArmToken = $content.access_token
    ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>Obtención de una credencial SAS desde Azure Resource Manager para realizar llamadas de almacenamiento 

Ahora vamos a utilizar PowerShell para realizar una llamada a Resource Manager mediante el token de acceso que se recuperó en la sección anterior, a fin de crear una credencial SAS de almacenamiento. Una vez que tenemos la credencial SAS de almacenamiento, podemos hacer llamadas de almacenamiento.

Para esta solicitud, vamos a usar los parámetros de solicitud HTTP de seguimiento para crear la credencial SAS:

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite. Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

Estos parámetros se incluyen en el cuerpo POST de la solicitud para la credencial SAS. Para más información sobre los parámetros para crear una credencial SAS, consulte la [referencia de REST de SAS del servicio de lista](/rest/api/storagerp/storageaccounts/listservicesas).

En primer lugar, convierta los parámetros a JSON y después llame al punto de conexión `listServiceSas` del almacenamiento para crear las credenciales SAS:

```powershell
$params = @{canonicalizedResource="/blob/<STORAGE-ACCOUNT-NAME>/<CONTAINER-NAME>";signedResource="c";signedPermission="rcw";signedProtocol="https";signedExpiry="2017-09-23T00:00:00Z"}
$jsonParams = $params | ConvertTo-Json
```

```powershell
$sasResponse = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE-ACCOUNT-NAME>/listServiceSas/?api-version=2017-06-01 -Method POST -Body $jsonParams -Headers @{Authorization="Bearer $ArmToken"}
```
> [!NOTE] 
> La dirección URL distingue mayúsculas de minúsculas, por lo tanto, asegúrese de que usa las mismas mayúsculas y minúsculas que al asignar el nombre al grupo de recursos, así como la "G" mayúscula de "resourceGroups". 

Ahora podemos extraer la credencial SAS de la respuesta:

```powershell
$sasContent = $sasResponse.Content | ConvertFrom-Json
$sasCred = $sasContent.serviceSasToken
```

Si examina la credencial SAS, verá algo parecido a esto:

```powershell
PS C:\> $sasCred
sv=2015-04-05&sr=c&spr=https&se=2017-09-23T00%3A00%3A00Z&sp=rcw&sig=JVhIWG48nmxqhTIuN0uiFBppdzhwHdehdYan1W%2F4O0E%3D
```

A continuación, creamos un archivo llamado "test.txt". Seguidamente, use la credencial SAS para autenticarse con el cmdlet `New-AzStorageContent`, cargue el archivo en el contenedor de blobs y, después, descargue el archivo.

```bash
echo "This is a test text file." > test.txt
```

Asegúrese de instalar en primer lugar los cmdlets de Azure Storage, mediante `Install-Module Azure.Storage`. Después, cargue el blob recién creado utilizando el cmdlet `Set-AzStorageBlobContent` de PowerShell:

```powershell
$ctx = New-AzStorageContext -StorageAccountName <STORAGE-ACCOUNT-NAME> -SasToken $sasCred
Set-AzStorageBlobContent -File test.txt -Container <CONTAINER-NAME> -Blob testblob -Context $ctx
```

Respuesta:

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/21/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

También puede descargar el blob recién creado utilizando el cmdlet `Get-AzStorageBlobContent` de PowerShell:

```powershell
Get-AzStorageBlobContent -Blob testblob -Container <CONTAINER-NAME> -Destination test2.txt -Context $ctx
```

Respuesta:

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/21/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Storage utilizando las credenciales de SAS.  Para obtener más información sobre SAS de Azure Storage, vea:

> [!div class="nextstepaction"]
>[Uso de Firmas de acceso compartido (SAS)](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)


