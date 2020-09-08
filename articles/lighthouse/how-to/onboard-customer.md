---
title: Incorporación de un cliente a Azure Lighthouse
description: Obtenga información sobre cómo incorporar un cliente a Azure Lighthouse, lo que permite administrar sus recursos y acceder a ellos desde su propio inquilino mediante la administración de recursos delegados de Azure.
ms.date: 08/20/2020
ms.topic: how-to
ms.openlocfilehash: db6a819c72f1ef46f542ed47cad6caae23c0d191
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719060"
---
# <a name="onboard-a-customer-to-azure-lighthouse"></a>Incorporación de un cliente a Azure Lighthouse

En este artículo se explica cómo, como proveedor de servicios, puede incorporar un cliente a Azure Lighthouse. Al hacerlo, los recursos delegados del cliente (suscripciones o grupos de recursos) están accesibles y se pueden administrar desde su propio inquilino de Azure Active Directory (Azure AD) mediante la [Administración de recursos delegados de Azure](../concepts/azure-delegated-resource-management.md).

> [!TIP]
> Aunque en este tema nos referiremos a los proveedores de servicios y clientes, las [empresas que administren varios inquilinos](../concepts/enterprise.md) pueden usar el mismo proceso para configurar Azure Lighthouse y consolidar su experiencia de administración.

Puede repetir el proceso de incorporación de varios clientes. Cuando un usuario con permisos apropiados inicia sesión en el inquilino administrador, se puede autorizar a ese usuario en los ámbitos del inquilino del cliente para realizar operaciones de administración sin tener que iniciar sesión en todos y cada uno de los inquilinos de cliente individuales.

Para realizar el seguimiento del impacto en las involucraciones de los clientes y recibir el reconocimiento correspondiente, asocie el identificador de Microsoft Partner Network (MPN) con al menos una cuenta de usuario con acceso a cada una de las suscripciones incorporadas. Tendrá que realizar esta asociación en el inquilino del proveedor de servicios. Para simplificar, se recomienda crear en el inquilino una cuenta de entidad de servicio asociada al identificador de MPN y concederle acceso de lectura a todos los clientes que incorpore. Para obtener más información, vea [Vinculación de un Id. de partner a cuentas de Azure](../../cost-management-billing/manage/link-partner-id.md).

> [!NOTE]
> Los clientes también se pueden incorporar a Azure Lighthouse cuando compran una oferta de servicios administrados (pública o privada) que [haya publicado en Azure Marketplace](publish-managed-services-offers.md). También puede usar el proceso de incorporación que se describe aquí con una oferta publicada en Azure Marketplace.

El proceso de incorporación requiere que se tomen medidas desde el inquilino del proveedor de servicios y del inquilino del cliente. Todos estos pasos se describen en este artículo.

## <a name="gather-tenant-and-subscription-details"></a>Recopilación de los detalles del inquilino y la suscripción

Para incorporar el inquilino de un cliente, este debe tener una suscripción activa a Azure. Necesitará saber:

- El identificador del inquilino del proveedor de servicios (donde va a administrar los recursos del cliente)
- El identificador del inquilino del cliente (cuyos recursos administrará el proveedor de servicios)
- Los identificadores de suscripción de cada suscripción específica del inquilino del cliente que administrará el proveedor de servicios (o que contenga los grupos de recursos que administrará el proveedor de servicios).

> [!NOTE]
> Incluso si solo quiere incorporar uno o varios grupos de recursos dentro de una suscripción, la implementación debe realizarse en el nivel de suscripción, por lo que necesitará el identificador de la suscripción.

Si aún no tiene estos valores de identificador, puede recuperarlos de una de las siguientes maneras. Asegúrese y use estos valores exactos en su implementación.

### <a name="azure-portal"></a>Azure portal

