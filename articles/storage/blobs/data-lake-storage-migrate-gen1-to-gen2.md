---
title: Migración de Azure Data Lake Storage de Gen1 a Gen2
description: Migración de Azure Data Lake Storage de Gen1 a Gen2.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.date: 03/11/2020
ms.service: storage
ms.reviewer: rukmani-msft
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 2cfe5f763b3dedd68c5180f5a0a3c7a3f4ea3e93
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496430"
---
# <a name="migrate-azure-data-lake-storage-from-gen1-to-gen2"></a>Migración de Azure Data Lake Storage de Gen1 a Gen2

Puede migrar los datos, las cargas de trabajo y las aplicaciones de Data Lake Storage Gen1 a Data Lake Storage Gen2.

‎Azure Data Lake Storage Gen2 se basa en [Azure Blob Storage](storage-blobs-introduction.md) y proporciona un conjunto de funcionalidades dedicadas al análisis de macrodatos. [Data Lake Storage Gen2](https://azure.microsoft.com/services/storage/data-lake-storage/) combina características de [Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/index), como la semántica del sistema de archivos y la seguridad y escala de nivel de directorio y archivo con las funcionalidades de recuperación ante desastres y alta disponibilidad, así como de almacenamiento en niveles de bajo costo de [Azure Blob Storage](storage-blobs-introduction.md).

> [!NOTE]
> Para facilitar la lectura, en este artículo se usa el término *Gen1* para hacer referencia a Azure Data Lake Storage Gen1 y el término *Gen2* para hacer referencia a Azure Data Lake Storage Gen2.

## <a name="recommended-approach"></a>Enfoque recomendado

Para migrar a Gen2, se recomienda usar el siguiente método.

:heavy_check_mark: Paso 1: evaluar la preparación

:heavy_check_mark: Paso 2: preparar la migración

:heavy_check_mark: Paso 3: migrar los datos y las cargas de trabajo de las aplicaciones

:heavy_check_mark: Paso 4: pasar de Gen1 a Gen2

> [!NOTE]
> Gen1 y Gen2 son servicios diferentes, no hay ninguna experiencia de actualización local, se requiere un esfuerzo de migración intencional. 

### <a name="step-1-assess-readiness"></a>Paso 1: Evaluación de la preparación

1. Obtenga información sobre la [oferta de Data Lake Storage Gen2](https://azure.microsoft.com/services/storage/data-lake-storage/), sus beneficios, los costos y la arquitectura general. 

2. [Compare las funcionalidades](#gen1-gen2-feature-comparison) de Gen1 con las de Gen2. 

3. Revise una lista de [problemas conocidos](data-lake-storage-known-issues.md) para evaluar cualquier brecha que se produzca en la funcionalidad.

4. Gen2 admite las características de Blob Storage, como el [registro de diagnóstico](../common/storage-analytics-logging.md), los [niveles de acceso](storage-blob-storage-tiers.md) y las [directivas de administración del ciclo de vida de Blob Storage](storage-lifecycle-management-concepts.md). Si le interesa usar cualquiera de estas características, vea el [nivel actual de soporte técnico](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-supported-blob-storage-features).

5. Vea el estado actual de [compatibilidad con el ecosistema de Azure](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-multi-protocol-access) para asegurarse de que Gen2 admite todos los servicios de los que dependen sus soluciones.

### <a name="step-2-prepare-to-migrate"></a>Paso 2: Preparación de la migración

1. Identifique los conjuntos de datos que va a migrar.

   Aproveche esta oportunidad para limpiar los conjuntos de datos que ya no usa. A menos que planee migrar todos los datos al mismo tiempo, dedique este tiempo a identificar los grupos lógicos de datos que puede migrar en fases.
   
2. Determine el impacto que una migración tendrá en su empresa.

   Por ejemplo, piense si puede permitirse que haya un tiempo de inactividad mientras se realiza la migración. Estas consideraciones pueden ayudarle a identificar un patrón de migración adecuado y a elegir las herramientas más adecuadas.

3. Cree un plan de migración. 

   Se recomiendan estos [patrones de migración](#migration-patterns). Puede elegir uno de estos patrones, combinarlos o diseñar un patrón personalizado propio.

### <a name="step-3-migrate-data-workloads-and-applications"></a>Paso 3: Migración de datos, cargas de trabajo y aplicaciones

Migre los datos, las cargas de trabajo y las aplicaciones con el patrón que prefiera. Se recomienda validar los escenarios de forma incremental.

1. [Cree una cuenta de almacenamiento](data-lake-storage-quickstart-create-account.md) y habilite la característica de espacio de nombres jerárquico. 

2. Migre los datos. 

3. Configure los [servicios de las cargas de trabajo](data-lake-storage-integrate-with-azure-services.md) para que apunten a su punto de conexión de Gen2. 
   
4. Actualice las aplicaciones para que usen las API de Gen2. Vea las guías de [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md), [JavaScript](data-lake-storage-directory-file-acl-javascript.md) y [REST](https://docs.microsoft.com/rest/api/storageservices/data-lake-storage-gen2). 
   
5. Actualice los scripts para usar los [cmdlets de PowerShell](data-lake-storage-directory-file-acl-powershell.md) y los [comandos de la CLI de Azure](data-lake-storage-directory-file-acl-cli.md) de Data Lake Storage Gen2.
   
6. Busque referencias del identificador URI que contengan la cadena `adl://` en los archivos de código o en los cuadernos de Databricks, archivos HQL de Apache Hive o cualquier otro archivo que se use como parte de las cargas de trabajo. Sustituya estas referencias por el [identificador URI con formato de Gen2](data-lake-storage-introduction-abfs-uri.md) de la nueva cuenta de almacenamiento. Por ejemplo: el identificador URI de Gen1: `adl://mydatalakestore.azuredatalakestore.net/mydirectory/myfile` podría convertirse en `abfss://myfilesystem@mydatalakestore.dfs.core.windows.net/mydirectory/myfile`. 

7. Configure la seguridad en su cuenta para que incluya [roles de Azure](../common/storage-auth-aad-rbac-portal.md), [seguridad de nivel de archivo y carpeta](data-lake-storage-access-control.md), así como [redes virtuales y firewalls de Azure Storage](../common/storage-network-security.md).

### <a name="step-4-cutover-from-gen1-to-gen2"></a>Paso 4: Paso de Gen1 a Gen2

Una vez que esté seguro de que tanto las aplicaciones como las cargas de trabajo son estables en Gen2, puede empezar a usar Gen2 para satisfacer sus escenarios empresariales. Desactive las restantes canalizaciones que se estén ejecutando en Gen1 y dé de baja su cuenta de Gen1. 

<a id="gen1-gen2-feature-comparison"></a>

## <a name="gen1-vs-gen2-capabilities"></a>Funcionalidades de Gen1 frente a las de Gen2

En esta tabla se comparan las funcionalidades de Gen1 con las de Gen2.

|Área |Gen1   |Gen2 |
|---|---|---|
|Organización de datos|[Espacio de nombres jerárquico](data-lake-storage-namespace.md)<br>Compatibilidad con archivos y carpetas|[Espacio de nombres jerárquico](data-lake-storage-namespace.md)<br>Compatibilidad con contenedores, archivos y carpetas |
|Redundancia geográfica| [LRS](../common/storage-redundancy.md#locally-redundant-storage)| [LRS](../common/storage-redundancy.md#locally-redundant-storage), [ZRS](../common/storage-redundancy.md#zone-redundant-storage), [GRS](../common/storage-redundancy.md#geo-redundant-storage), [RA-GRS](../common/storage-redundancy.md#read-access-to-data-in-the-secondary-region) |
|Authentication|[Identidad administrada de AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Entidades de servicio](../../active-directory/develop/app-objects-and-service-principals.md)|[Identidad administrada de AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Entidades de servicio](../../active-directory/develop/app-objects-and-service-principals.md)<br>[Clave de acceso compartido](https://docs.microsoft.com/rest/api/storageservices/authorize-with-shared-key)|
|Authorization|Administración: [RBAC](../../role-based-access-control/overview.md)<br>Datos: [ACL](data-lake-storage-access-control.md)|Administración: [RBAC](../../role-based-access-control/overview.md)<br>Datos: [ACL](data-lake-storage-access-control.md) y [RBAC](../../role-based-access-control/overview.md) |
|Cifrado: datos en reposo|Lado del servidor: con claves [administradas por Microsoft](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) o [administradas por el cliente](../common/encryption-customer-managed-keys.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|Lado del servidor: con claves [administradas por Microsoft](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) o [administradas por el cliente](../common/encryption-customer-managed-keys.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Compatibilidad de redes virtuales|[Integración con red virtual](../../data-lake-store/data-lake-store-network-security.md)|[Puntos de conexión de servicio](../common/storage-network-security.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), [puntos de conexión privados](../common/storage-private-endpoints.md)|
|Experiencia del desarrollador|[REST](../../data-lake-store/data-lake-store-data-operations-rest-api.md), [.NET](../../data-lake-store/data-lake-store-data-operations-net-sdk.md), [Java](../../data-lake-store/data-lake-store-get-started-java-sdk.md), [Python](../../data-lake-store/data-lake-store-data-operations-python.md), [PowerShell](../../data-lake-store/data-lake-store-get-started-powershell.md), [CLI de Azure](../../data-lake-store/data-lake-store-get-started-cli-2.0.md)|Disponible con carácter general: [REST](/rest/api/storageservices/data-lake-storage-gen2), [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md)<br>Versión preliminar pública: [JavaScript](data-lake-storage-directory-file-acl-javascript.md), [PowerShell](data-lake-storage-directory-file-acl-powershell.md), [CLI de Azure](data-lake-storage-directory-file-acl-cli.md)|
|Registros del recurso|Registros clásicos<br>[Integración de Azure Monitor](../../data-lake-store/data-lake-store-diagnostic-logs.md)|[Registros clásicos](../common/storage-analytics-logging.md): disponibilidad general<br>Integración de Azure Monitor: escala de tiempo por determinar|
|Ecosistema|[HDInsight (3.6)](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md), [Azure Databricks (3.1 y superior)](https://docs.databricks.com/data/data-sources/azure/azure-datalake.html), [SQL DW](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store), [ADF](../../data-factory/load-azure-data-lake-store.md)|[HDInsight (3.6, 4.0)](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md), [Azure Databricks (5.1 y superior)](https://docs.microsoft.com/azure/databricks/data/data-sources/azure/azure-datalake-gen2), [SQL DW](../../azure-sql/database/vnet-service-endpoint-rule-overview.md), [ADF](../../data-factory/load-azure-data-lake-storage-gen2.md)|

<a id="migration-patterns"></a>

## <a name="gen1-to-gen2-patterns"></a>Patrones de Gen1 a Gen2

Elija un patrón de migración y, después, modifíquelo tanto como sea necesario.

|||
|---|---|
|**Lift-and-shift**|El patrón más simple. Ideal si sus canalizaciones de datos pueden permitirse un tiempo de inactividad.|
|**Copia incremental**|Similar a *lift-and-shift*, pero con menos tiempo de inactividad. Ideal para grandes cantidades de datos que tardan más tiempo en copiarse.|
|**Canalización dual**|Ideal para canalizaciones que no pueden permitirse ningún tiempo de inactividad.|
|**Sincronización bidireccional**|Es similar a la *canalización dual*, pero con un enfoque más escalonado, que es adecuado para canalizaciones más complicadas.|

Examinemos más detenidamente cada patrón.
 
### <a name="lift-and-shift-pattern"></a>Patrón lift-and-shift

Este es el patrón más sencillo.

1. Detenga todas las escrituras en Gen1.

2. Mueva los datos de Gen1 a Gen2. Se recomienda [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage). Las ACL se copian con los datos.

3. Apunte las operaciones de ingesta y las cargas de trabajo a Gen2.

4. Dé de baja Gen1.

> [!div class="mx-imgBorder"]
> ![Patrón lift-and-shift](./media/data-lake-storage-migrate-gen1-to-gen2/lift-and-shift.png)

#### <a name="considerations-for-using-the-lift-and-shift-pattern"></a>Consideraciones sobre el uso del patrón lift-and-shift

:heavy_check_mark: Pase de Gen1 a Gen2 en todas las cargas de trabajo al mismo tiempo.

:heavy_check_mark: Cabe esperar que se produzca un tiempo de inactividad durante la migración y el período de transferencia.

:heavy_check_mark: Ideal para canalizaciones que pueden permitirse un tiempo de inactividad y todas las aplicaciones se pueden actualizar a la vez.

### <a name="incremental-copy-pattern"></a>Patrón de copia incremental

1. Empiece a mover los datos de Gen1 a Gen2. Se recomienda [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage). Las ACL se copian con los datos.

2. Copie de forma incremental los datos nuevos de Gen1.

3. Después de que se copien todos los datos, detenga todas las escrituras en Gen1 y apunte las cargas de trabajo a Gen2.

4. Dé de baja Gen1.

> [!div class="mx-imgBorder"]
> ![Patrón de copia incremental](./media/data-lake-storage-migrate-gen1-to-gen2/incremental-copy.png)

#### <a name="considerations-for-using-the-incremental-copy-pattern"></a>Consideraciones sobre el uso del patrón de copia incremental:

:heavy_check_mark: Pase de Gen1 a Gen2 en todas las cargas de trabajo al mismo tiempo.

:heavy_check_mark: Cabe esperar un tiempo de inactividad solo durante el período de transferencia.

:heavy_check_mark: Ideal para canalizaciones en las que todas las aplicaciones se han actualizado al mismo tiempo, pero la copia de datos requiere más tiempo.

### <a name="dual-pipeline-pattern"></a>Patrón de canalización dual

1. Mueva los datos de Gen1 a Gen2. Se recomienda [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage). Las ACL se copian con los datos.

2. Ingiera los datos nuevos tanto a Gen1 como a Gen2.

3. Apunte las cargas de trabajo a Gen2.

4. Detenga todas las escrituras en Gen1 y, después, dé de baja Gen1.

> [!div class="mx-imgBorder"]
> ![Patrón de canalización dual](./media/data-lake-storage-migrate-gen1-to-gen2/dual-pipeline.png)

#### <a name="considerations-for-using-the-dual-pipeline-pattern"></a>Consideraciones sobre el uso del patrón de canalización dual:

:heavy_check_mark: Las canalizaciones de Gen1 y Gen2 se ejecutan en paralelo.

:heavy_check_mark: Admite el tiempos de inactividad cero.

:heavy_check_mark: Ideal en aquellas situaciones en las que las cargas de trabajo y las aplicaciones no se pueden permitir ningún tiempo de inactividad, y el usuario puede ingerir en ambas cuentas de almacenamiento.

### <a name="bi-directional-sync-pattern"></a>Patrón de sincronización bidireccional

1. Configure la replicación bidireccional entre Gen1 y Gen2. Se recomienda utilizar [WanDisco](https://docs.wandisco.com/bigdata/wdfusion/adls/). Ofrece una característica de reparación para los datos existentes.

3. Cuando se hayan completado todos los movimientos, detenga todas las escrituras en Gen1 y desactive la replicación bidireccional.

4. Dé de baja Gen1.

> [!div class="mx-imgBorder"]
> ![Patrón bidireccional](./media/data-lake-storage-migrate-gen1-to-gen2/bidirectional-sync.png)

#### <a name="considerations-for-using-the-bi-directional-sync-pattern"></a>Consideraciones sobre el uso del patrón de sincronización bidireccional:

:heavy_check_mark: Ideal para escenarios complejos que implican un gran número de canalizaciones y dependencias en las que un enfoque por fases podría tener más sentido.  

:heavy_check_mark: El esfuerzo de la migración es elevado, pero proporciona compatibilidad en paralelo para Gen1 y Gen2.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre las distintas partes de la configuración de la seguridad de una cuenta de almacenamiento. Consulte la [guía de seguridad de Azure Storage](../common/storage-security-guide.md).
- Optimice el rendimiento de Data Lake Store. Consulte Optimización de [Azure Data Lake Storage Gen2 para el rendimiento](data-lake-storage-performance-tuning-guidance.md)
- Revise los procedimientos recomendados para administrar Data Lake Store. Consulte [Procedimientos recomendados para usar Azure Data Lake Storage Gen2](data-lake-storage-best-practices.md)
