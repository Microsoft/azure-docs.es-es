---
title: Configuración de claves administradas por el cliente para cifrar datos en reposo en ISE
description: Cree y administre sus propias claves de cifrado para proteger los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, rarayudu, logicappspm
ms.topic: conceptual
ms.date: 03/11/2020
ms.openlocfilehash: ad5b4245cc445ecf8fae22c39db3365d71730a56
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400150"
---
# <a name="set-up-customer-managed-keys-to-encrypt-data-at-rest-for-integration-service-environments-ises-in-azure-logic-apps"></a>Configure claves administradas por el cliente para cifrar los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps

Azure Logic Apps se basa en Azure Storage para almacenar y [cifrar automáticamente los datos en reposo](../storage/common/storage-service-encryption.md). Este cifrado protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización. De forma predeterminada, Azure Storage usa claves que administra Microsoft para cifrar sus datos. Para más información sobre cómo funciona el cifrado de Azure Storage, consulte [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md) y [Cifrado de datos de Azure en reposo](../security/fundamentals/encryption-atrest.md).

Al crear un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) para hospedar las aplicaciones lógicas y desea tener un mayor control sobre las claves de cifrado que usa Azure Storage, puede configurar, usar y administrar su propia clave mediante [Azure Key Vault](../key-vault/general/overview.md). Esta funcionalidad también se conoce como "Bring Your Own Key" (BYOK) y la clave se denomina "clave administrada por el cliente".

En este tema se muestra cómo configurar y especificar su propia clave de cifrado que se usará al crear el ISE mediante la API REST de Logic Apps. Para ver los pasos generales para crear un ISE mediante la API REST de Logic Apps, consulte [Creación de un entorno del servicio de integración (ISE) mediante la API REST de Logic Apps](../logic-apps/create-integration-service-environment-rest-api.md).

## <a name="considerations"></a>Consideraciones

* En este momento, la compatibilidad con claves administradas por el cliente para un ISE solo está disponible en estas regiones de Azure: Oeste de EE. UU. 2, Este de EE. UU. y Centro-sur de EE. UU.

* Solo puede especificar una clave administrada por el cliente *cuando cree su ISE*, no después. No se puede deshabilitar esta clave una vez creado el ISE. Actualmente, no existe compatibilidad para rotar una clave administrada por el cliente para un ISE.

* Para admitir las claves administradas por el cliente, el ISE requiere tener su [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) habilitada. Esta identidad permite que el ISE autentique el acceso a los recursos de otros inquilinos de Azure Active Directory (Azure AD), de modo que no tenga que iniciar sesión con sus credenciales.

* Actualmente, para crear un ISE que admita claves administradas por el cliente y tenga habilitada la identidad asignada por el sistema, debe llamar a la API de REST de Logic Apps mediante una solicitud HTTPS PUT.

