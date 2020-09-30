---
title: PowerShell de Azure Data Lake Storage Gen2 para archivos y ACL
description: Use cmdlets de PowerShell para administrar directorios y listas de control de acceso (ACL) de archivos y directorios en cuentas de almacenamiento que tengan habilitado un espacio de nombres jerárquico (HNS).
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 08/26/2020
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3343f0e21cdf8873447bd448c200102940b632e8
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077099"
---
# <a name="use-powershell-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>Uso de PowerShell para administrar directorios, archivos y ACL en Azure Data Lake Storage Gen2

En este artículo se explica cómo usar PowerShell para crear y administrar directorios, archivos y permisos en cuentas de almacenamiento que tengan habilitado un espacio de nombres jerárquico (HNS). 

[Referencia](https://docs.microsoft.com/powershell/module/Az.Storage/?view=azps-4.5.0) | [Asignación de Gen1 a Gen2](#gen1-gen2-map) | [Envíenos su comentarios](https://github.com/Azure/azure-powershell/issues)

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
> * Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](data-lake-storage-quickstart-create-account.md) instrucciones para crear uno.
> * Debe tener instalada la versión de .NET Framework 4.7.2 o superior. Vea [Descargar .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).
> * La versión de PowerShell `5.1` o posterior.

## <a name="install-the-powershell-module"></a>Instalación del módulo de PowerShell

