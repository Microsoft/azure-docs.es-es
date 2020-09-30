---
title: Uso de la identidad de AAD con el servicio web
titleSuffix: Azure Machine Learning
description: Use la identidad de AAD con su servicio web en Azure Kubernetes Service para acceder a los recursos en la nube durante la puntuación.
services: machine-learning
ms.author: larryfr
author: BlackMist
ms.reviewer: aashishb
ms.service: machine-learning
ms.subservice: core
ms.date: 02/10/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: ac7420e47077e4e2b5bcfce0f33766554cd5c76d
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89647330"
---
# <a name="use-azure-ad-identity-with-your-machine-learning-web-service-in-azure-kubernetes-service"></a>Uso de la identidad de Azure AD con el servicio web Machine Learning en Azure Kubernetes Service

En este procedimiento, aprenderá a asignar una identidad Azure Active Directory (AAD) al modelo de Machine Learning implementado en Azure Kubernetes Service. El proyecto [AAD Pod Identity](https://github.com/Azure/aad-pod-identity) permite a las aplicaciones acceder a los recursos en la nube de forma segura con AAD mediante una [identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) y primitivas de Kubernetes. Esto permite que el servicio web acceda de forma segura a los recursos de Azure sin tener que insertar credenciales ni administrar tokens directamente en el script `score.py`. En este artículo se explican los pasos para crear e instalar una identidad de Azure en el clúster de Azure Kubernetes Service y asignar la identidad al servicio web implementado.

## <a name="prerequisites"></a>Prerrequisitos

- La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) o la [extensión de Visual Studio Code para Azure Machine Learning](tutorial-setup-vscode-extension.md).

