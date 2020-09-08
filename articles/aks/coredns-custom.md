---
title: Personalización de CoreDNS para Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo personalizar CoreDNS para agregar subdominios o ampliar puntos de conexión DNS personalizados con Azure Kubernetes Service (AKS).
services: container-service
author: jnoller
ms.topic: article
ms.date: 03/15/2019
ms.author: jenoller
ms.openlocfilehash: e99d841dcfb18b41df128283c37f46682e3fa129
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88257123"
---
# <a name="customize-coredns-with-azure-kubernetes-service"></a>Personalización de CoreDNS con Azure Kubernetes Service

Azure Kubernetes Service (AKS) usa el proyecto [CoreDNS][coredns] para la administración y la resolución DNS del clúster con todos los clústeres *1.12.x* y versiones superiores. Anteriormente, usaba el proyecto kube-dns. Actualmente, este proyecto kube-dns está en desuso. Para obtener más información sobre la personalización de CoreDNS y Kubernetes, consulte la [documentación oficial del canal de subida][corednsk8s].

Dado que AKS es un servicio administrado, no puede modificar la configuración principal para CoreDNS (un archivo *CoreFile*). En vez de eso, use un archivo *ConfigMap* de Kubernetes para invalidar la configuración predeterminada. Para ver los archivos ConfigMap de CoreDNS predeterminados de AKS, use el comando `kubectl get configmaps --namespace=kube-system coredns -o yaml`.

En este artículo se muestra cómo usar los archivos ConfigMap para las opciones de personalización básica de CoreDNS en AKS. Este enfoque difiere de la configuración de CoreDNS en otros contextos, como el uso de CoreFile. Compruebe la versión de CoreDNS que ejecuta, ya que los valores de configuración pueden cambiar entre versiones.

> [!NOTE]
> `kube-dns` ofrecía [opciones de personalización][kubednsblog] diferentes a través de una asignación de configuración de Kubernetes. CoreDNS **no** es compatible con versiones anteriores de kube-dns. Todas las personalizaciones que usó anteriormente deben actualizarse para su uso con CoreDNS.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

Al crear una configuración como en los ejemplos siguientes, los nombres de la sección *data* deben finalizar en *.server* u *.override*. Esta convención de nomenclatura se define en el valor predeterminado CoreDNS Configmap de AKS, que se puede ver con el comando `kubectl get configmaps --namespace=kube-system coredns -o yaml`.

## <a name="what-is-supportedunsupported"></a>Elementos admitidos y no admitidos

Se admiten todos los complementos de CoreDNS integrados. No se admiten los complementos de terceros o complementarios.

## <a name="rewrite-dns"></a>Reescritura de DNS

Un escenario que debe considerar es la reescritura de nombres DNS sobre la marcha. En el ejemplo siguiente, reemplace `<domain to be written>` por su propio nombre de dominio completo. Cree un archivo denominado `corednsms.yaml` y pegue la siguiente configuración de ejemplo:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    <domain to be rewritten>.com:53 {
        errors
        cache 30
        rewrite name substring <domain to be rewritten>.com default.svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        forward .  /etc/resolv.conf # you can redirect this to a specific DNS server such as 10.0.0.10, but that server must be able to resolve the rewritten domain name
    }
```

> [!IMPORTANT]
> Si redirige a un servidor DNS, como la dirección IP del servicio CoreDNS, ese servidor DNS debe poder resolver el nombre de dominio reescrito.

Cree el archivo ConfigMap mediante el comando [kubectl apply configmap][kubectl-apply] y especifique el nombre de su manifiesto de YAML:

```console
kubectl apply -f corednsms.yaml
```

Para comprobar que las personalizaciones se han aplicado, use el comando [kubectl get configmaps][kubectl-get] y especifique su archivo ConfigMap *coredns-custom*:

```
kubectl get configmaps --namespace=kube-system coredns-custom -o yaml
```

Ahora, fuerce a que CoreDNS vuelva a cargar el archivo ConfigMap. El comando [kubectl delete pod][kubectl delete] no es destructivo y no provoca tiempo de inactividad. Se eliminan los pods `kube-dns` y el Programador de Kubernetes vuelve a crearlos. Estos nuevos pods contienen el cambio en el valor TTL.

```console
kubectl delete pod --namespace kube-system -l k8s-app=kube-dns
```

> [!Note]
> El comando anterior es correcto. Mientras cambiamos `coredns`, la implementación tiene el nombre **kube-dns**.

## <a name="custom-forward-server"></a>Servidor de reenvío personalizado

Si tiene que especificar un servidor de reenvío para el tráfico de red, puede crear un elemento ConfigMap para personalizar el DNS. En el ejemplo siguiente, actualice el nombre y la dirección del `forward` con los valores de su propio entorno. Cree un archivo denominado `corednsms.yaml` y pegue la siguiente configuración de ejemplo:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    <domain to be rewritten>.com:53 {
        forward foo.com 1.1.1.1
    }
```

