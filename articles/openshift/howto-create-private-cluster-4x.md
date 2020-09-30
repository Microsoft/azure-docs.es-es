---
title: Creación de un clúster privado de la versión 4 de Red Hat OpenShift en Azure
description: Aprenda a crear un clúster privado de Red Hat OpenShift en Azure en el que se ejecute OpenShift 4
ms.service: container-service
ms.topic: article
ms.date: 03/12/2020
author: ms-jasondel
ms.author: jasondel
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 11343ba668a4b74c436313f0abd4daed577c36d4
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505359"
---
# <a name="create-an-azure-red-hat-openshift-4-private-cluster"></a>Creación de un clúster privado de la versión 4 de Red Hat OpenShift en Azure

En este artículo, preparará su entorno para crear clústeres privados de Red Hat OpenShift en Azure que ejecutan OpenShift 4. Aprenderá a:

> [!div class="checklist"]
> * Configurar los requisitos previos y crear la red virtual y las subredes necesarias.
> * Implementar un clúster con un punto de conexión de servidor de API privado y un controlador de entrada privado.

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure, versión 2.6.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Antes de empezar

### <a name="register-the-resource-providers"></a>Registro de los proveedores de recursos

1. Si tiene varias suscripciones de Azure, especifique el identificador de la relevante:

    ```azurecli-interactive
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Registre el proveedor de recursos `Microsoft.RedHatOpenShift`:

    ```azurecli-interactive
    az provider register -n Microsoft.RedHatOpenShift --wait
    ```

1. Registre el proveedor de recursos `Microsoft.Compute`:

    ```azurecli-interactive
    az provider register -n Microsoft.Compute --wait
    ```

1. Registre el proveedor de recursos `Microsoft.Storage`:

    ```azurecli-interactive
    az provider register -n Microsoft.Storage --wait
    ```

### <a name="get-a-red-hat-pull-secret-optional"></a>Obtención de un secreto de extracción de Red Hat (opcional)

Los secretos de extracción de Red Hat permiten al clúster acceder a los registros de contenedor de Red Hat junto con contenido adicional. Este paso es opcional pero recomendable.

1. **[Vaya al portal del administrador de clústeres de Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) e inicie sesión.**

   Tendrá que iniciar sesión en su cuenta de Red Hat, o bien crear una cuenta de Red Hat con su correo electrónico empresarial y aceptar los términos y condiciones.

2. **Haga clic en Download pull secret** (Descargar secreto de incorporación de cambios).

Mantenga el archivo `pull-secret.txt` guardado en un lugar seguro, ya que se usará cada vez que se cree un clúster.

Al ejecutar el comando `az aro create`, puede hacer referencia al secreto de incorporación de cambios mediante el parámetro `--pull-secret @pull-secret.txt`. Ejecute `az aro create` desde el directorio donde haya almacenado el archivo `pull-secret.txt`. De lo contrario, reemplace `@pull-secret.txt` por `@<path-to-my-pull-secret-file`.

Si va a copiar el secreto de incorporación de cambios o hacer referencia a él en otros scripts, debe tener el formato de una cadena JSON válida.

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Creación de una red virtual que contenga dos subredes vacías

A continuación, creará una red virtual que contenga dos subredes vacías.

1. **Establezca las siguientes variables.**

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP="v4-$LOCATION"    # the name of the resource group where you want to create your cluster
   CLUSTER=aro-cluster             # the name of your cluster
   ```

1. **Cree un grupo de recursos**

    Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando se crea un grupo de recursos, se le pide que especifique una ubicación. Dicha ubicación es donde se almacenan los metadatos del grupo de recursos, así como el lugar en el que los recursos se ejecutan en Azure si no se especifica otra región al crear los recursos. Cree un grupo de recursos mediante el comando [az group create][az-group-create].

    ```azurecli-interactive
    az group create --name $RESOURCEGROUP --location $LOCATION
    ```

    En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

    ```json
    {
    "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
    "location": "eastus",
    "managedBy": null,
    "name": "aro-rg",
    "properties": {
        "provisioningState": "Succeeded"
    },
    "tags": null
    }
    ```

2. **Cree una red virtual.**

    Los clústeres de Red Hat OpenShift en Azure que ejecutan OpenShift 4 requieren una red virtual con dos subredes vacías, una para los nodos maestros y otra para los nodos de trabajo.

    Cree una red virtual en el mismo grupo de recursos que creó anteriormente.

    ```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22
    ```

    En la siguiente salida de ejemplo se muestra la red virtual creada correctamente:

    ```json
    {
    "newVNet": {
        "addressSpace": {
        "addressPrefixes": [
            "10.0.0.0/22"
        ]
        },
        "id": "/subscriptions/<guid>/resourceGroups/aro-rg/providers/Microsoft.Network/virtualNetworks/aro-vnet",
        "location": "eastus",
        "name": "aro-vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "aro-rg",
        "type": "Microsoft.Network/virtualNetworks"
    }
    }
    ```

