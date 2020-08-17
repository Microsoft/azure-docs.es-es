---
title: Restauración a un momento dado para blobs en bloques (versión preliminar)
titleSuffix: Azure Storage
description: La restauración a un momento dado para blobs en bloques proporciona protección contra eliminaciones accidentales o daños, ya que le permite restaurar una cuenta de almacenamiento a su estado anterior en un momento dado en el tiempo.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 06/10/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: b9514bea1f9c34f0ed36bf530a7129b6fea46c4f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501974"
---
# <a name="point-in-time-restore-for-block-blobs-preview"></a>Restauración a un momento dado para blobs en bloques (versión preliminar)

La restauración a un momento dado proporciona protección contra eliminaciones accidentales o daños, ya que le permite restaurar los datos de blobs en bloques a un estado anterior. Asimismo, la restauración a un momento dado resulta útil en escenarios en los que un usuario o una aplicación eliminan datos accidentalmente o donde un error de la aplicación daña los datos. La restauración a un momento dado también permite habilitar escenarios de prueba que requieran revertir un conjunto de datos a un estado conocido antes de ejecutar otras pruebas.

Para obtener información sobre cómo habilitar la restauración a un momento dado para una cuenta de almacenamiento, consulte [Habilitación y administración de la restauración a un momento dado para blobs en bloques (versión preliminar)](point-in-time-restore-manage.md).

## <a name="how-point-in-time-restore-works"></a>Cómo funciona la restauración a un momento dado

Para habilitar la restauración a un momento dado, cree una directiva de administración para la cuenta de almacenamiento y especifique un período de retención. Durante el período de retención, puede restaurar los blobs en bloques desde el estado actual a un estado de un momento dado anterior.

Para iniciar una restauración a un momento dado, llame a la operación [Restore Blob Ranges](/rest/api/storagerp/storageaccounts/restoreblobranges) y especifique un punto de restauración en una hora UTC. Puede especificar rangos lexicográficos de nombres de contenedor y blob para restaurar u omitir el rango para restaurar todos los contenedores de la cuenta de almacenamiento. Se admiten hasta 10 rangos lexicográficos por operación de restauración.

Igualmente, Azure Storage analiza todos los cambios que se han realizado en los blobs especificados entre el punto de restauración solicitado,la hora UTC especificada y el momento presente. La operación de restauración es atómica, por lo que o todos los cambios de la restauración se realizan correctamente, o se produce un error. Si hay blobs que no se pueden restaurar, se produce un error en la operación y se reanudan las operaciones de lectura y escritura en los contenedores afectados.

Recuerde que solo se puede ejecutar una operación de restauración en una cuenta de almacenamiento a la vez. Una operación de restauración no se puede cancelar una vez está en curso, pero se puede realizar una segunda operación de restauración para deshacer la primera operación.

La operación **Restore Blob Ranges** devuelve un id. de restauración que identifica de forma única la operación. Para comprobar el estado de una restauración a un momento dado, llame a la operación **Get Restore Status** con el id. de restauración que se devolvió de la operación **Restore Blob Ranges**.

Tenga en cuenta las siguientes limitaciones en las operaciones de restauración:

- Un bloque que se ha cargado mediante la operación [Put Block](/rest/api/storageservices/put-block) o [Put Block from URL](/rest/api/storageservices/put-block-from-url), pero que no se ha confirmado a través de [Put Block List](/rest/api/storageservices/put-block-list), no es parte de un blob y, por tanto, no se restaura como parte de una operación de restauración.
- No se puede restaurar un blob con una concesión activa. Si un blob con una concesión activa se incluye en el rango de blobs que se van a restaurar, la operación de restauración producirá un error de forma atómica.
- Las instantáneas no se crean ni se eliminan como parte de una operación de restauración. Solo el blob base se restaura a su estado anterior.
- Si un blob se ha desplazado entre los niveles de acceso frecuente y esporádico en el período comprendido entre el momento actual y el punto de restauración, el blob se restaura a su nivel anterior. Sin embargo, no se restaurará un blob que se haya llevado al nivel de archivo.

> [!IMPORTANT]
> Cuando se realiza una operación de restauración, Azure Storage bloquea las operaciones de datos en los blobs de los rangos que se están restaurando mientras dure la operación. Las operaciones de lectura, escritura y eliminación se bloquean en la ubicación principal. Por este motivo, es posible que las operaciones como la enumeración de contenedores en Azure Portal no funcionen según lo esperado mientras se realiza la operación de restauración.
>
> Asimismo, las operaciones de lectura de la ubicación secundaria pueden continuar durante la operación de restauración si la cuenta de almacenamiento tiene replicación geográfica.

> [!CAUTION]
> La restauración a un momento dado solo admite la restauración de operaciones en blobs en bloques. No se pueden restaurar las operaciones en contenedores. Si elimina un contenedor de la cuenta de almacenamiento mediante una llamada a la operación [Delete Container](/rest/api/storageservices/delete-container) durante la vista previa de la restauración a un momento dado, ese contenedor no se puede restaurar con una operación de restauración. Durante la versión preliminar, en lugar de eliminar un contenedor, elimine los blobs individuales si quiere restaurarlos.

### <a name="prerequisites-for-point-in-time-restore"></a>Requisitos previos para la restauración a un momento dado

La restauración a un momento dado requiere que estén habilitadas las siguientes características de Azure Storage:

- [Eliminación temporal](soft-delete-overview.md)
- [Fuente de cambios (versión preliminar)](storage-blob-change-feed.md)
- [Versiones de blobs (versión preliminar)](versioning-overview.md)

Habilite estas características para la cuenta de almacenamiento antes de habilitar la restauración a un momento dado. Asegúrese de registrarse en la fuente de cambios y las versiones preliminares de los blob antes de habilitarlas.

