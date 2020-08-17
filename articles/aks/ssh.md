---
title: Conexión SSH con los nodos de un clúster de Azure Kubernetes Service (AKS)
description: Aprenda a crear una conexión SSH con los nodos de clúster de Azure Kubernetes Service (AKS) para la solución de problemas y tareas de mantenimiento.
services: container-service
ms.topic: article
ms.date: 07/31/2019
ms.openlocfilehash: 50a52584618e505aa2ae7bd9ed7e0a9f6bc330a9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015619"
---
# <a name="connect-with-ssh-to-azure-kubernetes-service-aks-cluster-nodes-for-maintenance-or-troubleshooting"></a>Conexión con SSH a los nodos de clúster de Azure Kubernetes Service (AKS) para mantenimiento o solución de problemas

Durante el ciclo de vida del clúster de Azure Kubernetes Service (AKS), es posible que necesite acceder a un nodo de AKS. Este acceso podría ser para mantenimiento, recopilación de registros u otras operaciones de solución de problemas. Puede acceder a los nodos de AKS mediante SSH, incluidos los nodos de Windows Server. También puede [conectarse a los nodos de Windows Server mediante conexiones de protocolo de escritorio remoto (RDP)][aks-windows-rdp]. Por motivos de seguridad, los nodos de AKS no están expuestos a Internet. Para conectarse mediante SSH a los nodos de AKS, use la dirección IP privada.

En este artículo se muestra cómo crear una conexión SSH con un nodo de AKS mediante sus direcciones IP privadas.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

De forma predeterminada, las claves SSH se obtienen, o generan, y luego se agregan a los nodos cuando crea un clúster de AKS. En este artículo se muestra cómo especificar claves SSH distintas de las que se usaron al crear el clúster de Azure Kubernetes Service. En el artículo también se muestra cómo determinar la dirección IP privada del nodo y cómo conectarse a el mediante SSH. Si no necesita especificar otra clave SSH, puede omitir el paso en el que se agrega la clave pública SSH al nodo.

En este artículo también se da por supuesto que tiene una clave SSH. Se pueden crear claves SSH en [macOS, Linux][ssh-nix] o [Windows][ssh-windows]. Si utiliza PuTTY Gen para crear el par de claves, guarde dicho par en un formato OpenSSH en lugar del formato de clave privada PuTTy predeterminado (archivo .ppk).

También es preciso tener instalada y configurada la versión 2.0.64 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="configure-virtual-machine-scale-set-based-aks-clusters-for-ssh-access"></a>Configuración de clústeres de AKS basados en conjuntos de escalado de máquinas virtuales para acceder a SSH

Para configurar un clúster de AKS basado en un conjunto de escalado de máquinas virtuales para acceder a SSH, busque el nombre del conjunto de escalado de máquinas virtuales del clúster y agregue la clave pública SSH a ese conjunto de escalado.

Use el comando [az aks show][az-aks-show] para obtener el nombre del grupo de recursos del clúster de AKS y, después, el comando [vmss list][az-vmss-list] para obtener el nombre del conjunto de escalado.

```azurecli-interactive
CLUSTER_RESOURCE_GROUP=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv)
SCALE_SET_NAME=$(az vmss list --resource-group $CLUSTER_RESOURCE_GROUP --query [0].name -o tsv)
```

En el ejemplo anterior se asigna el nombre del grupo de recursos del clúster para *myAKSCluster* en *myResourceGroup* a *CLUSTER_RESOURCE_GROUP*. Luego, el ejemplo usa *CLUSTER_RESOURCE_GROUP* para mostrar el nombre del conjunto de escalado y asignarlo a *SCALE_SET_NAME*.

> [!IMPORTANT]
> En este punto, solo debe actualizar las claves SSH de los clústeres de AKS basados en conjuntos de escalado de máquinas virtuales mediante la CLI de Azure.
> 
> En los nodos de Linux, las claves SSH actualmente solo se pueden agregar mediante la CLI de Azure. Si desea conectarse a un nodo de Windows Server mediante SSH, use las claves SSH que se proporcionan al crear el clúster de AKS y omita el siguiente conjunto de comandos para agregar la clave pública SSH. Seguirá necesitando la dirección IP del nodo cuyos problemas desea solucionar, que se muestra en el comando final de esta sección. Como alternativa, puede [conectarse a los nodos de Windows Server mediante conexiones RDP (protocolo de escritorio remoto)][aks-windows-rdp], en lugar de usar SSH.

