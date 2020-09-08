---
title: 'Tutorial: Desencadenamiento de un grupo de contenedores mediante una función de Azure'
description: Creación de una función de PowerShell sin servidor desencadenada por HTTP para automatizar la creación de instancias de contenedor de Azure
ms.topic: tutorial
ms.date: 06/10/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ec4b2273f6be6ea4aabed2b660e0b7553f861d0d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072050"
---
# <a name="tutorial-use-an-http-triggered-azure-function-to-create-a-container-group"></a>Tutorial: Uso de una función de Azure desencadenada por HTTP para crear un grupo de contenedores

[Azure Functions](../azure-functions/functions-overview.md) es un servicio de proceso sin servidor que puede ejecutar scripts o código como respuesta a diversos eventos, como una solicitud HTTP, un temporizador o un mensaje en una cola de Azure Storage.

En este tutorial se crea una función de Azure que toma una solicitud HTTP y desencadena la implementación de un [grupo de contenedores](container-instances-container-groups.md). En este ejemplo se muestran los fundamentos del uso de Azure Functions para crear automáticamente recursos en Azure Container Instances. Modifique o extienda el ejemplo a escenarios más complejos o a otros desencadenadores de eventos. 

Aprenderá a:

> [!div class="checklist"]
> * Usar Visual Studio Code con la [extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para crear una función básica de PowerShell desencadenada por HTTP.
> * Habilitar una identidad en la aplicación de funciones y a darle permisos para crear recursos de Azure.
> * Modificar y volver a publicar la función de PowerShell para automatizar la implementación de un grupo de contenedores con un solo contenedor.
> * Comprobar la implementación desencadenada por HTTP del contenedor.

## <a name="prerequisites"></a>Requisitos previos

Para saber cuáles son los requisitos previos para instalar y usar Visual Studio Code con la extensión de Azure Functions en su sistema operativo, consulte [Creación de su primera función en Azure con Visual Studio Code](../azure-functions/functions-create-first-function-vs-code.md?pivots=programming-language-powershell#configure-your-environment).

En los pasos adicionales de este artículo se usa Azure PowerShell. Si necesita instalar o actualizar, consulte [Instalación de Azure PowerShell][azure-powershell-install] e [Inicio de sesión en Azure](/powershell/azure/get-started-azureps#sign-in-to-azure).

## <a name="create-a-basic-powershell-function"></a>Creación de una función básica de PowerShell

Siga los pasos de [Creación de la primera función de PowerShell en Azure](../azure-functions/functions-create-first-function-vs-code.md?pivots=programming-language-powershell) para crear una función de PowerShell mediante la plantilla Desencadenador de HTTP. Use el nombre predeterminado de la función de Azure, **HttpTrigger**. Como se muestra en el inicio rápido, pruebe la función localmente y publique el proyecto en una aplicación de funciones en Azure. Este ejemplo es una función básica desencadenada por HTTP que devuelve una cadena de texto. En pasos posteriores de este artículo, se modificará la función para crear un grupo de contenedores.

En este artículo se supone que publica el proyecto con el nombre *myfunctionapp* en un grupo de recursos de Azure al que se asigna nombre automáticamente según el nombre de la aplicación de funciones (también *myfunctionapp*). Más adelante tendrá que sustituir tanto el nombre único de la aplicación de funciones como el nombre del grupo de recursos.

## <a name="enable-an-azure-managed-identity-in-the-function-app"></a>Habilitación de una identidad administrada por Azure en la aplicación de funciones

Los comandos siguientes habilitan una [identidad administrada](../app-service/overview-managed-identity.md?toc=/azure/azure-functions/toc.json#add-a-system-assigned-identity) asignada por el sistema en la aplicación de funciones. El host de PowerShell que ejecuta la aplicación se puede autenticar automáticamente en Azure mediante esta identidad, lo que permite que las funciones realicen acciones en los servicios de Azure a los que se ha concedido acceso a la identidad. En este tutorial, concederá permisos de identidad administrada para crear recursos en el grupo de recursos de la aplicación de funciones. 

[Agregue una identidad](../app-service/overview-managed-identity.md?tabs=dotnet#using-azure-powershell-1) a la aplicación de funciones:

```powershell
Update-AzFunctionApp -Name myfunctionapp `
    -ResourceGroupName myfunctionapp `
    -IdentityType SystemAssigned
```

Asigne a la identidad el rol de colaborador en el ámbito del grupo de recursos:

```powershell
$SP=(Get-AzADServicePrincipal -DisplayName myfunctionapp).Id
$RG=(Get-AzResourceGroup -Name myfunctionapp).ResourceId
New-AzRoleAssignment -ObjectId $SP -RoleDefinitionName "Contributor" -Scope $RG
```

## <a name="modify-httptrigger-function"></a>Modificación de la función HttpTrigger

Modifique el código de PowerShell de la función **HttpTrigger** para crear un grupo de contenedores. En el archivo `run.ps1` de la función, busque el siguiente bloque de código. Este código muestra el valor de un nombre, si se ha usado alguno como cadena de consulta en la dirección URL de la función:

```powershell
[...]
if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}
[...]
```

Reemplace este código por el siguiente bloque de ejemplo. Aquí, si se pasa el valor de un nombre en la cadena de consulta, se usa para asignar un nombre y crear un grupo de contenedores mediante el cmdlet [New-AzContainerGroup][new-azcontainergroup]. Asegúrese de reemplazar el nombre del grupo de recursos *myfunctionapp* por el nombre del grupo de recursos de la aplicación de funciones:

```powershell
[...]
if ($name) {
    New-AzContainerGroup -ResourceGroupName myfunctionapp -Name $name `
        -Image alpine -OsType Linux `
        -Command "echo 'Hello from an Azure container instance triggered by an Azure function'" `
        -RestartPolicy Never
    if ($?) {
        $body = "This HTTP triggered function executed successfully. Started container group $name"
    }
    else  {
        $body = "There was a problem starting the container group."
    }
[...]
```

En este ejemplo se crea un grupo de contenedores que consta de una única instancia de contenedor que ejecuta la imagen `alpine`. El contenedor ejecuta un único comando `echo` y, después, finaliza. En un ejemplo real, podría desencadenar la creación de uno o varios grupos de contenedores para ejecutar un trabajo por lotes.
 
## <a name="test-function-app-locally"></a>Prueba de una aplicación de funciones en un entorno local

Asegúrese de que la función se ejecuta en el entorno local antes de volver a publicar el proyecto de la aplicación de funciones en Azure. Cuando se ejecuta localmente, la función no crea recursos de Azure. Sin embargo, puede probar el flujo de la función con y sin pasar un valor de nombre en una cadena de consulta. Para depurar la función, consulte [Depuración local de Azure Functions de PowerShell](../azure-functions/functions-debug-powershell-local.md).

## <a name="republish-azure-function-app"></a>Republicación de una aplicación de funciones de Azure

Una vez que haya comprobado que la función se ejecuta localmente, vuelva a publicar el proyecto en la aplicación de funciones existente en Azure.

1. En Visual Studio Code, abra la paleta de comandos. Busque y seleccione `Azure Functions: Deploy to Function App...`.
1. Seleccione la carpeta de trabajo actual para comprimirla y realizar la implementación.
1. Seleccione la suscripción y, después, el nombre de la aplicación de funciones existente (*myfunctionapp*). Confirme que desea sobrescribir la implementación anterior.

Una vez que se haya creado la aplicación de función se mostrará una notificación y se aplicará el paquete de implementación. Seleccione **View Output** (Ver salida) en esta notificación para ver la creación y los resultados de la implementación, incluidos los recursos de Azure que ha actualizado.

## <a name="run-the-function-in-azure"></a>Ejecución de la función en Azure

Una vez que la implementación se complete correctamente, obtenga la dirección URL de la función. Por ejemplo, use el área **Azure: Functions** de Visual Studio Code para copiar la dirección URL de la función **HttpTrigger** u obtenga la dirección URL de la función en [Azure Portal](../azure-functions/functions-create-first-azure-function.md#test-the-function).

La dirección URL de la función tiene el formato:

```
https://myfunctionapp.azurewebsites.net/api/HttpTrigger
```

### <a name="run-function-without-passing-a-name"></a>Ejecución de la función sin pasar ningún nombre

Como primera prueba, ejecute el comando `curl` y pase la dirección URL de la función sin anexar una cadena de consulta `name`. 

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger"
```

La función devuelve el código de estado 200 y el texto `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response`:

```
[...]
> GET /api/HttpTrigger? HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/1.1 200 OK
< Content-Length: 135
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:50:27 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.* Closing connection 0
```

### <a name="run-function-and-pass-the-name-of-a-container-group"></a>Ejecución de la función y uso del nombre de un grupo de contenedores

Ahora, ejecute el comando `curl` y anexe el nombre de un grupo de contenedores (*mycontainergroup*) como una cadena de consulta `?name=mycontainergroup`:

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger?name=mycontainergroup"
```

La función devuelve el código de estado 200 y desencadena la creación del grupo de contenedores:

```
[...]
> GET /api/HttpTrigger1?name=mycontainergroup HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 92
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:54:31 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Started container group mycontainergroup* Closing connection 0
```

Compruebe que el contenedor se ha ejecutado con el comando [Get-AzContainerInstanceLog][get-azcontainerinstancelog]:

```azurecli
Get-AzContainerInstanceLog -ResourceGroupName myfunctionapp `
  -ContainerGroupName mycontainergroup 
```

Salida del ejemplo:

```console
Hello from an Azure container instance triggered by an Azure function
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos creados en este tutorial, puede ejecutar el comando [az group delete][az-group-delete] para eliminar el grupo de recursos y todos los recursos que contiene. Este comando elimina la aplicación de funciones que se ha creado, así como el contenedor en ejecución y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myfunctionapp
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una función de Azure que toma una solicitud HTTP y desencadena la implementación de un grupo de contenedores. Ha aprendido a:

> [!div class="checklist"]
> * Usar Visual Studio Code con la extensión de Azure Functions para crear una función básica de PowerShell desencadenada por HTTP.
> * Habilitar una identidad en la aplicación de funciones y a darle permisos para crear recursos de Azure.
> * Modificar el código de la función de PowerShell para automatizar la implementación de un grupo de contenedores con un solo contenedor.
> * Comprobar la implementación desencadenada por HTTP del contenedor.

Para obtener un ejemplo detallado de cómo iniciar y supervisar un trabajo con contenedores, consulte la entrada de blog sobre los [contenedores sin servidor basados en eventos con PowerShell Azure Functions y Azure Container Instances](https://dev.to/azure/event-driven-serverless-containers-with-powershell-azure-functions-and-azure-container-instances-e9b) y el [código de ejemplo](https://github.com/anthonychu/functions-powershell-run-aci) que lo acompaña.

Consulte la [documentación de Azure Functions](../azure-functions/index.yml) para obtener instrucciones detalladas sobre la creación de funciones de Azure y la publicación de un proyecto de Functions. 

<!-- IMAGES -->


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-powershell-install]: /powershell/azure/install-az-ps
[new-azcontainergroup]: /powershell/module/az.containerinstance/new-azcontainergroup
[get-azcontainerinstancelog]: /powershell/module/az.containerinstance/get-azcontainerinstancelog
