---
title: 'Tutorial: Acciones y recursos personalizados'
description: En este tutorial se describe cómo crear una aplicación administrada de Azure con un proveedor personalizado de Azure.
ms.topic: tutorial
ms.author: lazinnat
author: lazinnat
ms.date: 06/20/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: e8824f534f573d97353cc86d2a1b112b1acdb211
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494509"
---
# <a name="tutorial-create-managed-application-with-custom-actions-and-resources"></a>Tutorial: Creación de una aplicación administrada con acciones y recursos personalizados

En este tutorial, creará su propia aplicación administrada con acciones y recursos personalizados. La aplicación administrada contendrá una acción personalizada en la página `Overview`, un tipo de recurso personalizado que se muestra como un elemento de menú independiente en `Table of Content` y una acción de contexto personalizada en la página de recursos personalizados.

Este tutorial incluye los siguientes pasos:

> [!div class="checklist"]
> * Creación de un archivo de definición de la interfaz de usuario para crear una instancia de aplicación administrada.
> * Creación de una plantilla de implementación con el [proveedor personalizado de Azure](../custom-providers/overview.md), la cuenta de Azure Storage y una función de Azure.
> * Creación de un artefacto de definición de vistas con acciones y recursos personalizados.
> * Creación de una definición de aplicación administrada.
> * Implementación de una instancia de aplicación administrada.
> * Realización de acciones personalizadas y creación de recursos personalizados.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita saber lo siguiente:

* [Creación y publicación de una definición de aplicación administrada](publish-service-catalog-app.md).
* [Implementación de aplicaciones del catálogo de servicios mediante Azure Portal](deploy-service-catalog-quickstart.md).
* [Creación de la interfaz de usuario de Azure Portal para una aplicación administrada](create-uidefinition-overview.md).
* Funcionalidades de [artefactos de definición de vistas](concepts-view-definition.md).
* Funcionalidades del [proveedor personalizado de Azure](../custom-providers/overview.md).

## <a name="user-interface-definition"></a>Definición de la interfaz de usuario

En este tutorial, creará una aplicación administrada y su grupo de recursos administrados contendrá una instancia de proveedor personalizado, una cuenta de almacenamiento y una función. La función de Azure que se usa en este ejemplo implementa una API que controla las operaciones del proveedor personalizado para las acciones y los recursos. La cuenta de Azure Storage se usa como almacenamiento básico para los recursos del proveedor personalizado.

La definición de la interfaz de usuario para crear una instancia de aplicación administrada incluye los elementos de entrada `funcname` y `storagename`. El nombre de la cuenta de almacenamiento y el nombre de la función deben ser únicos globalmente. De forma predeterminada, los archivos de función se implementarán desde el [paquete de función de ejemplo](https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip), pero esto puede cambiarse si agrega un elemento de entrada de un vínculo de paquete en *createUIDefinition.json*:

```json
{
  "name": "funcname",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the function to be created",
  "toolTip": "Name of the function to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "storagename",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the storage to be created",
  "toolTip": "Name of the storage to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "zipFileBlobUri",
  "type": "Microsoft.Common.TextBox",
  "defaultValue": "https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip/functionpackage.zip",
  "label": "The Uri to the uploaded function zip file",
  "toolTip": "The Uri to the uploaded function zip file",
  "visible": true
}
```

y de salida en *createUIDefinition.json*:

```json
  "funcname": "[steps('applicationSettings').funcname]",
  "storageName": "[steps('applicationSettings').storagename]",
  "zipFileBlobUri": "[steps('applicationSettings').zipFileBlobUri]"
```

El ejemplo de *createUIDefinition.json* completo se puede encontrar en [Referencia: Artefactos de elementos de la interfaz de usuario](reference-createuidefinition-artifact.md).

## <a name="template-with-custom-provider"></a>Plantilla con proveedor personalizado

