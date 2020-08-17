---
title: Control de versiones de blobs (versión preliminar)
titleSuffix: Azure Storage
description: El control de versiones de Blob Storage (versión preliminar) conserva automáticamente las versiones anteriores de un objeto y las identifica con marcas de tiempo. Puede restaurar las versiones anteriores de un blob para recuperar los datos si se modifican o eliminan erróneamente.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2085f0e8a148e27914b517f25e48894009592dd2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498606"
---
# <a name="blob-versioning-preview"></a>Control de versiones de blobs (versión preliminar)

Puede habilitar el control de versiones de Blob Storage (versión preliminar) para conservar automáticamente las versiones anteriores de un objeto.  Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error.

El control de versiones de blobs está habilitado en la cuenta de almacenamiento y se aplica a todos los blobs de dicha cuenta. Después de habilitar el control de versiones de blobs para una cuenta de almacenamiento, Azure Storage conserva automáticamente las versiones de todos los blobs de la cuenta de almacenamiento.

Microsoft recomienda usar el control de versiones de blobs para conservar las versiones anteriores de un blob a fin de mejorar la protección de datos. Cuando sea posible, use el control de versiones de blobs en lugar de las instantáneas de blobs para conservar las versiones anteriores. Las instantáneas de blobs proporcionan una funcionalidad similar, ya que también conservan las versiones anteriores de los blobs, sin embargo, en este caso, la aplicación debe mantener las instantáneas manualmente.

> [!IMPORTANT]
> El control de versiones de blobs no puede ayudarle a recuperar una eliminación accidental de una cuenta de almacenamiento o un contenedor. Para evitar dicha eliminación, configure el bloqueo **CannotDelete** en el recurso de la cuenta de almacenamiento. Para obtener más información sobre el bloqueo de recursos de Azure, vea [Bloqueo de recursos para impedir cambios inesperados](../../azure-resource-manager/management/lock-resources.md).

## <a name="how-blob-versioning-works"></a>Funcionamiento del control de versiones de blobs

Una versión captura el estado de un blob en un momento dado. Cuando el control de versiones de blobs está habilitado para una cuenta de almacenamiento, Azure Storage crea automáticamente una nueva versión de un blob cada vez que este se modifica o se elimina.

Cuando crea un blob con el control de versiones habilitado, este pasa a ser la versión actual del blob (o blob base). Si posteriormente lo modifica, Azure Storage crea una versión que captura su estado antes de la modificación. El blob modificado se convierte en la nueva versión actual. Cada vez que modifique el blob, se creará una nueva versión.

Cuando elimine un blob con el control de versiones habilitado, Azure Storage creará una versión que capturará su estado antes de la eliminación. De este modo, se elimina la versión actual del blob, pero se conservan sus versiones, de modo que se puede volver a crear si es necesario. 

Las versiones del blob son inmutables. No puede modificar el contenido ni los metadatos de la versión de un blob existente.

### <a name="version-id"></a>Id. de la versión

Cada versión de blob se identifica mediante un identificador de versión. El valor del identificador de la versión es la marca de tiempo en la que se escribió o actualizó el blob. El identificador de la versión se asigna en el momento en que se crea la versión.

Para realizar operaciones de lectura o eliminación en una versión específica de un blob, puede proporcionar su identificador de versión. Si omite el identificador de la versión, la operación actúa con respecto a la versión actual (el blob base).

Cuando llama a una operación de escritura para crear o modificar un blob, Azure Storage devuelve el encabezado *x-ms-version-id* en la respuesta. Este encabezado contiene el identificador de la versión actual del blob creado mediante la operación de escritura.

El identificador de la versión es el mismo para toda la vigencia de esta.

### <a name="versioning-on-write-operations"></a>Control de versiones en operaciones de escritura

Cuando se activa el control de versiones de blobs, con cada operación de escritura en un blob se crea una nueva versión. Las operaciones de escritura incluyen [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list), [Copy Blob](/rest/api/storageservices/copy-blob) y [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata).