Para ver el identificador del inquilino, mantenga el mouse sobre el nombre de la cuenta en la parte superior derecha de Azure Portal o seleccione **Cambiar directorio**. Para seleccionar y copiar el identificador del inquilino, busque "Azure Active Directory" desde dentro del portal y, a continuación, seleccione **Propiedades** y copie el valor que se muestra en campo **Id. de directorio**. Para buscar el identificador de una suscripción en el inquilino del cliente, busque "Suscripciones" y, a continuación, seleccione el identificador de suscripción adecuado.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Select-AzSubscription <subscriptionId>
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account set --subscription <subscriptionId/name>
az account show
```

> [!NOTE]
> Al incorporar una suscripción (o uno o varios grupos de recursos dentro de una suscripción) mediante el proceso que se describe aquí, el proveedor de recursos **Microsoft.ManagedServices** se registrará para esa suscripción.

## <a name="define-roles-and-permissions"></a>Definir roles y permisos

Como proveedor de servicios, es posible que quiera realizar varias tareas para un solo cliente, lo que requiere un acceso diferente para distintos ámbitos. Puede definir tantas autorizaciones como necesite para asignar los [roles integrados de control de acceso basado en rol (RBAC)](../../role-based-access-control/built-in-roles.md) a los usuarios de su inquilino.

Para facilitar la administración, se recomienda usar grupos de usuarios de Azure AD para cada rol. Esto le ofrece la flexibilidad para agregar o quitar usuarios individuales del grupo que tiene acceso, de modo que no tenga que repetir el proceso de incorporación para realizar cambios en el usuario. Puede asignar roles a una entidad de servicio, lo que puede ser útil para escenarios de automatización.

Al definir las autorizaciones, asegúrese de seguir el principio de privilegios mínimos para que los usuarios solo tengan los permisos necesarios para completar su trabajo. Para obtener recomendaciones e información sobre los roles admitidos, consulte [Inquilinos, usuarios y roles en escenarios de Azure Lighthouse](../concepts/tenants-users-roles.md).

> [!IMPORTANT]
> Para agregar permisos a un grupo de Azure AD, el **tipo de grupo** debe ser **Seguridad** y no **Office 365**. Esta opción se selecciona cuando se crea el grupo. Para obtener más información vea [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Para definir las autorizaciones, tiene que conocer los valores de identificador de cada usuario, grupo de usuarios o entidad de servicio en el inquilino del proveedor de servicios al que quiera conceder acceso. También necesitará el identificador de definición de roles para cada rol integrado que quiera asignar. Si aún no los tiene, puede recuperarlos ejecutando los comandos a continuación en el inquilino del proveedor de servicios.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
(Get-AzADGroup -DisplayName '<yourGroupName>').id

# To retrieve the objectId for an Azure AD user
(Get-AzADUser -UserPrincipalName '<yourUPN>').id

# To retrieve the objectId for an SPN
(Get-AzADApplication -DisplayName '<appDisplayName>' | Get-AzADServicePrincipal).Id

# To retrieve role definition IDs
(Get-AzRoleDefinition -Name '<roleName>').id
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
az ad group list --query "[?displayName == '<yourGroupName>'].objectId" --output tsv

# To retrieve the objectId for an Azure AD user
az ad user show --id "<yourUPN>" --query "objectId" --output tsv

# To retrieve the objectId for an SPN
az ad sp list --query "[?displayName == '<spDisplayName>'].objectId" --output tsv

# To retrieve role definition IDs
az role definition list --name "<roleName>" | grep name
```

> [!TIP]
> Se recomienda asignar el [rol de eliminación de asignación del registro de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) al incorporar un cliente, de modo que los usuarios del inquilino puedan [quitar el acceso a la delegación](remove-delegation.md) más adelante si es necesario. Si este rol no está asignado, solo un usuario puede quitar los recursos delegados del inquilino del cliente.

## <a name="create-an-azure-resource-manager-template"></a>Creación de una plantilla de Azure Resource Manager

Para incorporar el cliente, deberá crear una plantilla de [Azure Resource Manager](../../azure-resource-manager/index.yml) para la oferta con la información siguiente. Los valores **mspOfferName** y **mspOfferDescription** serán visibles para el cliente en la [página de proveedores de servicios](view-manage-service-providers.md) de Azure Portal.

|Campo  |Definición  |
|---------|---------|
|**mspOfferName**     |Nombre que describe esta definición. Este valor se muestra al cliente como el título de la oferta y debe ser un valor único.        |
|**mspOfferDescription**     |Breve descripción de la oferta (por ejemplo, "oferta de administración de VM de Contoso").      |
|**managedByTenantId**     |El identificador de inquilino.          |
|**authorizations**     |Los valores de **principalId** para los usuarios/grupos/SPN del inquilino, cada uno de ellos con un valor de **principalIdDisplayName** para ayudar a su cliente a entender el propósito de la autorización y asignarla a un valor de **roleDefinitionId** integrado para especificar el nivel de acceso.      |

