---
title: Almacenamiento de gráficos de Helm
description: Información sobre cómo almacenar gráficos de Helm para las aplicaciones de Kubernetes mediante repositorios en Azure Container Registry
ms.topic: article
ms.date: 06/12/2020
ms.openlocfilehash: 69b16f35589586787e1c31a0e9755b9030af755d
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86537874"
---
# <a name="push-and-pull-helm-charts-to-an-azure-container-registry"></a>Inserción y extracción de gráficos de Helm en Azure Container Registry

Para administrar e implementar rápidamente aplicaciones de Kubernetes, puede usar el [administrador de paquetes de código abierto de Helm][helm]. Con Helm, los paquetes de aplicación se definen como [gráficos](https://helm.sh/docs/topics/charts/) que se recopilan y almacenan en un [repositorio de gráficos de Helm](https://helm.sh/docs/topics/chart_repository/).

En este artículo se muestra cómo hospedar repositorios gráficos de Helm en un registro de contenedor de Azure mediante comandos de Helm 3. En la mayoría de los escenarios, creará y cargará sus propios gráficos para las aplicaciones que desarrolla. Para obtener más información sobre cómo crear sus propios gráficos de Helm, consulte la [guía para desarrolladores de plantillas de gráficos][develop-helm-charts] (en inglés). También puede almacenar un gráfico de Helm existente desde otro repositorio de Helm.

> [!IMPORTANT]
> La compatibilidad con gráficos de Helm en Azure Container Registry está actualmente en versión preliminar. Las versiones preliminares están a su disposición a condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="helm-3-or-helm-2"></a>¿Helm 3 o Helm 2?

Para almacenar, administrar e instalar gráficos de Helm, se usa un cliente de Helm y la CLI de Helm. Las versiones principales del cliente de Helm incluyen Helm 3 y Helm 2. Para obtener más información sobre las diferencias de versión, vea las [preguntas más frecuentes sobre la versión](https://helm.sh/docs/faq/). 

Helm 3 se debe usar para hospedar gráficos de Helm en Azure Container Registry. Con Helm 3:

* Puede crear uno o varios repositorios de Helm en una instancia de Azure Container Registry.
* Almacene gráficos de Helm 3 en un registro como [artefactos OCI](container-registry-image-formats.md#oci-artifacts). Actualmente, la compatibilidad con Helm 3 para OCI es *experimental*.
* Autentíquese con el registro mediante el comando `helm registry login`.
* Use `helm chart` comandos de la CLI de Helm para insertar, extraer y administrar gráficos de Helm en un registro
* Use `helm install` para instalar gráficos en un clúster de Kubernetes desde una caché del repositorio local.
> [!NOTE]
> A partir de Helm 3, los comandos [az acr helm][az-acr-helm] que se usan con el cliente de Helm 2 están en desuso. Consulte la [Hoja de ruta del producto](https://github.com/Azure/acr/blob/master/docs/acr-roadmap.md#acr-helm-ga). Si ya ha implementado gráficos de Helm 2 anteriormente, consulte [Migración de Helm v2 a v3](https://helm.sh/docs/topics/v2_v3_migration/).

## <a name="prerequisites"></a>Requisitos previos

Los siguientes recursos son necesarios para el escenario de este artículo:

- **Una instancia de Azure Container Registry** en la suscripción a Azure. Si es necesario, cree un registro mediante [Azure Portal](container-registry-get-started-portal.md) o la [CLI de Azure](container-registry-get-started-azure-cli.md).
- **Versión de cliente de Helm 3.1.0 o posterior**: ejecute `helm version` para buscar la versión actual. Para más información sobre cómo instalar y usar Helm, consulte [Instalación de Helm][helm-install].
- **Un clúster de Kubernetes** donde instalar un gráfico de Helm. Si es necesario, cree [un clúster de Azure Kubernetes Service][aks-quickstart]. 
- **CLI de Azure versión 2.0.71 o posterior**: ejecute `az --version` para buscar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="enable-oci-support"></a>Habilitar la compatibilidad de OCI

Use el comando `helm version` para comprobar que tiene instalado Helm 3:

```console
helm version
```

Establezca la siguiente variable de entorno para habilitar la compatibilidad de OCI en el cliente de Helm 3. Actualmente, esta compatibilidad es experimental. 

```console
export HELM_EXPERIMENTAL_OCI=1
```

## <a name="create-a-sample-chart"></a>Crear un gráfico de ejemplo

Cree un gráfico de prueba con los siguientes comandos:

```console
mkdir helmtest

cd helmtest
helm create hello-world
```

Como ejemplo básico, cambie el directorio a la carpeta `templates` y, en primer lugar, elimine el contenido aquí:

```console
cd hello-world/templates
rm -rf *
```

En la carpeta `templates`, cree un archivo llamado `configmap.yaml` mediante la ejecución del comando siguiente:

```console
cat <<EOF > configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-configmap
data:
  myvalue: "Hello World"
EOF
```

Para más información sobre cómo crear y ejecutar este ejemplo, consulte [Introducción](https://helm.sh/docs/chart_template_guide/getting_started/) en los documentos de Helm.

## <a name="save-chart-to-local-registry-cache"></a>Guardar el gráfico en la caché del registro local

Cambie el directorio al subdirectorio `hello-world`. A continuación, ejecute `helm chart save` para guardar una copia del gráfico localmente y crear también un alias con el nombre completo del registro (todo en minúsculas) y el repositorio y la etiqueta de destino. 

En el ejemplo siguiente, el nombre del registro es *mycontainerregistry*, el repositorio de destino es *hello-world* y la etiqueta del gráfico de destino es *v1*, pero sustituye los valores de su entorno:

```console
cd ..
helm chart save . hello-world:v1
helm chart save . mycontainerregistry.azurecr.io/helm/hello-world:v1
```

Ejecute `helm chart list` para confirmar que ha guardado los gráficos en la caché del registro local. La salida es parecida a esta:

```console
REF                                                      NAME            VERSION DIGEST  SIZE            CREATED
hello-world:v1                                           hello-world       0.1.0   5899db0 3.2 KiB        2 minutes 
mycontainerregistry.azurecr.io/helm/hello-world:v1       hello-world       0.1.0   5899db0 3.2 KiB        2 minutes
```

## <a name="authenticate-with-the-registry"></a>Autenticación con el registro

Ejecute el comando `helm registry login` de la CLI de Helm 3 para [autenticarse con el del registro](container-registry-authentication.md) mediante las credenciales apropiadas para su escenario.

Por ejemplo, cree una entidad de servicio de Azure Active Directory [con permisos de extracción e inserción](container-registry-auth-service-principal.md#create-a-service-principal) (rol AcrPush) para el registro. A continuación, proporcione las credenciales de la entidad de servicio para `helm registry login`. En el ejemplo siguiente se proporciona la contraseña mediante una variable de entorno:

```console
echo $spPassword | helm registry login mycontainerregistry.azurecr.io \
  --username <service-principal-id> \
  --password-stdin
```

## <a name="push-chart-to-registry"></a>Inserción del gráfico en el registro

Ejecute el comando `helm chart push` de la CLI de Helm 3 para insertar el gráfico al repositorio de destino completo:

```console
helm chart push mycontainerregistry.azurecr.io/helm/hello-world:v1
```

Después de una inserción correcta, la salida es similar a:

```output
The push refers to repository [mycontainerregistry.azurecr.io/helm/hello-world]
ref:     mycontainerregistry.azurecr.io/helm/hello-world:v1
digest:  5899db028dcf96aeaabdadfa5899db025899db025899db025899db025899db02
size:    3.2 KiB
name:    hello-world
version: 0.1.0
```

## <a name="list-charts-in-the-repository"></a>Lista de gráficos en el repositorio

Al igual que con las imágenes almacenadas en una instancia de Azure Container Registry, puede usar comandos [az acr repository][az-acr-repository] para mostrar los repositorios que hospedan sus gráficos, así como los manifiestos y las etiquetas de estos. 

Por ejemplo, ejecute [az acr repository show][az-acr-repository-show] para ver las propiedades del repositorio que creó en el paso anterior:

```azurecli
az acr repository show \
  --name mycontainerregistry \
  --repository helm/hello-world
```

La salida es parecida a esta:

```output
{
  "changeableAttributes": {
    "deleteEnabled": true,
    "listEnabled": true,
    "readEnabled": true,
    "writeEnabled": true
  },
  "createdTime": "2020-03-20T18:11:37.6701689Z",
  "imageName": "helm/hello-world",
  "lastUpdateTime": "2020-03-20T18:11:37.7637082Z",
  "manifestCount": 1,
  "registry": "mycontainerregistry.azurecr.io",
  "tagCount": 1
}
```

Ejecute el comando [az acr repository show-manifests][az-acr-repository-show-manifests] para ver los detalles del gráfico almacenado en el repositorio. Por ejemplo:

```azurecli
az acr repository show-manifests \
  --name mycontainerregistry \
  --repository helm/hello-world --detail
```

La salida, abreviada en este ejemplo, muestra un valor de `configMediaType` de `application/vnd.cncf.helm.config.v1+json`:

```output
[
  {
    [...]
    "configMediaType": "application/vnd.cncf.helm.config.v1+json",
    "createdTime": "2020-03-20T18:11:37.7167893Z",
    "digest": "sha256:0c03b71c225c3ddff53660258ea16ca7412b53b1f6811bf769d8c85a1f0663ee",
    "imageSize": 3301,
    "lastUpdateTime": "2020-03-20T18:11:37.7167893Z",
    "mediaType": "application/vnd.oci.image.manifest.v1+json",
    "tags": [
      "v1"
    ]
```

## <a name="pull-chart-to-local-cache"></a>Extraer el gráfico en la caché local

Para instalar un gráfico de Helm en Kubernetes, el gráfico debe estar en la memoria caché local. En este ejemplo, ejecute primero `helm chart remove` para quitar el gráfico local existente denominado `mycontainerregistry.azurecr.io/helm/hello-world:v1`:

```console
helm chart remove mycontainerregistry.azurecr.io/helm/hello-world:v1
```

Ejecute `helm chart pull` para descargar el gráfico desde Azure Container Registry en la memoria caché local:

```console
helm chart pull mycontainerregistry.azurecr.io/helm/hello-world:v1
```

## <a name="export-helm-chart"></a>Exportar el gráfico de Helm

Para seguir trabajando con el gráfico, expórtelo a un directorio local mediante `helm chart export`. Por ejemplo, exporte el gráfico que extrajo al directorio `install`:

```console
helm chart export mycontainerregistry.azurecr.io/helm/hello-world:v1 \
  --destination ./install
```

Para ver información del gráfico exportado en el repositorio, ejecute el comando `helm show chart` en el directorio donde lo exportó.

```console
cd install
helm show chart hello-world
```

Helm devuelve información detallada sobre la versión más reciente del gráfico, tal como se muestra en la siguiente salida de ejemplo:

```output
apiVersion: v2
appVersion: 1.16.0
description: A Helm chart for Kubernetes
name: hello-world
type: application
version: 0.1.0    
```

## <a name="install-helm-chart"></a>Instalar el gráfico de Helm

Ejecute `helm install` para instalar el gráfico de Helm que extrajo en la caché local y exportó. Especifique un nombre de versión, como *myhelmtest* o bien pase el parámetro `--generate-name`. Por ejemplo:

```console
helm install myhelmtest ./hello-world
```

La salida después de la instalación correcta del gráfico es similar a:

```console
NAME: myhelmtest
LAST DEPLOYED: Fri Mar 20 14:14:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Para comprobar la instalación, ejecute el comando `helm get manifest`. 

```console
helm get manifest myhelmtest
```

El comando devuelve los datos de YAML en el archivo de plantilla `configmap.yaml`.

Ejecute `helm uninstall` para desinstalar la versión del gráfico en el clúster:

```console
helm uninstall myhelmtest
```

## <a name="delete-chart-from-the-registry"></a>Eliminación del gráfico del registro

Para eliminar un gráfico del registro de contenedor, use el comando [az acr repository delete][az-acr-repository-delete]. Ejecute el siguiente comando y confirme la operación cuando se le solicite:

```azurecli
az acr repository delete --name mycontainerregistry --image helm/hello-world:v1
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre cómo crear e implementar sus propios gráficos de Helm, consulte cómo [desarrollar gráficos de Helm][develop-helm-charts].
* Más información sobre la instalación de aplicaciones con Helm en [Azure Kubernetes Service (AKS)](../aks/kubernetes-helm.md).
* Los gráficos de helm pueden usarse como parte del proceso de compilación de contenedor. Para más información, consulte [Uso de Azure Container Registry Tasks][acr-tasks].

<!-- LINKS - external -->
[helm]: https://helm.sh/
[helm-install]: https://helm.sh/docs/intro/install/
[develop-helm-charts]: https://helm.sh/docs/chart_template_guide/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-quickstart]: ../aks/kubernetes-walkthrough.md
[acr-bestpractices]: container-registry-best-practices.md
[az-configure]: /cli/azure/reference-index#az-configure
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-helm]: /cli/azure/acr/helm
[az-acr-repository]: /cli/azure/acr/repository
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[acr-tasks]: container-registry-tasks-overview.md
