---
title: Uso de un controlador de entrada de Traefik personalizado y configuración de HTTPS
services: azure-dev-spaces
ms.date: 12/10/2019
ms.topic: conceptual
description: Aprenda a configurar Azure Dev Spaces para usar un controlador de entrada de Traefik personalizado y configurar HTTPS con ese controlador de entrada.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s
ms.custom: devx-track-javascript
ms.openlocfilehash: e6c50dd8d27af8276ddbf47f7a3719ab59d87a7d
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421593"
---
# <a name="use-a-custom-traefik-ingress-controller-and-configure-https"></a>Uso de un controlador de entrada de Traefik personalizado y configuración de HTTPS

En este artículo se muestra cómo configurar Azure Dev Spaces para usar un controlador de entrada de Traefik personalizado. En este artículo también se muestra cómo configurar el controlador de entrada personalizado para usar HTTPS.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. En caso de no tener ninguna, puede crear una [cuenta gratuita][azure-account-create].
* [La CLI de Azure instalada][az-cli].
* [Clúster de Azure Kubernetes Service (AKS) con Azure Dev Spaces habilitado][qs-cli].
* [kubectl][kubectl] instalado.
* [Helm 3 instalado][helm-installed].
* [Un dominio personalizado][custom-domain] con una [zona DNS][dns-zone]. En este artículo se da por supuesto que el dominio personalizado y la zona DNS están en el mismo grupo de recursos que el clúster de AKS, pero es posible usar un dominio personalizado y una zona DNS de otro grupo de recursos.

## <a name="configure-a-custom-traefik-ingress-controller"></a>Configuración de un controlador de entrada de Traefik personalizado

Conecte el clúster con [kubectl][kubectl], el cliente de la línea de comandos de Kubernetes. Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKS
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```console
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
```

Agregue el [repositorio Helm estable oficial][helm-stable-repo], que contiene el gráfico de Traefik del controlador de entrada de Helm.

```console
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Cree un espacio de nombres Kubernetes para el controlador de entrada de Traefik e instálelo mediante `helm`.

> [!NOTE]
> Si el clúster de AKS no tiene habilitado RBAC, quite el parámetro *--set rbac.enabled=true*.

```console
kubectl create ns traefik
helm install traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set fullnameOverride=customtraefik --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0
```

> [!NOTE]
> En el ejemplo anterior se crea un punto de conexión público para el controlador de entrada. Si en su lugar necesita usar un punto de conexión privado para el controlador de entrada, agregue el parámetro *--set service.annotations."service\\.beta\\.kubernetes\\.io/azure-load-balancer-internal"=true* al comando *helm install*.
> ```console
> helm install traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set fullnameOverride=customtraefik --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --set service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true --version 1.85.0
> ```
> Este punto de conexión privado se expone dentro de la red virtual donde se implementa el clúster de AKS.

Obtenga la dirección IP del servicio del controlador de entrada de Traefik mediante [kubectl get][kubectl-get].

```console
kubectl get svc -n traefik --watch
```

La salida de ejemplo muestra las direcciones IP de todos los servicios en el espacio de nombres de *Traefik*.

```console
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
traefik   LoadBalancer   10.0.205.78   <pending>     80:32484/TCP,443:30620/TCP   20s
...
traefik   LoadBalancer   10.0.205.78   MY_EXTERNAL_IP   80:32484/TCP,443:30620/TCP   60s
```

Agregue un registro *A* a la zona DNS con la dirección IP externa del servicio de Traefik mediante [az network dns record-set a add-record][az-network-dns-record-set-a-add-record].

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address MY_EXTERNAL_IP
```

En el ejemplo anterior se agrega un registro *A* a la zona DNS *MY_CUSTOM_DOMAIN*.

En este artículo, puede usar la [aplicación de ejemplo Bike Sharing de Azure Dev Spaces](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp) para demostrar el uso de Azure Dev Spaces. Clone la aplicación desde GitHub y vaya a su directorio:

```cmd
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/BikeSharingApp/charts
```

Abra [values.yaml][values-yaml] y realice las siguientes actualizaciones:
* Reemplace todas las instancias de *<REPLACE_ME_WITH_HOST_SUFFIX>* por *traefik.MY_CUSTOM_DOMAIN* mediante el dominio de *MY_CUSTOM_DOMAIN*. 
* Reemplace *kubernetes.io/ingress.class: traefik-azds  # Dev Spaces-specific* por *kubernetes.io/ingress.class: traefik  # Custom Ingress*. 

