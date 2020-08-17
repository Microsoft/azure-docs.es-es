---
title: Comparación de opciones de almacenamiento para los clústeres de Azure HDInsight
description: Proporciona información general sobre los tipos de almacenamiento y cómo funcionan con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: ce1c6bdfb38e37c18a18cf970d2dd08683967da3
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536755"
---
# <a name="compare-storage-options-for-use-with-azure-hdinsight-clusters"></a>Comparación de opciones de almacenamiento para los clústeres de Azure HDInsight

Puede elegir entre diferentes servicios de Azure Storage al crear clústeres de HDInsight:

* [Almacenamiento de Azure](./overview-azure-storage.md)
* [Azure Data Lake Storage Gen2](./overview-data-lake-storage-gen2.md)
* [Azure Data Lake Storage Gen1](./overview-data-lake-storage-gen1.md)

En este artículo se proporciona información general sobre estos tipos de almacenamiento y sus características exclusivas.

## <a name="storage-types-and-features"></a>Tipos de almacenamiento y características

En la tabla siguiente se resumen los servicios de Azure Storage que son compatibles con diferentes versiones de HDInsight:

| Servicio de Storage | Tipo de cuenta | Tipo de espacio de nombres | Servicios admitidos | Niveles de rendimiento admitidos | Niveles de acceso admitidos | Versión de HDInsight | Tipo de clúster |
|---|---|---|---|---|---|---|---|
|Azure Data Lake Storage Gen2| Uso general v2 | Jerárquico (sistema de archivos) | Blob | Estándar | Frecuente, esporádico, archivo | 3.6+ | Todos excepto Spark 2.1 y 2.2|
|Azure Storage| Uso general v2 | Object | Blob | Estándar | Frecuente, esporádico, archivo | 3.6+ | All |
|Azure Storage| Uso general v1 | Object | Blob | Estándar | N/D | All | All |
|Azure Storage| Blob Storage** | Object | Blob en bloques | Estándar | Frecuente, esporádico, archivo | All | All |
|Azure Data Lake Storage Gen1| N/D | Jerárquico (sistema de archivos) | N/D | N/D | N/D | Solo 3.6 | Todos excepto HBase |

**Para los clústeres de HDInsight, solo las cuentas de almacenamiento secundarias pueden ser de tipo BlobStorage; Blob en páginas no es una opción de almacenamiento admitida.

Para obtener más información sobre los tipos de cuentas de almacenamiento, consulte [Información general acerca de la cuenta de Azure Storage](../storage/common/storage-account-overview.md).

Para obtener más información sobre los niveles de acceso de Azure Storage, consulte [Azure Blob Storage: niveles de almacenamiento de archivo, esporádico, frecuente y Premium (versión preliminar)](../storage/blobs/storage-blob-storage-tiers.md).

Puede crear clústeres mediante combinaciones de servicios para el almacenamiento principal y el secundario opcional. En la tabla siguiente se resumen las configuraciones de almacenamiento de clúster que actualmente se admiten en HDInsight:

| Versión de HDInsight | Almacenamiento principal | Almacenamiento secundario | Compatible |
|---|---|---|---|
| 3.6 y 4.0 | Uso general V1, uso general V2 | Uso general V1, uso general V2, BlobStorage (blobs en bloques) | Sí |
| 3.6 y 4.0 | Uso general V1, uso general V2 | Data Lake Storage Gen2 | No |
| 3.6 y 4.0 | Data Lake Storage Gen2* | Data Lake Storage Gen2 | Sí |
| 3.6 y 4.0 | Data Lake Storage Gen2* | Uso general V1, uso general V2, BlobStorage (blobs en bloques) | Sí |
| 3.6 y 4.0 | Data Lake Storage Gen2 | Data Lake Storage Gen1 | No |
| 3.6 | Data Lake Storage Gen1 | Data Lake Storage Gen1 | Sí |
| 3.6 | Data Lake Storage Gen1 | Uso general V1, uso general V2, BlobStorage (blobs en bloques) | Sí |
| 3.6 | Data Lake Storage Gen1 | Data Lake Storage Gen2 | No |
| 4.0 | Data Lake Storage Gen1 | Any | No |
| 4.0 | Uso general V1, uso general V2 | Data Lake Storage Gen1 | No |

* Podría tratarse de una o varias cuentas de Data Lake Storage Gen2, siempre y cuando todas estén configuradas para usar la misma identidad administrada para el acceso al clúster.

> [!NOTE]
> El almacenamiento principal de Data Lake Storage Gen2 no es compatible con los clústeres de Spark 2.1 o 2.2.

## <a name="data-replication"></a>Replicación de datos

Azure HDInsight no almacena los datos de los clientes. Los principales medios de almacenamiento para un clúster son sus cuentas de almacenamiento asociadas. Puede asociar el clúster a una cuenta de almacenamiento existente o crear una nueva cuenta de almacenamiento durante el proceso de creación del clúster. Si se crea una nueva, será una cuenta de almacenamiento con redundancia local (LRS) y cumplirá los requisitos de residencia de datos en la región, incluidos los especificados en [Centro de confianza](https://azuredatacentermap.azurewebsites.net).

Puede asegurarse de que HDInsight está configurado correctamente para almacenar datos en una sola región, asegurándose de que la cuenta de almacenamiento asociada a HDInsight sea LRS u otra opción de almacenamiento mencionada en [Centro de confianza](https://azuredatacentermap.azurewebsites.net).
 
## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Storage](./overview-azure-storage.md)
* [Introducción a Azure Data Lake Storage Gen1](./overview-data-lake-storage-gen1.md)
* [Introducción a Azure Data Lake Storage Gen2](./overview-data-lake-storage-gen2.md)
* [Introducción a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Introducción a Almacenamiento de Azure](../storage/common/storage-introduction.md)
