---
title: Integración de Azure Key Vault con Kubernetes
description: En este tutorial, accederá a los secretos de Azure Key Vault y los recuperará mediante el controlador Secrets Store Container Storage Interface (CSI) para montarlos en los pods de Kubernetes.
author: ShaneBala-keyvault
ms.author: sudbalas
ms.service: key-vault
ms.topic: tutorial
ms.date: 08/25/2020
ms.openlocfilehash: bfcaf9d4b1d03457f2e4cddd2e0eaf9d9d58eee2
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869191"
---
# <a name="tutorial-configure-and-run-the-azure-key-vault-provider-for-the-secrets-store-csi-driver-on-kubernetes"></a>Tutorial: Configuración y ejecución del proveedor de Azure Key Vault para el controlador Secrets Store CSI en Kubernetes

> [!IMPORTANT]
> El controlador de CSI es un proyecto de código abierto que no es compatible con el soporte técnico de Azure. Informe de todos los comentarios y problemas relacionados con la integración de Key Vault para el controlador de CSI en el vínculo de GitHub en la parte inferior de la página. Esta herramienta se proporciona para que los usuarios se ocupen por sí mismos de su instalación en los clústeres y recopilen comentarios de nuestra comunidad.

En este tutorial, accederá a los secretos de Azure Key Vault y los recuperará mediante el controlador Secrets Store Container Storage Interface (CSI) para montar los secretos en los pods de Kubernetes.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una entidad de servicio o usar identidades administradas.
> * Implementar un clúster de Azure Kubernetes Service (AKS) mediante la CLI de Azure.
> * Instalar Helm y el controlador Secrets Store CSI.
> * Crear una instancia de Azure Key Vault y establecer secretos.
> * Crear su propio objeto SecretProviderClass.
> * Asignar la entidad de servicio o usar identidades administradas.
> * Implementar el pod con los secretos montados desde Key Vault.

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* Antes de iniciar este tutorial, instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest).

## <a name="create-a-service-principal-or-use-managed-identities"></a>Creación de una entidad de servicio o uso de identidades administradas

Si tiene planea usar identidades administradas, puede pasar a la sección siguiente.

Cree una entidad de servicio para controlar los recursos a los que se puede acceder desde Azure Key Vault. El acceso de esta entidad de servicio está restringido por los roles que se le hayan asignado. Esta característica permite controlar el modo en que la entidad de servicio puede administrar sus secretos. En el ejemplo siguiente, el nombre de la entidad de servicio es *contosoServicePrincipal*.

```azurecli
az ad sp create-for-rbac --name contosoServicePrincipal --skip-assignment
```
Esta operación devuelve una serie de pares clave-valor:

![Captura de pantalla que muestra el identificador de aplicación y la contraseña para contosoServicePrincipal](../media/kubernetes-key-vault-1.png)

Copie las credenciales **appId** y **password** para su uso posterior.

## <a name="deploy-an-azure-kubernetes-service-aks-cluster-by-using-the-azure-cli"></a>Implementación de un clúster de Azure Kubernetes Service (AKS) mediante la CLI de Azure

No es necesario utilizar Azure Cloud Shell. El símbolo del sistema (terminal) con la CLI de Azure instalada será suficiente. 

Complete las secciones "Creación de un grupo de recursos", "Creación de un clúster de AKS" y "Conexión con el clúster" de [Implementación de un clúster de Azure Kubernetes Service mediante la CLI de Azure](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough). 

> [!NOTE] 
> Si planea usar una identidad de pod en lugar de una entidad de servicio, asegúrese de habilitarla al crear el clúster de Kubernetes, como se muestra en el siguiente comando:
>
> ```azurecli
> az aks create -n contosoAKSCluster -g contosoResourceGroup --kubernetes-version 1.16.9 --node-count 1 --enable-managed-identity
> ```

