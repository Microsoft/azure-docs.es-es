---
title: Uso de la CLI de Azure con archivos y listas de control de acceso (ACL) en Azure Data Lake Storage Gen2
description: Use la CLI de Azure para administrar directorios y listas de control de acceso (ACL) de archivos y directorios en cuentas de almacenamiento que tengan un espacio de nombres jerárquico.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 05/18/2020
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6140260b75580270b365e59358d97e0a54c7b4a7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503946"
---
# <a name="use-azure-cli-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>Uso de la CLI de Azure para administrar directorios, archivos y ACL en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar la [Interfaz de la línea de comandos de Azure (CLI)](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) para crear y administrar directorios, archivos y permisos en cuentas de almacenamiento que tengan un espacio de nombres jerárquico. 

[Ejemplos](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md) | [Enviar comentarios](https://github.com/Azure/azure-cli-extensions/issues)

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
> * Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](data-lake-storage-quickstart-create-account.md) instrucciones para crear uno.
> * CLI de Azure versión `2.6.0` o posterior.

## <a name="ensure-that-you-have-the-correct-version-of-azure-cli-installed"></a>Asegúrese de que tiene instalada la versión correcta de la CLI de Azure.

1. Abra [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview?view=azure-cli-latest) o, si ha [instalado](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) la CLI de Azure localmente, abra una aplicación de consola de comandos como Windows PowerShell.

2. Use el comando siguiente para verificar que la versión de la CLI de Azure que ha instalado sea `2.6.0` o posterior.

   ```azurecli
    az --version
   ```
   Si la versión de la CLI de Azure es anterior a `2.6.0`, instale una versión posterior. Consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="connect-to-the-account"></a>Conexión con la cuenta

1. Si usa la CLI de Azure localmente, ejecute el comando de inicio de sesión.

   ```azurecli
   az login
   ```

   Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

   En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal. Inicie sesión con las credenciales de su cuenta en el explorador.

   Para obtener más información sobre los distintos métodos de autenticación, consulte [Autorización del acceso a los datos de blobs o colas con la CLI de Azure](../common/authorize-data-operations-cli.md).

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento que hospedará el sitio web estático.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

> [!NOTE]
> En el ejemplo que se presenta en este artículo se muestra la autorización de Azure Active Directory (AD). Para obtener más información sobre los métodos de autorización, consulte [Autorización del acceso a los datos de blobs o colas con la CLI de Azure](../common/authorize-data-operations-cli.md).

## <a name="create-a-container"></a>Crear un contenedor

Un contenedor actúa como sistema de archivos para sus archivos. Puede crear uno mediante el comando `az storage fs create`. 

En este ejemplo se crea un contenedor denominado `my-file-system`.

```azurecli
az storage fs create -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-container-properties"></a>Mostrar las propiedades del contenedor

Puede imprimir las propiedades de un contenedor en la consola mediante el comando `az storage fs show`.

```azurecli
az storage fs show -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="list-container-contents"></a>Enumerar el contenido del contenedor

Muestre el contenido de un directorio mediante el comando `az storage fs file list`.

En este ejemplo se muestra el contenido de un contenedor denominado `my-file-system`.

```azurecli
az storage fs file list -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-container"></a>Eliminación de un contenedor

Elimine el contenedor con el comando `az storage fs delete`.

En este ejemplo se elimina un contenedor denominado `my-file-system`. 

```azurecli
az storage fs delete -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="create-a-directory"></a>Creación de un directorio

Cree una referencia de directorio mediante el comando `az storage fs directory create`. 

En este ejemplo se agrega un directorio denominado `my-directory` a un contenedor denominado `my-file-system` que se encuentra en una cuenta denominada `mystorageaccount`.

```azurecli
az storage fs directory create -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-directory-properties"></a>Visualización de las propiedades de directorio

Puede imprimir las propiedades de un directorio en la consola mediante el comando `az storage fs directory show`.

```azurecli
az storage fs directory show -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="rename-or-move-a-directory"></a>Cambio de nombre o traslado de un directorio

Cambie el nombre de un directorio o muévalo mediante el comando `az storage fs directory move`.

En este ejemplo se cambia el nombre de un directorio de `my-directory` a `my-new-directory` en el mismo contenedor.

```azurecli
az storage fs directory move -n my-directory -f my-file-system --new-directory "my-file-system/my-new-directory" --account-name mystorageaccount --auth-mode login
```

En este ejemplo se mueve un directorio a un contenedor denominado `my-second-file-system`.

