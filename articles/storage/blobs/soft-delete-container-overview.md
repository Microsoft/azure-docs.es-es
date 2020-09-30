---
title: Eliminación temporal de contenedores (versión preliminar)
titleSuffix: Azure Storage
description: La eliminación temporal para contenedores (versión preliminar) protege los datos para que pueda recuperarlos más fácilmente si una aplicación u otro usuario de la cuenta de almacenamiento los modifican o eliminan por error.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 08/25/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: references_regions
ms.openlocfilehash: f41caf61ac23b05ba0ff95d785be2e7449adf75d
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89228637"
---
# <a name="soft-delete-for-containers-preview"></a>Eliminación temporal de contenedores (versión preliminar)

La eliminación temporal de contenedores (versión preliminar) evita que los datos se modifiquen o eliminen de forma accidental o errónea. Cuando la eliminación temporal de contenedores está habilitada para una cuenta de almacenamiento, los contenedores eliminados y su contenido se conservan en Azure Storage durante el período que especifique. Durante el período de retención, puede restaurar los contenedores previamente eliminados y los blobs que aquellos contienen.

Para una protección integral de los datos de blobs, Microsoft recomienda habilitar las siguientes características de protección de datos:

- Eliminación temporal de contenedores, para proteger frente a la eliminación o sobrescritura accidentales de un contenedor. Para obtener información sobre cómo habilitar la eliminación temporal de contenedores, consulte [Habilitación y administración de la eliminación temporal de contenedores](soft-delete-container-enable.md).
- Eliminación temporal de blobs, para proteger frente a la eliminación o sobrescritura accidentales de un blob individual. Para obtener información sobre cómo habilitar la eliminación temporal de blobs, consulte [Eliminación temporal para blobs](soft-delete-blob-overview.md).
- Control de versiones de blobs, para conservar automáticamente las versiones anteriores de un blob. Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error. Para obtener información sobre cómo habilitar el control de versiones de blobs, consulte [Habilitación y administración del control de versiones de blobs](versioning-enable.md).

> [!WARNING]
> No se puede deshacer la eliminación de una cuenta de almacenamiento. La eliminación temporal no protege contra la eliminación de una cuenta de almacenamiento. Para evitar la eliminación accidental de una cuenta de almacenamiento, configure el bloqueo **CannotDelete** en el recurso de la cuenta de almacenamiento. Para obtener más información sobre el bloqueo de recursos de Azure, vea [Bloqueo de recursos para impedir cambios inesperados](../../azure-resource-manager/management/lock-resources.md).

## <a name="how-container-soft-delete-works"></a>Funcionamiento de la eliminación temporal de contenedores

Al habilitar la eliminación temporal de contenedores, puede especificar un período de retención de 1 a 365 días para los contenedores eliminados. El período de retención predeterminado es de 7 días. Durante el período de retención, puede recuperar un contenedor eliminado mediante una llamada a la operación **Recuperar contenedor**.

Cuando restaure un contenedor, puede restaurarlo a su nombre original si ese nombre no se ha reutilizado. Si se ha usado el nombre del contenedor original, puede restaurar el contenedor con un nuevo nombre.

Una vez que haya expirado el período de retención, el contenedor se eliminará de manera permanente de Azure Storage y no se podrá recuperar. El reloj del período de retención comienza en el momento en que se elimina el contenedor. Puede cambiar el período de retención en cualquier momento, pero tenga en cuenta que un período de retención actualizado se aplica solo a los contenedores recién eliminados. Los contenedores previamente eliminados se eliminarán de forma permanente según el período de retención que estaba en vigor en el momento en que se eliminó el contenedor.

La deshabilitación de la eliminación temporal de contenedores no provoca la eliminación permanente de los contenedores que se eliminaron temporalmente. Los contenedores que ya haya eliminado temporalmente se eliminarán de forma permanente en el momento de expiración del período de retención que estaba en vigor en el momento en que se eliminó el contenedor.

## <a name="about-the-preview"></a>Acerca de la versión preliminar

La eliminación temporal de contenedores está disponible en versión preliminar en las siguientes regiones:

- Centro de Francia
- Este de Canadá
- Centro de Canadá

> [!IMPORTANT]
> La versión preliminar de la eliminación temporal de contenedores está pensada para usos que no tengan que ver con la producción. En este momento no hay contratos de nivel de servicio de producción disponibles.

La versión 2019-12-12 y posteriores de la API de REST de Azure Storage admiten la eliminación temporal de contenedores.

### <a name="storage-account-support"></a>Compatibilidad con la cuenta de almacenamiento

La eliminación temporal de contenedores está disponible para los siguientes tipos de cuentas de almacenamiento:

- Cuentas de almacenamiento de uso general v2
- Cuentas de almacenamiento de blob en bloques
- Cuentas de Almacenamiento de blobs

Si la cuenta de almacenamiento es una cuenta de uso general v1, use Azure Portal para realizar la actualización a una cuenta de uso general v2. Para más información sobre las cuentas de almacenamiento, consulte  [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

También se admiten las cuentas de almacenamiento con un espacio de nombres jerárquico habilitado para usarse con Azure Data Lake Storage Gen2.

### <a name="register-for-the-preview"></a>Registro para obtener la versión preliminar

Para inscribirse en la versión preliminar de la eliminación temporal de contenedores, use PowerShell o la CLI de Azure a fin de enviar una solicitud para registrar la característica en su suscripción. Una vez aprobada la solicitud, puede habilitar la eliminación temporal de contenedores con las cuentas de almacenamiento nuevas o existentes de blobs en bloques prémium, Blob Storage o de uso general v2.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para registrarse con PowerShell, llame al comando [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

```powershell
# Register for container soft delete (preview)
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName ContainerSoftDelete

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para registrarse con la CLI de Azure, llame al comando [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage --name ContainerSoftDelete
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-the-status-of-your-registration"></a>Comprobación del estado del registro

Para comprobar el estado del registro, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el estado del registro con PowerShell, llame al comando [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName ContainerSoftDelete
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar el estado del registro con la CLI de Azure, llame al comando [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage --name ContainerSoftDelete
```

---

## <a name="pricing-and-billing"></a>Precios y facturación

No hay ningún cargo adicional para habilitar la eliminación temporal de contenedores. Los datos en contenedores eliminados temporalmente se facturan a la misma tasa que los datos activos.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la eliminación temporal de contenedores](soft-delete-container-enable.md)
- [Eliminación temporal para blobs](soft-delete-blob-overview.md)
- [Control de versiones de blobs](versioning-overview.md)
