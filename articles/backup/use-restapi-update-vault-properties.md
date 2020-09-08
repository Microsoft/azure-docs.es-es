---
title: Actualización de la configuración del almacén de Recovery Services con REST API
description: En este artículo, aprenderá a actualizar la configuración del almacén mediante la API REST.
ms.topic: conceptual
ms.date: 12/06/2019
ms.assetid: 9aafa5a0-1e57-4644-bf79-97124db27aa2
ms.openlocfilehash: 3ee2d57b5589daa756020ebb787a5400ed244506
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88890049"
---
# <a name="update-azure-recovery-services-vault-configurations-using-rest-api"></a>Actualización de la configuración del almacén de Azure Recovery Services mediante la API REST

En este artículo se describe cómo actualizar las configuraciones relacionadas con la copia de seguridad en el almacén de Azure Recovery Services mediante la API REST.

## <a name="soft-delete-state"></a>Estado de eliminación temporal

La eliminación de las copias de seguridad de un elemento protegido es una operación importante que debe supervisarse. Para protegerse contra eliminaciones accidentales, el almacén de Azure Recovery Services tiene una funcionalidad de eliminación temporal. Esta funcionalidad le permite restaurar las copias de seguridad eliminadas, de ser necesario, dentro de un período de tiempo posterior a la eliminación.

Sin embargo, hay escenarios en los que esta funcionalidad no es necesaria. No es posible eliminar un almacén de Azure Recovery Services si contiene elementos de copia de seguridad, incluso si tienen el estado de eliminación temporal. Esto puede suponer un problema si el almacén debe eliminarse inmediatamente. Por ejemplo: las operaciones de implementación a menudo limpian los recursos creados en el mismo flujo de trabajo. Una implementación puede crear un almacén, configurar copias de seguridad para un elemento, realizar una restauración de prueba y, a continuación, eliminar los elementos de copia de seguridad y el almacén. Si se produce un error en la eliminación del almacén, se puede producir un error en toda la implementación. Deshabilitar la eliminación temporal es la única manera de garantizar la eliminación inmediata.

Por lo tanto, debe elegir cuidadosamente si quiere deshabilitar la eliminación temporal para un almacén determinado en función del escenario. Para obtener más información, consulte el [artículo sobre la eliminación temporal](backup-azure-security-feature-cloud.md).

### <a name="fetch-soft-delete-state-using-rest-api"></a>Recuperación del estado de eliminación temporal mediante la API REST

De forma predeterminada, el estado de eliminación temporal se habilitará para los almacenes de Recovery Services recién creados. Para recuperar o actualizar el estado de la eliminación temporal de un almacén, use el [documento de API REST](/rest/api/backup/backupresourcevaultconfigs) relacionado con la configuración del almacén de copia de seguridad.

Para recuperar el estado actual de la eliminación temporal de un almacén, use la siguiente operación *GET*.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupconfig/vaultconfig?api-version=2019-05-13
```

El URI de GET tiene los parámetros `{subscriptionId}`, `{vaultName}` y `{vaultresourceGroupName}`. En este ejemplo, `{vaultName}` es "testVault" y `{vaultresourceGroupName}` es "testVaultRG". Como todos los parámetros necesarios se proporcionan en el URI, no hay necesidad de tener un cuerpo de solicitud independiente.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupconfig/vaultconfig?api-version=2019-05-13
```

#### <a name="responses"></a>Respuestas

A continuación se muestra la respuesta correcta para la operación "GET":

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|200 OK     |   [BackupResourceVaultConfig](/rest/api/backup/backupresourcevaultconfigs/get#backupresourcevaultconfigresource)      | Aceptar        |

##### <a name="example-response"></a>Respuesta de ejemplo

Una vez que se emite la solicitud GET, se devuelve una respuesta 200 (correcta).

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testvaultRG/providers/Microsoft.RecoveryServices/vaults/testvault/backupconfig/vaultconfig",
  "name": "vaultconfig",
  "type": "Microsoft.RecoveryServices/vaults/backupconfig",
  "properties": {
    "enhancedSecurityState": "Enabled",
    "softDeleteFeatureState": "Enabled"
  }
}
```

### <a name="update-soft-delete-state-using-rest-api"></a>Actualización del estado de eliminación temporal mediante la API REST

Para actualizar el estado de eliminación temporal del almacén de Recovery Services con la API REST, use la siguiente operación *PATCH*.

```http
PATCH https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupconfig/vaultconfig?api-version=2019-05-13
```

El URI de PATCH tiene los parámetros `{subscriptionId}`, `{vaultName}` y `{vaultresourceGroupName}`. En este ejemplo, `{vaultName}` es "testVault" y `{vaultresourceGroupName}` es "testVaultRG". Si reemplazamos el URI con los valores anteriores, el URI tendrá el siguiente aspecto.

```http
PATCH https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupconfig/vaultconfig?api-version=2019-05-13
```

#### <a name="create-the-request-body"></a>Creación del cuerpo de la solicitud

Para crear el cuerpo de la solicitud, se usan las siguientes definiciones comunes.

Para obtener más información, consulte la [documentación sobre la API REST](/rest/api/backup/backupresourcevaultconfigs/update#request-body).

|Nombre  |Obligatorio  |Tipo  |Descripción  |
|---------|---------|---------|---------|
|eTag     |         |   String      |  eTag opcional       |
|ubicación     |  true       |String         |   Ubicación de los recursos      |
|properties     |         | [VaultProperties](/rest/api/recoveryservices/vaults/createorupdate#vaultproperties)        |  Propiedades del almacén       |
|etiquetas     |         | Object        |     Etiquetas del recurso    |

#### <a name="example-request-body"></a>Cuerpo de solicitud de ejemplo

El ejemplo siguiente se usa para actualizar el estado de eliminación temporal al estado deshabilitado.

```json
{
  "properties": {
    "enhancedSecurityState": "Enabled",
    "softDeleteFeatureState": "Disabled"
  }
}
```

#### <a name="responses-for-the-patch-operation"></a>Respuestas para la operación PATCH

A continuación se muestra la respuesta correcta para la operación "PATCH":

|Nombre  |Tipo  |Descripción  |
|---------|---------|---------|
|200 OK     |   [BackupResourceVaultConfig](/rest/api/backup/backupresourcevaultconfigs/get#backupresourcevaultconfigresource)      | Aceptar        |

##### <a name="example-response-for-the-patch-operation"></a>Respuesta de ejemplo para la operación PATCH

Una vez que se emite la solicitud "PATCH", se devuelve una respuesta 200 (correcta).

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testvaultRG/providers/Microsoft.RecoveryServices/vaults/testvault/backupconfig/vaultconfig",
  "name": "vaultconfig",
  "type": "Microsoft.RecoveryServices/vaults/backupconfig",
  "properties": {
    "enhancedSecurityState": "Enabled",
    "softDeleteFeatureState": "Disabled"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

[Creación de una directiva de copia de seguridad para la copia de seguridad de una máquina virtual de Azure en este almacén](backup-azure-arm-userestapi-createorupdatepolicy.md).

Para más información sobre las API REST de Azure, consulte los siguientes documentos:

- [API REST del proveedor de Azure Recovery Services](/rest/api/recoveryservices/)
- [Get started with Azure REST API](/rest/api/azure/) (Introducción a la API REST de Azure)