### <a name="retention-period-for-point-in-time-restore"></a>Período de retención para la restauración a un momento dado

Al habilitar la restauración a un momento dado de una cuenta de almacenamiento, se especifica un período de retención. Los blobs en bloques de la cuenta de almacenamiento se pueden restaurar durante el período de retención.

El período de retención comienza cuando se habilita la restauración a un momento dado. Tenga en cuenta que no se pueden restaurar blobs a un estado anterior al inicio del período de retención. Por ejemplo, si habilitó la restauración a un momento dado el 1 de mayo con una retención de 30 días, el 15 de mayo podrá restaurar hasta un máximo de 15 días. Así pues, el 1 de junio podrá restaurar los datos de entre 1 y 30 días.

El período de retención de la restauración a un momento dado debe ser al menos un día inferior que el período de retención especificado en la eliminación temporal. Por ejemplo, si el período de retención de eliminación temporal se establece en 7 días, el período de retención de restauración a un momento dado puede estar comprendido entre 1 y 6 días.

### <a name="permissions-for-point-in-time-restore"></a>Permisos para la restauración a un momento dado

Para iniciar una operación de restauración, un cliente debe tener permisos de escritura en todos los contenedores de la cuenta de almacenamiento. Para conceder permisos para autorizar una operación de restauración con Azure Active Directory (Azure AD), asigne el rol de **Colaborador de la cuenta de almacenamiento** a la entidad de seguridad en el nivel de la cuenta de almacenamiento, el grupo de recursos o la suscripción.

## <a name="about-the-preview"></a>Acerca de la versión preliminar

La restauración a un momento dado solo se admite en las cuentas de almacenamiento (V2) de uso general. Solo se pueden restaurar los datos de los niveles de almacenamiento de acceso frecuente y esporádico con la restauración a un momento dado.

Las siguientes regiones admiten la restauración a un momento dado en la versión preliminar:

- Centro de Canadá
- Este de Canadá
- Centro de Francia

La versión preliminar tiene las limitaciones siguientes:

- No se admite la restauración de blobs en bloques premium.
- No se admite la restauración de blobs en el nivel de archivo. Por ejemplo, si un blob en el nivel de acceso frecuente se movió al nivel de archivo hace dos días y se realiza una operación de restauración a un momento de hace tres días, el blob no se restaura en el nivel de acceso frecuente.
- No se admite la restauración de los espacios de nombres planos y jerárquicos de Azure Data Lake Storage Gen2.
- No se admite la restauración de cuentas de almacenamiento mediante claves proporcionadas por el cliente.

> [!IMPORTANT]
> La versión preliminar de la restauración a un momento dado está pensada para usos que no tengan que ver con la producción. En este momento no hay contratos de nivel de servicio de producción disponibles.

### <a name="register-for-the-preview"></a>Registro para obtener la versión preliminar

Para registrarse y obtener la versión preliminar, ejecute estos comandos:

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Register for the point-in-time restore preview
Register-AzProviderFeature -FeatureName RestoreBlobRanges -ProviderNamespace Microsoft.Storage

# Register for change feed (preview)
Register-AzProviderFeature -FeatureName Changefeed -ProviderNamespace Microsoft.Storage

# Register for blob versioning (preview)
Register-AzProviderFeature -FeatureName Versioning -ProviderNamespace Microsoft.Storage

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az feature register --namespace Microsoft.Storage --name RestoreBlobRanges
az feature register --namespace Microsoft.Storage --name Changefeed
az feature register --namespace Microsoft.Storage --name Versioning
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-registration-status"></a>Comprobación del estado de registro

El registro de la restauración a un momento dado es automático y debe tardar menos de 10 minutos. Ejecute los siguientes comandos para comprobar el estado del registro:

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName RestoreBlobRanges

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Changefeed

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Versioning
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/RestoreBlobRanges')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Changefeed')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Versioning')].{Name:name,State:properties.state}"
```

---

## <a name="pricing-and-billing"></a>Precios y facturación

La facturación de la restauración a un momento dado depende de la cantidad de datos procesados para realizar la operación de restauración. La cantidad de datos procesados se basa en el número de cambios que se produjeron entre el punto de restauración y el momento actual. Por ejemplo, suponiendo una tasa de cambio relativamente constante para los datos de blobs en bloques de una cuenta de almacenamiento, una operación de restauración para revertir los datos 1 día tendría un costo de 1/10 de una restauración que pueda retroceder 10 días.

Para calcular el costo de una operación de restauración, revise el registro de la fuente de cambios para calcular la cantidad de datos que se modificaron durante el período de restauración. Por ejemplo, si el período de retención de la fuente de cambios es de 30 días y el tamaño de la fuente de cambios es de 10 MB, la restauración a un punto anterior de 10 días costaría aproximadamente un tercio del precio de una cuenta de LRS en esa región. La restauración a un punto que sea de 27 días costaría aproximadamente nueve décimos del precio de la lista.

Para obtener más información sobre los precios de las restauraciones a un momento dado, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="ask-questions-or-provide-feedback"></a>Formule preguntas o envíe comentarios

Para formular preguntas sobre la versión preliminar de la restauración a un momento dado o para escribir comentarios, póngase en contacto con Microsoft en pitrdiscussion@microsoft.com.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación y administración de la restauración a un momento dado para blobs en bloques (versión preliminar)](point-in-time-restore-manage.md)
- [Compatibilidad con la fuente de cambios en Azure Blob Storage (versión preliminar)](storage-blob-change-feed.md)
- [Habilitación de la eliminación temporal para blobs](soft-delete-enable.md)
- [Habilitación y administración de las versiones de blob](versioning-enable.md)