El proceso de incorporación requiere una plantilla de Azure Resource Manager (proporcionada en el [repositorio de ejemplos](https://github.com/Azure/Azure-Lighthouse-samples/)) y un archivo de parámetros correspondiente que se modifica para que coincida con la configuración y para definir las autorizaciones.

La plantilla que elija dependerá de si se incorpora una suscripción completa, un grupo de recursos o varios grupos de recursos dentro de una suscripción. También proporcionamos una plantilla que se puede usar para los clientes que han adquirido una oferta de servicios administrados que ha publicado en Azure Marketplace, si prefiere incorporar sus suscripciones de esta manera.

|Para incorporar esto  |Use esta plantilla de Azure Resource Manager  |Y modifique este archivo de parámetros |
|---------|---------|---------|
|Suscripción   |[delegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.json)  |[delegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.parameters.json)    |
|Resource group   |[rgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.json)  |[rgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.parameters.json)    |
|Varios grupos de recursos de una suscripción   |[multipleRgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.json)  |[multipleRgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.parameters.json)    |
|Suscripción (al usar una oferta publicada en Azure Marketplace)   |[marketplaceDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.json)  |[marketplaceDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.parameters.json)    |

> [!IMPORTANT]
> El proceso que se describe aquí requiere una implementación independiente para cada suscripción que se está incorporando, incluso si va a incorporar suscripciones en el mismo inquilino de cliente. También se necesitan implementaciones independientes si se incorporan varios grupos de recursos en distintas suscripciones del mismo inquilino del cliente. Sin embargo, la incorporación de varios grupos de recursos dentro de una sola suscripción puede realizarse en una implementación.
>
> También se necesitan implementaciones independientes para varias ofertas que se aplican a la misma suscripción (o grupos de recursos dentro de una suscripción). Cada oferta aplicada debe usar un valor de **mspOfferName** diferente.

En el ejemplo siguiente se muestra un archivo **delegatedResourceManagement.parameters.json**, que se usará para incorporar una suscripción. Los archivos de parámetros del grupo de recursos (situados en la carpeta [rg-delegated-resource-management](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/rg-delegated-resource-management)) son similares, pero también incluyen un parámetro **rgName** para identificar los grupos de recursos específicos que se incorporarán.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "mspOfferDescription": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "managedByTenantId": {
            "value": "df4602a3-920c-435f-98c4-49ff031b9ef6"
        },
        "authorizations": {
            "value": [
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "36243c78-bf99-498c-9df9-86d9f8d28608"
                },
                {
                    "principalId": "0afd8497-7bff-4873-a7ff-b19a6b7b332c",
                    "principalIdDisplayName": "Tier 2 Support",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
                },
                {
                    "principalId": "9fe47fff-5655-4779-b726-2cf02b07c7c7",
                    "principalIdDisplayName": "Service Automation Account",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "3kl47fff-5655-4779-b726-2cf02b05c7c4",
                    "principalIdDisplayName": "Policy Automation Account",
                    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
                    "delegatedRoleDefinitionIds": [
                        "b24988ac-6180-42a0-ab88-20f7382dd24c",
                        "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
                    ]
                }
            ]
        }
    }
}
```

La última autorización del ejemplo anterior agrega un valor de **principalId** con el rol de administrador de acceso de usuario (18d7d88d-d35e-4fb5-a5c3-7773c20a72d9). Al asignar este rol, debe incluir la propiedad **delegatedRoleDefinitionIds** y uno o más roles integrados. El usuario creado en esta autorización podrá asignar los roles integrados a las [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md) en el inquilino del cliente, algo necesario para [implementar directivas que pueden corregirse](deploy-policy-remediation.md).  El usuario también puede crear incidentes de soporte técnico.  No se aplicará a este usuario ningún otro permiso asociado normalmente al rol Administrador de acceso de usuario.

## <a name="deploy-the-azure-resource-manager-templates"></a>Implementación de las plantillas de Azure Resource Manager

Una vez actualizado el archivo de parámetros, un usuario del inquilino del cliente debe implementar la plantilla de Azure Resource Manager en su inquilino. Se necesita una implementación independiente para cada suscripción que quiera incorporar (o para cada suscripción que contenga grupos de recursos que quiera incorporar).

> [!IMPORTANT]
> Esta implementación debe realizarse desde una cuenta que no sea de invitado en el inquilino del cliente que tenga el [rol Propietario integrado](../../role-based-access-control/built-in-roles.md#owner) para la suscripción que se va a incorporar (o que contiene los grupos de recursos que se están incorporando). Para ver todos los usuarios que puedan delegar la suscripción, cualquiera de los usuarios del inquilino del cliente puede seleccionar la suscripción en Azure Portal, abrir **Control de acceso (IAM)** y [ver todos los usuarios con el rol Propietario](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription). 
>
> Si la suscripción se creó con el [programa Proveedor de soluciones en la nube (CSP)](../concepts/cloud-solution-provider.md), cualquier usuario que tenga el rol de [agente de administración](/partner-center/permissions-overview#manage-commercial-transactions-in-partner-center-azure-ad-and-csp-roles) en el inquilino del proveedor de servicios puede realizar la implementación.

La implementación puede realizarse en Azure Portal, mediante PowerShell o la CLI de Azure, como se muestra a continuación.

### <a name="azure-portal"></a>Azure portal

1. En nuestro [repositorio de GitHub](https://github.com/Azure/Azure-Lighthouse-samples/), seleccione el botón **Deploy to Azure** (Implementar en Azure) que se muestra junto a la plantilla que quiera usar. La plantilla se abrirá en Azure Portal.
1. Escriba los valores de **Msp Offer Name** (nombre de oferta de MSP), **Msp Offer Description** (descripción de la oferta de MSP), **Managed by Tenant Id** (administrado por id. de inquilino) y **Authorizations** (autorizaciones). Si lo prefiere, puede seleccionar **Editar parámetros** para especificar valores de `mspOfferName`, `mspOfferDescription`, `managedbyTenantId` y `authorizations` directamente en el archivo de parámetros. Asegúrese de actualizar estos valores, en lugar de usar los valores predeterminados de la plantilla.
1. Seleccione **Revisar y crear** y, luego, **Crear**.

Transcurridos unos minutos, se mostrará una notificación indicando que la implementación se ha completado.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateFile <pathToTemplateFile> `
                 -TemplateParameterFile <pathToParameterFile> `
                 -Verbose

# Deploy Azure Resource Manager template that is located externally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateUri <templateUri> `
                 -TemplateParameterUri <parameterUri> `
                 -Verbose
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
az deployment create --name <deploymentName> \
                     --location <AzureRegion> \
                     --template-file <pathToTemplateFile> \
                     --parameters <parameters/parameterFile> \
                     --verbose

# Deploy external Azure Resource Manager template, with local parameter file
az deployment create --name <deploymentName> \
                     --location <AzureRegion> \
                     --template-uri <templateUri> \
                     --parameters <parameterFile> \
                     --verbose
```

