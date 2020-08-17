---
title: Creación de áreas de trabajo con la CLI de Azure
titleSuffix: Azure Machine Learning
description: Aprenda a usar la CLI de Azure para crear una nueva área de trabajo de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: larryfr
author: Blackmist
ms.date: 06/25/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.openlocfilehash: 4910dc03cc4ef24b8515271a9197650c4b041f01
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489612"
---
# <a name="create-a-workspace-for-azure-machine-learning-with-azure-cli"></a>Creación de un área de trabajo para Azure Machine Learning con la CLI de Azure
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a crear un área de trabajo de Azure Machine Learning mediante la CLI de Azure. La CLI de Azure proporciona comandos para administrar recursos de Azure. La extensión de aprendizaje automático de la CLI proporciona comandos para trabajar con recursos de Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

* Una **suscripción de Azure**. Si no tiene una ya, pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Para usar los comandos de la CLI de este documento desde su **entorno local**, necesita la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

    Si usa el [Azure Cloud Shell](https://azure.microsoft.com//features/cloud-shell/), la CLI es accesible a través del explorador y reside en la nube.

## <a name="connect-the-cli-to-your-azure-subscription"></a>Conexión de la CLI a una suscripción de Azure

> [!IMPORTANT]
> Puede omitir esta sección si usa Azure Cloud Shell. Cloud Shell se autentica automáticamente mediante la cuenta con la que inicia sesión en su suscripción a Azure.

Hay varias maneras de autenticarse en la suscripción a Azure desde la CLI. La manera más básica consiste en autenticarse interactivamente a través de un explorador. Para autenticarse de forma interactiva, abra una línea de comandos o un terminal y use el siguiente comando:

```azurecli-interactive
az login
```

Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión. De lo contrario, tendrá que abrir un explorador y seguir las instrucciones de la línea de comandos. Las instrucciones implican navegar a [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escribir un código de autorización.

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

Para obtener otros métodos de autenticación, consulte [Inicio de sesión con la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

## <a name="install-the-machine-learning-extension"></a>Instalación de la extensión de Machine Learning

Para instalar la extensión de Machine Learning, use el siguiente comando:

```azurecli-interactive
az extension add -n azure-cli-ml
```

## <a name="create-a-workspace"></a>Crear un área de trabajo

El área de trabajo de Azure Machine Learning se basa en los siguientes servicios o entidades de Azure:

> [!IMPORTANT]
> Si no especifica un servicio de Azure existente, se creará uno automáticamente durante la creación del área de trabajo. Siempre debe especificar un grupo de recursos. Al adjuntar su propia cuenta de almacenamiento, asegúrese de que cumple los siguientes criterios:
>
> * La cuenta de almacenamiento _no_ es una cuenta prémium (Premium_LRS y Premium_GRS)
> * Las funcionalidades Azure Blob y Azure File están habilitadas
> * El espacio de nombres jerárquico (ADLS Gen 2) está deshabilitado
>
> Estos requisitos son solo para la cuenta de almacenamiento _predeterminada_ utilizada por el área de trabajo.

| Servicio | Parámetro para especificar una instancia existente |
| ---- | ---- |
| **Grupo de recursos de Azure** | `-g <resource-group-name>`
| **Cuenta de Azure Storage** | `--storage-account <service-id>` |
| **Azure Application Insights** | `--application-insights <service-id>` |
| **Azure Key Vault** | `--keyvault <service-id>` |
| **Azure Container Registry** | `--container-registry <service-id>` |

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

El área de trabajo de Azure Machine Learning debe crearse dentro de un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo. Utilice el comando siguiente para __crear un nuevo grupo de recursos__. Reemplace `<resource-group-name>` por el nombre que va a usar para este grupo de recursos. Reemplace `<location>` por la región de Azure que va a usar para este grupo de recursos:

> [!TIP]
> Debe seleccionar una región en la que Azure Machine Learning esté disponible. Para obtener más información, consulte los [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service).

```azurecli-interactive
az group create --name <resource-group-name> --location <location>
```

La respuesta de este comando será similar al siguiente JSON:

```json
{
  "id": "/subscriptions/<subscription-GUID>/resourceGroups/<resourcegroupname>",
  "location": "<location>",
  "managedBy": null,
  "name": "<resource-group-name>",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

Para obtener más información sobre cómo trabajar con grupos de recursos, consulte [az group](https://docs.microsoft.com//cli/azure/group?view=azure-cli-latest).

### <a name="automatically-create-required-resources"></a>Creación automática de los recursos necesarios

Para crear una nueva área de trabajo en la que __los servicios se creen automáticamente__, use el siguiente comando:

> [!TIP]
> Los comandos de esta sección crean un área de trabajo de edición básica. Para crear un área de trabajo empresarial, use el modificador `--sku enterprise` con el comando `az ml workspace create`. Para más información sobre las ediciones de Azure Machine Learning, consulte [¿Qué es Azure Machine Learning?](overview-what-is-azure-ml.md#sku).

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name>
```

> [!NOTE]
> El nombre del área de trabajo no distingue mayúsculas de minúsculas.

La salida de este comando es similar al JSON siguiente:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

### <a name="use-existing-resources"></a>Uso de recursos existentes

Para crear un área de trabajo que use los recursos existentes, debe proporcionar el Id. de los recursos. Use los siguientes comandos para obtener el Id. de los servicios:

> [!IMPORTANT]
> No es necesario especificar todos los recursos existentes. Puede especificar uno o varios. Por ejemplo, puede especificar una cuenta de almacenamiento existente y el área de trabajo creará los demás recursos.

+ **Cuenta de Azure Storage**: `az storage account show --name <storage-account-name> --query "id"`

    La respuesta de este comando es similar al siguiente texto y es el Id. de la cuenta de almacenamiento:

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"`

    > [!IMPORTANT]
    > Si quiere usar una cuenta de Azure Storage existente, no puede ser una cuenta prémium (Premium_LRS o Premium_GRS). Tampoco puede tener un espacio de nombres jerárquico (usado con Azure Data Lake Storage Gen2). No se admite Premium Storage ni el espacio de nombres jerárquico con la cuenta de almacenamiento _predeterminada_ del área de trabajo. Puede usar Premium Storage o el espacio de nombres jerárquico con cuentas de almacenamiento _no predeterminadas_.

+ **Azure Application Insights**:

    1. Instale la extensión de Application Insights:

        ```azurecli-interactive
        az extension add -n application-insights
        ```

    2. Obtenga el Id. del servicio Application Insights:

        ```azurecli-interactive
        az monitor app-insights component show --app <application-insight-name> -g <resource-group-name> --query "id"
        ```

        La respuesta de este comando es similar al siguiente texto y es el Id. del servicio Application Insights:

        `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>"`

+ **Azure Key Vault**: `az keyvault show --name <key-vault-name> --query "ID"`

    La respuesta de este comando es similar al siguiente texto y es el Id. del almacén de claves:

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>"`

+ **Azure Container Registry**: `az acr show --name <acr-name> -g <resource-group-name> --query "id"`

    La respuesta de este comando es similar al siguiente texto y es el Id. del registro de contenedor:

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<acr-name>"`

    > [!IMPORTANT]
    > El registro de contenedor debe tener habilitada la [cuenta de administrador](/azure/container-registry/container-registry-authentication#admin-account) antes de que se pueda usar con un área de trabajo de Azure Machine Learning.

Una vez que tenga los id. de los recursos que desea usar con el área de trabajo, use el comando base `az workspace create -w <workspace-name> -g <resource-group-name>` y agregue los parámetros y los id. de los recursos existentes. Por ejemplo, el siguiente comando crea un área de trabajo que usa un registro de contenedor existente:

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name> --container-registry "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<acr-name>"
```

La salida de este comando es similar al JSON siguiente:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

## <a name="list-workspaces"></a>Enumeración de áreas de trabajo

Para enumerar todas las áreas de trabajo de su suscripción a Azure, use el siguiente comando:

```azurecli-interactive
az ml workspace list
```

La salida de este comando es similar al JSON siguiente:

```json
[
  {
    "resourceGroup": "myresourcegroup",
    "subscriptionId": "<subscription-id>",
    "workspaceName": "myml"
  },
  {
    "resourceGroup": "anotherresourcegroup",
    "subscriptionId": "<subscription-id>",
    "workspaceName": "anotherml"
  }
]
```

Para obtener más información, consulte la documentación de [az ml workspace list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-list).

## <a name="get-workspace-information"></a>Obtención de la información del área de trabajo

Para obtener información sobre un área de trabajo, use el siguiente comando:

```azurecli-interactive
az ml workspace show -w <workspace-name> -g <resource-group-name>
```

La salida de este comando es similar al JSON siguiente:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/application-insight-name>",
  "creationTime": "2019-08-30T18:55:03.1807976+00:00",
  "description": "",
  "friendlyName": "",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "tags": {},
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

Para obtener más información, consulte la documentación de [az ml workspace show](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-show).

## <a name="update-a-workspace"></a>Actualización de un área de trabajo

Use el comando siguiente para actualizar un área de trabajo:

```azurecli-interactive
az ml workspace update -w <workspace-name> -g <resource-group-name>
```

La salida de este comando es similar al JSON siguiente:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/application-insight-name>",
  "creationTime": "2019-08-30T18:55:03.1807976+00:00",
  "description": "",
  "friendlyName": "",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "tags": {},
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

Para obtener más información, consulte la documentación de [az ml workspace update](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-update).

## <a name="share-a-workspace-with-another-user"></a>Uso compartido de un área de trabajo con otro usuario

Para compartir un área de trabajo con otro usuario de la suscripción, use el siguiente comando:

```azurecli-interactive
az ml workspace share -w <workspace-name> -g <resource-group-name> --user <user> --role <role>
```

Para más información sobre el control de acceso basado en rol (RBAC) con Azure Machine Learning, consulte [Administración de usuarios y roles](how-to-assign-roles.md).

Para obtener más información, consulte la documentación de [az ml workspace share](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-share).

## <a name="sync-keys-for-dependent-resources"></a>Sincronización de claves para recursos dependientes

Si cambia las claves de acceso de uno de los recursos que usa el área de trabajo, esta tarda aproximadamente una hora en sincronizarse con las nuevas claves. Para forzar la sincronización inmediata del área de trabajo con las nuevas claves, use el siguiente comando:

```azurecli-interactive
az ml workspace sync-keys -w <workspace-name> -g <resource-group-name>
```

Para obtener más información sobre los cambios de clave, consulte [Regeneración de las claves de la cuenta de almacenamiento](how-to-change-storage-access-key.md).

Para obtener más información, consulte la documentación de [az ml workspace sync-keys](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-sync-keys).

## <a name="delete-a-workspace"></a>Eliminar un área de trabajo

Use el siguiente comando para eliminar un área de trabajo cuando ya no es necesaria:

```azurecli-interactive
az ml workspace delete -w <workspace-name> -g <resource-group-name>
```

> [!IMPORTANT]
> La eliminación de un área de trabajo no elimina la instancia de Application Insights, la cuenta de almacenamiento, el almacén de claves ni el registro de contenedor que usa el área de trabajo.

También puede eliminar el grupo de recursos; al hacerlo, se elimina el área de trabajo y todos los demás recursos de Azure en el grupo de recursos. Para eliminar el grupo de recursos, use el siguiente comando:

```azurecli-interactive
az group delete -g <resource-group-name>
```

Para obtener más información, consulte la documentación de [az ml workspace delete](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-delete).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="resource-provider-errors"></a>Errores del proveedor de recursos

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Movimiento del área de trabajo

> [!WARNING]
> No se admite mover el área de trabajo de Azure Machine Learning a otra suscripción ni mover la suscripción propietaria a un nuevo inquilino. Si lo hace, pueden producirse errores.

### <a name="deleting-the-azure-container-registry"></a>Eliminación de la instancia de Azure Container Registry

El área de trabajo de Azure Machine Learning usa Azure Container Registry (ACR) para algunas operaciones. La primera vez que se necesite una instancia de ACR, se creará automáticamente.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la extensión de la CLI de Azure para aprendizaje automático, consulte la documentación de [az ml](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml?view=azure-cli-latest).