Para crear una instancia de aplicación administrada con un proveedor personalizado, debe definir el recurso de proveedor personalizado con el nombre **public** y el tipo **Microsoft.CustomProviders/resourceProviders** en **mainTemplate.json** En ese recurso, defina los tipos de recurso y las acciones para el servicio. Para implementar las instancias de Azure Functions y de la cuenta de Azure Storage, defina recursos del tipo `Microsoft.Web/sites` y `Microsoft.Storage/storageAccounts` respectivamente.

En este tutorial, creará un tipo de recurso `users`, una acción personalizada `ping` y una acción personalizada `users/contextAction` que se realizarán en un contexto de un recurso personalizado `users`. Para cada tipo de recurso y acción, proporcione un punto de conexión que apunte a la función con el nombre proporcionado en [createUIDefinition.json](#user-interface-definition). Especifique **routingType** como `Proxy,Cache` para los tipos de recurso y `Proxy` para las acciones:

```json
{
  "apiVersion": "[variables('customrpApiversion')]",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "name": "[variables('customProviderName')]",
  "location": "[parameters('location')]",
  "properties": {
    "actions": [
      {
        "name": "ping",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      },
      {
        "name": "users/contextAction",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ],
    "resourceTypes": [
      {
        "name": "users",
        "routingType": "Proxy,Cache",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ]
  },
  "dependsOn": [
    "[concat('Microsoft.Web/sites/',parameters('funcname'))]"
  ]
}
```

El ejemplo de *mainTemplate.json* completo se puede encontrar en [Referencia: Artefacto de plantilla de implementación](reference-main-template-artifact.md).

## <a name="view-definition-artifact"></a>Ver definición de artefacto

Para definir una interfaz de usuario que incluya acciones personalizadas y recursos personalizados en la aplicación administrada, debe crear el artefacto **viewDefinition.json**. Para más información sobre el artefacto de definición de vistas, consulte [Artefacto de definición de vistas en Azure Managed Applications](concepts-view-definition.md).

En este tutorial, definirá:
* Una página de *información general* con un botón de la barra de herramientas que representa una acción personalizada `TestAction` con entrada de texto básica.
* Una página de *usuarios* que representa un tipo de recurso personalizado `users`.
* Una acción de recursos personalizada `users/contextAction` en la página de *usuarios* que se realizará en un contexto de recurso personalizado de tipo `users`.

En el ejemplo siguiente se muestra la configuración de vista de una página de información general:

```json
{
    "kind": "Overview",
    "properties": {
      "header": "Welcome to your Demo Azure Managed Application",
      "description": "This Managed application with Custom Provider is for demo purposes only.",
      "commands": [{
          "displayName": "Ping Action",
          "path": "/customping",
          "icon": "LaunchCurrent"
      }]
    }
  }
```

En el ejemplo siguiente se incluye la configuración de la página de recursos de usuarios con la acción de recursos personalizada:

```json
{
    "kind": "CustomResources",
    "properties": {
      "displayName": "Users",
      "version": "1.0.0.0",
      "resourceType": "users",
      "createUIDefinition": {
      },
      "commands": [{
        "displayName": "Custom Context Action",
        "path": "users/contextAction",
        "icon": "Start"
      }],
      "columns": [
        { "key": "properties.FullName", "displayName": "Full Name" },
        { "key": "properties.Location", "displayName": "Location", "optional": true }
      ]
    }
  }
```

El ejemplo de *viewDefinition.json* completo se puede encontrar en [Referencia: Artefacto de definición de vistas](reference-view-definition-artifact.md).

## <a name="managed-application-definition"></a>Definición de aplicación administrada

Empaquete los siguientes artefactos de aplicación administrada en un archivo ZIP y cárguelo en el almacenamiento:

* createUiDefinition.json
* mainTemplate.json
* viewDefinition.json

Todos los archivos deben estar en el nivel raíz. El paquete con artefactos se puede almacenar en cualquier almacenamiento, por ejemplo, un blob de GitHub o un blob de una cuenta de Azure Storage. Este es un script para cargar el paquete de aplicación en la cuenta de almacenamiento: 

```powershell
$resourceGroup="appResourcesGroup"
$storageName="mystorageaccount$RANDOM"

# Sign in to your Azure subscription
Connect-AzAccount
# Create resource group for managed application definition and application package
New-AzResourceGroup -Name $resourceGroup -Location eastus

# Create storage account for a package with application artifacts
$storageAccount=New-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -SkuName Standard_LRS `
  -Location eastus `
$ctx=$storageAccount.Context

# Create storage container and upload zip to blob
New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob
Set-AzStorageBlobContent `
  -File "path_to_your_zip_package" `
  -Container appcontainer `
  -Blob app.zip `
  -Context $ctx 

# Get blob absolute uri
$blobUri=(Get-AzureStorageBlob -Container appcontainer -Blob app.zip -Context $ctx).ICloudBlob.uri.AbsoluteUri
```

Ejecute el siguiente script de la CLI de Azure o siga los pasos descritos en Azure Portal para implementar una definición de aplicación administrada del catálogo de servicios:

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli-interactive)

