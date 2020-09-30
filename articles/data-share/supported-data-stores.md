---
title: Almacenes de datos admitidos en Azure Data Share
description: Obtenga información sobre los almacenes de datos que se admiten para usar Azure Data Share.
ms.service: data-share
author: jifems
ms.author: jife
ms.topic: conceptual
ms.date: 08/14/2020
ms.openlocfilehash: bb8b13e1141a8cb4610e15ed693e28042dd20d72
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259020"
---
# <a name="supported-data-stores-in-azure-data-share"></a>Almacenes de datos admitidos en Azure Data Share

Azure Data Share proporciona un uso compartido de datos abierto y flexible, lo que incluye la habilidad de compartir desde y hacia diferentes almacenes de datos. Los proveedores de datos pueden compartir datos de un tipo de almacén de datos y sus consumidores pueden elegir en qué almacén se van a recibir los datos. 

En este artículo, obtendrá información sobre el amplio conjunto de almacenes de datos de Azure que se admiten en Azure Data Share. También puede encontrar información sobre las combinaciones de almacenes de datos que pueden aprovechar los proveedores y los consumidores de datos. 

## <a name="what-data-stores-are-supported-in-azure-data-share"></a>¿Qué almacenes de datos se admiten en Azure Data Share? 

La siguiente tabla detalla los orígenes de datos admitidos en Azure Data Share. 

| Almacén de datos | Uso compartido basado en instantáneas | Uso compartido en contexto 
|:--- |:--- |:--- |:--- |:--- |:--- |
| Azure Blob Storage |✓ | |
| Azure Data Lake Storage Gen1 |✓ | |
| Azure Data Lake Storage Gen2 |✓ ||
| Azure SQL Database |Vista previa pública | |
| Azure Synapse Analytics (anteriormente Azure SQL DW) |Vista previa pública | |
| Explorador de datos de Azure | |✓ |

## <a name="data-store-support-matrix"></a>Matriz de soporte de almacén de datos

Azure Data Share ofrece a los consumidores de datos flexibilidad a la hora de decidir un almacén en donde aceptar datos. Por ejemplo, los datos compartidos de Azure SQL Database se pueden recibir en Azure Data Lake Store Gen2, Azure SQL Database o Azure Synapse Analytics. Los clientes pueden elegir en qué formato recibir los datos cuando configuran un recurso compartido de datos recibidos. 

La siguiente tabla detalla las diferentes combinaciones y opciones que tienen los consumidores de datos cuando aceptan y configuran sus datos compartidos. Para más información sobre cómo configurar asignaciones de conjuntos de datos, consulte [cómo configurar asignaciones de conjuntos de datos](how-to-configure-mapping.md).

| Almacén de datos | Azure Blob Storage | Azure Data Lake Storage Gen1 | Azure Data Lake Storage Gen2 | Azure SQL Database | Azure Synapse Analytics | Explorador de datos de Azure
|:--- |:--- |:--- |:--- |:--- |:--- |:--- |
| Azure Blob Storage | ✓ || ✓ ||
| Azure Data Lake Storage Gen1 | ✓ | | ✓ ||
| Azure Data Lake Storage Gen2 | ✓ | | ✓ ||
| Azure SQL Database | ✓ | | ✓ | ✓ | ✓ ||
| Azure Synapse Analytics (anteriormente Azure SQL DW) | ✓ | | ✓ | ✓ | ✓ ||
| Explorador de datos de Azure |||||| ✓ |

## <a name="share-from-a-storage-account"></a>Uso compartido desde una cuenta de almacenamiento
Azure Data Share admite el uso compartido de archivos, carpetas y sistemas de archivos desde Azure Data Lake Gen1 y Azure Data Lake Gen2. También admite el uso compartido de blobs, carpetas y contenedores desde Azure Blob Storage. Actualmente, solo se admiten blobs en bloques. Cuando los sistemas de archivos, contenedores o carpetas se comparten mediante el uso compartido basado en instantáneas, el consumidor de datos puede optar por hacer una copia completa de los datos de los recursos compartidos o aprovechar la capacidad de realizar instantáneas incrementales para copiar solo los archivos nuevos o actualizados. La instantánea incremental se basa en la hora de la última modificación de los archivos. Los archivos existentes con el mismo nombre se sobrescribirán.

Para obtener más detalles, consulte [Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage](how-to-share-from-storage.md).

## <a name="share-from-a-sql-based-source"></a>Uso compartido desde un origen basado en SQL
Azure Data Share admite el uso compartido de tablas o vistas desde Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL DW). Los consumidores de datos pueden elegir aceptar los datos en Azure Data Lake Storage Gen2 o Azure Blob Storage como archivo CSV o Parquet, así como en Azure SQL Database y Azure Synapse Analytics como tablas.

Al aceptar los datos en Azure Data Lake Store Gen2 o Azure Blob Storage, las instantáneas completas sobrescriben el contenido del archivo de destino si este ya existe.
Cuando se reciben los datos en la tabla y si la tabla de destino no existe todavía, Azure Data Share crea la tabla SQL con el esquema de origen. Si ya existe una tabla de destino con el mismo nombre, se quitará y se sobrescribirá con la instantánea completa más reciente. Actualmente no se admiten instantáneas incrementales.

Para obtener más detalles, consulte [Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics](how-to-share-from-sql.md).

## <a name="share-from-azure-data-explorer"></a>Uso compartido desde Azure Data Explorer
Azure Data Share permite compartir bases de datos en contexto desde los clústeres de Azure Data Explorer. El proveedor de datos puede compartir en el nivel de base de datos o de clúster. Cuando se comparta en el nivel de base de datos, el consumidor de datos solo podrá acceder a las bases de datos específicas compartidas por el proveedor de datos. Cuando se comparta en el nivel de clúster, el consumidor de datos puede acceder a todas las bases de datos desde el clúster del proveedor, incluidas todas las bases de datos futuras creadas por el proveedor de datos.

Para acceder a las bases de datos compartidas, el consumidor de datos debe tener su propio clúster de Azure Data Explorer. Un clúster de Azure Data Explorer del consumidor de datos tiene que encontrarse en el mismo centro de datos de Azure que el clúster de Data Explorer del proveedor de datos. Cuando se establece la relación de uso compartido, Azure Data Share crea un vínculo simbólico entre los clústeres de Azure Data Explorer del consumidor y el proveedor. Los datos ingeridos mediante el uso del modo por lotes en el clúster de Azure Data Explorer de origen se mostrarán en el clúster de destino en unos segundos o unos minutos.

Para obtener más detalles, consulte [Uso compartido y recepción de datos de Azure Data Explorer](/azure/data-explorer/data-share). 

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo empezar a compartir datos, vaya al tutorial [Uso compartido de datos](share-your-data.md).