* En un plazo de *30 minutos* después de enviar la solicitud HTTPS PUT que crea su ISE, debe [proporcionar acceso al almacén de claves a la identidad asignada por el sistema de ISE](#identity-access-to-key-vault). En caso contrario, se produce un error en la creación de ISE que tendrá como consecuencia un error de permisos.

## <a name="prerequisites"></a>Prerrequisitos

* Los mismos [requisitos previos](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#prerequisites) y [requisitos para habilitar el acceso a su ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access) que cuando se crea un ISE en Azure Portal

* Un almacén de claves de Azure con las propiedades **Soft Delete** y **Do Not Purge**

  Para obtener más información acerca de cómo habilitar estas propiedades, consulte [Información general sobre la eliminación temporal Azure Key Vault ](../key-vault/general/soft-delete-overview.md) y [Configurar las claves administradas por el cliente con Azure Key Vault](../storage/common/storage-encryption-keys-portal.md). Si no está familiarizado con Azure Key Vault, obtenga información acerca de [cómo crear un almacén de claves](../key-vault/secrets/quick-create-portal.md#create-a-vault) mediante el Azure Portal o mediante el comando de Azure PowerShell, [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault).

* En el almacén de claves, una clave se crea con estos valores de propiedad:

  | Propiedad | Value |
  |----------|-------|
  | **Tipo de clave** | RSA |
  | **Tamaño de la clave RSA** | 2048 |
  | **Enabled** | Sí |
  |||

  ![Creación de clave de cifrado administrada por el cliente](./media/customer-managed-keys-integration-service-environment/create-customer-managed-key-for-encryption.png)

  Para obtener más información, configure [Configurar las claves administradas por el cliente con Azure Key Vault](../storage/common/storage-encryption-keys-portal.md) o el comando de Azure PowerShell [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey).

* Una herramienta que puede usar para crear su ISE mediante una llamada a la API de REST de Logic Apps con una solicitud HTTPS PUT. Por ejemplo, puede usar [Postman](https://www.getpostman.com/downloads/) o puede compilar una aplicación lógica que realice esta tarea.

<a name="enable-support-key-system-identity"></a>

## <a name="create-ise-with-key-vault-and-managed-identity-support"></a>Creación de ISE con almacén de claves y compatibilidad con identidades administradas

Para crear su ISE mediante una llamada a la API de REST de Logic Apps, realice esta solicitud HTTPS PUT.

`PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01`

> [!IMPORTANT]
> La versión 2019-05-01 de la API de REST de Logic Apps requiere que realice su propia solicitud HTTP PUT para conectores ISE.

Normalmente, la implementación tarda un máximo de dos horas en completarse. En ocasiones, la implementación puede tardar hasta cuatro horas. Para comprobar el estado de implementación, en [Azure Portal](https://portal.azure.com), en la barra de herramientas de Azure, seleccione el icono de notificaciones, que abre el panel de notificaciones.

> [!NOTE]
> Si se produce un error en la implementación o se elimina el ISE, Azure podría tardar hasta una hora en liberar las subredes. Debido a este retraso, es posible que deba esperar antes de volver a usar esas subredes en otro ISE.
>
> Si elimina su red virtual, Azure generalmente tarda hasta dos horas antes de liberar las subredes, pero esta operación puede llevar más tiempo. 
> Cuando elimine redes virtuales, asegúrese de que no haya recursos conectados. 
> Consulte [Eliminar red virtual](../virtual-network/manage-virtual-network.md#delete-a-virtual-network).

### <a name="request-header"></a>Encabezado de solicitud

En el encabezado de solicitud, incluya estas propiedades:

* `Content-type`: Establezca el valor de propiedad en `application/json`.

* `Authorization`: Establezca este valor de propiedad en el token de portador para el cliente que tiene acceso al grupo de recursos o la suscripción de Azure que desea usar.

### <a name="request-body"></a>Cuerpo de la solicitud

En el cuerpo de la solicitud, habilite la compatibilidad con estos elementos adicionales proporcionando su información en la definición de ISE:

* La identidad administrada asignada por el sistema que el ISE usa para acceder al almacén de claves
* El almacén de claves y la clave administrada por el cliente que desea usar

#### <a name="request-body-syntax"></a>Sintaxis del cuerpo de la solicitud

Esta es la sintaxis del cuerpo de la solicitud, que describe las propiedades que se deben usar al crear el ISE:

```json
{
   "id": "/subscriptions/{Azure-subscription-ID/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "identity": {
      "type": "SystemAssigned"
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-1}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-2}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-3}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-4}",
            }
         ]
      },
      "encryptionConfiguration": {
         "encryptionKeyReference": {
            "keyVault": {
               "id": "subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.KeyVault/vaults/{key-vault-name}",
            },
            "keyName": "{customer-managed-key-name}",
            "keyVersion": "{key-version-number}"
         }
      }
   }
}
```

#### <a name="request-body-example"></a>Ejemplo de cuerpo de solicitud

En este cuerpo de solicitud de ejemplo se muestran los valores de ejemplo:

```json
{
   "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Logic/integrationServiceEnvironments/Fabrikam-ISE",
   "name": "Fabrikam-ISE",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "WestUS2",
   "identity": {
      "type": "SystemAssigned"
   },
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-1",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-2",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-3",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-4",
            }
         ]
      },
      "encryptionConfiguration": {
         "encryptionKeyReference": {
            "keyVault": {
               "id": "subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.KeyVault/vaults/FabrikamKeyVault",
            },
            "keyName": "Fabrikam-Encryption-Key",
            "keyVersion": "********************"
         }
      }
   }
}
```

<a name="identity-access-to-key-vault"></a>

## <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves

En un plazo de *30 minutos* después de enviar la solicitud HTTPS PUT que crea su ISE, debe proporcionar una directiva de acceso para el almacén de claves para la identidad asignada por el sistema de ISE. De lo contrario, se producirá un error en la creación del ISE y que tendrá como consecuencia un error de permisos. 

Para esta tarea, puede usar el comando de Azure PowerShell [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy), o bien puede seguir estos pasos en el Azure Portal:

1. Abra el almacén de claves de Azure en [Azure Portal](https://portal.azure.com).

1. En el menú del almacén de claves, seleccione **Directivas de acceso** > **Agregar directiva de acceso**, por ejemplo:

   ![Agregar directiva de acceso para la identidad administrada asignada por el sistema](./media/customer-managed-keys-integration-service-environment/add-ise-access-policy-key-vault.png)

1. Una vez que se abra el panel de **Agregar directiva de acceso**, siga estos pasos:

   1. Seleccione estas opciones:

      | Configuración | Valores |
      |---------|--------|
      | **Configurar a partir de una lista de plantilla (opcional)** : | Administración de claves |
      | **Permisos de claves** | - **Operaciones de administración de claves**: Get, List <p><p>- **Operaciones criptográficas**: Unwrap Key, Wrap Key |
      |||

      ![Seleccione "Administración de claves" > "Permisos de clave"](./media/customer-managed-keys-integration-service-environment/select-key-permissions.png)

   1. Para **Seleccionar entidad de seguridad**, seleccione **Ninguna seleccionada**. Una vez que se abra el panel **Entidad de seguridad**, en el cuadro de búsqueda, busque y seleccione el ISE. Cuando termine, haga clic en **Seleccionar** > **Agregar**.

      ![Seleccione su ISE para usarlo como entidad de seguridad](./media/customer-managed-keys-integration-service-environment/select-service-principal-ise.png)

   1. Cuando haya terminado con el panel **Directivas de acceso**, seleccione **Guardar**.

Para más información, consulte [Autenticación en Key Vault](/azure/key-vault/general/authentication) y [Asignación de una directiva de acceso de Key Vault](/azure/key-vault/general/assign-access-policy-portal).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Azure Key Vault](../key-vault/general/overview.md)
