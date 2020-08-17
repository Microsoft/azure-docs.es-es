---
title: Complemento de enrutamiento de aplicación HTTP en Azure Kubernetes Service (AKS)
description: Uso del complemento de enrutamiento de aplicación HTTP para acceder a aplicaciones en Azure Kubernetes Service (AKS).
services: container-service
author: lachie83
ms.topic: article
ms.date: 07/20/2020
ms.author: laevenso
ms.openlocfilehash: 08835bda959fb4fe261e86e4d519ab85bd2a4625
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495155"
---
# <a name="http-application-routing"></a>Enrutamiento de aplicación HTTP

La solución de enrutamiento de aplicación HTTP facilita el acceso a las aplicaciones implementadas en el clúster de Azure Kubernetes Service (AKS). Cuando se habilita la solución, configura un [controlador de entrada](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) en el clúster de AKS. A medida que se implementan aplicaciones, la solución también crea nombres DNS accesibles públicamente para los puntos de conexión de aplicación.

Cuando se habilita el complemento, crea una zona DNS en su suscripción. Para más información sobre el costo de DNS, consulte los [precios de DNS][dns-pricing].

> [!CAUTION]
> El complemento de enrutamiento de aplicaciones HTTP está diseñado para permitirle crear rápidamente un controlador de entrada y acceder a sus aplicaciones. Este complemento no está diseñado actualmente para su uso en un entorno de producción y no se recomienda usarlo en el mismo. Para información sobre las implementaciones de entrada listas para producción, consulte [Creación de un controlador de entrada HTTPS](./ingress-tls.md).

## <a name="http-routing-solution-overview"></a>Introducción a la solución de enrutamiento HTTP

El complemento implementa dos componentes, un [controlador de entrada de Kubernetes][ingress] y un controlador [DNS externo][external-dns].

- **Controlador de entrada**: el controlador de entrada está expuesto a Internet a través de un servicio de Kubernetes de tipo equilibrador de carga. El controlador de entrada supervisa e implementa [recursos de entrada de Kubernetes][ingress-resource], lo que crea rutas a los puntos de conexión de aplicación.
- **Controlador DNS externo**: inspecciona los recursos de entrada de Kubernetes y crea registros A de DNS en la zona DNS específica del clúster.

## <a name="deploy-http-routing-cli"></a>Implementación del enrutamiento HTTP: CLI

El complemento de enrutamiento de aplicación HTTP se puede habilitar con la CLI de Azure al implementar un clúster de AKS. Para hacerlo, use el comando [az aks create][az-aks-create] con el argumento `--enable-addons`.

```azurecli
az aks create --resource-group myResourceGroup --name myAKSCluster --enable-addons http_application_routing
```

> [!TIP]
> Si desea habilitar varios complementos, debe proporcionarlos como una lista separada por comas. Por ejemplo, para habilitar la supervisión y el enrutamiento de solicitudes HTTP, use el formato `--enable-addons http_application_routing,monitoring`.

También puede habilitar el enrutamiento de HTTP en un clúster de AKS existente mediante el comando [az aks enable-addons][az-aks-enable-addons]. Para habilitar el enrutamiento de HTTP en un clúster existente, agregue el parámetro `--addons` y especifique *http_application_routing* tal como se muestra en el ejemplo siguiente:

```azurecli
az aks enable-addons --resource-group myResourceGroup --name myAKSCluster --addons http_application_routing
```

Después de implementar o actualizar el clúster, use el comando [az aks show][az-aks-show] para recuperar el nombre de la zona DNS.

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table
```

Este nombre es necesario para implementar aplicaciones en el clúster de AKS y se muestra en la siguiente salida de ejemplo:

```console
9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io
```

## <a name="deploy-http-routing-portal"></a>Implementación del enrutamiento HTTP: Portal

El complemento de enrutamiento de aplicación HTTP se puede habilitar mediante Azure Portal al implementar un clúster de AKS.

![Habilitación de la característica de enrutamiento HTTP](media/http-routing/create.png)

Después de implementar el clúster, vaya al grupo de recursos de AKS que se creó automáticamente y seleccione la zona DNS. Tome nota del nombre de la zona DNS. Este nombre es necesario para implementar aplicaciones en el clúster de AKS.

![Obtención del nombre de la zona DNS](media/http-routing/dns.png)

## <a name="connect-to-your-aks-cluster"></a>Conectarse al clúster AKS

Para conectarse al clúster de Kubernetes desde su equipo local, use [kubectl][kubectl], el cliente de la línea de comandos de Kubernetes.

Si usa Azure Cloud Shell, `kubectl` ya está instalado. También lo puede instalar localmente. Para ello debe usar el comando [az aks install-cli][]:

```azurecli
az aks install-cli
```

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][]. En el ejemplo siguiente se obtienen las credenciales del clúster de AKS llamado *MyAKSCluster* en el grupo de recursos *MyResourceGroup*:

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKSCluster
```

