---
title: Establecimiento de listas de control de acceso (ACL) de forma recursiva para Azure Data Lake Storage Gen2 | Microsoft Docs
description: Puede agregar, actualizar y quitar listas de control de acceso de forma recursiva para los elementos secundarios existentes de un directorio primario.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 08/26/2020
ms.author: normesta
ms.reviewer: prishet
ms.openlocfilehash: cbf5d8286d6f181c69cd090df6cf595934cd547e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88942026"
---
# <a name="set-access-control-lists-acls-recursively-for-azure-data-lake-storage-gen2"></a>Establecimiento de listas de control de acceso (ACL) de forma recursiva para Azure Data Lake Storage Gen2

La herencia de ACL ya está disponible para los nuevos elementos secundarios que se crean en un directorio primario. Ahora también puede agregar, actualizar y quitar las ACL de forma recursiva para los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario.

> [!NOTE]
> La capacidad de establecer listas de acceso de forma recursiva se encuentra en versión preliminar pública y está disponible en todas las regiones.  

[Bibliotecas](#libraries) | [Ejemplos](#code-samples) | [Procedimientos recomendados](#best-practice-guidelines) | [Envíe sus comentarios](#provide-feedback)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](data-lake-storage-quickstart-create-account.md) instrucciones para crear uno.

- Los permisos correctos para ejecutar el proceso de ACL recursivo. El permiso correcto incluye una de las siguientes opciones: 

  - Una [entidad de seguridad](https://docs.microsoft.com/azure/role-based-access-control/overview#security-principal) aprovisionada de Azure Active Directory a la que se ha asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, el grupo de recursos primario o la suscripción.   

  - El usuario propietario del contenedor o directorio de destino al que va a aplicar el proceso de ACL recursivo. Esto incluye todos los elementos secundarios del contenedor o directorio de destino. 

- Una descripción de cómo se aplican las ACL a los directorios y archivos. Consulte [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control). 

Consulte la sección **Configurar su proyecto** de este artículo para ver la guía de instalación de PowerShell, el SDK de .NET y el SDK de Python.

## <a name="set-up-your-project"></a>Configurar su proyecto

Instale las bibliotecas necesarias.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Asegúrese de que .NET Framework está instalado. Vea [Descargar .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).
 
2. Use el siguiente comando para comprobar que la versión de PowerShell que ha instalado es `5.1` o posterior.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Para actualizar la versión de PowerShell, vea [Actualización de Windows PowerShell existente](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-6#upgrading-existing-windows-powershell).
    
3. Instale la versión más reciente del módulo PowerShellGet.

   ```powershell
   install-Module PowerShellGet –Repository PSGallery –Force  
   ```

4. Cierre la consola de PowerShell y vuelva a abrirla.

5. Instale el módulo de versión preliminar **Az.Storage**.

   ```powershell
   Install-Module Az.Storage -Repository PsGallery -RequiredVersion 2.5.2-preview -AllowClobber -AllowPrerelease -Force  
   ```

   Para más información sobre cómo instalar módulos de PowerShell, vea [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.0.0).

### <a name="net"></a>[.NET](#tab/dotnet)

1. Abra una ventana de comando (por ejemplo: Windows PowerShell).

2. En el directorio del proyecto, instale el paquete Azure.Storage.Files.DataLake de la versión preliminar mediante el comando `dotnet add package`.

   ```console
   dotnet add package Azure.Storage.Files.DataLake -v 12.3.0-dev.20200811.1 -s https://azuresdkartifacts.blob.core.windows.net/azure-sdk-for-net/index.json
   ```

3. Agregue estas instrucciones "using" al inicio del archivo de código.

   ```csharp
   using Azure;
   using Azure.Core;
   using Azure.Storage;
   using Azure.Storage.Files.DataLake;
   using Azure.Storage.Files.DataLake.Models;
   using System.Collections.Generic;
   using System.Threading.Tasks;
    ```

### <a name="python"></a>[Python](#tab/python)

1. Descargue la [biblioteca cliente de Azure Data Lake Storage para Python](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2Fazure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A47%253A01Z%26se%3D2021-08-25T07%253A47%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DH1XYw4FTLJse%252BYQ%252BfamVL21UPVIKRnnh2mfudA%252BfI0I%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C95a5966d938a4902560e08d84912fe32%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339693209725909&sdata=acv4KWZdzkITw1lP0%2FiA3lZuW7NF5JObjY26IXttfGI%3D&reserved=0).

2. Instale la biblioteca que descargó mediante [pip](https://pypi.org/project/pip/).

   ```
   pip install azure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl
   ```

Agregue estas instrucciones import al inicio del archivo de código.

```python
import os, uuid, sys
from azure.storage.filedatalake import DataLakeServiceClient
from azure.core._match_conditions import MatchConditions
from azure.storage.filedatalake._models import ContentSettings
```

---

## <a name="connect-to-your-account"></a>Conexión a su cuenta

Puede conectarse mediante Azure Active Directory o mediante una clave de cuenta. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Abra una ventana de comandos de Windows PowerShell y, a continuación, inicie sesión en su suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

```powershell
Connect-AzAccount
```

Si su identidad está asociada a más de una suscripción, establezca la suscripción activa en la suscripción de la cuenta de almacenamiento en la que quiere crear y administrar directorios. En este ejemplo, reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

```powershell
Select-AzSubscription -SubscriptionId <subscription-id>
```

A continuación, elija cómo desea que sus comandos obtengan autorización para la cuenta de almacenamiento. 

### <a name="option-1-obtain-authorization-by-using-azure-active-directory-ad"></a>Opción 1: Obtención de autorización mediante Azure Active Directory (AD)

Con este enfoque, el sistema garantiza que su cuenta de usuario tiene los permisos de ACL y las asignaciones de control de acceso basado en rol (RBAC). 

```powershell
$ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -UseConnectedAccount
```

En la tabla siguiente se muestra cada uno de los roles admitidos y su capacidad de configuración de ACL.

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Opción 2: Obtención de autorización mediante la clave de cuenta de almacenamiento

Con este enfoque, el sistema no comprueba los permisos ACL o de RBAC.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
$ctx = $storageAccount.Context
```

### <a name="net"></a>[.NET](#tab/dotnet)

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de [DataLakeServiceClient](https://docs.microsoft.com/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) que represente la cuenta de almacenamiento.

#### <a name="connect-by-using-azure-active-directory-ad"></a>Conexión con Azure Active Directory (AD)

Puede usar la [biblioteca cliente de identidad de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) para autenticar la aplicación con Azure AD.

Después de instalar el paquete, agregue esta instrucción "using" al principio del archivo de código.

```csharp
using Azure.Identity;
```

Obtenga un identificador de cliente, un secreto de cliente y un identificador de inquilino. Para ello, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md). Como parte de ese proceso, tendrá que asignar uno de los siguientes roles de [control de acceso basado en rol](../../role-based-access-control/overview.md) a la entidad de seguridad. 

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

En este ejemplo se crea una instancia de [DataLakeServiceClient](https://docs.microsoft.com/dotnet/api/azure.storage.files.datalake.datalakeserviceclient?) con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  

```cs
public void GetDataLakeServiceClient(ref DataLakeServiceClient dataLakeServiceClient, 
    String accountName, String clientID, string clientSecret, string tenantID)
{

    TokenCredential credential = new ClientSecretCredential(
        tenantID, clientID, clientSecret, new TokenCredentialOptions());

    string dfsUri = "https://" + accountName + ".dfs.core.windows.net";

    dataLakeServiceClient = new DataLakeServiceClient(new Uri(dfsUri), credential);
}

``` 

#### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Este método es la manera más sencilla de conectarse a una cuenta. 

En este ejemplo se crea una instancia de [DataLakeServiceClient](https://docs.microsoft.com/dotnet/api/azure.storage.files.datalake.datalakeserviceclient?) con una clave de cuenta.

```cs
public void GetDataLakeServiceClient(ref DataLakeServiceClient dataLakeServiceClient,
    string accountName, string accountKey)
{
    StorageSharedKeyCredential sharedKeyCredential =
        new StorageSharedKeyCredential(accountName, accountKey);

    string dfsUri = "https://" + accountName + ".dfs.core.windows.net";

    dataLakeServiceClient = new DataLakeServiceClient
        (new Uri(dfsUri), sharedKeyCredential);
}
```

> [!NOTE]
> Para ver más ejemplos, consulte la documentación de la [biblioteca cliente de identidad de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity).

### <a name="python"></a>[Python](#tab/python)

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de **DataLakeServiceClient** que represente la cuenta de almacenamiento. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Conexión con Azure Active Directory (AD)

Puede usar la [biblioteca cliente de identidad de Azure para Python](https://pypi.org/project/azure-identity/) para autenticar la aplicación con Azure AD.

En este ejemplo se crea una instancia de **DataLakeServiceClient** con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  Para obtener estos valores, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md). Como parte de ese proceso, tendrá que asignar uno de los siguientes roles de [control de acceso basado en rol](../../role-based-access-control/overview.md) a la entidad de seguridad. 

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

```python
def initialize_storage_account_ad(storage_account_name, client_id, client_secret, tenant_id):
    
    try:  
        global service_client

        credential = ClientSecretCredential(tenant_id, client_id, client_secret)

        service_client = DataLakeServiceClient(account_url="{}://{}.dfs.core.windows.net".format(
            "https", storage_account_name), credential=credential)
    
    except Exception as e:
        print(e)
```

> [!NOTE]
> Para ver más ejemplos, consulte la documentación de la [biblioteca cliente de identidad de Azure para Python](https://pypi.org/project/azure-identity/).

### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Es la manera más sencilla de conectarse a una cuenta. 

En este ejemplo se crea una instancia de **DataLakeServiceClient** con una clave de cuenta.

```python
try:  
    global service_client
        
    service_client = DataLakeServiceClient(account_url="{}://{}.dfs.core.windows.net".format(
        "https", storage_account_name), credential=storage_account_key)
    
except Exception as e:
    print(e)
```
 
- Reemplace el valor de marcador de posición `storage_account_name` por el nombre de la cuenta de almacenamiento.

- Reemplace el valor de marcador de posición `storage_account_key` por la clave de acceso de la cuenta de almacenamiento.

---

## <a name="set-an-acl-recursively"></a>Establecimiento de una ACL de forma recursiva

Puede establecer las ACL de forma recursiva.  

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Establezca una ACL de forma recursiva mediante el cmdlet `Set-AzDataLakeGen2AclRecursive`.

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. En estas entradas se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission r-x -InputObject $acl 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "---" -InputObject $acl
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission r-x -InputObject $acl 

Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

### <a name="net"></a>[.NET](#tab/dotnet)

Establezca una ACL de forma recursiva llamando al método **DataLakeDirectoryClient.SetAccessControlRecursiveAsync**. Pase a este método una [lista](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItems](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Cada [PathAccessControlItems](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) define una entrada de ACL.

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. En estas entradas se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

```cs
public async void SetACLRecursively(DataLakeServiceClient serviceClient)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
            GetDirectoryClient("my-parent-directory");

    List<PathAccessControlItem> accessControlList = 
        new List<PathAccessControlItem>() 
    {
        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read | 
            RolePermissions.Write | 
            RolePermissions.Execute),
                    
        new PathAccessControlItem(AccessControlType.Group, 
            RolePermissions.Read | 
            RolePermissions.Execute),
                    
        new PathAccessControlItem(AccessControlType.Other, 
            RolePermissions.None),

        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read | 
            RolePermissions.Execute, 
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
    };

    await directoryClient.SetAccessControlRecursiveAsync
        (accessControlList, null);
}

```

### <a name="python"></a>[Python](#tab/python)

Establezca una ACL de forma recursiva llamando al método **DataLakeDirectoryClient.set_access_control_recursive**.

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. En estas entradas se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

```python
def set_permission_recursively():
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user::rwx,group::rwx,other::rwx,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x'

        directory_client.set_access_control_recursive(acl=acl)
        
    except Exception as e:
     print(e)
```

---

## <a name="update-an-acl-recursively"></a>Actualización de una ACL de forma recursiva

Puede actualizar una ACL existente de forma recursiva.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Actualice una ACL de forma recursiva mediante el cmdlet **Update-AzDataLakeGen2AclRecursive**. 

En este ejemplo se actualiza una entrada de ACL con permiso de escritura.

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission rwx

Update-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

### <a name="net"></a>[.NET](#tab/dotnet)

Actualice una ACL de forma recursiva llamando al método **DataLakeDirectoryClient.UpdateAccessControlRecursiveAsync**. 

En este ejemplo se actualiza una entrada de ACL con permiso de escritura. 

```cs
public async void UpdateACLsRecursively(DataLakeServiceClient serviceClient)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
        GetDirectoryClient("my-parent-directory");

    List<PathAccessControlItem> accessControlListUpdate = 
        new List<PathAccessControlItem>()
    {
        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read |
            RolePermissions.Write | 
            RolePermissions.Execute, 
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
    };

    await directoryClient.UpdateAccessControlRecursiveAsync
        (accessControlListUpdate, null);

}
```

### <a name="python"></a>[Python](#tab/python)

Actualice una ACL de forma recursiva llamando al método **DataLakeDirectoryClient.update_access_control_recursive**. 

En este ejemplo se actualiza una entrada de ACL con permiso de escritura. 

```python
def update_permission_recursively():
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx'

        directory_client.update_access_control_recursive(acl=acl)

        acl_props = directory_client.get_access_control()
        
        print(acl_props['permissions'])

    except Exception as e:
     print(e)
```

---

## <a name="remove-acl-entries-recursively"></a>Eliminación de las entradas de ACL de forma recursiva

Puede quitar una o varias entradas de ACL de forma recursiva.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Quite las entradas de ACL mediante el cmdlet **Remove-AzDataLakeGen2AclRecursive**. 

En este ejemplo se quita una entrada de ACL del directorio raíz del contenedor.  

```powershell
$filesystemName = "my-container"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID

Remove-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName  -Acl $acl
```

### <a name="net"></a>[.NET](#tab/dotnet)

Elimine entradas de ACL llamando al método **DataLakeDirectoryClient.RemoveAccessControlRecursiveAsync**. 

En este ejemplo se quita una entrada de la ACL del directorio llamado `my-parent-directory`. 

```cs
public async void RemoveACLsRecursively(DataLakeServiceClient serviceClient)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
            GetDirectoryClient("my-parent-directory");

    List<RemovePathAccessControlItem> accessControlListForRemoval = 
        new List<RemovePathAccessControlItem>()
        {
            new RemovePathAccessControlItem(AccessControlType.User, 
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
        };

    await directoryClient.RemoveAccessControlRecursiveAsync
        (accessControlListForRemoval, null);

}
```

### <a name="python"></a>[Python](#tab/python)

Elimine entradas de ACL llamando al método **DataLakeDirectoryClient.remove_access_control_recursive**. 

En este ejemplo se quita una entrada de la ACL del directorio llamado `my-parent-directory`. 

```python
def remove_permission_recursively():
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user:4a9028cf-f779-4032-b09d-970ebe3db258'

        directory_client.remove_access_control_recursive(acl=acl)

    except Exception as e:
     print(e)
```

---

## <a name="recover-from-failures"></a>Recuperación de errores

Podría encontrar errores de tiempo de ejecución o de permisos. En el caso de los errores de tiempo de ejecución, reinicie el proceso desde el principio. Los errores de permisos pueden producirse si la entidad de seguridad no tiene permisos suficientes para modificar la ACL de un directorio o archivo que se encuentra en la jerarquía de directorios que se está modificando. Solucione el problema de permisos y, a continuación, elija reanudar el proceso desde el punto de error mediante un token de continuación, o bien reinicie el proceso desde el principio. No tiene que usar el token de continuación si prefiere reiniciar desde el principio. Puede volver a aplicar las entradas de ACL sin ningún efecto negativo.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los resultados se devuelven a la variable. La canalización no pudo enviar las entradas a una tabla con formato.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$result
$result.FailedEntries | ft 
```

Según la salida de la tabla, puede corregir todos los errores de permisos y, a continuación, reanudar la ejecución mediante el token de continuación.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl -ContinuationToken $result.ContinuationToken
$result

```

## <a name="net"></a>[.NET](#tab/dotnet)

Este ejemplo devuelve un token de continuación en caso de que se produzca un error. La aplicación puede volver a llamar a este método de ejemplo después de que se haya solucionado el error y pasar el token de continuación. Si se llama a este método de ejemplo por primera vez, la aplicación puede pasar un valor de ``null`` para el parámetro del token de continuación. 

```cs
public async Task<string> ResumeAsync(DataLakeServiceClient serviceClient,
    DataLakeDirectoryClient directoryClient,
    List<PathAccessControlItem> accessControlList, 
    string continuationToken)
{
    try
    {
        var accessControlChangeResult =
            await directoryClient.SetAccessControlRecursiveAsync(
                accessControlList, null, continuationToken: continuationToken);

        if (accessControlChangeResult.Value.Counters.FailedChangesCount > 0)
        {
            continuationToken =
                accessControlChangeResult.Value.ContinuationToken;
        }

        return continuationToken;
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        return continuationToken;
    }

}
```

### <a name="python"></a>[Python](#tab/python)

Este ejemplo devuelve un token de continuación en caso de que se produzca un error. La aplicación puede volver a llamar a este método de ejemplo después de que se haya solucionado el error y pasar el token de continuación. Si se llama a este método de ejemplo por primera vez, la aplicación puede pasar un valor de ``None`` para el parámetro del token de continuación. 

```python
def resume_set_acl_recursive(continuation_token):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user::rwx,group::rwx,other::rwx,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x'

        acl_change_result = directory_client.set_access_control_recursive(acl=acl, continuation=continuation_token)

        continuation_token = acl_change_result.continuation

        return continuation_token
        
    except Exception as e:
     print(e) 
     return continuation_token
```

---

## <a name="resources"></a>Recursos

Esta sección contiene vínculos a bibliotecas y ejemplos de código.

#### <a name="libraries"></a>Bibliotecas

- [PowerShell](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fwww.powershellgallery.com%2Fpackages%2FAz.Storage%2F2.5.2-preview&data=02%7C01%7Cnormesta%40microsoft.com%7Ccdabce06132c42132b4008d849a2dfb1%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637340311173215017&sdata=FWynO9UKTt7ESMCFgkWaL7J%2F%2BjODaRo5BD6G6yCx9os%3D&reserved=0)
- [.NET](https://azuresdkartifacts.blob.core.windows.net/azure-sdk-for-net/index.json)
- [Python](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2Fazure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A47%253A01Z%26se%3D2021-08-25T07%253A47%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DH1XYw4FTLJse%252BYQ%252BfamVL21UPVIKRnnh2mfudA%252BfI0I%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C95a5966d938a4902560e08d84912fe32%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339693209725909&sdata=acv4KWZdzkITw1lP0%2FiA3lZuW7NF5JObjY26IXttfGI%3D&reserved=0)

#### <a name="code-samples"></a>Ejemplos de código

- PowerShell: [Léame](https://recursiveaclpr.blob.core.windows.net/privatedrop/README.txt?sv=2019-02-02&st=2020-08-24T17%3A03%3A18Z&se=2021-08-25T17%3A03%3A00Z&sr=b&sp=r&sig=sPdKiCSXWExV62sByeOYqBTqpGmV2h9o8BLij3iPkNQ%3D) | [Ejemplo](https://recursiveaclpr.blob.core.windows.net/privatedrop/samplePS.ps1?sv=2019-02-02&st=2020-08-24T17%3A04%3A44Z&se=2021-08-25T17%3A04%3A00Z&sr=b&sp=r&sig=dNNKS%2BZcp%2F1gl6yOx6QLZ6OpmXkN88ZjBeBtym1Mejo%3D)

- NET: [Léame](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FREADME%2520for%2520net%3Fsv%3D2019-02-02%26st%3D2020-08-25T23%253A20%253A42Z%26se%3D2021-08-26T23%253A20%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DKrnHvasHoSoVeUyr2g%252FSc2aDVW3De4A%252Fvx0lFWZs494%253D&data=02%7C01%7Cnormesta%40microsoft.com%7Cda902e4fe6c24e6a07d908d8494fd4bd%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339954503767961&sdata=gd%2B2LphTtDFVb7pZko9rkGO9OG%2FVvmeXprHB9IOEYXE%3D&reserved=0) | [Ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0)

- Python: [Léame](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FREADME%2520for%2520python%3Fsv%3D2019-02-02%26st%3D2020-08-25T23%253A21%253A47Z%26se%3D2021-08-26T23%253A21%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DRq6Bl5lXrtYk79thy8wX7UTbjyd2f%252B6xzVBFFVYbdYg%253D&data=02%7C01%7Cnormesta%40microsoft.com%7Cda902e4fe6c24e6a07d908d8494fd4bd%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339954503777915&sdata=3e46Lp2miOHj755Gh0odH3M0%2BdTF3loGCCBENrulVTM%3D&reserved=0) | [Ejemplo](https://recursiveaclpr.blob.core.windows.net/privatedrop/datalake_samples_access_control_async.py?sv=2019-02-02&st=2020-08-24T07%3A48%3A10Z&se=2021-08-25T07%3A48%3A00Z&sr=b&sp=r&sig=%2F1c540%2BpXYyNcuTmWPWHg2m9SyClXLIMw7ChLZGsyD0%3D)

## <a name="best-practice-guidelines"></a>Guías de procedimientos recomendados

En esta sección se proporcionan algunas directrices de procedimientos recomendados para configurar las ACL de forma recursiva. 

#### <a name="handling-runtime-errors"></a>Manejo de errores de tiempo de ejecución

Se puede producir un error de tiempo de ejecución por muchos motivos (por ejemplo: una interrupción o un problema de conectividad de cliente). Si se produce un error de tiempo de ejecución, reinicie el proceso de ACL recursivo. Las ACL se pueden volver a aplicar a los elementos sin provocar ningún efecto negativo. 

#### <a name="handling-permission-errors-403"></a>Manejo de errores de permisos (403)

Si experimenta una excepción de control de acceso al ejecutar un proceso de ACL recursivo, es posible que la [entidad de seguridad](https://docs.microsoft.com/azure/role-based-access-control/overview#security-principal) de AD no tenga permisos suficientes para aplicar una ACL a uno o varios de los elementos secundarios de la jerarquía de directorios. Cuando se produce un error de permiso, el proceso se detiene y se proporciona un token de continuación. Repare el problema de permisos y, después, use el token de continuación para procesar el conjunto de datos restante. Los directorios y archivos que ya se han procesado correctamente no tendrán que procesarse de nuevo. También puede optar por reiniciar el proceso de ACL recursivo. Las ACL se pueden volver a aplicar a los elementos sin provocar ningún efecto negativo. 

#### <a name="credentials"></a>Credenciales 

Se recomienda que aprovisione una entidad de seguridad de Azure AD a la que se haya asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito de la cuenta de almacenamiento o el contenedor de destino. 

#### <a name="performance"></a>Rendimiento 

Para reducir la latencia, se recomienda ejecutar el proceso de ACL recursivo en una máquina virtual de Azure que se encuentre en la misma región que la cuenta de almacenamiento. 

#### <a name="acl-limits"></a>Límites de ACL

El número máximo de ACL que puede aplicar a un directorio o archivo es de 32 ACL de acceso y 32 ACL predeterminadas. Para más información, consulte [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control).

<a id="provide-feedback"></a>

### <a name="provide-feedback-or-report-issues"></a>Envío de comentarios o notificación de problemas

Puede enviar sus comentarios o notificar un problema mediante cualquiera de estas páginas: [PowerShell](https://github.com/Azure/azure-powershell/issues/new?assignees=&labels=triage&template=az-module-bug-report.md&title=), [.NET](https://github.com/Azure/azure-sdk-for-net/issues/new?assignees=&labels=&template=bug_report.md&title=), [Python](https://github.com/Azure/azure-sdk-for-python/issues/new?assignees=&labels=&template=bug_report.md&title=)

## <a name="see-also"></a>Consulte también

- [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control)
- [Problemas conocidos](data-lake-storage-known-issues.md)