- Acceda al clúster de AKS mediante el comando `kubectl`. Para obtener más información, vea [Conexión al clúster](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough#connect-to-the-cluster).

- Un servicio web de Azure Machine Learning implementado en el clúster de AKS.

## <a name="create-and-install-an-azure-identity-in-your-aks-cluster"></a>Creación e instalación de una identidad de Azure en el clúster de AKS

1. Para determinar si el clúster de AKS está habilitado para RBAC, use el comando siguiente:

    ```azurecli-interactive
    az aks show --name <AKS cluster name> --resource-group <resource group name> --subscription <subscription id> --query enableRbac
    ```

    Este comando devuelve un valor de `true` si RBAC está habilitado. Este valor determina el comando que se va a usar en el paso siguiente.

1. Para instalar [AAD Pod Identity](https://github.com/Azure/aad-pod-identity#getting-started) en el clúster de AKS, use uno de los comandos siguientes:

    * Si el clúster de AKS tiene **RBAC habilitado**, use el comando siguiente:
    
        ```azurecli-interactive
        kubectl apply -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml
        ```
    
    * Si el clúster de AKS **no tiene RBAC habilitado**, use el comando siguiente:
    
        ```azurecli-interactive
        kubectl apply -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment.yaml
        ```
    
        La salida del comando es similar al texto siguiente:

        ```text
        customresourcedefinition.apiextensions.k8s.io/azureassignedidentities.aadpodidentity.k8s.io created
        customresourcedefinition.apiextensions.k8s.io/azureidentitybindings.aadpodidentity.k8s.io created
        customresourcedefinition.apiextensions.k8s.io/azureidentities.aadpodidentity.k8s.io created
        customresourcedefinition.apiextensions.k8s.io/azurepodidentityexceptions.aadpodidentity.k8s.io created
        daemonset.apps/nmi created
        deployment.apps/mic created
        ```

1. [Cree una identidad de Azure](https://github.com/Azure/aad-pod-identity#2-create-an-azure-identity) siguiendo los pasos que se indican en la página del proyecto AAD Pod Identity.

1. [Instale la identidad de Azure](https://github.com/Azure/aad-pod-identity#3-install-the-azure-identity) siguiendo los pasos que se indican en la página del proyecto AAD Pod Identity.

1. [Instale el enlace de la identidad de Azure](https://github.com/Azure/aad-pod-identity#5-install-the-azure-identity-binding) siguiendo los pasos que se indican en la página del proyecto AAD Pod Identity.

1. Si la identidad de Azure creada en el paso anterior no se encuentra en el mismo grupo de recursos que el clúster de AKS, realice el procedimiento de [Definición de permisos para MIC](https://github.com/Azure/aad-pod-identity#6-set-permissions-for-mic) tras los pasos que se indican en la página del proyecto AAD Pod Identity.

## <a name="assign-azure-identity-to-machine-learning-web-service"></a>Asignación de la identidad de Azure al servicio web Machine Learning

En los pasos siguientes se usa la identidad de Azure creada en la sección anterior y se asigna a su servicio web de AKS a través de una **etiqueta selector**.

En primer lugar, identifique el nombre y el espacio de nombres de su implementación en el clúster de AKS que desea asignar a la identidad de Azure. Para obtener esta información, ejecute el comando siguiente. Los espacios de nombres deben ser el nombre del área de trabajo de Azure Machine Learning y el nombre de la implementación debe ser el nombre del punto de conexión, tal como se muestra en el portal.

```azurecli-interactive
kubectl get deployment --selector=isazuremlapp=true --all-namespaces --show-labels
```

Agregue la etiqueta selector de la identidad de Azure a su implementación. Para ello, edite la especificación de implementación. El valor de selector debe ser el que definió en el paso 5 de [Instalación del enlace de la identidad de Azure](https://github.com/Azure/aad-pod-identity#5-install-the-azure-identity-binding).

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: demo1-azure-identity-binding
spec:
  AzureIdentity: <a-idname>
  Selector: <label value to match>
```

Edite la implementación para agregar la etiqueta selector de la identidad de Azure. Diríjase a la siguiente sección en `/spec/template/metadata/labels`. Debería ver valores como `isazuremlapp: “true”`. Agregue la etiqueta add-pod-identity como se muestra a continuación.

```azurecli-interactive
    kubectl edit deployment/<name of deployment> -n azureml-<name of workspace>
```

```yaml
spec:
  template:
    metadata:
      labels:
       aadpodidbinding: "<value of Selector in AzureIdentityBinding>"
      ...
```

Para comprobar que la etiqueta se ha agregado correctamente, ejecute el siguiente comando.

```azurecli-interactive
   kubectl get deployment <name of deployment> -n azureml-<name of workspace> --show-labels
```

Para ver los estados de todos los pods, ejecute el siguiente comando.

```azurecli-interactive
    kubectl get pods -n azureml-<name of workspace>
```

Una vez que los pods estén en funcionamiento, los servicios web de esta implementación podrán acceder a los recursos de Azure a través de su identidad de Azure sin tener que insertar las credenciales en el código. 

## <a name="assign-the-appropriate-roles-to-your-azure-identity"></a>Asignación de los roles adecuados a la identidad de Azure

[Asigne su identidad administrada de Azure con los roles adecuados](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal) para acceder a otros recursos de Azure. Asegúrese de que los roles que está asignando tienen las **acciones de datos** correctas. Por ejemplo, el [rol de lector de datos de blobs de almacenamiento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-reader) tendrá permisos de lectura para el blob de almacenamiento, mientras que el [rol de lector](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#reader) genérico podría no tenerlos.

## <a name="use-azure-identity-with-your-machine-learning-web-service"></a>Uso de la identidad de Azure con el servicio web Machine Learning

Implemente un modelo en el clúster de AKS. El script `score.py` puede contener operaciones que apunten a los recursos de Azure a los que tiene acceso su identidad de Azure. Asegúrese de que ha instalado las dependencias de la biblioteca de cliente necesarias para el recurso al que intenta acceder. A continuación se muestran un par de ejemplos de cómo puede usar su identidad de Azure para acceder a distintos recursos de Azure desde su servicio.

### <a name="access-key-vault-from-your-web-service"></a>Acceso a Key Vault desde el servicio web

Si ha concedido a su identidad de Azure acceso de lectura a un secreto en una instancia de **Key Vault**, el script `score.py` puede acceder a este mediante el código siguiente.

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

my_vault_name = "yourkeyvaultname"
my_vault_url = "https://{}.vault.azure.net/".format(my_vault_name) 
my_secret_name = "sample-secret"

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
secret_client = SecretClient(
    vault_url=my_vault_url,
    credential=credential)
secret = secret_client.get_secret(my_secret_name)
```

> [!IMPORTANT]
> En este ejemplo se usa DefaultAzureCredential. Para conceder acceso a su identidad mediante una directiva de acceso específica, consulte [Asignación de una directiva de acceso de Key Vault mediante la CLI de Azure](/azure/key-vault/general/assign-access-policy-cli).

### <a name="access-blob-from-your-web-service"></a>Acceder al blob desde el servicio web

Si ha concedido a su identidad de Azure acceso de lectura a los datos de un **blob de almacenamiento**, el script `score.py` puede acceder a este mediante el código siguiente.

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

my_storage_account_name = "yourstorageaccountname"
my_storage_account_url = "https://{}.blob.core.windows.net/".format(my_storage_account_name)

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
blob_service_client = BlobServiceClient(
    account_url=my_storage_account_url,
    credential=credential
)
blob_client = blob_service_client.get_blob_client(container="some-container", blob="some_text.txt")
blob_data = blob_client.download_blob()
blob_data.readall()
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo usar la biblioteca de cliente de la identidad de Azure para Python, consulte el [repositorio](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/identity/azure-identity#azure-identity-client-library-for-python) en GitHub.
* Para obtener una guía detallada sobre la implementación de modelos en los clústeres de Azure Kubernetes Service, consulte la [guía paso a paso](how-to-deploy-azure-kubernetes-service.md).