## <a name="use-http-routing"></a>Uso del enrutamiento HTTP

La solución de enrutamiento de aplicación HTTP solo se puede desencadenar en los recursos de entrada que se anotan como sigue:

```yaml
annotations:
  kubernetes.io/ingress.class: addon-http-application-routing
```

Cree un archivo denominado **samples-http-application-routing.yaml** y cópielo en el siguiente código YAML. En la línea 43, actualice `<CLUSTER_SPECIFIC_DNS_ZONE>` con el nombre de la zona DNS que se recopiló en el paso anterior de este artículo.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld
  template:
    metadata:
      labels:
        app: aks-helloworld
    spec:
      containers:
      - name: aks-helloworld
        image: neilpeterson/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: aks-helloworld
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: aks-helloworld.<CLUSTER_SPECIFIC_DNS_ZONE>
    http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /
```

Utilice el comando [kubectl apply][kubectl-apply] para crear los recursos.

```bash
kubectl apply -f samples-http-application-routing.yaml
```

En el siguiente ejemplo se muestran los recursos creados:

```bash
$ kubectl apply -f samples-http-application-routing.yaml

deployment.apps/aks-helloworld created
service/aks-helloworld created
ingress.networking.k8s.io/aks-helloworld created
```

Abra un explorador web a *aks-helloworld.\<CLUSTER_SPECIFIC_DNS_ZONE\>* , por ejemplo *aks-helloworld.9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io* y compruebe que ve la aplicación demo. La aplicación puede tardar unos minutos en aparecer.

## <a name="remove-http-routing"></a>Eliminación del enrutamiento HTTP

La solución de enrutamiento HTTP se puede eliminar mediante la CLI de Azure. Para ello, ejecute el siguiente comando, sustituyendo su clúster de AKS y el nombre del grupo de recursos.

```azurecli
az aks disable-addons --addons http_application_routing --name myAKSCluster --resource-group myResourceGroup --no-wait
```

Cuando se deshabilita el complemento de enrutamiento de aplicación HTTP, algunos recursos de Kubernetes pueden permanecer en el clúster. Estos recursos incluyen a los archivos *configMaps* y a los *secretos* y se crean en el espacio de nombres *kube-system*. Para mantener un clúster limpio, es posible que quiera quitar estos recursos.

Busque los recursos *addon-http-application-routing* con los siguientes comandos [kubectl get][kubectl-get]:

```console
kubectl get deployments --namespace kube-system
kubectl get services --namespace kube-system
kubectl get configmaps --namespace kube-system
kubectl get secrets --namespace kube-system
```

La salida del ejemplo siguiente muestra los archivos configMaps que deben eliminarse:

```
$ kubectl get configmaps --namespace kube-system

NAMESPACE     NAME                                                       DATA   AGE
kube-system   addon-http-application-routing-nginx-configuration         0      9m7s
kube-system   addon-http-application-routing-tcp-services                0      9m7s
kube-system   addon-http-application-routing-udp-services                0      9m7s
```

Para eliminar los recursos, use el comando [kubectl delete][kubectl-delete]. Especifique el tipo de recurso, el nombre del recurso y el espacio de nombres. En el ejemplo siguiente se elimina uno de los archivos configmaps anteriores:

```console
kubectl delete configmaps addon-http-application-routing-nginx-configuration --namespace kube-system
```

Repita el paso `kubectl delete` anterior para todos los recursos *addon-http-application-routing* que aún se encuentran en el clúster.

## <a name="troubleshoot"></a>Solución de problemas

Use el comando [kubectl logs][kubectl-logs] para ver los registros de aplicación para la aplicación de DNS externo. Los registros deben confirmar que crearon correctamente unos registros DNS A y TXT.

```
$ kubectl logs -f deploy/addon-http-application-routing-external-dns -n kube-system