1. [Establezca la variable de entorno PATH](https://www.java.com/en/download/help/path.xml) en el archivo *kubectl.exe* que descargó.
1. Compruebe la versión de Kubernetes con el siguiente comando, que genera la versión del cliente y el servidor. La versión del cliente es el archivo *kubectl.exe* que instaló y la versión del servidor es la instancia de Azure Kubernetes Services (AKS) en la que se ejecuta el clúster.
    ```azurecli
    kubectl version
    ```
1. Asegúrese de que la versión de Kubernetes sea la 1.16.0 o posterior. El comando siguiente actualiza tanto el clúster de Kubernetes como el grupo de nodos. El comando puede tardar un par de minutos en ejecutarse. En este ejemplo, el grupo de recursos es *contosoResourceGroup* y el clúster de Kubernetes es *contosoAKSCluster*.
    ```azurecli
    az aks upgrade --kubernetes-version 1.16.9 --name contosoAKSCluster --resource-group contosoResourceGroup
    ```
1. Para mostrar los metadatos del clúster de AKS que ha creado, utilice el comando siguiente. Copie los valores de **principalId**, **clientId**, **subscriptionId**y **nodeResourceGroup** para su uso posterior. Si el clúster de ASK no se creó con las identidades administradas habilitadas, los identificadores **principalId** y **clientId** serán null. 

    ```azurecli
    az aks show --name contosoAKSCluster --resource-group contosoResourceGroup
    ```

    La salida muestra ambos parámetros resaltados:
    
    ![Captura de pantalla de la CLI de Azure con los valores de principalId y clientId resaltados](../media/kubernetes-key-vault-2.png) ![Captura de pantalla de la CLI de Azure con los valores de subscriptionId y nodeResourceGroup resaltados](../media/kubernetes-key-vault-3.png)
    
## <a name="install-helm-and-the-secrets-store-csi-driver"></a>Instalación de Helm y el controlador Secrets Store CSI

Para instalar el controlador Secrets Store CSI, primero debe instalar [Helm](https://helm.sh/docs/intro/install/).

Con la interfaz del controlador [Secrets Store CSI](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/charts/csi-secrets-store-provider-azure/README.md) puede obtener los secretos almacenados en la instancia de Azure Key Vault y usar la interfaz del controlador para montar el contenido de los secretos en los pods de Kubernetes.

1. Asegúrese de que la versión de Helm sea la v3 o posterior:
    ```azurecli
    helm version
    ```
1. Instale el controlador Secrets Store CSI y el proveedor de Azure Key Vault para el controlador:
    ```azurecli
    helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts

    helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
    ```

## <a name="create-an-azure-key-vault-and-set-your-secrets"></a>Creación de una instancia de Azure Key Vault y establecimiento de secretos

Para crear su propio almacén de claves y establecer los secretos, siga las instrucciones de [Establecimiento y recuperación de un secreto desde Azure Key Vault mediante la CLI de Azure](https://docs.microsoft.com/azure/key-vault/secrets/quick-create-cli).

> [!NOTE] 
> No es necesario usar Azure Cloud Shell ni crear un nuevo grupo de recursos. Puede usar el grupo de recursos creado anteriormente para el clúster de Kubernetes.

## <a name="create-your-own-secretproviderclass-object"></a>Creación de su propio objeto SecretProviderClass

[Use esta plantilla](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/test/bats/tests/azure_v1alpha1_secretproviderclass.yaml) para crear su propio objeto SecretProviderClass personalizado con parámetros específicos del proveedor para el controlador Secrets Store CSI. Este objeto proporcionará acceso de identidad a Key Vault.

En el archivo YAML SecretProviderClass de ejemplo, rellene los parámetros que faltan. Se necesitan los siguientes parámetros:

* **userAssignedIdentityID**: identificador de cliente de la entidad de servicio.
* **keyvaultName**: nombre del almacén de claves.
* **objects**: contenedor para todo el contenido secreto que desea montar.
    * **objectName**: nombre del contenido del secreto.
    * **objectType**: tipo de objeto (secreto, clave o certificado).
* **resourceGroup**: nombre del grupo de recursos.
* **subscriptionId**: identificador de la suscripción del almacén de claves.
* **tenantID**: identificador de inquilino o identificador de directorio del almacén de claves.

La plantilla actualizada se muestra en el código siguiente. Descárguela como un archivo YAML y rellene los campos obligatorios. En este ejemplo, el almacén de claves es **contosoKeyVault5**. Tiene dos secretos, **secret1** y **secret2**.

> [!NOTE] 
> Si utiliza identidades administradas, establezca el valor de **usePodIdentity** en *true* y establezca el valor de **userAssignedIdentityID** como un par de comillas ( **""** ). 

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"                   # [REQUIRED] Set to "true" if using managed identities
    useVMManagedIdentity: "false"             # [OPTIONAL] if not provided, will default to "false"
    userAssignedIdentityID: "servicePrincipalClientID"       # [REQUIRED] If you're using a service principal, use the client id to specify which user-assigned managed identity to use. If you're using a user-assigned identity as the VM's managed identity, specify the identity's client id. If the value is empty, it defaults to use the system-assigned identity on the VM
                                                             #     az ad sp show --id http://contosoServicePrincipal --query appId -o tsv
                                                             #     the preceding command will return the client ID of your service principal
    keyvaultName: "contosoKeyVault5"          # [REQUIRED] the name of the key vault
                                              #     az keyvault show --name contosoKeyVault5
                                              #     the preceding command will display the key vault metadata, which includes the subscription ID, resource group name, key vault 
    cloudName: ""                                # [OPTIONAL for Azure] if not provided, Azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1                 # [REQUIRED] object name
                                              #     az keyvault secret list --vault-name "contosoKeyVault5"
                                              #     the above command will display a list of secret names from your key vault
          objectType: secret                  # [REQUIRED] object types: secret, key, or cert
          objectVersion: ""                   # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: secret2
          objectType: secret
          objectVersion: ""
    resourceGroup: "contosoResourceGroup"     # [REQUIRED] the resource group name of the key vault
    subscriptionId: "subscriptionID"          # [REQUIRED] the subscription ID of the key vault
    tenantId: "tenantID"                      # [REQUIRED] the tenant ID of the key vault
```
En la imagen siguiente se muestra la salida de consola de **az keyvault show --name contosoKeyVault5** con los metadatos pertinentes resaltados:

![Captura de pantalla que muestra la salida de consola de "az keyvault show --name contosoKeyVault5"](../media/kubernetes-key-vault-4.png)

## <a name="assign-your-service-principal-or-use-managed-identities"></a>Asignación de una entidad de servicio o uso de identidades administradas

### <a name="assign-a-service-principal"></a>Asignación de una entidad de servicio

Si usa una entidad de servicio, deberá conceder permisos para acceder al almacén de claves y recuperar los secretos. Asigne el rol *Lector* y conceda a la entidad de servicio permisos para *obtener* los secretos del almacén de claves; para ello, debe ejecutar este comando:

1. Asigne la entidad de servicio al almacén de claves existente. El parámetro **$AZURE_CLIENT_ID** es el valor de **appId** que copió después de crear la entidad de servicio.
    ```azurecli
    az role assignment create --role Reader --assignee $AZURE_CLIENT_ID --scope /subscriptions/$SUBID/resourcegroups/$KEYVAULT_RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$KEYVAULT_NAME
    ```

    La salida del comando se muestra en la siguiente imagen: 

    ![Captura de pantalla que muestra el valor de principalId](../media/kubernetes-key-vault-5.png)

1. Conceda a la entidad de servicio permisos para obtener secretos:
    ```azurecli
    az keyvault set-policy -n $KEYVAULT_NAME --secret-permissions get --spn $AZURE_CLIENT_ID
    ```

1. Ya ha configurado la entidad de servicio con permisos para leer secretos desde el almacén de claves. **$AZURE_CLIENT_SECRET** es la contraseña de la entidad de servicio. Agregue las credenciales de la entidad de servicio como un secreto de Kubernetes al que pueda acceder el controlador Secrets Store CSI:
    ```azurecli
    kubectl create secret generic secrets-store-creds --from-literal clientid=$AZURE_CLIENT_ID --from-literal clientsecret=$AZURE_CLIENT_SECRET
    ```

> [!NOTE] 
> Si va a implementar el pod de Kubernetes y recibe un error sobre un identificador de secreto de cliente no válido, es posible que tenga un identificador de secreto de cliente anterior que haya expirado o se haya restablecido. Para resolver este problema, elimine el secreto *secrets-store-creds* y cree uno nuevo con el identificador de secreto de cliente actual. Ejecute el siguiente comando para eliminar *secrets-store-creds*:
>
> ```azurecli
> kubectl delete secrets secrets-store-creds
> ```

Si olvidó el identificador de secreto de cliente de la entidad de servicio, puede restablecerlo con el siguiente comando:

```azurecli
az ad sp credential reset --name contosoServicePrincipal --credential-description "APClientSecret" --query password -o tsv
```

### <a name="use-managed-identities"></a>Uso de identidades administradas

Si usa identidades administradas, asigne roles específicos al clúster de AKS que ha creado. 

1. Para crear, enumerar o leer una identidad administrada asignada por el usuario, es preciso asignar el rol [Operador de identidades administradas](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#managed-identity-operator) al clúster de AKS. Asegúrese de que **$clientId** sea el identificador de cliente del clúster de Kubernetes. Con respecto al ámbito, estará en el servicio de suscripción de Azure, específicamente el grupo de recursos de nodo que se creó al tiempo que el clúster de AKS. Este ámbito garantizará que solo los recursos de ese grupo se vean afectados por los roles asignados a continuación. 

    ```azurecli
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/$SUBID/resourcegroups/$NODE_RESOURCE_GROUP
    
    az role assignment create --role "Virtual Machine Contributor" --assignee $clientId --scope /subscriptions/$SUBID/resourcegroups/$NODE_RESOURCE_GROUP
    ```

1. Instale la identidad de Azure Active Directory (Azure AD) en AKS.
    ```azurecli
    helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

    helm install pod-identity aad-pod-identity/aad-pod-identity
    ```

1. Cree una identidad de Azure AD. En la salida, copie el valor de **clientId** y **principalId** para su uso posterior.
    ```azurecli
    az identity create -g $resourceGroupName -n $identityName
    ```

1. Asigne el rol *Lector* a la identidad de Azure AD que creó en el paso anterior para el almacén de claves y, a continuación, conceda permisos a la identidad para obtener los secretos del almacén de claves. Use los valores de **clientId** y **principalId** de la identidad de Azure AD.
    ```azurecli
    az role assignment create --role "Reader" --assignee $principalId --scope /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/contosoResourceGroup/providers/Microsoft.KeyVault/vaults/contosoKeyVault5

    az keyvault set-policy -n contosoKeyVault5 --secret-permissions get --spn $clientId
    ```

## <a name="deploy-your-pod-with-mounted-secrets-from-your-key-vault"></a>Implementación del pod con los secretos montados desde Key Vault

Para configurar el objeto SecretProviderClass, ejecute el siguiente comando:
```azurecli
kubectl apply -f secretProviderClass.yaml
```

### <a name="use-a-service-principal"></a>Uso de una entidad de servicio

Si usa una entidad de servicio, utilice el siguiente comando para implementar los pods de Kubernetes con los objetos SecretProviderClass y secrets-store-creds que configuró anteriormente. Estas son las plantillas de implementación:
* Para [Linux](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/nginx-pod-inline-volume-service-principal.yaml)
* Para [Windows](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/windows-pod-secrets-store-inline-volume-secret-providerclass.yaml)

```azurecli
kubectl apply -f updateDeployment.yaml
```

### <a name="use-managed-identities"></a>Uso de identidades administradas

Si utiliza identidades administradas, cree un elemento *AzureIdentity* en el clúster que haga referencia a la identidad que creó anteriormente. A continuación, cree un elemento *AzureIdentityBinding* que haga referencia al elemento AzureIdentity que ha creado. Rellene los parámetros de la plantilla siguiente y guárdela como *podIdentityAndBinding.yaml*.  

```yml
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentity
metadata:
    name: "azureIdentityName"               # The name of your Azure identity
spec:
    type: 0                                 # Set type: 0 for managed service identity
    resourceID: /subscriptions/<SUBID>/resourcegroups/<RESOURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<AZUREIDENTITYNAME>
    clientID: "managedIdentityClientId"     # The clientId of the Azure AD identity that you created earlier
---
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentityBinding
metadata:
    name: azure-pod-identity-binding
spec:
    azureIdentity: "azureIdentityName"      # The name of your Azure identity
    selector: azure-pod-identity-binding-selector
```
    
Ejecute el siguiente comando para ejecutar el enlace:

```azurecli
kubectl apply -f podIdentityAndBinding.yaml
```

A continuación, implemente el pod. El código siguiente es el archivo YAML de implementación, que usa el enlace de la identidad del pod del paso anterior. Guarde el archivo como *podBindingDeployment.yaml*.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secrets-store-inline
  labels:
    aadpodidbinding: azure-pod-identity-binding-selector
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: azure-kvname
```

Ejecute el siguiente comando para implementar el pod:

```azurecli
kubectl apply -f podBindingDeployment.yaml
```

### <a name="check-the-pod-status-and-secret-content"></a>Comprobación del estado del pod y del contenido del secreto 

Para mostrar los pods que ha implementado, ejecute el siguiente comando:
```azurecli
kubectl get pods
```

Para comprobar el estado del pod, ejecute el siguiente comando:
```azurecli
kubectl describe pod/nginx-secrets-store-inline
```

![Captura de pantalla de la salida de la CLI de Azure que muestra el estado "En ejecución" del pod y muestra todos los eventos como "Normal" ](../media/kubernetes-key-vault-6.png)

En la ventana de salida, el pod implementado debe estar en estado *Running* (En ejecución). En la sección **Events** de la parte inferior, todos los tipos de evento se muestran como *Normal*.

Una vez que haya comprobado que el pod está en ejecución, puede comprobar que contiene los secretos del almacén de claves.

Para mostrar todos los secretos contenidos en el pod, ejecute el siguiente comando:
```azurecli
kubectl exec -it nginx-secrets-store-inline -- ls /mnt/secrets-store/
```

Para mostrar el contenido de un secreto específico, ejecute el siguiente comando:
```azurecli
kubectl exec -it nginx-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

Compruebe que se muestra el contenido del secreto.

## <a name="next-steps"></a>Pasos siguientes

Para asegurarse de que el almacén de claves es recuperable, consulte:
> [!div class="nextstepaction"]
> [Activación de la eliminación temporal](https://docs.microsoft.com/azure/key-vault/general/soft-delete-cli)
