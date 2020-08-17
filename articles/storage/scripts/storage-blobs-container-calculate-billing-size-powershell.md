---
title: 'Ejemplo de script de Azure PowerShell: Cálculo del tamaño total de facturación de un contenedor de blobs | Microsoft Docs'
description: Calcule el tamaño total de un contenedor de Azure Blob Storage a efectos de facturación.
services: storage
author: fhryo-msft
ms.service: storage
ms.subservice: blobs
ms.devlang: powershell
ms.topic: sample
ms.date: 11/07/2017
ms.author: fryu
ms.openlocfilehash: f8bd47a8fde700382f9789ab1c77ff35e9b1f4f5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999537"
---
# <a name="calculate-the-total-billing-size-of-a-blob-container"></a>Cálculo del tamaño total de facturación de un contenedor de blobs

Este script calcula el tamaño de un contenedor de Azure Blob Storage con la finalidad de estimar los costos de facturación. El script suma el total de los tamaños de los blobs del contenedor.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Este script de PowerShell calcula el tamaño de un contenedor a efectos de facturación. Si va a calcular el tamaño de un contenedor para otros fines, consulte [Cálculo del tamaño total de un contenedor de Blob Storage](../scripts/storage-blobs-container-calculate-size-powershell.md) para obtener un script más sencillo que proporcione una estimación.

## <a name="determine-the-size-of-the-blob-container"></a>Determinación del tamaño de un contenedor de blobs

El tamaño total de un contenedor de blobs incluye el tamaño del contenedor y el tamaño de todos los blobs del contenedor.

En las secciones siguientes se describe cómo se calcula la capacidad de almacenamiento de los blobs y los contenedores de blobs. En la sección siguiente, Len(X) hace referencia al número de caracteres de la cadena.

### <a name="blob-containers"></a>Contenedores de blobs

En el siguiente cálculo se describe cómo calcular la cantidad de almacenamiento utilizado por cada contenedor de blobs:

```
48 bytes + Len(ContainerName) * 2 bytes +
For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
For-Each Signed Identifier[512 bytes]
```

A continuación, se explica con detalle:
* En los 48 bytes de sobrecarga para cada contenedor se incluye la hora de la última modificación, los permisos, la configuración públicos y algunos metadatos del sistema.

* El nombre del contenedor se almacena como Unicode, por lo que se toma el número de caracteres y se multiplica por dos.

* Para cada bloque de metadatos almacenado del contenedor de blobs, se almacena la longitud del nombre (ASCII), así como la longitud del valor de cadena.

* En los 512 bytes por identificador firmado se incluyen el nombre del identificador firmado, la hora de inicio, la hora de expiración y los permisos.

### <a name="blobs"></a>Datos BLOB

En los cálculos siguientes se muestra cómo calcular la cantidad de almacenamiento usado por blob.

* Blob en bloques (blob base o instantánea):

   ```
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   8 bytes + number of committed and uncommitted blocks * Block ID Size in bytes +
   SizeInBytes(data in unique committed data blocks stored) +
   SizeInBytes(data in uncommitted data blocks)
   ```

* Blob en páginas (blob base o instantánea):

   ```
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   number of nonconsecutive page ranges with data * 12 bytes +
   SizeInBytes(data in unique pages stored)
   ```

A continuación, se explica con detalle:

* 124 bytes de sobrecarga del blob, que incluye:
    - Hora de la última modificación
    - Size
    - Cache-Control
    - Content-Type
    - Content-Language
    - Content-Encoding
    - Content-MD5
    - Permisos
    - Información de la instantánea
    - Concesión
    - Algunos metadatos del sistema

* El nombre del blob se almacena como Unicode, por lo que se toma el número de caracteres y se multiplica por dos.

* Para cada bloque de metadatos almacenado, agregue la longitud del nombre (guardado como ASCII), así como la longitud del valor de cadena.

* Para los blobs en bloques:
  * 8 bytes para la lista de bloques.
  * El número de bloques multiplicado por el tamaño del identificador de bloque en bytes.
  * El tamaño de los datos de todos los bloques confirmados y sin confirmar.

    >[!NOTE]
    >Cuando se usan instantáneas, este tamaño solo incluye los datos únicos de este blob base o de instantánea. Si no se utilizan los bloques sin confirmar después de una semana, se recolectan como elementos no utilizados. Después de eso, no cuentan para la facturación.

* Para los blobs en páginas:
  * El número de intervalos de páginas no consecutivas con datos multiplicado por 12 bytes. Este es el número de intervalos de páginas únicas que se ven al llamar a la API **GetPageRanges**.

  * El tamaño de los datos en bytes de todas las páginas almacenadas.

    >[!NOTE]
    >Cuando se usan instantáneas, este tamaño solo incluye las páginas únicas del blob base o de instantánea que se cuenta.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/storage/calculate-container-size/calculate-container-size-ex.ps1 "Calculate container size")]

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Cálculo del tamaño total de un contenedor de Blob Storage](../scripts/storage-blobs-container-calculate-size-powershell.md) para obtener un script sencillo que proporciona una estimación del tamaño del contenedor.

- Para más información sobre la facturación de Azure Storage, vea la [descripción de la facturación de Microsoft Azure Storage](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/07/08/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity/).

- Para más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

- Puede encontrar ejemplos de script adicionales de PowerShell para Storage en los [ejemplos de PowerShell para Azure Storage](../blobs/storage-samples-blobs-powershell.md).