Si la operación de escritura crea un nuevo blob, el resultante es su versión actual. Si la operación de escritura modifica un blob existente, los nuevos datos se capturan en el blob actualizado, que es la versión actual, y Azure Storage crea una versión que guarda el estado anterior del blob.

Por motivos de simplicidad, en los diagramas que se incluyen en este artículo el identificador de la versión se muestra como un valor entero simple. En realidad, el identificador de la versión es una marca de tiempo. La versión actual se muestra en azul y las anteriores, en gris.

En el diagrama siguiente se muestra cómo las operaciones de escritura afectan a las versiones de los blobs. Cuando se crea un blob, ese pasa a ser la versión actual. Cuando se modifica el mismo blob, se crea una nueva versión para guardar el estado anterior del blob y el blob actualizado se convierte en la versión actual.

:::image type="content" source="media/versioning-overview/write-operations-blob-versions.png" alt-text="Diagrama que muestra cómo las operaciones de escritura afectan a los blobs con versiones":::

> [!NOTE]
> Un blob creado antes de habilitar el control de versiones para la cuenta de almacenamiento no tiene ningún identificador de versión. Cuando se modifica, se convierte en la versión actual y se crea una versión para guardar el estado del blob antes de la actualización. Se asigna un identificador a la versión, que corresponde a la hora de creación.

### <a name="versioning-on-delete-operations"></a>Control de versiones en operaciones de eliminación

Cuando elimina un blob, la versión actual de este se convierte en una versión anterior y se elimina el blob base. Todas sus versiones anteriores existentes se conservan tras su eliminación.

Cuando se llama a la operación [Delete Blob](/rest/api/storageservices/delete-blob) sin ningún identificador de versión, se elimina el blob base. Para eliminar una versión específica, proporcione el identificador de esa versión en la operación de eliminación.

En el diagrama siguiente se muestra el efecto de una operación de eliminación en un blob con versiones:

:::image type="content" source="media/versioning-overview/delete-versioned-base-blob.png" alt-text="Diagrama que muestra la eliminación de un blob con versiones":::

Al escribir nuevos datos en el blob, se crea una nueva versión de este. Las versiones existentes no se verán afectadas, tal como se muestra en el diagrama siguiente.

:::image type="content" source="media/versioning-overview/recreate-deleted-base-blob.png" alt-text="Diagrama que muestra la nueva creación del blob con versiones después de la eliminación":::

### <a name="blob-types"></a>Tipos de blobs

Cuando se habilita el control de versiones de blobs para una cuenta de almacenamiento, todas las operaciones de escritura y eliminación en blobs en bloques desencadenan la creación de una nueva versión, con la excepción de la operación [Put Block](/rest/api/storageservices/put-block).

En el caso de los blobs en páginas y en anexos, solo un subconjunto de operaciones de escritura y eliminación desencadena la creación de una versión. Entre las operaciones se incluyen:

- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Block List](/rest/api/storageservices/put-block-list)
- [Delete Blob](/rest/api/storageservices/delete-blob)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Copy Blob](/rest/api/storageservices/copy-blob)

Las siguientes operaciones no la desencadenan. Para capturar los cambios de esas operaciones, realice una instantánea manual:

- [Put Page](/rest/api/storageservices/put-page) (blob en páginas)
- [Append Block](/rest/api/storageservices/append-block) (blob en anexos)

Todas las versiones de un blob deben ser del mismo tipo de blob. Si un blob tiene versiones anteriores, no puede sobrescribirlo con uno de otro tipo, a menos que primero se elimine este y todas sus versiones.

### <a name="access-tiers"></a>Niveles de acceso

Puede trasladar cualquier versión de un blob en bloques, incluida la actual, a otro nivel de acceso de blobs mediante una llamada a la operación [Set Blob Tier](/rest/api/storageservices/set-blob-tier). Para aprovechar los precios de menor capacidad, traslade las versiones anteriores de un blob al nivel de acceso esporádico o de archivo. Para obtener más información, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

Para automatizar el proceso de trasladar blobs en bloques al nivel adecuado, use la administración del ciclo de vida del blob. Para obtener más información sobre la administración del ciclo de vida, consulte [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md).

