---
title: Problemas conocidos con Azure Data Lake Storage Gen2 | Microsoft Docs
description: Más información sobre las limitaciones y problemas conocidos con Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 07/21/2020
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 0309f4d07056e3986138139e10ab29faa675cfcd
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056085"
---
# <a name="known-issues-with-azure-data-lake-storage-gen2"></a>Problemas conocidos con Azure Data Lake Storage Gen2

En este artículo se describen las limitaciones y problemas conocidos con Azure Data Lake Storage Gen2.

## <a name="supported-blob-storage-features"></a>Características de Blob Storage compatibles

Ahora, un número creciente de características de Blob Storage funciona con cuentas que tienen un espacio de nombres jerárquico. Para obtener una lista completa, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="supported-azure-service-integrations"></a>Integraciones de servicios de Azure admitidos

Azure Data Lake Storage Gen2 admite varios servicios de Azure que se pueden usar para ingerir datos, realizar análisis y crear representaciones visuales. Para ver una lista de los servicios de Azure admitidos, consulte el documento en que se indica cuáles son los [servicios de Azure que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

Consulte [Servicios de Azure que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

## <a name="supported-open-source-platforms"></a>Plataformas de código abierto compatibles

Varias plataformas de código abierto son compatibles con Data Lake Storage Gen2. Para ver una lista completa, consulte [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

Consulte [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

## <a name="blob-storage-apis"></a>API de Blob Storage

Las API de Blob y Data Lake Storage Gen2 API pueden operar en los mismos datos.

En esta sección se describen los problemas y las limitaciones del uso de las API de blob y de las API de Data Lake Storage Gen2 para operar en los mismos datos.

* No puede usar las API de blob y las API de Data Lake Storage para escribir en la misma instancia de un archivo. Si escribe en un archivo con la API de Data Lake Storage Gen2, los bloques de ese archivo no estarán visibles para las llamadas a la API de blob [Get Block List](https://docs.microsoft.com/rest/api/storageservices/get-block-list). Puede sobrescribir un archivo con las API de Data Lake Storage Gen2 o con las API de blob. No afectará a las propiedades del archivo.

* Cuando use la operación [List Blobs](https://docs.microsoft.com/rest/api/storageservices/list-blobs) (Enumerar blobs) sin especificar un delimitador, los resultados incluirán tanto los directorios como los blobs. Si decide usar un delimitador, use solo una barra diagonal (`/`). Este es el único delimitador admitido.

* Si usa la API [Delete Blob](https://docs.microsoft.com/rest/api/storageservices/delete-blob) para eliminar un directorio, solo se eliminará si está vacío. Esto significa que no puede utilizar la API de blob para eliminar los directorios de forma recursiva.

Las siguientes API REST de blob no se admiten:

* [Put Blob (Page)](https://docs.microsoft.com/rest/api/storageservices/put-blob) [Poner blob (página)]
* [Put Page](https://docs.microsoft.com/rest/api/storageservices/put-page)
* [Get Page Ranges](https://docs.microsoft.com/rest/api/storageservices/get-page-ranges) (Conseguir intervalos de páginas)
* [Incremental Copy Blob](https://docs.microsoft.com/rest/api/storageservices/incremental-copy-blob) (Copia incremental del blob)
* [Put Page from URL](https://docs.microsoft.com/rest/api/storageservices/put-page-from-url) (Poner página de dirección URL)
* [Append Block from URL](https://docs.microsoft.com/rest/api/storageservices/append-block-from-url) (Anexar bloque desde dirección URL)

No se admiten discos de máquina virtual no administrados en cuentas que tengan un espacio de nombres jerárquico. Si desea habilitar un espacio de nombres jerárquico en una cuenta de almacenamiento, coloque los discos de máquina virtual no administrados en una cuenta de almacenamiento que no tenga habilitada la característica de espacio de nombres jerárquico.

<a id="api-scope-data-lake-client-library"></a>

## <a name="file-system-support-in-sdks-powershell-and-azure-cli"></a>Compatibilidad del sistema de archivos en los SDK, PowerShell y la CLI de Azure

- Las operaciones de ACL Get y Set no son recursivas actualmente.

<a id="known-issues-tools"></a>

## <a name="azcopy"></a>AzCopy

Use solo la versión más reciente de AzCopy ([AzCopy v10](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2ftables%2ftoc.json)). No se admiten versiones anteriores de AzCopy, como AzCopy v8.1.

<a id="storage-explorer"></a>

## <a name="azure-storage-explorer"></a>Explorador de Azure Storage

Use solo las versiones  `1.6.0`  o superiores.

<a id="explorer-in-portal"></a>

## <a name="storage-explorer-in-the-azure-portal"></a>Explorador de Storage en Azure Portal

Las ACL aún no se admiten.

<a id="third-party-apps"></a>

## <a name="thirdpartyapplications"></a>Aplicaciones de terceros

Las aplicaciones de terceros que usan las API REST para funcionar seguirán funcionando si se usan con Data Lake Storage Gen2. Lo más probable es que las aplicaciones que llaman a las API de Blob funcionen.

## <a name="access-control-lists-acl-and-anonymous-read-access"></a>Listas de control de acceso (ACL) y acceso de lectura anónimo

Si se ha concedido [acceso de lectura anónimo](storage-manage-access-to-resources.md) a un contenedor, las ACL no tendrán ningún efecto en ese contenedor o los archivos de ese contenedor.

## <a name="premium-performance-blockblobstorage-storage-accounts"></a>Cuentas de almacenamiento BlockBlobStorage de alto rendimiento

### <a name="diagnostic-logs"></a>Registros de diagnóstico

Los registros de diagnóstico aún no se pueden habilitar mediante Azure Portal. Puede habilitarlos mediante PowerShell. Por ejemplo:

```powershell
#To login
Connect-AzAccount

#Set default block blob storage account.
Set-AzCurrentStorageAccount -Name premiumGen2Account -ResourceGroupName PremiumGen2Group

#Enable logging
Set-AzStorageServiceLoggingProperty -ServiceType Blob -LoggingOperations read,write,delete -RetentionDays 14
```

Todavía no se admite la configuración de los días de retención, pero puede eliminar los registros manualmente mediante cualquier herramienta compatible como Explorador de Azure Storage, REST o un SDK.

### <a name="lifecycle-management-policies"></a>Directivas de administración del ciclo de vida

- Las directivas de administración del ciclo de vida todavía no se admiten en cuentas de almacenamiento BlockBlobStorage premium. 

- Los datos no se pueden pasar del nivel Premium a niveles inferiores. 

- La acción **Eliminar blob** no se admite actualmente. 

### <a name="hdinsight-support"></a>Compatibilidad con HDInsight

Al crear un clúster de HDInsight, todavía no se puede seleccionar una cuenta BlockBlobStorage con la característica de espacio de nombres jerárquico habilitada en ella. Sin embargo, puede asociar la cuenta al clúster después de haberla creado.

### <a name="dremio-support"></a>Compatibilidad con Dremio

Dremio todavía no se conecta a una cuenta BlockBlobStorage con la característica de espacio de nombres jerárquico habilitada en ella. 

## <a name="windows-azure-storage-blob-wasb-driver-unsupported-with-data-lake-storage-gen2"></a>Controlador Windows Azure Storage Blob (WASB) (no compatible con Data Lake Storage Gen2)

Actualmente, el controlador WASB, que se diseñó para funcionar solo con la API Blob, experimenta problemas en algunos escenarios comunes. En concreto, cuando es un cliente en una cuenta de almacenamiento habilitada para espacios de nombres jerárquicos. El acceso multiprotocolo en Data Lake Storage no mitigará estos problemas. 

Por el momento (y probablemente en un futuro inmediato), no se ofrecerá compatibilidad para el controlador WASB como cliente en una cuenta de almacenamiento habilitada para espacios de nombres jerárquicos. En su lugar, se recomienda que utilice el controlador de [Azure Blob File System (ABFS)](data-lake-storage-abfs-driver.md) en su entorno de Hadoop. Si está intentando migrar de un entorno de Hadoop local con una versión anterior a la rama de Hadoop 3, abra una incidencia de soporte técnico de Azure para que podamos ponernos en contacto con usted para indicarle el camino correcto para usted y su organización.
