---
title: Rotación de certificados en Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo rotar los certificados en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 11/15/2019
ms.openlocfilehash: 90526b78e65c335f07a2a9d2d152b54b47233082
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88211032"
---
# <a name="rotate-certificates-in-azure-kubernetes-service-aks"></a>Rotación de certificados en Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) usa certificados para la autenticación con muchos de sus componentes. Periódicamente, puede que necesite rotar esos certificados por motivos de seguridad o de directivas. Por ejemplo, puede establecer una directiva para girar todos los certificados cada 90 días.

En este artículo se muestra cómo rotar los certificados en el clúster de AKS.

## <a name="before-you-begin"></a>Antes de empezar

Para este artículo, es preciso usar la versión 2.0.77 de la CLI de Azure o cualquier versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="aks-certificates-certificate-authorities-and-service-accounts"></a>Certificados de AKS, entidades de certificación y cuentas de servicio

AKS genera y usa los siguientes certificados, entidades de certificación y cuentas de servicio:

* El servidor de API de AKS crea una entidad de certificación (CA) denominada CA del clúster.
* El servidor de API tiene una CA del clúster que firma los certificados para la comunicación unidireccional desde el servidor de API a kubelets.
* Cada kubelet también crea una solicitud de firma de certificado (CSR), que está firmada por la CA del clúster, para la comunicación desde el kubelet al servidor de la API.
* El almacén de valores de clave de etcd tiene un certificado firmado por la CA del clúster para la comunicación desde etcd al servidor de la API.
* El almacén de valores de clave de etcd crea una CA que firma los certificados para autenticar y autorizar la replicación de datos entre las réplicas de etcd en el clúster de AKS.
* El agregador de API usa la CA del clúster para emitir certificados para la comunicación con otras API. El agregador de API también puede tener su propia CA para emitir esos certificados, pero actualmente usa la CA del clúster.
* Cada nodo usa un token de cuenta de servicio (SA), que está firmado por la CA del clúster.
* El cliente `kubectl` tiene un certificado para comunicarse con el clúster de AKS.

> [!NOTE]
> Los clústeres de AKS creados antes de marzo de 2019 tienen certificados que expiran después de dos años. Los clústeres creados después de marzo del 2019 o cualquier clúster que tenga rotados sus certificados tienen certificados de CA del clúster que expiran a los 30 años. El resto de certificados expira transcurridos dos años. Para comprobar cuándo se creó el clúster, use `kubectl get nodes` para ver la *Antigüedad* de los grupos de nodos.
> 
> Además, puede comprobar la fecha de expiración del certificado del clúster. Por ejemplo, el comando de Bash siguiente muestra los detalles del certificado para el clúster *myAKSCluster*.
> ```console
> kubectl config view --raw -o jsonpath="{.clusters[?(@.name == 'myAKSCluster')].cluster.certificate-authority-data}" | base64 -d | openssl x509 -text | grep -A2 Validity
> ```

## <a name="rotate-your-cluster-certificates"></a>Rotar los certificados de clúster

> [!WARNING]
> La rotación de los certificados con `az aks rotate-certs` puede producir un tiempo de inactividad de hasta 30 minutos para el clúster de AKS.

Use [az aks get-credentials][az-aks-get-credentials] para iniciar sesión en el clúster de AKS. Este comando también descarga y configura el certificado de cliente `kubectl` en el equipo local.

```azurecli
az aks get-credentials -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME
```

Use `az aks rotate-certs` para rotar todos los certificados, entidades de certificación (CA) y cuentas de servicio (SA) del clúster.

```azurecli
az aks rotate-certs -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME
```

> [!IMPORTANT]
> Este proceso puede tardar hasta 30 minutos para que `az aks rotate-certs` se complete. Si se produce un error en el comando antes de finalizar, use `az aks show` para comprobar que el estado del clúster es *Certificate Rotating* (Rotación de certificados). Si el clúster se encuentra en un estado de error, vuelva a ejecutar `az aks rotate-certs` para rotar los certificados otra vez.

Para comprobar que los certificados antiguos ya no son válidos, ejecute un comando `kubectl`. Como no ha actualizado los certificados usados por `kubectl`, verá que aparece un error.  Por ejemplo:

```console
$ kubectl get no
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "ca")
```

Actualice el certificado usado por `kubectl` ejecutando `az aks get-credentials`.

```azurecli
az aks get-credentials -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME --overwrite-existing
```

Compruebe que los certificados se han actualizado mediante la ejecución de un comando `kubectl`, que ahora se realizará correctamente. Por ejemplo:

```console
kubectl get no
```

> [!NOTE]
> Si tiene algún servicio que se ejecute sobre AKS, como [Azure Dev Spaces][dev-spaces], es posible que también tenga que [actualizar los certificados relacionados con esos servicios][dev-spaces-rotate].

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo rotar automáticamente los certificados, las entidades de certificación (CA) y las cuentas de servicio (SA) del clúster. Para obtener más información sobre los procedimientos recomendados de seguridad de AKS, puede consultar las [Prácticas recomendadas para la seguridad de los clústeres y las actualizaciones en Azure Kubernetes Service (AKS)][aks-best-practices-security-upgrades].


[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[aks-best-practices-security-upgrades]: operator-best-practices-cluster-security.md
[dev-spaces]: ../dev-spaces/index.yml
[dev-spaces-rotate]: ../dev-spaces/troubleshooting.md#error-using-dev-spaces-after-rotating-aks-certificates
