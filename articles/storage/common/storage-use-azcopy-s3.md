---
title: Copia de datos desde Amazon S3 a Azure Storage con AzCopy | Microsoft Docs
description: Use AzCopy para copiar datos de Amazon S3 a Azure Storage. AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: ac093f707167160e916c15b935cb3d8ff6bbc748
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037123"
---
# <a name="copy-data-from-amazon-s3-to-azure-storage-by-using-azcopy"></a>Copia de datos desde Amazon S3 a Azure Storage con AzCopy

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. Este artículo le ayuda a copiar objetos, directorios y cubos desde Amazon Web Services (AWS) S3 a Azure Blob Storage mediante AzCopy.

## <a name="choose-how-youll-provide-authorization-credentials"></a>Elección del modo de proporcionar las credenciales de autorización

* Para la autorización con Azure Storage, use Azure Active Directory (AD) o un token de firma de acceso compartido (SAS).

* Para la autorización con AWS S3, use una clave de acceso de AWS y una clave de acceso secreta.

### <a name="authorize-with-azure-storage"></a>Autorización con Azure Storage

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y elegir cómo proporcionará las credenciales de autorización para el servicio de almacenamiento.

> [!NOTE]
> En los ejemplos de este artículo se supone que ha autenticado la identidad mediante el comando `AzCopy login`. Después, AzCopy usa la cuenta de Azure AD para autorizar el acceso a los datos en Blob Storage.
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy.
>
> Por ejemplo: `https://mystorageaccount.blob.core.windows.net/mycontainer?<SAS-token>`.

### <a name="authorize-with-aws-s3"></a>Autorización con AWS S3

Obtenga la clave de acceso de AWS y la clave de acceso secreta y, después, establezca estas variables de entorno:

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set AWS_ACCESS_KEY_ID=<access-key>`<br>`set AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **Linux** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **macOS** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>`|

## <a name="copy-objects-directories-and-buckets"></a>Copia de objetos, directorios y cubos

AzCopy usa la API [Put Block From URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url), por lo que los datos se copian directamente entre AWS S3 y los servidores de almacenamiento. En estas operaciones de copia no se usa el ancho de banda de red del equipo.

> [!TIP]
> En los ejemplos de esta sección se delimitan los argumentos de ruta de acceso con comillas simples (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

 Estos ejemplos también funcionan con las cuentas que tienen un espacio de nombres jerárquico. El [acceso multiprotocolo en Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) le permite usar la misma sintaxis de URL (`blob.core.windows.net`) en esas cuentas. 

### <a name="copy-an-object"></a>Copia de un objeto

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'` |
| **Ejemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |

> [!NOTE]
> En los ejemplos de este artículo se usan direcciones URL de estilo de ruta de acceso para los cubos de AWS S3 (por ejemplo: `http://s3.amazonaws.com/<bucket-name>`). 
>
> También puede usar direcciones URL similares a las de hospedaje virtual (por ejemplo: `http://bucket.s3.amazonaws.com`). 
>
> Para más información sobre el hospedaje virtual de los cubos, vea [Hospedaje virtual de cubos]](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html).

### <a name="copy-a-directory"></a>Copia de un directorio

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Ejemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

> [!NOTE]
> En este ejemplo se anexa la marca `--recursive` para copiar archivos en todos los subdirectorios.

### <a name="copy-the-contents-of-a-directory"></a>Copia de los contenidos de un directorio

Puede copiar el contenido de un directorio sin copiar el propio directorio contenedor mediante el carácter comodín (*).

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>/*' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Ejemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-a-bucket"></a>Copia de un cubo

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive=true` |
| **Ejemplo** | `azcopy copy 'https://s3.amazonaws.com/mybucket' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive=true` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-all-regions"></a>Copia de todos los cubos de todas las regiones

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Ejemplo** | `azcopy copy 'https://s3.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-a-specific-s3-region"></a>Copia de todos los cubos de una región específica de S3

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://s3-<region-name>.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Ejemplo** | `azcopy copy 'https://s3-rds.eu-north-1.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

## <a name="handle-differences-in-object-naming-rules"></a>Control de las diferencias en las reglas de nomenclatura de objetos

AWS S3 tiene otro conjunto de convenciones de nomenclatura para los nombres de cubo en comparación con los contenedores de blobs de Azure. [Aquí](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) puede leer más al respecto. Si elige copiar un grupo de cubos en una cuenta de Azure Storage, es posible que se produzca un error en la operación de copia debido a las diferencias de nomenclatura.

AzCopy controla dos de los problemas más comunes que pueden surgir; cubos que contienen puntos y cubos que pueden contener guiones consecutivos. Los nombres de cubo de AWS S3 pueden contener puntos y guiones consecutivos, pero un contenedor de Azure no. AzCopy reemplaza los puntos con guiones y los guiones consecutivos con un número que representa el número de guiones consecutivos (por ejemplo: un cubo denominado `my----bucket` se convierte en `my-4-bucket`). 

Además, como AzCopy copia sobre los archivos, comprueba si hay conflictos de nombres e intenta resolverlos. Por ejemplo, si hay cubos con el nombre `bucket-name` y `bucket.name`, en AzCopy un cubo con el nombre `bucket.name` primero se resuelve como `bucket-name` y después como `bucket-name-2`.

## <a name="handle-differences-in-object-metadata"></a>Control de las diferencias en los metadatos de objetos

AWS S3 y Azure permiten distintos conjuntos de caracteres en los nombres de las claves de objeto. [Aquí](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-keys) puede leer sobre los caracteres que se usan en AWS S3. En Azure, las claves de objeto de blob se adhieren a las reglas de nomenclatura para los [ identificadores de C#](https://docs.microsoft.com/dotnet/csharp/language-reference/).

Como parte de un comando `copy` de AzCopy, puede proporcionar un valor opcional para la marca `s2s-handle-invalid-metadata` que especifique cómo quiere controlar los archivos donde los metadatos del archivo contienen nombres de clave no compatibles. En la tabla siguiente se describe cada valor de marca.

| Valor de marca | Descripción  |
|--------|-----------|
| **ExcludeIfInvalid** | (Opción predeterminada) Los metadatos no se incluyen en el objeto transferido. AzCopy registra una advertencia. |
| **FailIfInvalid** | Los objetos no se copian. AzCopy registra un error y lo incluye en el recuento de errores que aparece en el resumen de la transferencia.  |
| **RenameIfInvalid**  | AzCopy resuelve la clave de metadatos no válida y copia el objeto en Azure mediante el par clave-valor de metadatos resuelto. Para saber exactamente qué pasos realiza AzCopy para cambiar el nombre de las claves de objeto, vea la sección [Cómo cambia AzCopy el nombre de las claves de objeto](#rename-logic) más adelante. Si AzCopy no puede cambiar el nombre de la clave, el objeto no se copiará. |

<a id="rename-logic"></a>

### <a name="how-azcopy-renames-object-keys"></a>Cómo cambia AzCopy el nombre de las claves de objeto

AzCopy realiza estos pasos:

1. Reemplaza los caracteres no válidos con "_".

2. Agrega la cadena `rename_` al principio de una nueva clave válida.

   Esta clave se usará para guardar el **valor** de los metadatos originales.

3. Agrega la cadena `rename_key_` al principio de una nueva clave válida.
   Esta clave se usará para guardar la **clave** no válida de los metadatos originales.
   Puede usar esta clave para intentar recuperar los metadatos en Azure, ya que la clave de metadatos se conserva como un valor en el servicio Blob Storage.

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar más ejemplos en cualquiera de estos artículos:

- [Introducción a AzCopy](storage-use-azcopy-v10.md)

- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)

- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)

- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)