Como en los ejemplos anteriores, cree el archivo ConfigMap mediante el comando [kubectl apply configmap][kubectl-apply] y especifique el nombre de su manifiesto de YAML. Después, fuerce a que CoreDNS vuelva a cargar el archivo ConfigMap con el comando [kubectl delete pod][kubectl delete] para que el programador de Kubernetes vuelva a crearlo:

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="use-custom-domains"></a>Uso de dominios personalizados

Es posible que le interese configurar dominios personalizados que solo se puedan resolver internamente. Por ejemplo, tal vez quiera resolver el dominio personalizado *puglife.local*, que no es un dominio de nivel superior válido. Sin un archivo ConfigMap de dominio personalizado, el clúster de AKS no puede resolver la dirección.

En el ejemplo siguiente, actualice con los valores de su propio entorno el dominio personalizado y la dirección IP a la que se va a dirigir el tráfico. Cree un archivo denominado `corednsms.yaml` y pegue la siguiente configuración de ejemplo:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  puglife.server: | # you may select any name here, but it must end with the .server file extension
    puglife.local:53 {
        errors
        cache 30
        forward . 192.11.0.1  # this is my test/dev DNS server
    }
```

Como en los ejemplos anteriores, cree el archivo ConfigMap mediante el comando [kubectl apply configmap][kubectl-apply] y especifique el nombre de su manifiesto de YAML. Después, fuerce a que CoreDNS vuelva a cargar el archivo ConfigMap con el comando [kubectl delete pod][kubectl delete] para que el programador de Kubernetes vuelva a crearlo:

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="stub-domains"></a>Dominios de código auxiliar

También se puede usar CoreDNS para configurar dominios de código auxiliar. En el ejemplo siguiente, actualice con los valores de su propio entorno los dominios personalizados y las direcciones IP. Cree un archivo denominado `corednsms.yaml` y pegue la siguiente configuración de ejemplo:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    abc.com:53 {
        errors
        cache 30
        forward . 1.2.3.4
    }
    my.cluster.local:53 {
        errors
        cache 30
        forward . 2.3.4.5
    }

```

Como en los ejemplos anteriores, cree el archivo ConfigMap mediante el comando [kubectl apply configmap][kubectl-apply] y especifique el nombre de su manifiesto de YAML. Después, fuerce a que CoreDNS vuelva a cargar el archivo ConfigMap con el comando [kubectl delete pod][kubectl delete] para que el programador de Kubernetes vuelva a crearlo:

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="hosts-plugin"></a>Complemento de hosts

Dado que se admiten todos los complementos integrados, también se puede personalizar el complemento de [Hosts][coredns hosts] de CoreDNS:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom # this is the name of the configmap you can overwrite with your changes
  namespace: kube-system
data:
    test.override: | # you may select any name here, but it must end with the .override file extension
          hosts example.hosts example.org { # example.hosts must be a file
              10.0.0.1 example.org
              fallthrough
          }
```

## <a name="enable-logging-for-dns-query-debugging"></a>Habilitación del registro de depuración de consultas de DNS 

Para habilitar el registro de consultas de DNS, aplique la siguiente configuración en la instancia coredns-custom de ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  log.override: | # you may select any name here, but it must end with the .override file extension
        log
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se han mostrado algunos escenarios de ejemplo para la personalización de CoreDNS. Para obtener información sobre el proyecto CoreDNS, consulte la [página de proyecto del canal de subida de CoreDNS][coredns].

Para obtener más información sobre conceptos básicos de red, consulte [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)][concepts-network].

<!-- LINKS - external -->
[kubednsblog]: https://www.danielstechblog.io/using-custom-dns-server-for-domain-specific-name-resolution-with-azure-kubernetes-service/
[coredns]: https://coredns.io/
[corednsk8s]: https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#coredns
[dnscache]: https://coredns.io/plugins/cache/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[coredns hosts]: https://coredns.io/plugins/hosts/

<!-- LINKS - internal -->
[concepts-network]: concepts-network.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
