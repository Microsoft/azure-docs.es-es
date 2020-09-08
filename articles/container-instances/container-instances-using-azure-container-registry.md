---
title: Implementación de una imagen de contenedor de Azure Container Registry
description: Obtenga información acerca de cómo implementar contenedores en Azure Container Instances mediante la extracción de imágenes de contenedor de Azure Container Registry.
services: container-instances
ms.topic: article
ms.date: 07/02/2020
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 0a997733e015a9f65b59ffc99cc137dae3d2d62a
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927443"
---
# <a name="deploy-to-azure-container-instances-from-azure-container-registry"></a>Implementación en Azure Container Instances desde Azure Container Registry

[Azure Container Registry](../container-registry/container-registry-intro.md) es un servicio de registro de contenedores administrado basado en Azure que se usa para almacenar imágenes de contenedor de Docker privadas. En este artículo se describe cómo extraer imágenes de contenedor almacenadas en una instancia de Azure Container Registry al implementarlas en Azure Container Instances. Un método recomendado para configurar el acceso al registro consiste en crear una entidad de servicio y una contraseña de Azure Active Directory, así como almacenar las credenciales de inicio de sesión en Azure Key Vault.

## <a name="prerequisites"></a>Prerrequisitos

**Azure Container Registry**: se necesita una instancia de Azure Container Registry, y al menos una imagen de contenedor en el registro, para completar los pasos descritos en este artículo. Si necesita un registro, consulte [Creación de un registro de contenedor con la CLI de Azure](../container-registry/container-registry-get-started-azure-cli.md).

**CLI de Azure**: los ejemplos de línea de comandos de este artículo usan la [CLI de Azure](/cli/azure/) y tienen un formato adecuado para el shell de Bash. Puede [instalar la CLI de Azure](/cli/azure/install-azure-cli) localmente o usar [Azure Cloud Shell][cloud-shell-bash].

## <a name="configure-registry-authentication"></a>Configurar la autenticación del registro

En un escenario de producción donde se brinda acceso a servicios y aplicaciones "sin periféricos", se recomienda configurar el acceso al registro mediante una [entidad de servicio](../container-registry/container-registry-auth-service-principal.md). Una entidad de servicio permite proporcionar el [control de acceso basado en rol (RBAC de Azure)](../container-registry/container-registry-roles.md) a las imágenes de contenedor. Por ejemplo, puede configurar una entidad de servicio con acceso de solo extracción a un registro.

Azure Container Registry ofrece [opciones de autenticación](../container-registry/container-registry-authentication.md) adicionales.

> [!NOTE]
> No se puede autenticar en Azure Container Registry para extraer imágenes durante la implementación del grupo de contenedores mediante una [identidad administrada](container-instances-managed-identity.md) configurada en el mismo grupo de contenedores.

En la siguiente sección, cree un almacén de claves de Azure y una entidad de servicio, y almacene las credenciales de la entidad de servicio en el almacén. 

### <a name="create-key-vault"></a>Creación del almacén de claves

Si todavía no tiene un almacén en [Azure Key Vault](../key-vault/general/overview.md), cree uno con la CLI de Azure mediante los siguientes comandos.

Actualice la variable `RES_GROUP` con el nombre de un grupo de recursos existente en el que crear el almacén de claves, y la variable `ACR_NAME` con el nombre de su registro de contenedor. Para abreviar, los comandos de este artículo suponen que el registro, el almacén de claves y las instancias de contenedor se han creado en el mismo grupo de recursos.

 Especifique un nombre para el nuevo almacén de claves en `AKV_NAME`. El nombre del almacén debe ser exclusivo en Azure, tener entre 3 y 24 caracteres alfanuméricos, comenzar con una letra y terminar con una letra o un número y no puede contener guiones consecutivos.

```azurecli
RES_GROUP=myresourcegroup # Resource Group name
ACR_NAME=myregistry       # Azure Container Registry registry name
AKV_NAME=mykeyvault       # Azure Key Vault vault name

az keyvault create -g $RES_GROUP -n $AKV_NAME
```

### <a name="create-service-principal-and-store-credentials"></a>Creación de la entidad de servicio y almacenamiento de credenciales

Ahora cree una entidad de servicio y almacene sus credenciales en el almacén de claves.

El siguiente comando usa [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] para crear la entidad de servicio y [az keyvault secret set][az-keyvault-secret-set] para almacenar la **contraseña** de la entidad de servicio en el almacén.

```azurecli
# Create service principal, store its password in vault (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name http://$ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

El argumento `--role` en el comando anterior configura la entidad de servicio con el rol *acrpull*, que le concede de solo extracción acceso al registro. Para conceder acceso de inserción y extracción, cambie el argumento `--role` a *acrpush*.

A continuación, almacene el *appId* de la entidad de servicio en el almacén, que es el **nombre de usuario** que pasa a Azure Container Registry para la autenticación.

```azurecli
# Store service principal ID in vault (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

Ha creado una instancia de Azure Key Vault y almacenado dos secretos allí:

* `$ACR_NAME-pull-usr`: el identificador de la entidad de servicio, para su uso como **nombre de usuario** del registro de contenedor.
* `$ACR_NAME-pull-pwd`: la contraseña de la entidad de servicio, para su uso como **contraseña** del registro de contenedor.

