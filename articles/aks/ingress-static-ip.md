---
title: Uso del controlador de entrada con una IP estática
titleSuffix: Azure Kubernetes Service
description: Aprenda a instalar y configurar un controlador de entrada NGINX con una dirección IP pública estática en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: be4856beac69d11de12ec764f313fa59f3b24e9f
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89290555"
---
# <a name="create-an-ingress-controller-with-a-static-public-ip-address-in-azure-kubernetes-service-aks"></a>Cree un controlador de entrada con una dirección IP pública estática en Azure Kubernetes Service (AKS)

Un controlador de entrada es un software que proporciona el proxy inverso, el enrutamiento del tráfico configurable y la terminación de TLS para los servicios de Kubernetes. Los recursos de entrada de Kubernetes se usan para configurar las reglas de entrada y las rutas de los distintos servicios de Kubernetes. Mediante reglas de entrada y un controlador de entrada, se puede usar una sola dirección IP para enrutar el tráfico a varios servicios en un clúster de Kubernetes.

En este artículo se muestra cómo implementar el [controlador de entrada NGINX][nginx-ingress] en un clúster de Azure Kubernetes Service (AKS). El controlador de entrada se configura con una dirección IP pública estática. El proyecto [cert-manager][cert-manager] se usa para generar y configurar automáticamente certificados [Let's Encrypt][lets-encrypt]. Por último, en el clúster de AKS se ejecutan dos aplicaciones, a las que se puede acceder con una sola dirección IP.

También puede:

- [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
- [Habilitación del complemento de enrutamiento de aplicación HTTP][aks-http-app-routing]
- [Crear un controlador de entrada que usa sus propios certificados TLS][aks-ingress-own-tls]
- [Crear un controlador de entrada que usa Let's Encrypt para generar automáticamente certificados TLS con una dirección IP pública dinámica][aks-ingress-tls]

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

En este artículo se usa [Helm 3][helm] para instalar el controlador de entrada NGINX y cert-manager. Asegúrese de que usa la versión más reciente de Helm y de que tiene acceso a los repositorios de Helm *ingress-nginx* y *jetstack*. Para instrucciones de actualización, consulte la [documentación de instalación de Helm][helm-install]. Para obtener más información sobre cómo configurar y usar Helm, consulte [Instalación de aplicaciones con Helm en Azure Kubernetes Service (AKS)][use-helm].

En este artículo también se requiere que ejecute la versión 2.0.64 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="create-an-ingress-controller"></a>Crear un controlador de entrada

De manera predeterminada, se crea un controlador de entrada NGINX con una asignación de una nueva dirección IP pública. Esta dirección IP pública solo es estática mientras esté vigente el controlador de entrada y se pierde si el controlador se elimina y se vuelve a crear. Un requisito de configuración común es proporcionar una dirección IP pública estática existente al controlador de entrada NGINX. La dirección IP pública estática se conserva si se elimina el controlador de entrada. Este enfoque permite usar los registros DNS y las configuraciones de red existentes de manera coherente durante el ciclo de vida de las aplicaciones.

Si debe crear una dirección IP pública estática, primero obtenga el nombre del grupo de recursos del clúster AKS con el comando [az aks show][az-aks-show]:

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

Después, cree una dirección IP pública con el método de asignación *estático* mediante el comando [az network public-ip create][az-network-public-ip-create]. En el ejemplo siguiente se crea una dirección IP pública denominada *myAKSPublicIP* en el grupo de recursos del clúster AKS obtenido en el paso anterior:

```azurecli-interactive
az network public-ip create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv
```

> [!NOTE]
> Los comandos anteriores crean una dirección IP que se eliminará si elimina el clúster de AKS. Como alternativa, puede crear una dirección IP en otro grupo de recursos que se pueda administrar de forma independiente del clúster de AKS. Si crea una dirección IP en otro grupo de recursos, asegúrese de que la entidad de servicio usada por el clúster de AKS tenga permisos delegados para el otro grupo de recursos, como *Colaborador de la red*. Para obtener más información, consulte [Uso de una dirección IP pública estática y una etiqueta DNS con el equilibrador de carga de AKS][aks-static-ip].

Ahora implemente el gráfico *nginx-ingress* con Helm. Para obtener redundancia adicional, se implementan dos réplicas de los controladores de entrada NGINX con el parámetro `--set controller.replicaCount`. Para sacar el máximo provecho de las réplicas en ejecución del controlador de entrada, asegúrese de que hay más de un nodo en el clúster de AKS.

Debe pasar dos parámetros adicionales a la versión de Helm para que el controlador de entrada tenga en cuenta la dirección IP estática del equilibrador de carga que se va a asignar al servicio de controlador de entrada y la etiqueta de nombre DNS que se está aplicando al recurso de dirección IP pública. Para que los certificados HTTPS funcionen correctamente, se usa una etiqueta de nombre DNS para configurar un FQDN para la dirección IP del controlador de entrada.

1. Agregue el parámetro `--set controller.service.loadBalancerIP`. Especifique su propia dirección IP pública creada en el paso anterior.
1. Agregue el parámetro `--set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"`. Especifique una etiqueta de nombre DNS para aplicarla a la dirección IP pública que se creó en el paso anterior.

El controlador de entrada también debe programarse en un nodo de Linux. Los nodos de Windows Server no deben ejecutar el controlador de entrada. Un selector de nodos se especifica mediante el parámetro `--set nodeSelector` para indicar al programador de Kubernetes que ejecute el controlador de entrada NGINX en un nodo basado en Linux.

> [!TIP]
> En el siguiente ejemplo se crea un espacio de nombres de Kubernetes para los recursos de entrada denominado *ingress-basic*. Especifique un espacio de nombres para su propio entorno según sea necesario. Si su clúster de AKS no tiene RBAC habilitado, agregue `--set rbac.create=false` a los comandos de Helm.

> [!TIP]
> Si quiere habilitar la [conservación de direcciones IP de origen del cliente][client-source-ip] para las solicitudes a los contenedores de su clúster, agregue `--set controller.service.externalTrafficPolicy=Local` al comando de instalación de Helm. La dirección IP de origen del cliente se almacena en el encabezado de la solicitud en *X-Forwarded-For*. Al usar un controlador de entrada con la conservación de direcciones IP de origen del cliente habilitada, el paso a través de TLS no funciona.

Actualice el siguiente script con la **dirección IP** del controlador de entrada y el **nombre único** que le gustaría usar para el prefijo de FQDN.

> [!IMPORTANT]
> Debe reemplazar mediante actualización *STATIC_IP* y *DNS_LABEL* por su propia dirección IP y nombre único cuando ejecute el comando.

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
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.service.loadBalancerIP="STATIC_IP" \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"="DNS_LABEL"
```

Cuando se crea el servicio del equilibrador de carga de Kubernetes para el controlador de entrada NGINX, se asigna la dirección IP estática, como se muestra en la salida del ejemplo siguiente:

```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

No se han creado reglas de entrada aún, por lo que aparece la página 404 predeterminada del controlador de entrada NGINX si navega a la dirección IP pública. Las reglas de entrada se configuran en los pasos siguientes.

Para comprobar que se ha aplicado la etiqueta de nombre DNS, consulte el FQDN en la dirección IP pública como se indica a continuación:

```azurecli-interactive
az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query "[?name=='myAKSPublicIP'].[dnsSettings.fqdn]" -o tsv
```

El controlador de entrada ahora es accesible mediante la dirección IP o el FQDN.

## <a name="install-cert-manager"></a>Instalar cert-manager

El controlador de entrada NGINX es compatible con la terminación de TLS. Hay varias maneras de recuperar y configurar certificados para HTTPS. En este artículo se muestra cómo utilizar [cert manager][cert-manager], que proporciona la generación automática de certificados [Lets Encrypt][lets-encrypt] y la funcionalidad de administración.

> [!NOTE]
> En este artículo se usa el entorno `staging` para Let's Encrypt. En las implementaciones de producción, use `letsencrypt-prod` y `https://acme-v02.api.letsencrypt.org/directory` en las definiciones de recursos y al instalar el gráfico de Helm.

Para instalar el controlador de cert-manager en un clúster habilitado para RBAC, use el comando `helm install` siguiente:

```console
# Label the cert-manager namespace to disable resource validation
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
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: user@contoso.com
    privateKeySecretRef:
      name: letsencrypt-staging
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

```
$ kubectl apply -f cluster-issuer.yaml --namespace ingress-basic

clusterissuer.cert-manager.io/letsencrypt-staging created
```

## <a name="run-demo-applications"></a>Ejecución de aplicaciones de demostración

Se deben haber configurado una solución de administración de certificados y un controlador de entrada. Ahora vamos a ejecutar dos aplicaciones de demostración en el clúster de AKS. En este ejemplo, Helm se usa para implementar dos instancias de una aplicación "Hola mundo" sencilla.

Para ver el controlador de entrada en acción, ejecute dos aplicaciones de demostración en el clúster de AKS. En este ejemplo, usar `kubectl apply` para implementar dos instancias de una aplicación *Hola mundo* sencilla.

Crear un archivo *aks-helloworld.yaml* y copiarlo en el ejemplo siguiente de YAML:

```yml
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
```

Crear un archivo *ingress-demo.yaml* y copiarlo en el ejemplo siguiente de YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-demo
  template:
    metadata:
      labels:
        app: ingress-demo
    spec:
      containers:
      - name: ingress-demo
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
  name: ingress-demo
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: ingress-demo
```

Ejecutar las dos aplicaciones de demostración mediante `kubectl apply`:

```console
kubectl apply -f aks-helloworld.yaml --namespace ingress-basic
kubectl apply -f ingress-demo.yaml --namespace ingress-basic
```

## <a name="create-an-ingress-route"></a>Creación de una ruta de entrada

Ambas aplicaciones se ejecutan ahora en el clúster de Kubernetes, sin embargo, están configuradas con un servicio de tipo `ClusterIP`. Por lo tanto, no se puede acceder a ellas desde Internet. Para que estén disponibles de manera pública, cree un recurso de entrada de Kubernetes. El recurso de entrada configura las reglas de enrutamiento del tráfico a una de las dos aplicaciones.

En el ejemplo siguiente, el tráfico a la dirección `https://demo-aks-ingress.eastus.cloudapp.azure.com/` se enruta al servicio denominado `aks-helloworld`. El tráfico a la dirección `https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two` se enruta al servicio `ingress-demo`. Actualice los *hosts* y el *host* al nombre DNS que creó en un paso anterior.

Cree un archivo denominado `hello-world-ingress.yaml` y cópielo en el ejemplo siguiente de YAML.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - demo-aks-ingress.eastus.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: demo-aks-ingress.eastus.cloudapp.azure.com
    http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /hello-world-one(/|$)(.*)
      - backend:
          serviceName: ingress-demo
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /(.*)
```

Cree el recurso de entrada con el comando `kubectl apply`.

```
$ kubectl apply -f hello-world-ingress.yaml --namespace ingress-basic

ingress.extensions/hello-world-ingress created
```

## <a name="create-a-certificate-object"></a>Creación de un objeto de certificado

A continuación, se debe crear un recurso de certificado. El recurso de certificado define el certificado X.509 deseado. Para obtener más información, consulte los [certificados de cert-manager][cert-manager-certificates].

Es probable que el Administrador de certificados haya creado automáticamente un objeto de certificado mediante la corrección de compatibilidad (shim) de entrada, que se implementa automáticamente con el Administrador de certificados desde v0.2.2. Para obtener más información, consulte la [documentación sobre corrección de compatibilidad (shim) de entrada][ingress-shim].

Para comprobar que el certificado se creó correctamente, use el comando `kubectl describe certificate tls-secret --namespace ingress-basic`.

Si se ha emitido el certificado, verá un resultado similar al siguiente:
```
Type    Reason          Age   From          Message
----    ------          ----  ----          -------
  Normal  CreateOrder     11m   cert-manager  Created new ACME order, attempting validation...
  Normal  DomainVerified  10m   cert-manager  Domain "demo-aks-ingress.eastus.cloudapp.azure.com" verified with "http-01" validation
  Normal  IssueCert       10m   cert-manager  Issuing certificate...
  Normal  CertObtained    10m   cert-manager  Obtained certificate from ACME server
  Normal  CertIssued      10m   cert-manager  Certificate issued successfully
```

Si necesita crear un recurso de certificado adicional, puede hacerlo con el siguiente ejemplo de manifiesto. Actualice *dnsNames* y los *dominios* al nombre DNS que creó en un paso anterior. Si usa un controlador de entrada solo para uso interno, especifique el nombre DNS interno para el servicio.

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: tls-secret
  namespace: ingress-basic
spec:
  secretName: tls-secret
  dnsNames:
  - demo-aks-ingress.eastus.cloudapp.azure.com
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - demo-aks-ingress.eastus.cloudapp.azure.com
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

Para crear el recurso de certificado, use el comando `kubectl apply`.

```
$ kubectl apply -f certificates.yaml

certificate.cert-manager.io/tls-secret created
```

## <a name="test-the-ingress-configuration"></a>Prueba de la configuración de entrada

Abra un explorador web en el FQDN del controlador de entrada de Kubernetes, como *`https://demo-aks-ingress.eastus.cloudapp.azure.com`* .

Dado que estos ejemplos usan `letsencrypt-staging`, el explorador no confía en el certificado TLS/SSL emitido. Acepte el mensaje de advertencia para continuar a la aplicación. La información de certificado muestra que Let’s Encrypt emite este certificado *Fake LE Intermediate X1*. Este certificado falso indica que `cert-manager` procesó correctamente la solicitud y recibió un certificado del proveedor:

![Certificado de almacenamiento provisional de Let's Encrypt](media/ingress/staging-certificate.png)

Cuando modifica Let's Encrypt para usar `prod` en lugar de `staging`, se usa un certificado de confianza que Let's Encrypt emite, tal como se muestra en el ejemplo siguiente:

![Certificado de Let's Encrypt](media/ingress/certificate.png)

La aplicación de demostración se muestra en el explorador web:

![Primer ejemplo de aplicación](media/ingress/app-one.png)

A continuación, agregue la ruta de acceso */hello-world-two* al FQDN, como *`https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two`* . Se muestra la segunda aplicación de demostración con el título personalizado:

![Segundo ejemplo de aplicación](media/ingress/app-two.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

En este artículo, se usa Helm para instalar los componentes de entrada, los certificados y las aplicaciones de ejemplo. Al implementar un gráfico de Helm, se crean algunos recursos de Kubernetes. Estos recursos incluyen pods, implementaciones y servicios. Para limpiar estos recursos, puede eliminar el espacio de nombres de ejemplo completo o los recursos individuales.

### <a name="delete-the-sample-namespace-and-all-resources"></a>Eliminación del espacio de nombres de ejemplo y de todos los recursos

Para eliminar el espacio de nombres de ejemplo completo, use el comando `kubectl delete` y especifique el nombre del espacio de nombres. Todos los recursos del espacio de nombres se eliminan.

```console
kubectl delete namespace ingress-basic
```

### <a name="delete-resources-individually"></a>Eliminación de recursos individualmente

Como alternativa, un enfoque más pormenorizado consiste en eliminar los recursos individuales creados. Primero, quite los recursos de certificado:

```console
kubectl delete -f certificates.yaml
kubectl delete -f cluster-issuer.yaml
```

Ahora, despliegue una lista de las versiones de Helm con el comando `helm list`. Busque los gráficos denominados *nginx-ingress* y *cert-manager*, tal y como se muestra en la salida del ejemplo siguiente:

```
$ helm list --all-namespaces

NAME                    NAMESPACE       REVISION        UPDATED                        STATUS          CHART                   APP VERSION
nginx-ingress           ingress-basic   1               2020-01-11 14:51:03.454165006  deployed        nginx-ingress-1.28.2    0.26.2
cert-manager            ingress-basic   1               2020-01-06 21:19:03.866212286  deployed        cert-manager-v0.13.0    v0.13.0
```

Desinstale las versiones con el comando `helm uninstall`. En el ejemplo siguiente se desinstala la implementación de entrada de NGINX y las implementaciones del administrador de certificados.

```
$ helm uninstall nginx-ingress cert-manager -n ingress-basic

release "nginx-ingress" deleted
release "cert-manager" deleted
```

Luego, quite las dos aplicaciones de ejemplo:

```console
kubectl delete -f aks-helloworld.yaml --namespace ingress-basic
kubectl delete -f ingress-demo.yaml --namespace ingress-basic
```

Elimine el propio espacio de nombres. Use el comando `kubectl delete` y especifique el nombre del espacio de nombres:

```console
kubectl delete namespace ingress-basic
```

Por último, elimine la dirección IP pública estática que creó para el controlador de entrada. Proporcione su nombre del grupo de recursos de clúster *MC_* obtenido en el primer paso de este artículo; por ejemplo, *MC_miGrupoDeRecursos_miClusterAKS_eastus*:

```azurecli-interactive
az network public-ip delete --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP
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
- [Creación de un controlador de entrada con una dirección IP pública dinámica y configuración de Let's Encrypt para generar certificados TLS de forma automática][aks-ingress-tls]

<!-- LINKS - external -->
[helm-cli]: ./kubernetes-helm.md
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm]: https://helm.sh/
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm
[ingress-shim]: https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[client-source-ip]: concepts-network.md#ingress-controllers
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-static-ip]: static-ip.md