## <a name="enable-or-disable-blob-versioning"></a>Habilitación o deshabilitación del control de versiones de blobs

Para obtener información sobre cómo habilitar o deshabilitar el control de versiones de blobs, consulte [Habilitación o deshabilitación del control de versiones de blobs](versioning-enable.md).

Al deshabilitar el control de versiones de blobs, no se eliminan los blobs, las versiones ni las instantáneas existentes. Cuando desactiva el control de versiones de blobs, todas las versiones existentes permanecen accesibles en la cuenta de almacenamiento. No se crean nuevas versiones posteriormente.

Si un blob se creó o modificó después de que se deshabilitara el control de versiones en la cuenta de almacenamiento, se creará una nueva versión cuando se sobrescriba. El blob actualizado ya no es la versión actual ni tiene ningún identificador de versión. Todas las actualizaciones posteriores del blob sobrescribirán sus datos sin guardar el estado anterior.

Puede leer o eliminar versiones con el identificador de la versión después de deshabilitar el control de versiones. Así mismo, después de deshabilitarlo, también puede enumerar las versiones de un blob.

En el diagrama siguiente se muestra cómo se crea un blob sin versiones cuando se modifica un blob después de deshabilitar el control de versiones. Las versiones existentes asociadas con el blob se conservan.

:::image type="content" source="media/versioning-overview/modify-base-blob-versioning-disabled.png" alt-text="Diagrama que muestra el blob base modificado después de deshabilitar el control de versiones":::

## <a name="blob-versioning-and-soft-delete"></a>Control de versiones de blobs y eliminación temporal

El control de versiones y la eliminación temporal de blobs funcionan de forma conjunta para proporcionarle una protección de datos óptima. Al habilitar la eliminación temporal, debe especificar el tiempo durante el que Azure Storage debe conservar un blob eliminado temporalmente. Cualquier versión de un blob eliminada temporalmente permanece en el sistema y se puede recuperar dentro del período de retención de eliminación temporal. Para obtener más información sobre la eliminación temporal, consulte [Eliminación temporal de blobs de Azure Storage](storage-blob-soft-delete.md).

### <a name="deleting-a-blob-or-version"></a>Eliminación de un blob o una versión

La eliminación temporal ofrece protección adicional para eliminar versiones de blobs. Si se habilita tanto el control de versiones como la eliminación temporal en la cuenta de almacenamiento, cuando se elimina un blob, Azure Storage crea una versión nueva para guardar el estado del blob inmediatamente antes de la eliminación y elimina la versión actual. La nueva versión no se elimina de forma temporal ni se quita cuando expira el período de retención de eliminación temporal.

Cuando se elimina una versión anterior del blob, esta se elimina de forma temporal. La versión eliminada temporalmente se conserva durante el período de retención especificado en la configuración de eliminación temporal de la cuenta de almacenamiento y se elimina de forma permanente cuando expira dicho período de retención.

Para quitar una versión anterior de un blob, especifique el identificador de la versión para eliminarlo explícitamente.

En el diagrama siguiente se muestra lo que ocurre cuando elimina un blob o una versión de este.

:::image type="content" source="media/versioning-overview/soft-delete-historical-version.png" alt-text="Diagrama que muestra la eliminación de una versión con la eliminación temporal habilitada":::

Si el control de versiones y la eliminación temporal están habilitados en una cuenta de almacenamiento, no se crea ninguna instantánea de eliminación temporal cuando se modifica o elimina una versión de un blob o un blob.

### <a name="restoring-a-soft-deleted-version"></a>Restauración de una versión eliminada temporalmente

Puede restaurar una versión de blob eliminada temporalmente mediante una llamada a la operación [Undelete blob](/rest/api/storageservices/undelete-blob) en la versión durante el período de retención de eliminación temporal. La operación **Undelete Blob** restaura todas las versiones eliminadas temporalmente del blob.

Al realizar dicha restauración con la operación **Undelete Blob**, no se promueve ninguna versión para que sea la actual. Para restaurar la versión actual, primero restaure todas las versiones eliminadas temporalmente y, a continuación, use la operación [Copy Blob](/rest/api/storageservices/copy-blob) para copiar una versión anterior a fin de restaurar el blob.

