---
title: Creación de la primera función mediante plantillas de Azure Resource Manager
description: Cree e implemente en Azure una función sencilla sin servidor desencadenada por HTTP mediante una plantilla de Azure Resource Manager (plantilla de ARM).
ms.date: 3/5/2020
ms.topic: quickstart
ms.service: azure-functions
ms.custom: subject-armqs
ms.openlocfilehash: 362ff5cd59982c1d848ed59af8381090344f5c5e
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642272"
---
# <a name="quickstart-create-and-deploy-azure-functions-resources-from-an-arm-template"></a>Inicio rápido: Creación e implementación de recursos de Azure Functions a partir de una plantilla de Resource Manager

En este artículo va a utilizar una plantilla de Azure Resource Manager (plantilla de ARM) para crear una función que responda a solicitudes HTTP. 

Este inicio rápido supone un pequeño costo en su cuenta de Azure. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-function-app-create-dynamic%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

### <a name="azure-account"></a>Cuenta de Azure 

Antes de empezar, debe tener una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

### <a name="create-a-local-functions-project"></a>Creación de un proyecto local de Functions

En este artículo se requiere un proyecto de código local de Functions para que se ejecute en los recursos de Azure que cree. Si no crea primero un proyecto para su publicación, no podrá completar la sección de implementación de este artículo. 

Elija una de las siguientes pestañas, siga el vínculo y complete la sección para crear una aplicación de funciones en el lenguaje de su elección:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[Creación de un proyecto de Functions local en Visual Studio Code](functions-create-first-function-vs-code.md#create-an-azure-functions-project)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[Creación de un proyecto de Functions local en Visual Studio](functions-create-your-first-function-visual-studio.md#create-a-function-app-project)

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

[Creación de un proyecto de Functions local desde la línea de comandos](functions-create-first-azure-function-azure-cli.md#create-a-local-function-project)

---

Una vez que haya creado el proyecto de forma local, debe crear los recursos necesarios para ejecutar la nueva función en Azure. 

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-function-app-create-dynamic/).

:::code language="json" source="~/quickstart-templates/101-function-app-create-dynamic/azuredeploy.json":::

Los siguientes cuatro recursos de Azure se han creado mediante esta plantilla:

+ [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts): permite crear una cuenta de Azure Storage como requiere Functions.
+ [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): permite crear un plan de hospedaje de consumo sin servidor para la aplicación de funciones.
+ [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites): permite la creación de una aplicación de funciones.
+ [**microsoft.insights/components**](/azure/templates/microsoft.insights/components): permite la creación de una instancia de Application Insights para la supervisión.

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli-interactive
read -p "Enter a resource group name that is used for generating resource names:" resourceGroupName &&
read -p "Enter the location (like 'eastus' or 'northeurope'):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter a resource group name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (like 'eastus' or 'northeurope')"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```
---

## <a name="validate-the-deployment"></a>Validación de la implementación

A continuación, puede validar los recursos de hospedaje de la aplicación de funciones que creó mediante la publicación del proyecto en Azure y la llamada al punto de conexión HTTP de la función.

### <a name="publish-the-function-project-to-azure"></a>Publicación del proyecto de Functions en Azure

Siga estos pasos para publicar el proyecto en los nuevos recursos de Azure:

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [functions-republish-vscode](../../includes/functions-republish-vscode.md)]

En la salida, copie la dirección URL del desencadenador HTTP. Esto se usa para probar la función que se ejecuta en Azure. 

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En **Elegir un destino de publicación**, elija **Plan de consumo de Azure Functions** con las opciones **Seleccionar existente** y **Crear perfil**.

    :::image type="content" source="media/functions-create-first-function-arm/choose-publish-target-visual-studio.png" alt-text="Elección de un destino de publicación existente":::

1. Elija la **suscripción**, expanda el grupo de recursos, seleccione la aplicación de funciones y seleccione **Aceptar**.

1. Una vez finalizada la publicación, copie la **dirección URL del sitio**.

    :::image type="content" source="media/functions-create-first-function-arm/publish-summary-site-url.png" alt-text="Copia de la dirección URL del sitio desde el resumen de publicación":::

1. Anexe la ruta de acceso `/api/<FUNCTION_NAME>?name=Functions`, en la que `<FUNCTION_NAME>` es el nombre de la función. La dirección URL que llama a la función de desencadenador HTTP tiene el formato siguiente:

    `http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?name=Functions`

Puede usar esta dirección URL para probar la función desencadenada por HTTP que se ejecuta en Azure.

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

Para publicar su código local en una aplicación de funciones en Azure, use el comando `publish`:

```cmd
func azure functionapp publish <FUNCTION_APP_NAME>
```

En este ejemplo, reemplace `<FUNCTION_APP_NAME>` por el nombre de la aplicación de funciones. Es posible que tenga que iniciar sesión de nuevo mediante `az login`. 

En la salida, copie la dirección URL del desencadenador HTTP. Esto se usa para probar la función que se ejecuta en Azure.

---

### <a name="invoke-the-function-on-azure"></a>Invocación de la función en Azure

Pegue la dirección URL que copió de la solicitud HTTP en la barra de direcciones del navegador, asegúrese de que se ha anexado la cadena de consulta `name` como `?name=Functions` y, después, ejecute la solicitud. 

Debería obtener una respuesta similar a la siguiente:

<pre>Hello Functions!</pre>

## <a name="clean-up-resources"></a>Limpieza de recursos

Si continúa con el paso siguiente y agrega un enlace de salida de la cola de Azure Storage, conserve todos los recursos intactos, ya que va a crear a partir de lo que ya ha hecho.

De lo contrario, use el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene para evitar incurrir en costos adicionales.

```azurecli
az group delete --name <RESOURCE_GROUP_NAME>
```

Reemplace `<RESOURCE_GROUP_NAME>` por el nombre del grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha publicado la primera función, agregue un enlace de salida a la función para obtener más información.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-vs-code.md)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-vs.md)

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md)

---