```azurecli
az storage fs directory move -n my-directory -f my-file-system --new-directory "my-second-file-system/my-new-directory" --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-directory"></a>Eliminación de un directorio

Elimine un directorio mediante el comando `az storage fs directory delete`.

En este ejemplo se elimina un directorio denominado `my-directory`. 

```azurecli
az storage fs directory delete -n my-directory -f my-file-system  --account-name mystorageaccount --auth-mode login 
```

## <a name="check-if-a-directory-exists"></a>Comprobación de si existe un directorio

Determine si existe un directorio concreto en el contenedor mediante el comando `az storage fs directory exists`.

En este ejemplo se ve si existe un directorio denominado `my-directory` en el contenedor `my-file-system`. 

```azurecli
az storage fs directory exists -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login 
```

## <a name="download-from-a-directory"></a>Descarga de un directorio

Descargue un archivo de un directorio mediante el comando `az storage fs file download`.

En este ejemplo se descarga un archivo denominado `upload.txt` de un directorio denominado `my-directory`. 

```azurecli
az storage fs file download -p my-directory/upload.txt -f my-file-system -d "C:\myFolder\download.txt" --account-name mystorageaccount --auth-mode login
```

## <a name="list-directory-contents"></a>Lista del contenido del directorio

Muestre el contenido de un directorio mediante el comando `az storage fs file list`.

En este ejemplo se muestra el contenido de un directorio denominado `my-directory` que se encuentra en el contenedor `my-file-system` de una cuenta de almacenamiento denominada `mystorageaccount`. 

```azurecli
az storage fs file list -f my-file-system --path my-directory --account-name mystorageaccount --auth-mode login
```

## <a name="upload-a-file-to-a-directory"></a>Carga de un archivo en un directorio

Cargue un archivo en un directorio mediante el comando `az storage fs directory upload`.

En este ejemplo se carga un archivo denominado `upload.txt` en un directorio denominado `my-directory`. 

```azurecli
az storage fs file upload -s "C:\myFolder\upload.txt" -p my-directory/upload.txt  -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-file-properties"></a>Visualización de las propiedades del archivo

Puede imprimir las propiedades de un archivo en la consola mediante el comando `az storage fs file show`.

```azurecli
az storage fs file show -p my-file.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="rename-or-move-a-file"></a>Cambio de nombre o traslado de un archivo

Cambie el nombre de un archivo o muévalo mediante el comando `az storage fs file move`.

En este ejemplo se cambia el nombre de un archivo de `my-file.txt` a `my-file-renamed.txt`.

```azurecli
az storage fs file move -p my-file.txt -f my-file-system --new-path my-file-system/my-file-renamed.txt --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-file"></a>Eliminación de un archivo

Elimine un archivo mediante el comando `az storage fs file delete`.

En este ejemplo se elimina un archivo denominado `my-file.txt`.

```azurecli
az storage fs file delete -p my-directory/my-file.txt -f my-file-system  --account-name mystorageaccount --auth-mode login 
```

## <a name="manage-permissions"></a>Administración de permisos

Puede obtener, establecer y actualizar los permisos de acceso de los directorios y archivos.

> [!NOTE]
> Si usa Azure Active Directory (Azure AD) para autorizar comandos, asegúrese de que la entidad de seguridad tiene asignado el [rol Propietario de datos de blobs de almacenamiento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, vea [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control).

### <a name="get-an-acl"></a>Obtención de una ACL

Obtenga la ACL de un **directorio** mediante el comando `az storage fs access show`.

En este ejemplo se obtiene la ACL de un directorio y luego se imprime la ACL en la consola.

```azurecli
az storage fs access show -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Obtenga los permisos de acceso de un **archivo** mediante el comando `az storage fs access show`. 

En este ejemplo se obtiene la ACL de un archivo y luego se imprime la ACL en la consola.

```azurecli
az storage fs access show -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

En la siguiente imagen se muestra la salida después de obtener la ACL de un directorio.

![Obtención de una salida de ACL](./media/data-lake-storage-directory-file-acl-cli/get-acl.png)

En este ejemplo, el usuario propietario tiene permisos de lectura, escritura y ejecución. El grupo propietario tiene permisos de solo lectura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="set-an-acl"></a>Establecimiento de una ACL

Use el comando `az storage fs access set` para establecer la ACL de un **directorio**. 

En este ejemplo se establece la ACL en un directorio del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo, se establece la ACL *predeterminada* en un directorio del usuario propietario, el grupo propietario o de otros usuarios y, luego, se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "default:user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Use el comando `az storage fs access set` para establecer la ACL de un **archivo**. 

En este ejemplo se establece la ACL en un archivo del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

En la siguiente imagen se muestra la salida después de establecer la ACL de un archivo.

![Obtención de una salida de ACL](./media/data-lake-storage-directory-file-acl-cli/set-acl-file.png)

En este ejemplo, el usuario propietario y el grupo propietario tienen permisos de solo lectura y escritura. Los demás usuarios tienen permisos de escritura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="update-an-acl"></a>Actualización de una ACL

Otra manera de establecer este permiso es usar el comando `az storage fs access set`. 

Actualice la ACL de un directorio o archivo estableciendo el parámetro `-permissions` en el formato abreviado de una ACL.

En este ejemplo se actualiza la ACL de un **directorio**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo se actualiza la ACL de un **archivo**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

También puede actualizar el usuario propietario y el grupo de un directorio o archivo estableciendo los parámetros `--owner` o `group` en el identificador de entidad o en el nombre principal de usuario (UPN) de un usuario. 

En este ejemplo se cambia el propietario de un directorio. 

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo se cambia el propietario de un archivo. 

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="see-also"></a>Consulte también

* [Muestras](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)
* [Envíenos sus comentarios](https://github.com/Azure/azure-cli-extensions/issues)
* [Problemas conocidos](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)