En el diagrama siguiente se muestra cómo restaurar las versiones de blobs eliminadas temporalmente con la operación **Undelete blob** y cómo restaurar la versión actual del blob con la operación **Copy blob**.

:::image type="content" source="media/versioning-overview/undelete-version.png" alt-text="Diagrama que muestra cómo restaurar versiones eliminadas temporalmente":::

Una vez transcurrido el período de retención de eliminación temporal, cualquier versión del blob eliminada temporalmente se eliminará de forma permanente.

## <a name="blob-versioning-and-blob-snapshots"></a>Control de versiones e instantáneas de blobs

Una instantánea de blob es una copia de solo lectura de un blob que se ha realizado en un momento dado específico. Las instantáneas y las versiones de blobs son similares, pero, las instantáneas las crea el usuario o la aplicación de forma manual, mientras que una versión de blob se crea automáticamente en una operación de escritura o eliminación cuando el control de versiones de blobs está habilitado para la cuenta de almacenamiento.

> [!IMPORTANT]
> Microsoft recomienda que, después de habilitar el control de versiones de blobs, también actualice la aplicación para que deje de tomar instantáneas de blobs en bloques. Si está habilitado el control de versiones para la cuenta de almacenamiento, las versiones capturan y conservan todas las actualizaciones y eliminaciones de los blobs en bloques. La toma de instantáneas no ofrece ninguna protección adicional a los datos de los blobs en bloques si el control de versiones de blobs está habilitado, y puede aumentar los costos y la complejidad de la aplicación.

### <a name="snapshot-a-blob-when-versioning-is-enabled"></a>Instantánea de un blob cuando el control de versiones está habilitado

Aunque no se recomienda, puede tomar una instantánea de un blob que también tenga versiones. Si no puede actualizar la aplicación para que deje de tomar instantáneas de blobs al habilitar el control de versiones, es posible que la aplicación admita tanto instantáneas como versiones.

Cuando toma una instantánea de un blob con versiones, se crea una nueva versión al mismo tiempo que se crea la instantánea. También se crea una nueva versión actual cuando se toma una instantánea.

En el diagrama siguiente se muestra lo que ocurre cuando toma una instantánea de un blob con versiones. En el diagrama, las versiones del blob y las instantáneas con el identificador de versión 2 y 3 contienen datos idénticos.

:::image type="content" source="media/versioning-overview/snapshot-versioned-blob.png" alt-text="Diagrama que muestra instantáneas de un blob con versiones ":::

## <a name="authorize-operations-on-blob-versions"></a>Autorización de operaciones en versiones de blobs

Puede autorizar el acceso a las versiones de blobs mediante uno de los enfoques siguientes:

- Control de acceso basado en roles (RBAC) para conceder permisos a una entidad de seguridad de Azure Active Directory (Azure AD). Microsoft recomienda usar Azure AD para mayor seguridad y facilidad de uso. Para obtener más información sobre el uso de Azure AD con las operaciones de blobs, consulte [Autorización del acceso a los blobs y las colas con Azure Active Directory](../common/storage-auth-aad.md).
- Firma de acceso compartido (SAS), para delegar el acceso a las versiones de blobs. Especifique el identificador de versión del tipo de recurso firmado `bv`, que representa una versión de blob, a fin de crear un token de SAS para las operaciones en una versión específica. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).
- Claves de acceso a la cuenta, para autorizar las operaciones mediante versiones de blobs con la clave compartida. Para más información, consulte el artículo sobre la [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key).

El control de versiones de blobs está diseñado para proteger los datos de la eliminación accidental o malintencionada. A fin de mejorar la protección, la eliminación de una versión de blob requiere permisos especiales. En las secciones siguientes se describen los permisos necesarios para eliminar una versión de un blob.

### <a name="rbac-action-to-delete-a-blob-version"></a>Acción de RBAC para eliminar una versión de un blob

En la tabla siguiente se muestran las acciones de RBAC que admiten la eliminación de un blob o una versión de blob.

