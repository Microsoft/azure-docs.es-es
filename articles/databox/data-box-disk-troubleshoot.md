---
title: Solución de problemas de Azure Data Box Disk | Microsoft Docs
description: Obtenga información acerca de cómo usar los registros para solucionar los problemas de validación que pueden aparecer al implementar Azure Data Box Disk.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/14/2019
ms.author: alkohli
ms.openlocfilehash: 1c1b38c4021660b9f59098f8442d16bfd0ecc582
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925549"
---
# <a name="use-logs-to-troubleshoot-validation-issues-in-azure-data-box-disk"></a>Uso de registros para solucionar incidencias de validación en Azure Data Box Disk

Este artículo se aplica a Microsoft Azure Data Box Disk. En el artículo se describe cómo usar los registros para solucionar incidencias de validación que podría observar al implementar esta solución.

## <a name="validation-tool-log-files"></a>Archivos de registro de herramienta de validación

Cuando se validan los datos en los discos mediante la [herramienta de validación](data-box-disk-deploy-copy-data.md#validate-data), se genera un archivo *error.xml* para registrar los errores. El archivo de registro se ubica en la carpeta `Drive:\DataBoxDiskImport\logs` de la unidad. Cuando se ejecuta la validación, se proporciona un vínculo al registro de errores.

<!--![Validation tool with link to error log](media/data-box-disk-troubleshoot/validation-tool-link-error-log.png)-->

Si se ejecutan varias sesiones para la validación, se genera un registro de errores por cada sesión.

- Este es un ejemplo del registro de errores cuando los datos cargados en la carpeta `PageBlob` no son de 512 bytes alineados. Los datos cargados a PageBlob deben ser de 512 bytes alineados, por ejemplo, VHD o VHDX. Los errores en este archivo se encuentran en `<Errors>` y las advertencias en `<Warnings>`.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
        <ErrorLog Version="2018-10-01">
            <SessionId>session#1</SessionId>
            <ItemType>PageBlob</ItemType>
            <SourceDirectory>D:\Dataset\TestDirectory</SourceDirectory>
            <Errors>
                <Error Code="Not512Aligned">
                    <Description>The file is not 512 bytes aligned.</Description>
                    <List>
                        <File Path="\Practice\myScript.ps1" />
                    </List>
                    <Count>1</Count>
                </Error>
            </Errors>
            <Warnings />
        </ErrorLog>
    ```

- Este es un ejemplo de registro de errores cuando el nombre del contenedor no es válido. La carpeta que se crea en las carpetas del disco `BlockBlob`, `PageBlob`, o `AzureFile` se convierte en un contenedor en la cuenta Azure Storage. El nombre del contenedor debe seguir las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions).

    ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <ErrorLog Version="2018-10-01">
          <SessionId>bbsession</SessionId>
          <ItemType>BlockBlob</ItemType>
          <SourceDirectory>E:\BlockBlob</SourceDirectory>
          <Errors>
            <Error Code="InvalidShareContainerFormat">
              <List>
                <Container Name="Azu-reFile" />
                <Container Name="bbcont ainer1" />
              </List>
              <Count>2</Count>
            </Error>
          </Errors>
          <Warnings />
    </ErrorLog>
    ```

## <a name="validation-tool-errors"></a>Errores de herramienta de validación

En la tabla siguiente se resumen los errores incluidos en el archivo *error.xml* y las acciones recomendadas correspondientes.

| Código de error| Descripción                       | Acciones recomendadas               |
|------------|--------------------------|-----------------------------------|
| `None` | Se han validado correctamente los datos. | No se requiere ninguna acción. |
| `InvalidXmlCharsInPath` |No se ha podido crear un archivo de manifiesto, ya que la ruta de acceso tiene caracteres que no son válidos. | Quite estos caracteres para continuar.  |
| `OpenFileForReadFailed`| No se ha podido procesar el archivo. Esto puede deberse a una incidencia de acceso o a daños en el sistema de archivos.|No se ha podido leer el archivo debido a un error. Los detalles del error se encuentran en la excepción. |
| `Not512Aligned` | Este archivo no tiene un formato válido para la carpeta PageBlob.| Cargue solo los datos que estén alineados en 512 bytes en la carpeta `PageBlob`. Quite el archivo de la carpeta PageBlob o muévalo a la carpeta BlockBlob. Vuelva a intentar la validación.|
| `InvalidBlobPath` | La ruta de acceso del archivo no se asigna a una ruta de acceso de blob válida en la nube según las convenciones de nomenclatura de Azure Blob.|Siga las instrucciones de nomenclatura de Azure para cambiar el nombre de la ruta de acceso del archivo. |
| `EnumerationError` | No se ha podido enumerar el archivo para la validación. |Los motivos para que se produzca este error pueden ser varios. El motivo más probable es el acceso al archivo. |
| `ShareSizeExceeded` | Este archivo ha producido que el tamaño del recurso compartido de Azure supere el límite de 5 TB de Azure.|Reduzca el tamaño de los datos en el recurso compartido de modo que cumpla los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Vuelva a intentar la validación. |
| `AzureFileSizeExceeded` | El tamaño de archivo supera los límites de tamaño de archivo de Azure.| Reduzca el tamaño del archivo o de los datos de modo que cumpla los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Vuelva a intentar la validación.|
| `BlockBlobSizeExceeded` | El tamaño de archivo supera los límites de tamaño del blob en bloques de Azure. | Reduzca el tamaño del archivo o de los datos de modo que cumpla los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Vuelva a intentar la validación. |
| `ManagedDiskSizeExceeded` | El tamaño de archivo supera los límites de tamaño del disco administrado de Azure. | Reduzca el tamaño del archivo o de los datos de modo que cumpla los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Vuelva a intentar la validación. |
| `PageBlobSizeExceeded` | El tamaño de archivo supera los límites de tamaño del disco administrado de Azure. | Reduzca el tamaño del archivo o de los datos de modo que cumpla los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Vuelva a intentar la validación. |
| `InvalidShareContainerFormat`  |Los nombres de directorio no cumplen las convenciones de nomenclatura de Azure para contenedores o recursos compartidos.         |La primera carpeta que ha creado en las carpetas ya existentes en el disco se convierte en un contenedor en la cuenta de almacenamiento. El nombre de este recurso compartido o contenedor no cumple las convenciones de nomenclatura de Azure. Cambie el nombre del archivo para que cumpla las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Vuelva a intentar la validación.   |
| `InvalidBlobNameFormat` | La ruta de acceso del archivo no se asigna a una ruta de acceso de blob válida en la nube según las convenciones de nomenclatura de Azure Blob.|Cambie el nombre del archivo para que cumpla las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Vuelva a intentar la validación. |
| `InvalidFileNameFormat` | La ruta de acceso del archivo no se asigna a una ruta de acceso del archivo válida en la nube según las convenciones de nomenclatura de los archivos de Azure. |Cambie el nombre del archivo para que cumpla las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Vuelva a intentar la validación. |
| `InvalidDiskNameFormat` | La ruta de acceso del archivo no se asigna a un nombre de disco válido en la nube según las convenciones de nomenclatura del disco administrado de Azure. |Cambie el nombre del archivo para que cumpla las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Vuelva a intentar la validación.       |
| `NotPartOfFileShare` | La ruta de acceso de carga de los archivos no es válida. Cargue los archivos en una carpeta de Azure Files.   | Quite los archivos con errores y cárguelos en una carpeta creada previamente. Vuelva a intentar la validación. |
| `NonVhdFileNotSupportedForManagedDisk` | No se ha podido cargar un archivo que no es VHD como disco administrado. |Quite los archivos que no son VHD de la carpeta `ManagedDisk`, ya que no se admiten, o mueva dichos archivos a una carpeta `PageBlob`. Vuelva a intentar la validación. |


## <a name="next-steps"></a>Pasos siguientes

- Solucionar problemas de [errores al cargar los datos](data-box-disk-troubleshoot-upload.md).