Para agregar sus claves SSH a los nodos de un conjunto de escalado de máquinas virtuales, use los comandos [az vmss extension set][az-vmss-extension-set] y [az vmss update-instances][az-vmss-update-instances].

```azurecli-interactive
az vmss extension set  \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --vmss-name $SCALE_SET_NAME \
    --name VMAccessForLinux \
    --publisher Microsoft.OSTCExtensions \
    --version 1.4 \
    --protected-settings "{\"username\":\"azureuser\", \"ssh_key\":\"$(cat ~/.ssh/id_rsa.pub)\"}"

az vmss update-instances --instance-ids '*' \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --name $SCALE_SET_NAME
```

En el ejemplo anterior se usan las variables *CLUSTER_RESOURCE_GROUP* y *SCALE_SET_NAME* de los comandos anteriores. En el ejemplo anterior también se usa *~/.ssh/id_rsa.pub* como ubicación de la clave pública SSH.

> [!NOTE]
> De forma predeterminada, el nombre de usuario para los nodos de AKS es *azureuser*.

Después de agregar la clave pública SSH al conjunto de escalado, puede usar SSH en una máquina virtual del nodo de ese conjunto de escalado mediante su dirección IP. Vea las direcciones IP privadas de los nodos del clúster de AKS mediante el comando [kubectl get][kubectl-get].

```console
kubectl get nodes -o wide
```

La salida del ejemplo siguiente muestra las direcciones IP internas de todos los nodos del clúster, incluido un nodo de Windows Server.

```console
$ kubectl get nodes -o wide

NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                    KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-42485177-vmss000000   Ready    agent   18h   v1.12.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS          4.15.0-1040-azure   docker://3.0.4
aksnpwin000000                      Ready    agent   13h   v1.12.7   10.240.0.67   <none>        Windows Server Datacenter   10.0.17763.437
```

Registre la dirección IP interna del nodo que desea solucionar.