```azurecli-interactive
resourceGroup="appResourcesGroup"
# Select subscription and create resource group (if you have not created yet)
az account set --subscription <subscriptionID>
az group create --name $resourceGroup --location eastus

# Get object ID of your identity
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
# Get role definition ID for the Owner role
roleid=$(az role definition list --name Owner --query [].name --output tsv)

# Create managed application definition resource
az managedapp definition create \
  --name "ManagedUsersAppDefinition" \
  --location "eastus" \
  --resource-group $resourceGroup \
  --lock-level ReadOnly \
  --display-name "Managed users app definition" \
  --description "Managed application with Azure Custom Provider" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "path to your app.zip package"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba y seleccione **Centro de Managed Applications**.
2. En **Centro de Managed Applications**, seleccione **Definición de aplicación del catálogo de servicios** y haga clic en **Agregar**. 
    
    ![Adición de catálogo de servicios](./media/tutorial-create-managed-app-with-custom-provider/service-catalog-managed-application.png)

3. Proporcione valores para crear una definición del catálogo de servicios:

    * Proporcione un **nombre** único para la definición del catálogo de servicios, un **nombre para mostrar** y una *descripción* (opcional).
    * Seleccione la **suscripción**, el **grupo de recursos** y la **ubicación** donde se creará la definición de la aplicación. Puede usar el mismo grupo de recursos que se usa para el paquete ZIP o crear un nuevo grupo de recursos.
    * Para el **identificador URI del archivo de paquete**, proporcione la ruta de acceso al archivo ZIP que creó en el paso anterior.

    ![Proporcionar valores](./media/tutorial-create-managed-app-with-custom-provider/add-service-catalog-managed-application.png)

4. Cuando llegue a la sección de Autenticación y Nivel de bloqueo, seleccione **Agregar autorización**.

    ![Agregar autorización](./media/tutorial-create-managed-app-with-custom-provider/add-authorization.png)

5. Seleccione un grupo de Azure Active Directory para administrar los recursos y seleccione **Aceptar**.

   ![Agregar grupo de autorización](./media/tutorial-create-managed-app-with-custom-provider/add-auth-group.png)

6. Cuando haya proporcionado todos los valores, seleccione **Crear**.

   ![Creación de la definición de aplicación administrada](./media/tutorial-create-managed-app-with-custom-provider/create-service-catalog-definition.png)

---

## <a name="managed-application-instance"></a>Instancia de la aplicación administrada

Una vez implementada la definición de la aplicación administrada, ejecute el siguiente script o siga los pasos descritos en Azure Portal para implementar la instancia de la aplicación administrada con el proveedor personalizado:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli-interactive)

```azurecli-interactive
appResourcesGroup="appResourcesGroup"
applicationGroup="usersApplicationGroup"

# Create resource group for managed application instance
az group create --name $applicationGroup --location eastus

# Get ID of managed application definition
appid=$(az managedapp definition show --name ManagedUsersAppDefinition --resource-group $appResourcesGroup --query id --output tsv)