time="2018-04-26T20:36:19Z" level=info msg="Updating A record named 'aks-helloworld' to '52.242.28.189' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
time="2018-04-26T20:36:21Z" level=info msg="Updating TXT record named 'aks-helloworld' to '"heritage=external-dns,external-dns/owner=default"' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
```

Estos registros también se pueden ver en el recurso de zona DNS en Azure Portal.

![Obtención de los registros DNS](media/http-routing/clippy.png)

Use el comando [kubectl logs][kubectl-logs] para ver los registros de aplicación para el controlador de entrada Nginx. Los registros deben confirmar la `CREATE` de un recurso de entrada y la recarga del controlador. Se registra toda la actividad HTTP.

```bash
$ kubectl logs -f deploy/addon-http-application-routing-nginx-ingress-controller -n kube-system

-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.13.0
  Build:      git-4bc943a
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------

I0426 20:30:12.212936       9 flags.go:162] Watching for ingress class: addon-http-application-routing
W0426 20:30:12.213041       9 flags.go:165] only Ingress with class "addon-http-application-routing" will be processed by this ingress controller
W0426 20:30:12.213505       9 client_config.go:533] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0426 20:30:12.213752       9 main.go:181] Creating API client for https://10.0.0.1:443
I0426 20:30:12.287928       9 main.go:225] Running in Kubernetes Cluster version v1.8 (v1.8.11) - git (clean) commit 1df6a8381669a6c753f79cb31ca2e3d57ee7c8a3 - platform linux/amd64
I0426 20:30:12.290988       9 main.go:84] validated kube-system/addon-http-application-routing-default-http-backend as the default backend
I0426 20:30:12.294314       9 main.go:105] service kube-system/addon-http-application-routing-nginx-ingress validated as source of Ingress status
I0426 20:30:12.426443       9 stat_collector.go:77] starting new nginx stats collector for Ingress controller running in namespace  (class addon-http-application-routing)
I0426 20:30:12.426509       9 stat_collector.go:78] collector extracting information from port 18080
I0426 20:30:12.448779       9 nginx.go:281] starting Ingress controller
I0426 20:30:12.463585       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-nginx-configuration", UID:"2588536c-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"559", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-nginx-configuration
I0426 20:30:12.466945       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-tcp-services", UID:"258ca065-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"561", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-tcp-services
I0426 20:30:12.467053       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-udp-services", UID:"259023bc-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"562", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-udp-services
I0426 20:30:13.649195       9 nginx.go:302] starting NGINX process...
I0426 20:30:13.649347       9 leaderelection.go:175] attempting to acquire leader lease  kube-system/ingress-controller-leader-addon-http-application-routing...
I0426 20:30:13.649776       9 controller.go:170] backend reload required
I0426 20:30:13.649800       9 stat_collector.go:34] changing prometheus collector from  to default
I0426 20:30:13.662191       9 leaderelection.go:184] successfully acquired lease kube-system/ingress-controller-leader-addon-http-application-routing
I0426 20:30:13.662292       9 status.go:196] new leader elected: addon-http-application-routing-nginx-ingress-controller-5cxntd6
I0426 20:30:13.763362       9 controller.go:179] ingress backend successfully reloaded...
I0426 21:51:55.249327       9 event.go:218] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"aks-helloworld", UID:"092c9599-499c-11e8-a5e1-0a58ac1f0ef2", APIVersion:"extensions", ResourceVersion:"7346", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/aks-helloworld
W0426 21:51:57.908771       9 controller.go:775] service default/aks-helloworld does not have any active endpoints
I0426 21:51:57.908951       9 controller.go:170] backend reload required
I0426 21:51:58.042932       9 controller.go:179] ingress backend successfully reloaded...
167.220.24.46 - [167.220.24.46] - - [26/Apr/2018:21:53:20 +0000] "GET / HTTP/1.1" 200 234 "" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)" 197 0.001 [default-aks-helloworld-80] 10.244.0.13:8080 234 0.004 200
```

## <a name="clean-up"></a>Limpieza

Quite los objetos Kubernetes asociados creados en este artículo mediante `kubectl delete`.

```bash
kubectl delete -f samples-http-application-routing.yaml
```

En la salida del ejemplo se muestra que los objetos Kubernetes se han quitado.

```bash
$ kubectl delete -f samples-http-application-routing.yaml

deployment "aks-helloworld" deleted
service "aks-helloworld" deleted
ingress "aks-helloworld" deleted
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo instalar un controlador de entrada protegido con HTTPS en AKS, consulte[Entrada HTTPS en Azure Kubernetes Service (AKS)][ingress-https].

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-show]: /cli/azure/aks?view=azure-cli-latest#az-aks-show
[ingress-https]: ./ingress-tls.md
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials

<!-- LINKS - external -->
[dns-pricing]: https://azure.microsoft.com/pricing/details/dns/
[external-dns]: https://github.com/kubernetes-incubator/external-dns
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[ingress-resource]: https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource
