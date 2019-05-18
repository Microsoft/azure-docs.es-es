---
title: Solucionar problemas en Azure Data Box | Microsoft Docs
description: Describe cómo solucionar problemas en Azure Data Box que se ven al cargar datos en Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 05/13/2019
ms.author: alkohli
ms.openlocfilehash: 1126002a93419371be3216c55114385c9c600419
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65594005"
---
# <a name="troubleshoot-issues-related-to-azure-data-box"></a>Solucionar problemas relacionados con Azure Data Box

Este artículo detalla información sobre cómo solucionar problemas que puede ver al usar Azure Data Box.

## <a name="errors-during-data-copy"></a>Errores durante la copia de datos

En las secciones siguientes se resumen todos los errores que se ven durante la copia de datos.

### <a name="errorcontainerorsharenamelength"></a>ERROR_CONTAINER_OR_SHARE_NAME_LENGTH 

**Descripción del error:** El nombre del contenedor o recurso compartido debe tener entre 3 y 63 caracteres.

**Resolución sugerida:** La carpeta en el recurso compartido de Data Box (SMB o NFS) a la que ha copiado los datos se convierte en un contenedor de Azure en su cuenta de almacenamiento. 

- En el **Connect y copie** página de la IU web local, descarga y revise los nombres de los archivos de error para identificar la carpeta con problemas de Data Box.
- Cambiar el nombre de carpeta en el recurso compartido de Data Box para asegurarse de:

    - El nombre tiene entre 3 y 63 caracteres.
    - Los nombres solo pueden tener letras, números y guiones.
    - Los nombres no pueden comenzar ni terminar con guiones.
    - Los nombres no pueden tener guiones consecutivos.
    - Ejemplos de nombres válidos: `my-folder-1`, `my-really-extra-long-folder-111`.
    - Ejemplos de nombres que no son válidos: `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Para obtener más información, vea las convenciones de nomenclatura de Azure para [los nombres de contenedor](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) y [comparten nombres](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).

### <a name="errorcontainerorsharenamelength"></a>ERROR_CONTAINER_OR_SHARE_NAME_LENGTH     

**Descripción del error:** El nombre del contenedor o recurso compartido debe tener entre 3 y 63 caracteres. 

**Resolución sugerida:** La carpeta en el recurso compartido de Data Box (SMB o NFS) a la que ha copiado los datos se convierte en un contenedor de Azure en su cuenta de almacenamiento. 

- En el **Connect y copie** página de la IU web local, descarga y revise los nombres de los archivos de error para identificar la carpeta con problemas de Data Box.
- Cambiar el nombre de carpeta en el recurso compartido de Data Box para asegurarse de:

    - El nombre tiene entre 3 y 63 caracteres.
    - Los nombres solo pueden tener letras, números y guiones.
    - Los nombres no pueden comenzar ni terminar con guiones.
    - Los nombres no pueden tener guiones consecutivos.
    - Ejemplos de nombres válidos: `my-folder-1`, `my-really-extra-long-folder-111`
    - Ejemplos de nombres que no son válidos: `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Para obtener más información, vea las convenciones de nomenclatura de Azure para [los nombres de contenedor](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) y [comparten nombres](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).


### <a name="errorcontainerorsharenamealphanumericdash"></a>ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH

**Descripción del error:** El nombre del contenedor o recurso compartido debe constar solo de letras, números o guiones.

**Resolución sugerida:** La carpeta en el recurso compartido de Data Box (SMB o NFS) a la que ha copiado los datos se convierte en un contenedor de Azure en su cuenta de almacenamiento. 