| Descripción | Operación de servicio de blob | Acción de datos de RBAC necesaria | Informe de rol integrado de RBAC |
|----------------------------------------------|------------------------|---------------------------------------------------------------------------------------|-------------------------------|
| Eliminación de la versión actual del blob | Delete Blob | **Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete** | Colaborador de datos de blobs de almacenamiento |
| Eliminación de una versión | Delete Blob | **Microsoft.Storage/storageAccounts/blobServices/containers/blobs/deleteBlobVersion/action** | Propietario de datos de blobs de almacenamiento |

### <a name="shared-access-signature-sas-parameters"></a>Parámetros de la Firma de acceso compartido (SAS)

El recurso firmado de una versión de blob es `bv`. Para obtener más información, vea [Creación de un SAS de servicio](/rest/api/storageservices/create-service-sas) o [Creación de un SAS de delegación de usuario](/rest/api/storageservices/create-user-delegation-sas).

En la tabla siguiente se muestra el permiso necesario en un SAS para eliminar una versión de un blob.

| **Permiso** | **Símbolo de URI** | **Operaciones permitidas** |
|----------------|----------------|------------------------|
| Eliminar         | x              | Eliminar una versión de un blob |

## <a name="about-the-preview"></a>Acerca de la versión preliminar

El control de versiones de blobs está disponible en versión preliminar en las siguientes regiones:

- Este de EE. UU. 2
- Centro de EE. UU.
- Norte de Europa
- Oeste de Europa
- Centro de Francia
- Este de Canadá
- Centro de Canadá

La versión preliminar está pensada para usos distintos del de producción.

La versión 2019-10-10 y las posteriores de la API REST de Azure Storage admiten el control de versiones de blobs.

### <a name="storage-account-support"></a>Compatibilidad con la cuenta de almacenamiento

El control de versiones de blobs está disponible para los siguientes tipos de cuentas de almacenamiento:

- Cuentas de almacenamiento de uso general v2
- Cuentas de almacenamiento de blob en bloques
- Cuentas de Almacenamiento de blobs

Si la cuenta de almacenamiento es una cuenta de uso general v1, use Azure Portal para realizar la actualización a una cuenta de uso general v2. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

Actualmente, no se admiten las cuentas de almacenamiento con un espacio de nombres jerárquico habilitado para usarse con Azure Data Lake Storage Gen2.

### <a name="register-for-the-preview"></a>Registro para obtener la versión preliminar