A continuación se muestra un ejemplo de un archivo `values.yaml` actualizado:

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
    hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
    hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
```

Guarde los cambios y cierre el archivo.

Cree el espacio *dev* con la aplicación de ejemplo mediante `azds space select`.

```console
azds space select -n dev -y
```

Implemente la aplicación de ejemplo mediante `helm install`.

```console
helm install bikesharingsampleapp . --dependency-update --namespace dev --atomic
```

En el ejemplo anterior se implementa la aplicación de ejemplo en el espacio de nombres *dev*.

Muestre las direcciones URL para acceder a la aplicación de ejemplo mediante `azds list-uris`.

```console
azds list-uris
```

La siguiente salida muestra las direcciones URL de ejemplo de `azds list-uris`.

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/  Available
http://dev.gateway.traefik.MY_CUSTOM_DOMAIN/         Available
```

Navegue hasta el servicio *bikesharingweb* abriendo la dirección URL pública con el comando `azds list-uris`. En el ejemplo anterior, la dirección URL pública para el servicio *bikesharingweb* es `http://dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/`.

> [!NOTE]
> Si ve una página de error en lugar del servicio *bikesharingweb*, compruebe que ha actualizado **tanto** la anotación *kubernetes.io/ingress.class* como el host en el archivo *values.yaml*.

Use el comando `azds space select` para crear un espacio secundario en *dev* y mostrar las direcciones URL para acceder al espacio de desarrollo secundario.

```console
azds space select -n dev/azureuser1 -y
azds list-uris
```

La salida siguiente muestra las direcciones URL de ejemplo de `azds list-uris` para tener acceso a la aplicación de ejemplo en el espacio de desarrollo secundario *azureuser1*.

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/  Available
http://azureuser1.s.dev.gateway.traefik.MY_CUSTOM_DOMAIN/         Available
```

Navegue hasta el servicio *bikesharingweb* en el espacio de desarrollo secundario *azureuser1* abriendo la dirección URL pública con el comando `azds list-uris`. En el ejemplo anterior, la dirección URL pública para el servicio *bikesharingweb* en el espacio de desarrollo secundario *azureuser1* es `http://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/`.

## <a name="configure-the-traefik-ingress-controller-to-use-https"></a>Configuración del controlador de entrada de Traefik para usar HTTPS

Use [cert-manager][cert-manager] para automatizar la administración del certificado TLS al configurar el controlador de entrada de Traefik para que use HTTPS. Use `helm` para instalar el gráfico *certmanager*.

```console
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace traefik
kubectl label namespace traefik certmanager.k8s.io/disable-validation=true
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace traefik --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
```

Cree un archivo `letsencrypt-clusterissuer.yaml` y actualice el campo de correo electrónico con la dirección de correo electrónico.

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
            class: traefik
```

> [!NOTE]
> Para las pruebas, también hay un [servidor de ensayo][letsencrypt-staging-issuer] que puede usar para *ClusterIssuer*.

Use `kubectl` para aplicar `letsencrypt-clusterissuer.yaml`.

```console
kubectl apply -f letsencrypt-clusterissuer.yaml --namespace traefik
```

Quite los *traefik* *ClusterRole* y *ClusterRoleBinding* anteriores y, a continuación, actualice traefik para usar HTTPS mediante `helm`.

> [!NOTE]
> Si el clúster de AKS no tiene habilitado RBAC, quite el parámetro *--set rbac.enabled=true*.

```console
kubectl delete ClusterRole traefik
kubectl delete ClusterRoleBinding traefik
helm upgrade traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0 --set ssl.enabled=true --set ssl.enforced=true --set ssl.permanentRedirect=true
```

Obtenga la dirección IP actualizada del servicio del controlador de entrada de Traefik mediante [kubectl get][kubectl-get].

```console
kubectl get svc -n traefik --watch
```

La salida de ejemplo muestra las direcciones IP de todos los servicios en el espacio de nombres de *Traefik*.

```console
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP          PORT(S)                      AGE
traefik   LoadBalancer   10.0.205.78   <pending>            80:32484/TCP,443:30620/TCP   20s
...
traefik   LoadBalancer   10.0.205.78   MY_NEW_EXTERNAL_IP   80:32484/TCP,443:30620/TCP   60s
```

Agregue un registro *A* a la zona DNS con la dirección IP externa del servicio de Traefik mediante [az network dns record-set a add-record][az-network-dns-record-set-a-add-record] y quite el registro *A* anterior mediante [az network dns record-set a remove-record][az-network-dns-record-set-a-remove-record].

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address MY_NEW_EXTERNAL_IP

az network dns record-set a remove-record \
    --resource-group myResourceGroup \
    --zone-name  MY_CUSTOM_DOMAIN \
    --record-set-name *.traefik \
    --ipv4-address PREVIOUS_EXTERNAL_IP
```