- En el **Connect y copie** página de la IU web local, descarga y revise los nombres de los archivos de error para identificar la carpeta con problemas de Data Box.
- Cambiar el nombre de carpeta en el recurso compartido de Data Box para asegurarse de:

    - El nombre tiene entre 3 y 63 caracteres.
    - Los nombres solo pueden tener letras, números y guiones.
    - Los nombres no pueden comenzar ni terminar con guiones.
    - Los nombres no pueden tener guiones consecutivos.
    - Ejemplos de nombres válidos: `my-folder-1`, `my-really-extra-long-folder-111`
    - Ejemplos de nombres que no son válidos: `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Para obtener más información, vea las convenciones de nomenclatura de Azure para [los nombres de contenedor](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) y [comparten nombres](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).

### <a name="errorcontainerorsharenameimproperdash"></a>ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH

**Descripción del error:** Los nombres de contenedor y los nombres de recurso compartido no se pueden iniciar o finalizar con guiones y no puede tener guiones consecutivos.

**Resolución sugerida:** La carpeta en el recurso compartido de Data Box (SMB o NFS) a la que ha copiado los datos se convierte en un contenedor de Azure en su cuenta de almacenamiento. 

- En el **Connect y copie** página de la IU web local, descarga y revise los nombres de los archivos de error para identificar la carpeta con problemas de Data Box.
- Cambiar el nombre de carpeta en el recurso compartido de Data Box para asegurarse de:

    - El nombre tiene entre 3 y 63 caracteres.
    - Los nombres solo pueden tener letras, números y guiones.
    - Los nombres no pueden comenzar ni terminar con guiones.
    - Los nombres no pueden tener guiones consecutivos.
    - Ejemplos de nombres válidos: `my-folder-1`, `my-really-extra-long-folder-111`
    - Ejemplos de nombres que no son válidos: `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Para obtener más información, vea las convenciones de nomenclatura de Azure para [los nombres de contenedor](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) y [comparten nombres](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).

### <a name="errorcontainerorsharenamedisallowedfortype"></a>ERROR_CONTAINER_OR_SHARE_NAME_DISALLOWED_FOR_TYPE

**Descripción del error:** Se especifican los nombres de contenedor incorrecto para los recursos compartidos de disco administrado.

**Resolución sugerida:** Para los discos administrados, dentro de cada recurso compartido, se crean las carpetas siguientes que corresponden a los contenedores de la cuenta de almacenamiento: SSD Premium, estándar HDD y SSD estándar. Estas carpetas se corresponden con el nivel de rendimiento para el disco administrado.

- Asegúrese de que copia los datos de blob de página (VHD) en una de estas carpetas existentes. Solo los datos de estos contenedores existentes se cargan en Azure.
- Cualquier otra carpeta que se crea en el mismo nivel que SSD Premium, estándar de HDD y SSD estándar no se corresponde con un nivel de rendimiento válido y no se puede usar.
- Eliminar archivos o carpetas creadas fuera de los niveles de rendimiento.

Para obtener más información, consulte [copia a discos administrados](data-box-deploy-copy-data-from-vhds.md#connect-to-data-box).

### <a name="errorcontainerorsharecapacityexceeded"></a>ERROR_CONTAINER_OR_SHARE_CAPACITY_EXCEEDED

**Descripción del error:** Recurso compartido de archivos de Azure limita a un recurso compartido de 5 TB de datos. Ha superado este límite para algunos recursos compartidos.

**Resolución sugerida:** En el **Connect y copie** página de la Data Box IU web local, descargue y revise los archivos de error.

Identifique las carpetas que tienen este problema de los registros de error y asegúrese de que los archivos en esa carpeta son inferior a 5 TB.

### <a name="errorbloborfilenamecharactercontrol"></a>ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL

**Descripción del error:** Los nombres de blob o archivo contienen caracteres de control no admitido.

**Resolución sugerida:** Los blobs o los archivos que ha copiado contienen nombres con caracteres no admitidos.

En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
Quitar o cambiar el nombre de los archivos para quitar caracteres no admitidos.

Para obtener más información, vea las convenciones de nomenclatura de Azure para [nombres de blob](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) y [los nombres de archivo](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).

### <a name="errorbloborfilenamecharacterillegal"></a>ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL

**Descripción del error:** Los nombres de archivo o blob contienen caracteres no válidos.

**Resolución sugerida:** Los blobs o los archivos que ha copiado contienen nombres con caracteres no admitidos.

En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
Quitar o cambiar el nombre de los archivos para quitar caracteres no admitidos.

Para obtener más información, vea las convenciones de nomenclatura de Azure para [nombres de blob](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) y [los nombres de archivo](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).


### <a name="errorbloborfilenameending"></a>ERROR_BLOB_OR_FILE_NAME_ENDING

**Descripción del error:** Los nombres de blob o archivo finalizan con caracteres incorrectos.

**Resolución sugerida:** Los blobs o los archivos que ha copiado contienen nombres con caracteres no admitidos.

En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
Quitar o cambiar el nombre de los archivos para quitar caracteres no admitidos.

Para obtener más información, vea las convenciones de nomenclatura de Azure para [nombres de blob](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) y [los nombres de archivo](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).


### <a name="errorbloborfilenamesegmentcount"></a>ERROR_BLOB_OR_FILE_NAME_SEGMENT_COUNT

**Descripción del error:** El nombre de archivo o blob contiene demasiados segmentos de ruta de acceso.

**Resolución sugerida:** Los blobs o los archivos que ha copiado superan el número máximo de segmentos de ruta de acceso. Un segmento de ruta de acceso es la cadena entre caracteres delimitadores consecutivos, por ejemplo, la barra diagonal /.

- En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
- Asegúrese de que el [nombres de blob](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) y [los nombres de archivo](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names) se ajustan a las convenciones de nomenclatura de Azure.

### <a name="errorbloborfilenameaggregatelength"></a>ERROR_BLOB_OR_FILE_NAME_AGGREGATE_LENGTH

**Descripción del error:** El nombre de blob o archivo es demasiado largo.

**Resolución sugerida:** El blob o los nombres de archivo superan la longitud máxima.

- En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
- El nombre del blob no debe superar los 1024 caracteres.
- Quitar o cambiar el nombre del blob o los archivos para que los nombres no supere los 1024 caracteres.

Para obtener más información, vea las convenciones de nomenclatura de Azure para los nombres de blob y los nombres de archivo.

### <a name="errorbloborfilenamecomponentlength"></a>ERROR_BLOB_OR_FILE_NAME_COMPONENT_LENGTH

**Descripción del error:** Uno de los segmentos del nombre de blob o archivo es demasiado largo.

**Resolución sugerida:** Uno de los segmentos de ruta de acceso en el nombre de archivo o blob supera el número máximo de caracteres. Un segmento de ruta de acceso es la cadena entre caracteres delimitadores consecutivos, por ejemplo, la barra diagonal /.

- En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
- Asegúrese de que el [nombres de blob](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) y [los nombres de archivo](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names) se ajustan a las convenciones de nomenclatura de Azure.

### <a name="errorbloborfilesizelimit"></a>ERROR_BLOB_OR_FILE_SIZE_LIMIT

**Descripción del error:** El tamaño del archivo supera el tamaño máximo para la carga.

**Resolución sugerida:** El blob o los tamaños de archivo superan el límite máximo permitido para la carga.

- En el **Connect y copie** página de la interfaz de usuario de web local, descargue y revise los archivos de error.
- Asegúrese de que los tamaños de blob y archivo no superen los límites de tamaño de objeto de Azure.

### <a name="errorbloborfilesizealignment"></a>ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT

**Descripción del error:** El blob o archivo no está alineado correctamente.

**Resolución sugerida:** Alinea el recurso compartido de blobs de página en el cuadro de datos solo admite archivos que son de 512 bytes (por ejemplo, VHD/VHDX). Los datos que se copian en el recurso compartido de blobs de página se cargan en Azure como blobs en páginas.

Quitar todos los datos que no son VHD/VHDX desde el recurso compartido de blobs de página. Puede usar recursos compartidos de blob en bloques o los archivos de Azure para datos genéricos.

Para obtener más información, consulte [blobs de página de información general de](../storage/blobs/storage-blob-pageblob-overview.md).

### <a name="errorbloborfiletypeunsupported"></a>ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED

**Descripción del error:** Un tipo de archivo no admitido está presente en un recurso compartido de disco administrado. Se permiten solo los VHD fijos.

**Resolución sugerida:**

- Asegúrese de que sólo pueda cargar los VHD fijos para crear discos administrados.
- Los archivos VHDX o **dinámica** y **diferenciación** no se admiten los discos duros virtuales.

### <a name="errordirectorydisallowedfortype"></a>ERROR_DIRECTORY_DISALLOWED_FOR_TYPE

**Descripción del error:** Un directorio no se permite en cualquiera de las carpetas ya existentes para los discos administrados. Solo los VHD fijos se permiten en estas carpetas.

**Resolución sugerida:** Para los discos administrados, dentro de cada recurso compartido, las tres carpetas siguientes se crean que corresponden a los contenedores de la cuenta de almacenamiento: SSD Premium, estándar HDD y SSD estándar. Estas carpetas se corresponden con el nivel de rendimiento para el disco administrado.

- Asegúrese de que copia los datos de blob de página (VHD) en una de estas carpetas existentes.
- No se permite una carpeta o un directorio en estas carpetas existentes. Quite las carpetas que ha creado dentro de las carpetas ya existentes.

Para obtener más información, consulte [copia a discos administrados](data-box-deploy-copy-data-from-vhds.md#connect-to-data-box).


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la [los requisitos del sistema de almacenamiento Blob de datos cuadro](data-box-system-requirements-rest.md).