Para inscribirse para obtener la versión preliminar del control de versiones de blobs, use PowerShell o la CLI de Azure a fin de enviar una solicitud para registrar la característica con su suscripción. Una vez aprobada la solicitud, puede habilitar el control de versiones de blobs con las cuentas de almacenamiento nuevas o existentes de blobs en bloque prémium, Blob Storage o de uso general v2.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para registrarse con PowerShell, llame al comando [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
# Register for blob versioning (preview)
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Versioning

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para registrarse con la CLI de Azure, llame al comando [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage \
    --name Versioning
```

---

### <a name="check-the-status-of-your-registration"></a>Comprobación del estado del registro

Para comprobar el estado del registro, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el estado del registro con PowerShell, llame al comando [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Versioning
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar el estado del registro con la CLI de Azure, llame al comando [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage \
    --name Versioning
```

---

## <a name="pricing-and-billing"></a>Precios y facturación

La habilitación del control de versiones de blobs puede dar lugar a cargos de almacenamiento de datos adicionales en su cuenta. Al diseñar una aplicación, es importante tener en cuenta cómo se pueden acumular estos gastos para que pueda de minimizar los costos.

Las versiones de blobs, como las instantáneas de blobs, se facturan con la misma tarifa que los datos activos. Si una versión comparte bloques o páginas con su blob base, solo pagará por los bloques o páginas adicionales que no se compartan entre la versión y el blob base.

> [!NOTE]
> Habilitar el control de versiones de los datos que se sobrescriben con frecuencia puede provocar un aumento de los cargos por capacidad de almacenamiento y una mayor latencia durante las operaciones de enumeración. A fin de mitigar estos problemas, almacene los datos que se sobrescriben con frecuencia en una cuenta de almacenamiento independiente con el control de versiones deshabilitado.

### <a name="important-billing-considerations"></a>Consideraciones importantes sobre facturación

Asegúrese de tener en cuenta los siguientes puntos cuando habilite el control de versiones de blobs:

- La cuenta de almacenamiento genera cargos para páginas o bloques únicos, tanto si se encuentran en el blob cono en una versión anterior del blob. La cuenta no generará gastos adicionales para versiones asociadas a un blob hasta que actualice el blob en el que se basan. Después de actualizar el blob, este difiere de sus versiones anteriores. Cuando esto sucede, se le cobra por las páginas o bloques únicos en cada blob o versión.
- Al reemplazar un bloque en un blob en bloques, ese bloque será considerado como un bloque único a la hora de aplicar cargos. Esto es así aunque el bloque tenga el mismo identificador de bloque y los mismos datos que tiene en la versión. Cuando se vuelva a confirmar el bloque, diferirá de su equivalente en cualquier versión y se le aplicarán cargos por sus datos. Lo mismo sucede con una página de un blob en páginas que se haya actualizado con datos idénticos.
- Blob Storage no dispone de medios para determinar si dos bloques contienen datos idénticos. Cada bloque que se carga y confirma se trata como único, incluso si tiene los mismos datos y el mismo identificador de bloque. Dado que las cargas se incrementan con los bloques únicos, es importante considerar que cuando se actualiza un blob con el control de versiones habilitado se generarán más bloques únicos y se aplicarán cargos adicionales.
- Cuando el control de versiones de blobs esté habilitado, diseñe las operaciones de actualización en blobs en bloques para que actualicen el menor número posible de bloques. Las operaciones de escritura que permiten un control más exhaustivo de los bloques son [Put Block](/rest/api/storageservices/put-block) y [Put Block List](/rest/api/storageservices/put-block-list). Por otro lado, la operación [Put Blob](/rest/api/storageservices/put-blob) reemplaza todo el contenido de un blob, lo que puede dar lugar a cargos adicionales.

### <a name="versioning-billing-scenarios"></a>Escenarios de facturación del control de versiones

En las siguientes situaciones, se muestra cómo se ven incrementados los cargos de un blob en bloques y sus versiones.

#### <a name="scenario-1"></a>Escenario 1

En el escenario 1, el blob tiene una versión anterior. Este no se ha actualizado desde que se creó la versión, así que se generarán cargos únicamente para los bloques únicos 1, 2 y 3.

![Recursos de Azure Storage](./media/versioning-overview/versions-billing-scenario-1.png)

#### <a name="scenario-2"></a>Escenario 2

En el escenario 2, se ha actualizado un bloque (bloque 3 en el diagrama) en el blob. Aunque dicho bloque actualizado contiene los mismos datos y el mismo identificador, no es igual que el bloque 3 de la versión anterior. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Recursos de Azure Storage](./media/versioning-overview/versions-billing-scenario-2.png)

#### <a name="scenario-3"></a>Escenario 3

En el escenario 3, el blob se ha actualizado, pero la versión no. El bloque 3 se reemplazó por el bloque 4 en el blob base, pero la versión anterior sigue reflejando el bloque 3. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Recursos de Azure Storage](./media/versioning-overview/versions-billing-scenario-3.png)

#### <a name="scenario-4"></a>Escenario 4

En la situación 4, el blob de base se ha actualizado totalmente y no contiene ninguno de los bloques originales. Como resultado, los cargos de la cuenta se aplicarán por los ocho bloques únicos, cuatro de los cuales se encuentran en el blob base y, los demás, en la versión anterior. Este escenario puede producirse si escribe en un blob con la operación Put Blob, ya que reemplaza todo el contenido del blob base.

![Recursos de Azure Storage](./media/versioning-overview/versions-billing-scenario-4.png)

## <a name="see-also"></a>Consulte también

- [Habilitación del control de versiones de blobs](versioning-enable.md)
- [Creación de una instantánea de un blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob)
- [Eliminación temporal de blobs de Azure Storage](storage-blob-soft-delete.md)