# Create the managed application
az managedapp create \
  --name ManagedUsersApp \
  --location "eastus" \
  --kind "Servicecatalog" \
  --resource-group $applicationGroup \
  --managedapp-definition-id $appid \
  --managed-rg-id "managedResourcesGroup" \
  --parameters "{\"funcname\": {\"value\": \"managedusersappfunction\"}, \"storageName\": {\"value\": \"managedusersappstorage\"}}"
```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba y seleccione **Centro de Managed Applications**.
2. En **Centro de Managed Applications**, seleccione **Aplicaciones del catálogo de servicios** y haga clic en **Agregar**. 

    ![Adición de aplicación administrada](./media/tutorial-create-managed-app-with-custom-provider/add-managed-application.png)

3. En la página **Aplicaciones del catálogo de servicios**, escriba el nombre para mostrar de la definición del catálogo de servicios en el cuadro de búsqueda. Seleccione la definición creada en el paso anterior y haga clic en **Crear**.

    ![Selección del catálogo de servicios](./media/tutorial-create-managed-app-with-custom-provider/select-service-catalog-definition.png)

4. Proporcione valores para crear una instancia de aplicación administrada a partir de la definición del catálogo de servicios:

    * Seleccione la **suscripción**, el **grupo de recursos** y la **ubicación** donde se creará la instancia de la aplicación.
    * Proporcione un nombre de función de Azure y un nombre de cuenta de Azure Storage únicos.

    ![Configuración de la aplicación](./media/tutorial-create-managed-app-with-custom-provider/application-settings.png)

5. Cuando se supere la validación, haga clic en **Aceptar** para implementar una instancia de una aplicación administrada. 
    
    ![Implementación de una aplicación administrada](./media/tutorial-create-managed-app-with-custom-provider/deploy-managed-application.png)

---

## <a name="custom-actions-and-resources"></a>Acciones y recursos personalizados

Una vez implementada la instancia de la aplicación del catálogo de servicios, tendrá dos nuevos grupos de recursos. El primer grupo de recursos `applicationGroup` contiene una instancia de la aplicación administrada y el segundo grupo de recursos `managedResourceGroup` contiene los recursos de la aplicación administrada, incluido el **proveedor personalizado**.

![Grupos de recursos de la aplicación](./media/tutorial-create-managed-app-with-custom-provider/application-resource-groups.png)

Puede ir a la instancia de la aplicación administrada y realizar una **acción personalizada** en la página de información general, crear un recurso personalizado de **usuarios** en la página de usuarios y ejecutar una **acción de contexto personalizada** en un recurso personalizado.

* Vaya a la página de información general y haga clic en el botón "Ping Action" (Acción ping):

![Realización de acción personalizada](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-action.png)

* Vaya a la página de usuarios y haga clic en el botón "Agregar". Proporcione entradas para crear un recurso y envíe el formulario:

![Creación de un recurso personalizado](./media/tutorial-create-managed-app-with-custom-provider/create-custom-resource.png)

* Vaya a la página de usuarios, seleccione un recurso "users" y haga clic en "Custom Context Action" (Acción de contexto personalizado):

![Creación de un recurso personalizado](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-resource-action.png)

[!INCLUDE [clean-up-section-portal](../../../includes/clean-up-section-portal.md)]

## <a name="looking-for-help"></a>¿Busca ayuda?

Si tiene alguna pregunta sobre Azure Managed Applications, pregunte en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-managedapps). Puede que ya se haya formulado y respondido una pregunta similar, así que compruébelo antes de publicar la suya. Agregue la etiqueta `azure-managedapps` para obtener una respuesta rápida.

## <a name="next-steps"></a>Pasos siguientes

Para publicar la aplicación administrada en Azure Marketplace, consulte [Aplicaciones administradas de Azure en Marketplace](publish-marketplace-app.md).

Más información sbre los [proveedores personalizados de Azure](../custom-providers/overview.md).