Ahora puede hacer referencia a estos secretos por nombre cuando usted o sus aplicaciones y servicios extraigan imágenes del registro.

## <a name="deploy-container-with-azure-cli"></a>Implementación de un contenedor con la CLI de Azure

Ahora que las credenciales de la entidad de servicio están almacenadas en secretos de Azure Key Vault, las aplicaciones y servicios pueden utilizarlas para acceder a su registro privado.

Obtenga primero el nombre del servidor de inicio de sesión del Registro con el comando [az acr show][az-acr-show]. El nombre del servidor de inicio de sesión se escribe todo en minúsculas y es similar a `myregistry.azurecr.io`.

```azurecli
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --resource-group $RES_GROUP --query "loginServer" --output tsv)
```

Ejecute el comando [az container create][az-container-create] siguiente para implementar una instancia de contenedor. El comando usa las credenciales de la entidad de servicio almacenadas en Azure Key Vault para autenticarse en el registro de contenedor y da por supuesto que ha insertado previamente la imagen [aci-helloworld](container-instances-quickstart.md) en el registro. Actualice el valor `--image` si desea usar otra imagen del registro.

```azurecli
az container create \
    --name aci-demo \
    --resource-group $RES_GROUP \
    --image $ACR_LOGIN_SERVER/aci-helloworld:v1 \
    --registry-login-server $ACR_LOGIN_SERVER \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label aci-demo-$RANDOM \
    --query ipAddress.fqdn
```

El valor `--dns-name-label` debe ser único en Azure, por lo que el comando anterior anexa un número aleatorio a la etiqueta de nombre DNS del contenedor. La salida del comando muestra el nombre de dominio completo (FQDN) del contenedor, por ejemplo:

```output
"aci-demo-25007.eastus.azurecontainer.io"
```

Una vez que se ha iniciado correctamente el contenedor, puede desplazarse a su FQDN en el explorador para comprobar que la aplicación se ejecuta correctamente.

## <a name="deploy-with-azure-resource-manager-template"></a>Implementación con plantillas de Azure Resource Manager

Puede especificar las propiedades de Azure Container Registry en una plantilla de Azure Resource Manager; para ello, debe incluir la propiedad `imageRegistryCredentials` en la definición del grupo de contenedores. Por ejemplo, puede especificar las credenciales del registro directamente:

```JSON
[...]
"imageRegistryCredentials": [
  {
    "server": "imageRegistryLoginServer",
    "username": "imageRegistryUsername",
    "password": "imageRegistryPassword"
  }
]
[...]
```

Para obtener la configuración de grupo de contenedores completa, consulte la [referencia de plantillas de Resource Manager](/azure/templates/Microsoft.ContainerInstance/2019-12-01/containerGroups).    

Para más información acerca de cómo hacer referencia a secretos de Azure Key Vault en una plantilla de Resource Manager, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](../azure-resource-manager/templates/key-vault-parameter.md).

## <a name="deploy-with-azure-portal"></a>Implementación con Azure Portal

Si se mantienen imágenes de contenedor en un registro de contenedor de Azure, se puede crear fácilmente un contenedor en Azure Container Instances a través de Azure Portal. Al usar el portal para implementar una instancia de contenedor desde un registro de contenedor, tiene que habilitar la [cuenta de administrador](../container-registry/container-registry-authentication.md#admin-account) del registro. La cuenta de administrador está diseñada para que un solo usuario acceda al registro, principalmente con fines de prueba. 

1. En Azure Portal, vaya al registro de contenedor.

1. Para confirmar que la cuenta de administrador está habilitada, seleccione **Claves de acceso**y en **Usuario administrador** seleccione **Habilitar**.

1. Seleccione **Repositorios**, elija el repositorio desde el que va a implementar, haga clic con el botón derecho en la etiqueta de la imagen de contenedor que desea implementar y seleccione **Ejecutar instancia**.

    !["Ejecutar instancia" en Azure Container Registry de Azure Portal][acr-runinstance-contextmenu]

1. Escriba un nombre para el contenedor y un nombre para el grupo de recursos. También puede cambiar los valores predeterminados si lo desea.

    ![Menú Crear para Azure Container Instances][acr-create-deeplink]

1. Una vez completada la implementación, puede ir al grupo de contenedores desde el panel de notificaciones para buscar la dirección IP y otras propiedades.

    ![Vista de detalles del grupo de contenedores de Azure Container Instances][aci-detailsview]

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de la autenticación de Azure Container Registry, consulte [Autenticación con un registro de contenedor de Azure](../container-registry/container-registry-authentication.md).

<!-- IMAGES -->
[acr-create-deeplink]: ./media/container-instances-using-azure-container-registry/acr-create-deeplink.png
[aci-detailsview]: ./media/container-instances-using-azure-container-registry/aci-detailsview.png
[acr-runinstance-contextmenu]: ./media/container-instances-using-azure-container-registry/acr-runinstance-contextmenu.png

<!-- LINKS - External -->
[cloud-shell-bash]: https://shell.azure.com/bash
[cloud-shell-try-it]: https://shell.azure.com/powershell

<!-- LINKS - Internal -->
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-create]: /cli/azure/container#az-container-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