3. **Agregue una subred vacía para los nodos maestros.**

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

4. **Agregue una subred vacía para los nodos de trabajo.**

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

5. **[Deshabilite las directivas de punto de conexión privado](../private-link/disable-private-link-service-network-policy.md) en la subred maestra.** Esto es necesario para poder conectarse al clúster y administrarlo.

    ```azurecli-interactive
    az network vnet subnet update \
    --name master-subnet \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --disable-private-link-service-network-policies true
    ```

## <a name="create-the-cluster"></a>Creación del clúster

Ejecute el siguiente comando para crear un clúster. También puede [pasar el secreto de incorporación de cambios de Red Hat](#get-a-red-hat-pull-secret-optional), que permite al clúster acceder a los registros de contenedor de Red Hat junto con contenido adicional.

>[!NOTE]
> Si va a copiar o pegar comandos, y usa uno de los parámetros opcionales, asegúrese de eliminar los hashtags iniciales y el texto de comentario final. Además, cierre el argumento de la línea anterior del comando con una barra diagonal inversa al final.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  --apiserver-visibility Private \
  --ingress-visibility Private
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

Después de ejecutar el comando `az aro create`, se tarda aproximadamente 35 minutos en crear un clúster.

>[!IMPORTANT]
> Si elige especificar un dominio personalizado, por ejemplo **foo.example.com**, la consola de OpenShift estará disponible en una dirección URL como `https://console-openshift-console.apps.foo.example.com`, en lugar del dominio integrado `https://console-openshift-console.apps.<random>.<location>.aroapp.io`.
>
> De forma predeterminada, OpenShift usa certificados autofirmados para todas las rutas creadas en `*.apps.<random>.<location>.aroapp.io`.  Si elige Custom DNS (DNS personalizado), después de conectarse al clúster, tendrá que seguir la documentación de OpenShift para [configurar una entidad de certificación personalizada para el controlador de entrada ](https://docs.openshift.com/container-platform/4.3/authentication/certificates/replacing-default-ingress-certificate.html) y [una entidad de certificación personalizada para el servidor de API](https://docs.openshift.com/container-platform/4.3/authentication/certificates/api-server.html).

## <a name="connect-to-the-private-cluster"></a>Conexión al clúster privado

Puede iniciar sesión en el clúster con el usuario `kubeadmin`.  Ejecute el siguiente comando para buscar la contraseña del usuario `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

En la siguiente salida de ejemplo se muestra que la contraseña estará en `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Para encontrar la dirección URL de la consola del clúster, ejecute el siguiente comando, que será similar a `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

>[!IMPORTANT]
> Para conectarse a un clúster privado de Red Hat OpenShift en Azure, tendrá que realizar el siguiente paso desde un host que esté en la red virtual que ha creado o en una red virtual que esté [emparejada](../virtual-network/virtual-network-peering-overview.md) con la red virtual en la que se implementó el clúster.

Inicie la dirección URL de la consola en un explorador e inicie sesión con las credenciales de `kubeadmin`.

![Pantalla de inicio de sesión de Red Hat OpenShift en Azure](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Instalación de la CLI de OpenShift

Cuando haya iniciado sesión en la consola web de OpenShift, haga clic en el signo **?** en la parte superior derecha y, luego, en **Command Line Tools** (Herramientas de línea de comandos). Descargue la versión adecuada para su máquina.

![Pantalla de inicio de sesión de Red Hat OpenShift en Azure](media/aro4-download-cli.png)

También puede descargar la versión más reciente de la CLI adecuada en su máquina desde <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

## <a name="connect-using-the-openshift-cli"></a>Conexión mediante la CLI de OpenShift

Recupere la dirección del servidor de API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

>[!IMPORTANT]
> Para conectarse a un clúster privado de Red Hat OpenShift en Azure, tendrá que realizar el siguiente paso desde un host que se encuentre en la red virtual que ha creado o en una red virtual que esté [emparejada](../virtual-network/virtual-network-peering-overview.md) con la red virtual en la que se implementó el clúster.

Inicie sesión en el servidor de API del clúster de OpenShift mediante el siguiente comando. Reemplace **\<kubeadmin password>** por la contraseña que acaba de recuperar.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se ha implementado un clúster de Red Hat OpenShift en Azure que ejecuta OpenShift 4. Ha aprendido a:

> [!div class="checklist"]
> * Configurar los requisitos previos y crear la red virtual y las subredes necesarias.
> * Implementación de un clúster
> * Conectarse al clúster mediante el usuario `kubeadmin`.

Pase al siguiente artículo, donde aprenderá a configurar el clúster para la autenticación mediante Azure Active Directory.


* [Configuración la autenticación con Azure Active Directory desde la línea de comandos](configure-azure-ad-cli.md)


* [Configuración la autenticación con Azure Active Directory desde Azure Portal y la consola web de OpenShift](configure-azure-ad-cli.md)