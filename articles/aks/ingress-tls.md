---
title: Creación de entrada con TLS automático
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre cómo instalar y configurar un controlador de entrada NGINX que usa Let’s Encrypt para la generación automática de certificados TLS en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: 452e7d1e8dad0a3ae3d6393598f5f24ef2153aa8
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855940"
---
# <a name="create-an-https-ingress-controller-on-azure-kubernetes-service-aks"></a>Creación de un controlador de entrada HTTPS en Azure Kubernetes Service (AKS)

Un controlador de entrada es un software que proporciona el proxy inverso, el enrutamiento del tráfico configurable y la terminación de TLS para los servicios de Kubernetes. Los recursos de entrada de Kubernetes se usan para configurar las reglas de entrada y las rutas de los distintos servicios de Kubernetes. Mediante reglas de entrada y un controlador de entrada, se puede usar una sola dirección IP para enrutar el tráfico a varios servicios en un clúster de Kubernetes.

En este artículo se muestra cómo implementar el [controlador de entrada NGINX][nginx-ingress] en un clúster de Azure Kubernetes Service (AKS). El proyecto [cert-manager][cert-manager] se usa para generar y configurar automáticamente certificados [Let's Encrypt][lets-encrypt]. Por último, en el clúster de AKS se ejecutan dos aplicaciones, a las que se puede acceder con una sola dirección IP.

También puede:

- [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
- [Habilitación del complemento de enrutamiento de aplicación HTTP][aks-http-app-routing]
- [Creación de un controlador de entrada que use una red privada interna y una dirección IP][aks-ingress-internal]
- [Crear un controlador de entrada que usa sus propios certificados TLS][aks-ingress-own-tls]
- [Crear un controlador de entrada que usa Let's Encrypt para generar automáticamente certificados TLS con una dirección IP pública estática][aks-ingress-static-tls]

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

En este artículo se supone que tiene [un dominio personalizado][custom-domain] con una [zona DNS][dns-zone] en el mismo grupo de recursos que el clúster de AKS.

En este artículo se usa [Helm 3][helm] para instalar el controlador de entrada NGINX y cert-manager. Asegúrese de que usa la versión más reciente de Helm y de que tiene acceso a los repositorios de Helm *ingress-nginx* y *jetstack*. Para instrucciones de actualización, consulte la [documentación de instalación de Helm][helm-install]. Para obtener más información sobre cómo configurar y usar Helm, consulte [Instalación de aplicaciones con Helm en Azure Kubernetes Service (AKS)][use-helm].

En este artículo también se requiere que ejecute la versión 2.0.64 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="create-an-ingress-controller"></a>Crear un controlador de entrada

Para crear el controlador de entrada, use el comando `helm` para instalar *nginx-ingress*. Para obtener redundancia adicional, se implementan dos réplicas de los controladores de entrada NGINX con el parámetro `--set controller.replicaCount`. Para sacar el máximo provecho de las réplicas en ejecución del controlador de entrada, asegúrese de que hay más de un nodo en el clúster de AKS.

El controlador de entrada también debe programarse en un nodo de Linux. Los nodos de Windows Server no deben ejecutar el controlador de entrada. Un selector de nodos se especifica mediante el parámetro `--set nodeSelector` para indicar al programador de Kubernetes que ejecute el controlador de entrada NGINX en un nodo basado en Linux.

> [!TIP]
> En el siguiente ejemplo se crea un espacio de nombres de Kubernetes para los recursos de entrada denominado *ingress-basic*. Especifique un espacio de nombres para su propio entorno según sea necesario.

> [!TIP]
> Si quiere habilitar la [conservación de direcciones IP de origen del cliente][client-source-ip] para las solicitudes a los contenedores de su clúster, agregue `--set controller.service.externalTrafficPolicy=Local` al comando de instalación de Helm. La dirección IP de origen del cliente se almacena en el encabezado de la solicitud en *X-Forwarded-For*. Al usar un controlador de entrada con la conservación de direcciones IP de origen del cliente habilitada, el paso a través de TLS no funciona.

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

Durante la instalación se crea una dirección IP pública de Azure para el controlador de entrada. Esta dirección IP pública es estática durante el período de vida del controlador de entrada. Si elimina el controlador de entrada, se pierde la asignación de dirección IP pública. Si después crea un controlador de entrada adicional, se asigna una dirección IP pública nueva. Si quiere conservar el uso de la dirección IP pública, en su lugar puede [crear un controlador de entrada con una dirección IP pública estática][aks-ingress-static-tls].

Para obtener la dirección IP pública, use el comando `kubectl get service`. La asignación de la dirección IP al servicio puede tardar hasta un minuto.

```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

Todavía no se creó ninguna regla de entrada. Si navega a la dirección IP pública, se muestra la página 404 predeterminada del controlador de entrada NGINX.

## <a name="add-an-a-record-to-your-dns-zone"></a>Adición de un registro A a la zona DNS

Agregue un registro *A* a la zona DNS con la dirección IP externa del servicio de NGINX mediante [az network dns record-set a add-record][az-network-dns-record-set-a-add-record].

```console
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name '*' \
    --ipv4-address MY_EXTERNAL_IP
```

> [!NOTE]
> Si quiere, puede configurar un FQDN para la dirección IP del controlador de entrada en lugar de un dominio personalizado. Tenga en cuenta que este ejemplo es para un shell de Bash.
> 
> ```azurecli-interactive
> # Public IP address of your ingress controller
> IP="MY_EXTERNAL_IP"
> 
> # Name to associate with public IP address
> DNSNAME="demo-aks-ingress"
> 
> # Get the resource-id of the public ip
> PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)
> 
> # Update public ip address with DNS name
> az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
> 
> # Display the FQDN
> az network public-ip show --ids $PUBLICIPID --query "[dnsSettings.fqdn]" --output tsv
> ```

## <a name="install-cert-manager"></a>Instalar cert-manager

El controlador de entrada NGINX es compatible con la terminación de TLS. Hay varias maneras de recuperar y configurar certificados para HTTPS. En este artículo se muestra cómo utilizar [cert manager][cert-manager], que proporciona la generación automática de certificados [Lets Encrypt][lets-encrypt] y la funcionalidad de administración.

Para instalar el controlador cert-manager:

```console
# Label the ingress-basic namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager \
  --namespace ingress-basic \
  --version v0.16.1 \
  --set installCRDs=true \
  --set nodeSelector."beta\.kubernetes\.io/os"=linux \
  jetstack/cert-manager
```

Para obtener más información sobre la configuración cert-manager, consulte el [proyecto cert-manager][cert-manager].

## <a name="create-a-ca-cluster-issuer"></a>Creación de un emisor de clúster de entidad de certificación

Para poder emitir certificados, cert-manager requiere un recurso [Issuer][cert-manager-issuer] o [ClusterIssuer][cert-manager-cluster-issuer]. La funcionalidad de los recursos de Kubernetes es idéntica, pero `Issuer` funciona en un espacio de nombres único, mientras que `ClusterIssuer` funciona en todos los espacios de nombres. Para obtener más información, consulte la documentación de [cert-manager issuer][cert-manager-issuer].

Cree un emisor de clúster, como `cluster-issuer.yaml`, con el manifiesto de ejemplo siguiente. Actualice la dirección de correo electrónico con una dirección válida de su organización:

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx
          podTemplate:
            spec:
              nodeSelector:
                "kubernetes.io/os": linux
```

Para crear el emisor, use el comando `kubectl apply`.

```console
kubectl apply -f cluster-issuer.yaml
```

## <a name="run-demo-applications"></a>Ejecución de aplicaciones de demostración

Se deben haber configurado una solución de administración de certificados y un controlador de entrada. Ahora vamos a ejecutar dos aplicaciones de demostración en el clúster de AKS. En este ejemplo, Helm se usa para implementar dos instancias de una aplicación *Hola mundo* sencilla.

Para ver el controlador de entrada en acción, ejecute dos aplicaciones de demostración en el clúster de AKS. En este ejemplo, use `kubectl apply` para implementar dos instancias de una aplicación *Hola mundo* sencilla.

Cree un archivo *aks-helloworld-one.yaml* y cópielo en el ejemplo siguiente de YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
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
  name: aks-helloworld-one
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
```

Cree un archivo *aks-helloworld-two.yaml* y cópielo en el ejemplo siguiente de YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-two
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-two
  template:
    metadata:
      labels:
        app: aks-helloworld-two
    spec:
      containers:
      - name: aks-helloworld-two
        image: neilpeterson/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "AKS Ingress Demo"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-two
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-two
```

Ejecute las dos aplicaciones de demostración mediante `kubectl apply`:

```console
kubectl apply -f aks-helloworld-one.yaml --namespace ingress-basic
kubectl apply -f aks-helloworld-two.yaml --namespace ingress-basic
```

## <a name="create-an-ingress-route"></a>Creación de una ruta de entrada

Ambas aplicaciones ahora se ejecutan en el clúster de Kubernetes. Sin embargo, están configuradas con un servicio de tipo `ClusterIP` y no son accesibles desde Internet. Para que estén disponibles de manera pública, cree un recurso de entrada de Kubernetes. El recurso de entrada configura las reglas de enrutamiento del tráfico a una de las dos aplicaciones.

En el ejemplo siguiente, el tráfico a la dirección *hello-world-ingress.MY_CUSTOM_DOMAIN* se enruta al servicio *aks-helloworld*. El tráfico a la dirección *hello-world-ingress.MY_CUSTOM_DOMAIN/hello-world-two* se enruta al servicio *aks-helloworld-two*. El tráfico a *hello-world-ingress.MY_CUSTOM_DOMAIN/static* se enruta al servicio denominado *aks-helloworld*  para los recursos estáticos.

> [!NOTE]
> Si ha configurado FQDN para la dirección IP del controlador de entrada en lugar de un dominio personalizado, use FQDN en lugar de *hello-world-ingress.MY_CUSTOM_DOMAIN*. Por ejemplo, si FQDN es *demo-aks-ingress.eastus.cloudapp.azure.com*, reemplace *hello-world-ingress.MY_CUSTOM_DOMAIN* por *demo-aks-ingress.eastus.cloudapp.azure.com* en `hello-world-ingress.yaml`.

Cree un archivo denominado `hello-world-ingress.yaml` mediante el ejemplo de YAML siguiente. Actualice los *hosts* y el *host* al nombre DNS que creó en un paso anterior.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - hello-world-ingress.MY_CUSTOM_DOMAIN
    secretName: tls-secret
  rules:
  - host: hello-world-ingress.MY_CUSTOM_DOMAIN
    http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /hello-world-one(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-two
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /(.*)
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress-static
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /static/$2
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - hello-world-ingress.MY_CUSTOM_DOMAIN
    secretName: tls-secret
  rules:
  - host: hello-world-ingress.MY_CUSTOM_DOMAIN
    http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /static(/|$)(.*)
```

Cree el recurso de entrada con el comando `kubectl apply`.

```console
kubectl apply -f hello-world-ingress.yaml --namespace ingress-basic
```

## <a name="verify-a-certificate-object-has-been-created"></a>Verificación de la creación de un objeto de certificado

A continuación, se debe crear un recurso de certificado. El recurso de certificado define el certificado X.509 deseado. Para obtener más información, consulte los [certificados de cert-manager][cert-manager-certificates]. Es probable que el elemento cert-manager haya creado automáticamente un objeto de certificado mediante las correcciones de compatibilidad (shim) de entrada, que se implementan automáticamente con el elemento cert-manager desde v0.2.2. Para obtener más información, consulte la [documentación sobre corrección de compatibilidad (shim) de entrada][ingress-shim].

Para verificar que el certificado se haya creado correctamente, use el comando `kubectl get certificate --namespace ingress-basic` y compruebe que la opción *LISTO*  esté establecida en *True*, lo que puede tardar varios minutos.

```
$ kubectl get certificate --namespace ingress-basic

NAME         READY   SECRET       AGE
tls-secret   True    tls-secret   11m
```

## <a name="test-the-ingress-configuration"></a>Prueba de la configuración de entrada

Abra un explorador web a la dirección *hello-world-ingress.MY_CUSTOM_DOMAIN* del controlador de entrada de Kubernetes. Tenga en cuenta que se le redirige a usar HTTPS, que el certificado es de confianza y la aplicación de demostración se muestra en el explorador web. Agregue la ruta de acceso */hello-world-two* y observe que se muestra la segunda aplicación de demostración con el título personalizado.

## <a name="clean-up-resources"></a>Limpieza de recursos

En este artículo, se usa Helm para instalar los componentes de entrada, los certificados y las aplicaciones de ejemplo. Al implementar un gráfico de Helm, se crean algunos recursos de Kubernetes. Estos recursos incluyen pods, implementaciones y servicios. Para limpiar estos recursos, puede eliminar el espacio de nombres de ejemplo completo o los recursos individuales.

### <a name="delete-the-sample-namespace-and-all-resources"></a>Eliminación del espacio de nombres de ejemplo y de todos los recursos

Para eliminar el espacio de nombres de ejemplo completo, use el comando `kubectl delete` y especifique el nombre del espacio de nombres. Todos los recursos del espacio de nombres se eliminan.

```console
kubectl delete namespace ingress-basic
```

### <a name="delete-resources-individually"></a>Eliminación de recursos individualmente

Como alternativa, un enfoque más pormenorizado consiste en eliminar los recursos individuales creados. En primer lugar, quite los recursos del emisor del clúster:

```console
kubectl delete -f cluster-issuer.yaml --namespace ingress-basic
```

Despliegue una lista de las versiones de Helm con el comando `helm list`. Busque los gráficos denominados *nginx* y *cert-manager*, tal y como se muestra en la salida del ejemplo siguiente:

```
$ helm list --namespace ingress-basic

NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
cert-manager            ingress-basic   1               2020-01-15 10:23:36.515514 -0600 CST    deployed        cert-manager-v0.13.0    v0.13.0    
nginx                   ingress-basic   1               2020-01-15 10:09:45.982693 -0600 CST    deployed        nginx-ingress-1.29.1    0.27.0  
```

Desinstale las versiones con el comando `helm uninstall`. En el ejemplo siguiente se desinstalan las implementaciones de entrada de NGINX y de cert-manager.

```
$ helm uninstall cert-manager nginx --namespace ingress-basic

release "cert-manager" uninstalled
release "nginx" uninstalled
```

Luego, quite las dos aplicaciones de ejemplo:

```console
kubectl delete -f aks-helloworld-one.yaml --namespace ingress-basic
kubectl delete -f aks-helloworld-two.yaml --namespace ingress-basic
```

Elimine la ruta de entrada que dirige el tráfico a las aplicaciones de ejemplo:

```console
kubectl delete -f hello-world-ingress.yaml --namespace ingress-basic
```

Por último, puede eliminar el propio espacio de nombres. Use el comando `kubectl delete` y especifique el nombre del espacio de nombres:

```console
kubectl delete namespace ingress-basic
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se incluyen algunos componentes externos a AKS. Para más información sobre estos componentes, consulte las siguientes páginas del proyecto:

- [CLI de Helm][helm-cli]
- [Controlador de entrada NGINX][nginx-ingress]
- [cert-manager][cert-manager]

También puede:

- [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
- [Habilitación del complemento de enrutamiento de aplicación HTTP][aks-http-app-routing]
- [Creación de un controlador de entrada que use una red privada interna y una dirección IP][aks-ingress-internal]
- [Crear un controlador de entrada que usa sus propios certificados TLS][aks-ingress-own-tls]
- [Crear un controlador de entrada que usa Let's Encrypt para generar automáticamente certificados TLS con una dirección IP pública estática][aks-ingress-static-tls]

<!-- LINKS - external -->
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-add-record
[custom-domain]: ../app-service/manage-custom-dns-buy-domain.md#buy-the-domain
[dns-zone]: ../dns/dns-getstarted-cli.md
[helm]: https://helm.sh/
[helm-cli]: ./kubernetes-helm.md
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[ingress-shim]: https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[client-source-ip]: concepts-network.md#ingress-controllers
[install-azure-cli]: /cli/azure/install-azure-cli
