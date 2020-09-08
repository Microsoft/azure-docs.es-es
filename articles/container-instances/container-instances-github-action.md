---
title: Implementación de una instancia de contenedor mediante una acción de GitHub
description: Configure una acción de GitHub que automatice los pasos necesarios para compilar, enviar e implementar una imagen de contenedor a Azure Container Instances.
ms.topic: article
ms.date: 08/20/2020
ms.custom: ''
ms.openlocfilehash: 8da72d3911797e8e3a4551f2af100afb0d7ea0fb
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88755014"
---
# <a name="configure-a-github-action-to-create-a-container-instance"></a>Configuración de una acción de GitHub para crear una instancia de contenedor

[Acciones de GitHub](https://help.github.com/actions/getting-started-with-github-actions/about-github-actions) es un conjunto de características de GitHub para automatizar los flujos de trabajo de desarrollo de software en el mismo lugar donde almacena el código y colabora en las solicitudes de incorporación de cambios y problemas.

Use la acción de GitHub [Deploy to Azure Container Instances](https://github.com/azure/aci-deploy) para automatizar la implementación de un contenedor único en Azure Container Instances. La acción le permite establecer las propiedades de una instancia de contenedor de manera similar a las del comando [az container create][az-container-create].

En este artículo se muestra cómo configurar un flujo de trabajo en un repositorio de GitHub que realiza las acciones siguientes:

* Creación de una imagen desde Dockerfile
* Inserción de la imagen en una instancia de Azure Container Registry
* Implementación de la imagen de contenedor en una instancia de Azure Container Instance

En este artículo se muestran dos maneras de configurar el flujo de trabajo:

* [Configuración de un flujo de trabajo de GitHub](#configure-github-workflow): cree un flujo de trabajo en un repositorio de GitHub con la acción Deploy to Azure Container Instances y otras acciones.  
* [Uso de la extensión de la CLI](#use-deploy-to-azure-extension): use el comando `az container app up` en la extensión [Deploy to Azure](https://github.com/Azure/deploy-to-azure-cli-extension) en la CLI de Azure. Este comando simplifica la creación de un flujo de trabajo de GitHub y los pasos de implementación.

> [!IMPORTANT]
> La acción de GitHub para Azure Container Instances está actualmente en versión preliminar. Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="prerequisites"></a>Prerrequisitos

* **Cuenta de GitHub**: Cree una cuenta en https://github.com si aún no tiene una.
* **CLI de Azure**: Para completar los pasos de la CLI de Azure, puede usar Azure Cloud Shell o una instalación local de la CLI de Azure. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].
* **Registro de contenedor de Azure**: Si no tiene uno, cree un registro de contenedor de Azure en el nivel Básico mediante la [CLI de Azure](../container-registry/container-registry-get-started-azure-cli.md), [Azure Portal](../container-registry/container-registry-get-started-portal.md) u otros métodos. Tome nota del grupo de recursos que se utiliza para la implementación, que se usa para el flujo de trabajo de GitHub.

## <a name="set-up-repo"></a>Configuración del repositorio

* Para ver los ejemplos de este artículo, utilice GitHub para bifurcar el siguiente repositorio: https://github.com/Azure-Samples/acr-build-helloworld-node

  Este repositorio contiene un Dockerfile y archivos de código fuente para crear una imagen de contenedor de una pequeña aplicación web.

  ![Captura de pantalla del botón Bifurcar (resaltado) en GitHub](../container-registry/media/container-registry-tutorial-quick-build/quick-build-01-fork.png)

* Asegúrese de Acciones esté habilitado para el repositorio. Navegue al repositorio bifurcado y seleccione **Configuración** > **Acciones**. En **Actions permissions** (Permisos de acciones), asegúrese de que **Enable local and third party Actions for this repository** (Habilitar Acciones locales y de terceros para este repositorio) esté seleccionado.

## <a name="configure-github-workflow"></a>Configuración de flujo de trabajo de GitHub

### <a name="create-service-principal-for-azure-authentication"></a>Creación de una entidad de servicio para la autenticación de Azure

En el flujo de trabajo de GitHub, debe proporcionar las credenciales de Azure para autenticarse en la CLI de Azure. En el ejemplo siguiente se crea una entidad de servicio con el rol Colaborador limitado al grupo de recursos del registro de contenedor.

En primer lugar, obtenga el identificador de recurso del grupo de recursos. Sustituya el nombre del grupo en el siguiente comando [az group show][az-group-show]:

```azurecli
groupId=$(az group show \
  --name <resource-group-name> \
  --query id --output tsv)
```

Utilice [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] para crear la entidad de servicio:

```azurecli
az ad sp create-for-rbac \
  --scope $groupId \
  --role Contributor \
  --sdk-auth
```

La salida es parecida a esta:

```json
{
  "clientId": "xxxx6ddc-xxxx-xxxx-xxx-ef78a99dxxxx",
  "clientSecret": "xxxx79dc-xxxx-xxxx-xxxx-aaaaaec5xxxx",
  "subscriptionId": "xxxx251c-xxxx-xxxx-xxxx-bf99a306xxxx",
  "tenantId": "xxxx88bf-xxxx-xxxx-xxxx-2d7cd011xxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

Guarde la salida de JSON porque se usará en un paso posterior. Además, tome nota de `clientId`, que necesita para actualizar la entidad de servicio en la sección siguiente.

### <a name="update-service-principal-for-registry-authentication"></a>Actualización de la entidad de servicio para la autenticación del registro

Actualice las credenciales de la entidad de servicio de Azure para habilitar el acceso de inserción y extracción en el registro de contenedor. Este paso permite que el flujo de trabajo de GitHub use la entidad de servicio para [autenticarse en el registro de contenedor](../container-registry/container-registry-auth-service-principal.md) y realizar la inserción y extracción de una imagen de Docker. 

Obtenga el identificador de recurso del registro de contenedor. Sustituya el nombre del registro en el siguiente comando [az acr show][az-acr-show]:

```azurecli
registryId=$(az acr show \
  --name <registry-name> \
  --query id --output tsv)
```

Utilice [az role assignment create][az-role-assignment-create] para asignar el rol AcrPush, que proporciona acceso de inserción y extracción al registro. Sustituya el id. de cliente de la entidad de servicio:

```azurecli
az role assignment create \
  --assignee <ClientId> \
  --scope $registryId \
  --role AcrPush
```

### <a name="save-credentials-to-github-repo"></a>Almacenamiento de las credenciales en el repositorio de GitHub

1. En la UI de GitHub, navegue al repositorio bifurcado y seleccione **Configuración** > **Secretos**. 

1. Seleccione **Add a new secret** (Agregar un nuevo secreto) para agregar los siguientes secretos:

|Secreto  |Value  |
|---------|---------|
|`AZURE_CREDENTIALS`     | Toda la salida JSON del paso de creación de la entidad de servicio |
|`REGISTRY_LOGIN_SERVER`   | Nombre del servidor de inicio de sesión del registro (todo en minúsculas). Ejemplo: *myregistry.azurecr.io*        |
|`REGISTRY_USERNAME`     |  `clientId` de la salida JSON de la creación de la entidad de servicio       |
|`REGISTRY_PASSWORD`     |  `clientSecret` de la salida JSON de la creación de la entidad de servicio |
| `RESOURCE_GROUP` | Nombre del grupo de recursos que usó para limitar el ámbito de la entidad de servicio |

### <a name="create-workflow-file"></a>Creación del archivo de flujo de trabajo

1. En la UI de GitHub, seleccione **Acciones** > **Nuevo flujo de trabajo**.
1. Seleccione **Set up a workflow yourself** (Configurar un flujo de trabajo personalmente).
1. En **Edit new file** (Editar nuevo archivo), pegue el contenido de YAML siguiente para sobrescribir el código de ejemplo. Acepte el nombre de archivo predeterminado `main.yml` o proporcione un nombre de archivo a su elección.
1. Seleccione **Start commit** (Iniciar confirmación) y, opcionalmente, proporcione descripciones breves y ampliadas de la confirmación, y seleccione **Commit new file** (Confirmar nuevo archivo).

```yml
on: [push]
name: Linux_Container_Workflow

jobs:
    build-and-deploy:
        runs-on: ubuntu-latest
        steps:
        # checkout the repo
        - name: 'Checkout GitHub Action'
          uses: actions/checkout@master
          
        - name: 'Login via Azure CLI'
          uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
        
        - name: 'Build and push image'
          uses: azure/docker-login@v1
          with:
            login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            username: ${{ secrets.REGISTRY_USERNAME }}
            password: ${{ secrets.REGISTRY_PASSWORD }}
        - run: |
            docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}

        - name: 'Deploy to Azure Container Instances'
          uses: 'azure/aci-deploy@v1'
          with:
            resource-group: ${{ secrets.RESOURCE_GROUP }}
            dns-name-label: ${{ secrets.RESOURCE_GROUP }}${{ github.run_number }}
            image: ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            registry-login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            registry-username: ${{ secrets.REGISTRY_USERNAME }}
            registry-password: ${{ secrets.REGISTRY_PASSWORD }}
            name: aci-sampleapp
            location: 'west us'
```

### <a name="validate-workflow"></a>Validación del flujo de trabajo

Después de confirmar el archivo de flujo de trabajo, este se desencadena. Para revisar el progreso del flujo de trabajo, vaya a **Acciones** > **Flujos de trabajo**. 

![Visualización del progreso del flujo de trabajo](./media/container-instances-github-action/github-action-progress.png)

Consulte [Managing a workflow run](https://help.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run) para obtener información sobre cómo ver el estado y los resultados de cada paso del flujo de trabajo. Si el flujo de trabajo no se completa, consulte [Visualización de registros para diagnosticar errores](https://docs.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run#viewing-logs-to-diagnose-failures).

Cuando se complete correctamente el flujo de trabajo, obtenga información sobre la instancia de contenedor denominada *aci-sampleapp* mediante la ejecución del comando [az container show][az-container-show]. Sustituya el nombre del grupo de recursos: 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name aci-sampleapp \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

La salida es parecida a esta:

```console
FQDN                                   ProvisioningState
---------------------------------      -------------------
aci-action01.westus.azurecontainer.io  Succeeded
```

Una vez aprovisionada la instancia, navegue hasta el FQDN del contenedor en el explorador para ver la aplicación web en ejecución.

![Ejecución de la aplicación web en el explorador](./media/container-instances-github-action/github-action-container.png)

## <a name="use-deploy-to-azure-extension"></a>Uso de la extensión Implementar en Azure

Como alternativa, utilice la [extensión Implementar en Azure](https://github.com/Azure/deploy-to-azure-cli-extension) en la CLI de Azure para configurar el flujo de trabajo. El comando `az container app up` de la extensión toma los parámetros de entrada del usuario para configurar un flujo de trabajo que se va a implementar en Azure Container Instances. 

El flujo de trabajo creado por la CLI de Azure es similar al flujo de trabajo que puede [crear manualmente mediante GitHub](#configure-github-workflow).

### <a name="additional-prerequisite"></a>Requisitos previos adicionales

Además de los [requisitos previos](#prerequisites) y de la [configuración del repositorio](#set-up-repo) para este escenario, debe instalar la **extensión Implementar en Azure** para la CLI de Azure.

Ejecute el comando [az extension add][az-extension-add] para instalar la extensión:

```azurecli
az extension add \
  --name deploy-to-azure
```

Para obtener información sobre cómo buscar, instalar y administrar extensiones, consulte [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview).

### <a name="run-az-container-app-up"></a>Ejecute `az container app up`:

Para ejecutar el comando [az container app up][az-container-app-up], proporcione como mínimo:

* El nombre de la instancia de Azure Container Registry, por ejemplo, *miregistro*
* La dirección URL del repositorio de GitHub, por ejemplo, `https://github.com/<your-GitHub-Id>/acr-build-helloworld-node`

Comando de ejemplo:

```azurecli
az container app up \
  --acr myregistry \
  --repository https://github.com/myID/acr-build-helloworld-node
```

### <a name="command-progress"></a>Progreso del comando

* Cuando se le solicite, proporcione sus credenciales de GitHub o proporcione un [token de acceso personal (PAT) de GitHub](https://help.github.com/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) que tenga ámbitos de *repositorio* y de *usuario* para autenticarse en la cuenta de GitHub. Si proporciona las credenciales de GitHub, el comando crea un PAT automáticamente. Siga las indicaciones adicionales para configurar el flujo de trabajo.

* El comando crea secretos del repositorio para el flujo de trabajo:

  * Credenciales de la entidad de servicio para la CLI de Azure
  * Credenciales para el acceso a la instancia de Azure Container Registry

* Después de que el comando confirme el archivo de flujo de trabajo en el repositorio, se desencadena el flujo de trabajo. 

La salida es parecida a esta:

```console
[...]
Checking in file github/workflows/main.yml in the Github repository myid/acr-build-helloworld-node
Creating workflow...
GitHub Action Workflow has been created - https://github.com/myid/acr-build-helloworld-node/runs/515192398
GitHub workflow completed.
Workflow succeeded
Your app is deployed at:  http://acr-build-helloworld-node.eastus.azurecontainer.io:8080/
```

Para ver el estado del flujo de trabajo y los resultados de cada paso en la UI de GitHub, consulte [Managing a workflow run](https://help.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run).

### <a name="validate-workflow"></a>Validación del flujo de trabajo

El flujo de trabajo implementa una instancia de Azure Container Instance con el nombre base del repositorio de GitHub, en este caso *acr-build-helloworld-node*. Cuando se complete el flujo de trabajo correctamente, obtenga información sobre la instancia de contenedor denominada *acr-build-helloworld-node* mediante la ejecución del comando [az container show][az-container-show]. Sustituya el nombre del grupo de recursos: 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name acr-build-helloworld-node \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

La salida es parecida a esta:

```console
FQDN                                                   ProvisioningState
---------------------------------                      -------------------
acr-build-helloworld-node.westus.azurecontainer.io     Succeeded
```

Una vez aprovisionada la instancia, navegue hasta el FQDN del contenedor en el explorador para ver la aplicación web en ejecución.

## <a name="clean-up-resources"></a>Limpieza de recursos

Detenga la instancia de contenedor con el comando [az container delete][az-container-delete]:

```azurecli
az container delete \
  --name <instance-name>
  --resource-group <resource-group-name>
```

Para eliminar el grupo de recursos y todos los recursos que contiene, ejecute el comando [az group delete][az-group-delete]:

```azurecli
az group delete \
  --name <resource-group-name>
```

## <a name="next-steps"></a>Pasos siguientes

Explore [GitHub Marketplace](https://github.com/marketplace?type=actions) para ver más acciones para automatizar el flujo de trabajo de desarrollo.


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->

[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-show]: /cli/azure/group#az-group-show
[az-group-delete]: /cli/azure/group#az-group-delete
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-container-create]: /cli/azure/container#az-container-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-container-show]: /cli/azure/container#az-container-show
[az-container-delete]: /cli/azure/container#az-container-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-container-app-up]: /cli/azure/ext/deploy-to-azure/container/app#ext-deploy-to-azure-az-container-app-up