En el ejemplo anterior se actualiza el registro *A* en la zona DNS *MY_CUSTOM_DOMAIN* para usar *PREVIOUS_EXTERNAL_IP*.

Actualice [values.yaml][values-yaml] para incluir los detalles de uso de *cert-manager* y HTTPS. A continuación se muestra un ejemplo de un archivo `values.yaml` actualizado:

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN
      secretName: dev-bikesharingweb-secret

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: traefik  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.gateway.traefik.MY_CUSTOM_DOMAIN
      secretName: dev-gateway-secret
```

Actualice la aplicación de ejemplo mediante `helm`:

```console
helm upgrade bikesharingsampleapp . --namespace dev --atomic
```

Navegue hasta la aplicación de ejemplo en el espacio secundario *dev/azureuser1* y observe que se le redirige para usar HTTPS.

> [!IMPORTANT]
> Los cambios en DNS pueden tardar 30 minutos o más en completarse, lo mismo que su aplicación de ejemplo en estar accesible.

Observe también que la página se carga, pero el explorador muestra algunos errores. Al abrir la consola del explorador, se muestra el error relacionado con una página HTTPS que intenta cargar recursos HTTP. Por ejemplo:

```console
Mixed Content: The page at 'https://azureuser1.s.dev.bikesharingweb.traefik.MY_CUSTOM_DOMAIN/devsignin' was loaded over HTTPS, but requested an insecure resource 'http://azureuser1.s.dev.gateway.traefik.MY_CUSTOM_DOMAIN/api/user/allUsers'. This request has been blocked; the content must be served over HTTPS.
```

Para corregir este error, actualice [BikeSharingWeb/azds.yaml][azds-yaml] para usar *traefik* para *kubernetes.io/ingress.class* y el dominio personalizado para *$(hostSuffix)* . Por ejemplo:

```yaml
...
    ingress:
      annotations:
        kubernetes.io/ingress.class: traefik
      hosts:
      # This expands to [space.s.][rootSpace.]bikesharingweb.<random suffix>.<region>.azds.io
      - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.traefik.MY_CUSTOM_DOMAIN
...
```

Actualice [BikeSharingWeb/package.json][package-json] con una dependencia para el paquete *url*.

```json
{
...
    "react-responsive": "^6.0.1",
    "universal-cookie": "^3.0.7",
    "url": "0.11.0"
  },
...
```

Actualice el método *getApiHostAsync* en [BikeSharingWeb/lib/helpers.js][helpers-js] para usar HTTPS:

```javascript
...
    getApiHostAsync: async function() {
        const apiRequest = await fetch('/api/host');
        const data = await apiRequest.json();
        
        var urlapi = require('url');
        var url = urlapi.parse(data.apiHost);

        console.log('apiHost: ' + "https://"+url.host);
        return "https://"+url.host;
    },
...
```

Navegue hasta el directorio `BikeSharingWeb` y use `azds up` para ejecutar el servicio de BikeSharingWeb actualizado.

```console
cd ../BikeSharingWeb/
azds up
```

Navegue hasta la aplicación de ejemplo en el espacio secundario *dev/azureuser1* y observe que se le redirige para usar HTTPS sin ningún error.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de la forma en que Azure Dev Spaces le ayuda a desarrollar aplicaciones más complejas en varios contenedores y cómo puede simplificar el desarrollo en colaboración mediante el uso de distintas versiones o bifurcaciones del código en distintos espacios.

> [!div class="nextstepaction"]
> [Desarrollo en equipo en Azure Dev Spaces][team-development-qs]


[az-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-add-record
[az-network-dns-record-set-a-remove-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-remove-record
[custom-domain]: ../../app-service/manage-custom-dns-buy-domain.md#buy-the-domain
[dns-zone]: ../../dns/dns-getstarted-cli.md
[qs-cli]: ../quickstart-cli.md
[team-development-qs]: ../quickstart-team-development.md

[azds-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/azds.yaml
[azure-account-create]: https://azure.microsoft.com/free
[cert-manager]: https://cert-manager.io/
[helm-installed]: https://helm.sh/docs/intro/install/
[helm-stable-repo]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helpers-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/lib/helpers.js#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[letsencrypt-staging-issuer]: https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer
[package-json]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/package.json
[values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml