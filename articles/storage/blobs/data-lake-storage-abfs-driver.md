---
title: El controlador Azure Blob Filesystem para Azure Data Lake Storage Gen2
description: 'El controlador Azure Blob Filesystem (ABFS): un controlador de Azure Storage dedicado para Hadoop. Acceda a los datos de Azure Data Lake Storage Gen2 con este controlador.'
author: normesta
ms.topic: conceptual
ms.author: normesta
ms.reviewer: jamesbak
ms.date: 12/06/2018
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: e9c56f59d6dd64002632a5f74e9f39c51293caab
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836367"
---
# <a name="the-azure-blob-filesystem-driver-abfs-a-dedicated-azure-storage-driver-for-hadoop"></a>El controlador Azure Blob Filesystem (ABFS): un controlador de Azure Storage dedicado para Hadoop

Uno de los principales métodos de acceso para los datos en Azure Data Lake Storage Gen2 es mediante [Hadoop FileSystem](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/index.html). Data Lake Storage Gen2 permite a los usuarios de Azure Blob Storage obtener acceso a un nuevo controlador, el controlador del sistema de archivos de blob de Azure o `ABFS`. ABFS forma parte de Apache Hadoop y se incluye en muchas de las distribuciones comerciales de Hadoop. Con este controlador, muchas aplicaciones y plataformas pueden acceder a datos de Azure Blob Storage sin código que haga referencia explícita a Data Lake Storage Gen2.

## <a name="prior-capability-the-windows-azure-storage-blob-driver"></a>Funcionalidad anterior: el controlador Azure Storage Blob para Windows

El controlador de Azure Storage Blob para Windows o [controlador WASB](https://hadoop.apache.org/docs/current/hadoop-azure/index.html) ofrecía la compatibilidad original con Azure Storage Blob. Este controlador realizaba la compleja tarea de asignar semántica de sistema de archivos (como requería la interfaz de Hadoop FileSystem) a la de la interfaz de estilo de almacén de objetos expuesta por Azure Blob Storage. Aunque este controlador aún admite este modelo, y proporciona acceso de alto rendimiento a los datos almacenados en blobs, contiene una importante cantidad de código que realiza esta asignación, lo que hace que sea difícil de mantener. Además, algunas operaciones como [FileSystem.rename()](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/filesystem.html#boolean_renamePath_src_Path_d) y [FileSystem.delete()](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/filesystem.html#boolean_deletePath_p_boolean_recursive), cuando se aplican a directorios, necesitan el controlador para realizar un gran número de operaciones (debido a la falta de compatibilidad de los almacenes de objetos con los directorios), lo que con frecuencia conduce a una degradación en el rendimiento. El controlador ABFS se diseñó para superar las deficiencias inherentes de WASB.

## <a name="the-azure-blob-file-system-driver"></a>El controlador Azure Blob File System

La [interfaz de REST de Azure Data Lake Storage](https://docs.microsoft.com/rest/api/storageservices/data-lake-storage-gen2) está diseñada para admitir la semántica de sistema de archivos a través de Azure Blob Storage. Dado que Hadoop FileSystem también está diseñado para admitir la misma semántica, no hay ningún requisito de una asignación compleja en el controlador. Por tanto, el controlador Azure Blob File System (o ABFS) es una mera corrección de compatibilidad (shim) de cliente para la API REST.

Sin embargo, hay algunas funciones que el controlador debe seguir realizando:

### <a name="uri-scheme-to-reference-data"></a>Esquema de URI para los datos de referencia

En consonancia con otras implementaciones de FileSystem en Hadoop, el controlador ABFS define su propio esquema de URI de modo que los recursos (directorios y archivos) se puedan direccionar claramente. El esquema de URI se documenta en [Use the Azure Data Lake Storage Gen2 URI](./data-lake-storage-introduction-abfs-uri.md) (Uso del URI de Azure Data Lake Storage Gen2). La estructura del URI es: `abfs[s]://file_system@account_name.dfs.core.windows.net/<path>/<path>/<file_name>`

Mediante el formato de URI anterior, se pueden usar herramientas y plataformas estándar de Hadoop para hacer referencia a estos recursos:

```bash
hdfs dfs -mkdir -p abfs://fileanalysis@myanalytics.dfs.core.windows.net/tutorials/flightdelays/data
hdfs dfs -put flight_delays.csv abfs://fileanalysis@myanalytics.dfs.core.windows.net/tutorials/flightdelays/data/
```

Internamente, el controlador ABFS convierte los recursos especificados en el URI en archivos y directorios y realiza llamadas a la API REST de Azure Data Lake Storage con esas referencias.

### <a name="authentication"></a>Autenticación

El controlador ABFS admite dos formas de autenticación, de modo que la aplicación de Hadoop puede acceder de forma segura a los recursos contenidos en la cuenta compatible de Data Lake Storage Gen2. Se proporcionan detalles completos de los esquemas de autenticación disponibles en la [guía de seguridad de Azure Storage](security-recommendations.md). Son las siguientes:

- **Clave compartida:** esto proporciona a los usuarios acceso a todos los recursos de la cuenta. La clave se cifra y almacena en la configuración de Hadoop.

- **Token de portador de OAuth de Azure Active Directory:** el controlador adquiere y actualiza los tokens de portador de Azure AD con la identidad del usuario final o una entidad de servicio configurada. Al usar este modelo de autenticación, se autoriza todo acceso en función de cada llamada, mediante la identidad asociada con el token suministrado y se evalúa en relación con la lista de control de acceso POSIX (ACL) asignada.

   > [!NOTE]
   > Azure Data Lake Storage Gen2 solo admite puntos de conexión de Azure AD v1.0.

### <a name="configuration"></a>Configuración

Toda la configuración del controlador ABFS se almacena en el archivo de configuración <code>core-site.xml</code>. En las distribuciones de Hadoop con [Ambari](https://ambari.apache.org/), la configuración también puede administrarse mediante el portal web o la API REST de Ambari.

Se puede encontrar información detallada de todas las entradas de configuración admitidas en la [documentación oficial de Hadoop](https://hadoop.apache.org/docs/stable/hadoop-azure/abfs.html).

### <a name="hadoop-documentation"></a>Documentación de Hadoop

El controlador ABFS está completamente documentado en la [documentación oficial de Hadoop](https://hadoop.apache.org/docs/stable/hadoop-azure/abfs.html).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un clúster de Azure Databricks](./data-lake-storage-quickstart-create-databricks-account.md)
- [Uso del URI de Azure Data Lake Storage Gen2](./data-lake-storage-introduction-abfs-uri.md)