## <a name="confirm-successful-onboarding"></a>Confirmar la incorporación correcta

Cuando una suscripción de cliente se incorpora correctamente a Azure Lighthouse, los usuarios del inquilino del proveedor de servicios pueden ver la suscripción y sus recursos (si se les ha concedido acceso a ellos mediante el proceso anterior, ya sea individualmente o como miembro de un grupo de Azure AD con los permisos adecuados). Para confirmarlo, compruebe que la suscripción aparece de una de las siguientes maneras.  

### <a name="azure-portal"></a>Azure portal

En el inquilino del proveedor de servicios:

1. Vaya a la página [Mis clientes](view-manage-customers.md).
2. Seleccione **Clientes**.
3. Confirme que puede ver las suscripciones con el nombre de la oferta que proporcionó en la plantilla de Resource Manager.

> [!IMPORTANT]
> Para ver la suscripción delegada en [Mis clientes](view-manage-customers.md). Se debe haber concedido a los usuarios del inquilino del proveedor de servicios el rol [Lector](../../role-based-access-control/built-in-roles.md#reader) (u otro rol integrado que incluya acceso de lectura) al incorporar la suscripción.

En el inquilino del cliente:

1. Vaya a la [página Proveedores de servicios](view-manage-service-providers.md).
2. Seleccione **Ofertas del proveedor de servicios**.
3. Confirme que puede ver las suscripciones con el nombre de la oferta que proporcionó en la plantilla de Resource Manager.

> [!NOTE]
> Una vez completada la implementación, las actualizaciones pueden tardar unos minutos en verse reflejadas en Azure Portal.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Get-AzContext
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account list
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md).
- Puede [ver y administrar clientes](view-manage-customers.md) desde **Mis clientes**, en Azure Portal.
- Obtenga información sobre cómo [quitar el acceso a una delegación](remove-delegation.md) que se incorporó previamente.