1. Use el siguiente comando para comprobar que la versión de PowerShell que ha instalado es `5.1` o posterior.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Para actualizar la versión de PowerShell, vea [Actualización de Windows PowerShell existente](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-6#upgrading-existing-windows-powershell).
    
2. Instale el módulo **Az.Storage**.

   ```powershell
   Install-Module Az.Storage -Repository PSGallery -Force  
   ```

   Para más información sobre cómo instalar módulos de PowerShell, vea [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.0.0).

## <a name="connect-to-the-account"></a>Conexión con la cuenta

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

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Opción 2: Obtención de autorización mediante la clave de cuenta de almacenamiento

Con este enfoque, el sistema no comprueba los permisos ACL o de RBAC.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
$ctx = $storageAccount.Context
```

## <a name="create-a-container"></a>Crear un contenedor

Un contenedor actúa como sistema de archivos para sus archivos. Puede crear uno con el cmdlet `New-AzStorageContainer`. 

En este ejemplo se crea un contenedor denominado `my-file-system`.

```powershell
$filesystemName = "my-file-system"
New-AzStorageContainer -Context $ctx -Name $filesystemName
```

## <a name="create-a-directory"></a>Creación de un directorio

Cree una referencia de directorio con el cmdlet `New-AzDataLakeGen2Item`. 

En este ejemplo se agrega un directorio denominado `my-directory` a un contenedor.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory
```

En este ejemplo se agrega el mismo directorio, pero también se establecen los permisos, la máscara de permisos, los valores de propiedad y los valores de metadatos. 

```powershell
$dir = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory -Permission rwxrwxrwx -Umask ---rwx---  -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
```

## <a name="show-directory-properties"></a>Visualización de las propiedades de directorio

En este ejemplo se obtiene un directorio con el cmdlet `Get-AzDataLakeGen2Item` y, después, se imprimen los valores de propiedad en la consola.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir =  Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
$dir.Permissions
$dir.Group
$dir.Owner
$dir.Properties
$dir.Properties.Metadata
```
> [!NOTE]
> Para obtener el directorio raíz del contenedor, omita el parámetro `-Path`.

## <a name="rename-or-move-a-directory"></a>Cambio de nombre o traslado de un directorio

Cambie el nombre de un directorio o muévalo con el cmdlet `Move-AzDataLakeGen2Item`.

En este ejemplo se cambia el nombre de un subdirectorio de `my-directory` a `my-new-directory`.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-new-directory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -DestFileSystem $filesystemName -DestPath $dirname2
```

> [!NOTE]
> Use el parámetro `-Force` si desea sobrescribir sin preguntar.

En este ejemplo se mueve un directorio denominado `my-directory` a un subdirectorio de `my-directory-2` denominado `my-subdirectory`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-directory-2/my-subdirectory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname1 -DestFileSystem $filesystemName -DestPath $dirname2
```

## <a name="delete-a-directory"></a>Eliminación de un directorio

Elimine un directorio con el cmdlet `Remove-AzDataLakeGen2Item`.

En este ejemplo se elimina un directorio denominado `my-directory`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $dirname 
```

Puede usar el parámetro `-Force` para quitar el archivo sin preguntar.

## <a name="download-from-a-directory"></a>Descarga de un directorio

Descargue un archivo de un directorio con el cmdlet `Get-AzDataLakeGen2ItemContent`.

En este ejemplo se descarga un archivo denominado `upload.txt` de un directorio denominado `my-directory`. 

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$downloadFilePath = "download.txt"
Get-AzDataLakeGen2ItemContent -Context $ctx -FileSystem $filesystemName -Path $filePath -Destination $downloadFilePath
```

## <a name="list-directory-contents"></a>Lista del contenido del directorio

Muestre el contenido de un directorio con el cmdlet `Get-AzDataLakeGen2ChildItem`. Puede usar el parámetro opcional `-OutputUserPrincipalName` para obtener el nombre (en lugar del identificador de objeto) de los usuarios.

En este ejemplo se muestra el contenido de un directorio denominado `my-directory`.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -OutputUserPrincipalName
```

En el ejemplo siguiente se enumeran las propiedades `ACL`, `Permissions`, `Group` y `Owner` de cada elemento del directorio. Se necesita el parámetro `-FetchProperty` para obtener los valores de la propiedad `ACL`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$properties = Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -Recurse -FetchProperty
$properties.ACL
$properties.Permissions
$properties.Group
$properties.Owner
```

> [!NOTE]
> Para mostrar el contenido del directorio raíz del contenedor, omita el parámetro `-Path`.

## <a name="upload-a-file-to-a-directory"></a>Carga de un archivo en un directorio

Cargue un archivo en un directorio con el cmdlet `New-AzDataLakeGen2Item`.

En este ejemplo se carga un archivo denominado `upload.txt` en un directorio denominado `my-directory`. 

```powershell
$localSrcFile =  "upload.txt"
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$destPath = $dirname + (Get-Item $localSrcFile).Name
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Force 
```

En este ejemplo se carga el mismo archivo, pero después se establecen los permisos, la máscara de permisos, los valores de propiedad y los valores de metadatos del archivo de destino. En este ejemplo también se imprimen estos valores en la consola.

```powershell
$file = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Permission rwxrwxrwx -Umask ---rwx--- -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
$file1
$file1.Properties
$file1.Properties.Metadata

```

> [!NOTE]
> Para cargar un archivo en el directorio raíz del contenedor, omita el parámetro `-Path`.

## <a name="show-file-properties"></a>Visualización de las propiedades del archivo

En este ejemplo se obtiene un archivo con el cmdlet `Get-AzDataLakeGen2Item` y, después, se imprimen los valores de propiedad en la consola.

```powershell
$filepath =  "my-directory/upload.txt"
$filesystemName = "my-file-system"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filepath
$file
$file.ACL
$file.Permissions
$file.Group
$file.Owner
$file.Properties
$file.Properties.Metadata
```

## <a name="delete-a-file"></a>Eliminación de un archivo

Elimine un archivo con el cmdlet `Remove-AzDataLakeGen2Item`.

En este ejemplo se elimina un archivo denominado `upload.txt`. 

```powershell
$filesystemName = "my-file-system"
$filepath = "upload.txt"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $filepath 
```

Puede usar el parámetro `-Force` para quitar el archivo sin preguntar.

## <a name="manage-access-permissions"></a>Administración de permisos de acceso

Puede obtener, establecer y actualizar los permisos de acceso de los directorios y archivos. Estos permisos se capturan en las listas de control de acceso (ACL).

> [!NOTE]
> Si usa Azure Active Directory (Azure AD) para autorizar comandos, asegúrese de que la entidad de seguridad tiene asignado el [rol Propietario de datos de blobs de almacenamiento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, vea [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control).

### <a name="get-an-acl"></a>Obtención de una ACL

Obtenga la ACL de un directorio o de un archivo con el cmdlet `Get-AzDataLakeGen2Item`.

En este ejemplo se obtiene la ACL del directorio raíz de un **contenedor** y luego se imprime la ACL en la consola.

```powershell
$filesystemName = "my-file-system"
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

En este ejemplo se obtiene la ACL de un **directorio** y luego se imprime la ACL en la consola.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```

En este ejemplo se obtiene la ACL de un **archivo** y luego se imprime la ACL en la consola.

```powershell
$filePath = "my-directory/upload.txt"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

En la siguiente imagen se muestra la salida después de obtener la ACL de un directorio.

![Obtención de una salida de ACL para el directorio](./media/data-lake-storage-directory-file-acl-powershell/get-acl.png)

En este ejemplo, el usuario propietario tiene permisos de lectura, escritura y ejecución. El grupo propietario tiene permisos de solo lectura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="set-an-acl"></a>Establecimiento de una ACL

Use el cmdlet `set-AzDataLakeGen2ItemAclObject` para crear una ACL para el usuario propietario, el grupo propietario u otros usuarios. Luego, use el cmdlet `Update-AzDataLakeGen2Item` para confirmar la ACL.

En este ejemplo se establece la ACL en el directorio raíz de un **contenedor** del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```powershell
$filesystemName = "my-file-system"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Acl $acl
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

En este ejemplo se establece la ACL en un **directorio** del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```
En este ejemplo se establece la ACL en un **archivo** del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "-wx" -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath -Acl $acl
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

En la siguiente imagen se muestra la salida después de establecer la ACL de un archivo.

![Obtención de una salida de ACL para el archivo](./media/data-lake-storage-directory-file-acl-powershell/set-acl.png)

En este ejemplo, el usuario propietario y el grupo propietario tienen permisos de solo lectura y escritura. Los demás usuarios tienen permisos de escritura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="add-or-update-an-acl-entry"></a>Incorporación o actualización de una entrada de ACL

En primer lugar, obtenga la lista de control de acceso. Luego, use el cmdlet `set-AzDataLakeGen2ItemAclObject` para agregar o actualizar una entrada de ACL. Luego, use el cmdlet `Update-AzDataLakeGen2Item` para confirmar la lista de control de acceso.

En este ejemplo se crea o actualiza la lista de control de acceso en un **directorio** para un usuario.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = (Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname).ACL
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityID xxxxxxxx-xxxx-xxxxxxxxxxx -Permission r-x -InputObject $acl 
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
```

### <a name="remove-an-acl-entry"></a>Eliminación de una entrada de ACL

En este ejemplo se quita una entrada de una lista de control de acceso existente.

```powershell
$id = "xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Create the new ACL object.
[Collections.Generic.List[System.Object]]$aclnew =$acl

foreach ($a in $aclnew)
{
    if ($a.AccessControlType -eq "User"-and $a.DefaultScope -eq $false -and $a.EntityId -eq $id)
    {
        $aclnew.Remove($a);
        break;
    }
}
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $aclnew
```

### <a name="set-an-acl-recursively-preview"></a>Establecimiento de una ACL de forma recursiva (versión preliminar)

Puede agregar, actualizar y quitar las ACL de forma recursiva para los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario. Para obtener más información, consulte [Establecimiento de listas de control de acceso (ACL) de forma recursiva para Azure Data Lake Storage Gen2](recursive-access-control-lists.md).

<a id="gen1-gen2-map"></a>

## <a name="gen1-to-gen2-mapping"></a>Asignación de Gen1 a Gen2

En la siguiente tabla se refleja la asignación de cmdlets usados con Data Lake Storage Gen1 a cmdlets de Data Lake Storage Gen2.

|Cmdlet de Data Lake Storage Gen1| Cmdlet de Data Lake Storage Gen2| Notas |
|--------|---------|-----|
|Get-AzDataLakeStoreChildItem|Get-AzDataLakeGen2ChildItem|De forma predeterminada, el cmdlet Get-AzDataLakeGen2ChildItem solo muestra los elementos secundarios de primer nivel. El parámetro -Recurse muestra elementos secundarios de forma recursiva. |
|Get-AzDataLakeStoreItem<br>Get-AzDataLakeStoreItemAclEntry<br>Get-AzDataLakeStoreItemOwner<br>Get-AzDataLakeStoreItemPermission|Get-AzDataLakeGen2Item|Los elementos de salida del cmdlet Get-AzDataLakeGen2Item tiene estas propiedades: Acl, Owner, Group y Permission.|
|Get-AzDataLakeStoreItemContent|Get-AzDataLakeGen2FileContent|El cmdlet Get-AzDataLakeGen2FileContent descarga el contenido del archivo en el archivo local.|
|Move-AzDataLakeStoreItem|Move-AzDataLakeGen2Item||
|New-AzDataLakeStoreItem|New-AzDataLakeGen2Item|Este cmdlet carga el contenido del archivo nuevo desde un archivo local.|
|Remove-AzDataLakeStoreItem|Remove-AzDataLakeGen2Item||
|Set-AzDataLakeStoreItemOwner<br>Set-AzDataLakeStoreItemPermission<br>Set-AzDataLakeStoreItemAcl|Update-AzDataLakeGen2Item|El cmdlet Update-AzDataLakeGen2Item actualiza solo un único elemento, y no de forma recursiva. Si desea actualizar de forma recursiva, muestre los elementos mediante el cmdlet Get-AzDataLakeStoreChildItem y, a continuación, canalice al cmdlet Update-AzDataLakeGen2Item.|
|Test-AzDataLakeStoreItem|Get-AzDataLakeGen2Item|El cmdlet Get-AzDataLakeGen2Item notificará un error si el elemento no existe.|

## <a name="see-also"></a>Consulte también

* [Problemas conocidos](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
* [Cmdlets de PowerShell de almacenamiento](/powershell/module/az.storage)