Para acceder a su nodo mediante SSH, siga los pasos que se indican en [Creación de la conexión SSH](#create-the-ssh-connection).

## <a name="configure-virtual-machine-availability-set-based-aks-clusters-for-ssh-access"></a>Configuración de clústeres de AKS basados en conjuntos de disponibilidad de máquinas virtuales para acceder a SSH

Para configurar un clúster de AKS basado en un conjunto de disponibilidad de máquinas virtuales para acceder a SSH, busque el nombre del nodo Linux del clúster y agregue su clave pública SSH a ese nodo.

Use el comando [az aks show][az-aks-show] para obtener el nombre del grupo de recursos del clúster de AKS y, después, el comando [az vm list][az-vm-list] para enumerar el nombre de la máquina virtual del nodo de Linux del clúster.

```azurecli-interactive
CLUSTER_RESOURCE_GROUP=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv)
az vm list --resource-group $CLUSTER_RESOURCE_GROUP -o table
```

En el ejemplo anterior se asigna el nombre del grupo de recursos del clúster para *myAKSCluster* en *myResourceGroup* a *CLUSTER_RESOURCE_GROUP*. Luego, el ejemplo usa *CLUSTER_RESOURCE_GROUP* para enumerar el nombre de la máquina virtual. La salida del ejemplo muestra el nombre de la máquina virtual:

```
Name                      ResourceGroup                                  Location
------------------------  ---------------------------------------------  ----------
aks-nodepool1-79590246-0  MC_myResourceGroupAKS_myAKSClusterRBAC_eastus  eastus
```

Para agregar las claves SSH al nodo, utilice el comando [az vm user update][az-vm-user-update].

```azurecli-interactive
az vm user update \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --name aks-nodepool1-79590246-0 \
    --username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

En el ejemplo anterior se usa la variable *CLUSTER_RESOURCE_GROUP* y el nombre de la máquina virtual del nodo de los comandos anteriores. En el ejemplo anterior también se usa *~/.ssh/id_rsa.pub* como ubicación de la clave pública SSH. También puede usar el contenido de la clave pública SSH, en lugar de especificar una ruta de acceso.

> [!NOTE]
> De forma predeterminada, el nombre de usuario para los nodos de AKS es *azureuser*.

Después de agregar la clave pública SSH a la máquina virtual del nodo, puede usar conectarse con SSH a esa máquina virtual mediante su dirección IP. Vea la dirección IP privada de un nodo de clúster de AKS con el comando [az vm list-ip-addresses][az-vm-list-ip-addresses].

```azurecli-interactive
az vm list-ip-addresses --resource-group $CLUSTER_RESOURCE_GROUP -o table
```

En el ejemplo anterior se usa la variable *CLUSTER_RESOURCE_GROUP* establecida en los comandos anteriores. En la siguiente salida de ejemplo se muestran las direcciones IP privadas de los nodos de AKS:

```
VirtualMachine            PrivateIPAddresses
------------------------  --------------------
aks-nodepool1-79590246-0  10.240.0.4
```

## <a name="create-the-ssh-connection"></a>Creación de la conexión SSH

Para crear una conexión SSH a un nodo AKS, ejecute un pod asistente en el clúster de AKS. Este pod asistente proporciona acceso SSH al clúster y después acceso adicional al nodo SSH. Para crear y utilizar este pod asistente, siga estos pasos:

1. Ejecute una imagen de contenedor `debian` y asocie a ella una sesión de terminal. Este contenedor se puede usar para crear una sesión de SSH con cualquier nodo del clúster de AKS:

    ```console
    kubectl run -it --rm aks-ssh --image=debian
    ```

    > [!TIP]
    > Si usa nodos de Windows Server, agregue un selector de nodo al comando para programar el contenedor de Debian en un nodo de Linux:
    >
    > ```console
    > kubectl run -it --rm aks-ssh --image=debian --overrides='{"apiVersion":"v1","spec":{"nodeSelector":{"beta.kubernetes.io/os":"linux"}}}'
    > ```

1. Una vez que la sesión de terminal esté conectada al contenedor, instale un cliente SSH mediante `apt-get`:

    ```console
    apt-get update && apt-get install openssh-client -y
    ```

1. Abra una nueva ventana de terminal, no conectada al contenedor, y copie la clave SSH privada en el pod asistente. Esta clave privada se utiliza para crear el SSH en el nodo de Azure Kubernetes Service. 

   Si es necesario, cambie *~/.ssh/id_rsa* a la ubicación de la clave SSH privada:

    ```console
    kubectl cp ~/.ssh/id_rsa $(kubectl get pod -l run=aks-ssh -o jsonpath='{.items[0].metadata.name}'):/id_rsa
    ```

1. Vuelva a la sesión de terminal en el contenedor, actualice los permisos de la clave SSH privada copiada `id_rsa` para que sea de solo lectura para el usuario:

    ```console
    chmod 0600 id_rsa
    ```

1. Cree una conexión SSH al nodo de Azure Kubernetes Service. De nuevo, el nombre de usuario para los nodos de AKS es *azureuser*. Acepte el aviso para continuar con la conexión, ya que la clave SSH es la primera en la que se confía. A continuación, se proporcionan con el aviso de bash del nodo de AKS:

    ```console
    $ ssh -i id_rsa azureuser@10.240.0.4

    ECDSA key fingerprint is SHA256:A6rnRkfpG21TaZ8XmQCCgdi9G/MYIMc+gFAuY9RUY70.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.240.0.4' (ECDSA) to the list of known hosts.

    Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1018-azure x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

      Get cloud support with Ubuntu Advantage Cloud Guest:
        https://www.ubuntu.com/business/services/cloud

    [...]

    azureuser@aks-nodepool1-79590246-0:~$
    ```

## <a name="remove-ssh-access"></a>Eliminación del acceso SSH

Cuando haya finalizado, `exit` la sesión de SSH y, a continuación, `exit` la sesión del contenedor interactiva. Cuando esta sesión de contenedor se cierra, se elimina el pod usado para el acceso SSH desde el clúster de AKS.

## <a name="next-steps"></a>Pasos siguientes

Si necesita datos adicionales para la solución de problemas, puede [ver los registros de kubelet][view-kubelet-logs] o [ver los registros del nodo maestro de Kubernetes][view-master-logs].

<!-- EXTERNAL LINKS -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- INTERNAL LINKS -->
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-vm-list]: /cli/azure/vm#az-vm-list
[az-vm-user-update]: /cli/azure/vm/user#az-vm-user-update
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[view-kubelet-logs]: kubelet-logs.md
[view-master-logs]: view-master-logs.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-windows-rdp]: rdp.md
[ssh-nix]: ../virtual-machines/linux/mac-create-ssh-keys.md
[ssh-windows]: ../virtual-machines/linux/ssh-from-windows.md
[az-vmss-list]: /cli/azure/vmss#az-vmss-list
[az-vmss-extension-set]: /cli/azure/vmss/extension#az-vmss-extension-set
[az-vmss-update-instances]: /cli/azure/vmss#az-vmss-update-instances
