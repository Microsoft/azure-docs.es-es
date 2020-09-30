---
title: Uso de un contenedor de Computer Vision con Kubernetes y Helm
titleSuffix: Azure Cognitive Services
description: Aprenda a implementar el contenedor de Computer Vision con Kubernetes y Helm.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 9a8e0dde8b24c39180a584c26af725ab82ea0176
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907109"
---
# <a name="use-computer-vision-container-with-kubernetes-and-helm"></a>Uso de un contenedor de Computer Vision con Kubernetes y Helm

Una opción para administrar los contenedores de Computer Vision es usar Kubernetes y Helm. Vamos a crear un paquete de Kubernetes usando Kubernetes y Helm para definir una imagen de contenedor de Computer Vision. Este paquete se va a implementar en un clúster de Kubernetes en el entorno local. Por último, exploraremos cómo probar los servicios implementados. Para más información sobre la ejecución de contenedores de Docker sin orquestación de Kubernetes, consulte [Instalación y ejecución de contenedores de Computer Vision](computer-vision-how-to-install-containers.md).

## <a name="prerequisites"></a>Requisitos previos

Estos son los requisitos previos para poder usar contenedores de Computer Vision en el entorno local:

| Obligatorio | Propósito |
|----------|---------|
| Cuenta de Azure | Si no tiene una suscripción a Azure, cree una [cuenta gratuita][free-azure-account] antes de empezar. |
| CLI de Kubernetes | La [CLI de Kubernetes][kubernetes-cli] es necesaria para administrar las credenciales compartidas desde el registro de contenedor. Kubernetes también se necesita antes que Helm, que es el administrador de paquetes de Kubernetes. |
| CLI de Helm | Instale la [CLI de Helm][helm-install], que se usa para instalar un gráfico de Helm (definición de paquete de contenedor). |
| Recurso de Computer Vision |Para poder usar el contenedor, debe tener:<br><br>Un recurso de **Computer Vision** de Azure y la clave de API asociada con el URI del punto de conexión. Ambos valores están disponibles en las páginas de introducción y claves del recurso y son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**.|

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

[!INCLUDE [Container requirements and recommendations](includes/container-requirements-and-recommendations.md)]

## <a name="connect-to-the-kubernetes-cluster"></a>Conectar al clúster de Kubernetes

Se espera que el equipo host tenga un clúster de Kubernetes disponible. Vea este tutorial sobre la [implementación de un clúster de Kubernetes](../../aks/tutorial-kubernetes-deploy-cluster.md) para lograr un reconocimiento conceptual de cómo implementar un clúster de Kubernetes en un equipo host. Encontrará más información sobre las implementaciones en la [documentación de Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### <a name="sharing-docker-credentials-with-the-kubernetes-cluster"></a>Uso compartido de credenciales de Docker con el clúster de Kubernetes

Para permitir que el clúster de Kubernetes `docker pull` las imágenes configuradas del registro de contenedor `containerpreview.azurecr.io`, debe transferir las credenciales de Docker al clúster. Ejecute el comando [`kubectl create`][kubectl-create] siguiente para crear un *secreto de registro de Docker* basado en las credenciales proporcionadas en el requisito previo de acceso al registro de contenedor.

Ejecute el siguiente comando desde la interfaz de la línea de comandos que prefiera. Asegúrese de reemplazar `<username>`, `<password>` y `<email-address>` por las credenciales del registro de contenedor.

```console
kubectl create secret docker-registry containerpreview \
    --docker-server=containerpreview.azurecr.io \
    --docker-username=<username> \
    --docker-password=<password> \
    --docker-email=<email-address>
```

> [!NOTE]
> Si ya tiene acceso al registro de contenedor `containerpreview.azurecr.io`, puede crear un secreto de Kubernetes con la marca genérica en su lugar. Tenga en cuenta el siguiente comando que se ejecuta en el JSON de configuración de Docker.
> ```console
>  kubectl create secret generic containerpreview \
>      --from-file=.dockerconfigjson=~/.docker/config.json \
>      --type=kubernetes.io/dockerconfigjson
> ```

El siguiente resultado se imprime en la consola una vez que el secreto se ha creado correctamente.

```console
secret "containerpreview" created
```

Para comprobar que el secreto se ha creado, ejecute [`kubectl get`][kubectl-get] con la marca `secrets`.

```console
kubectl get secrets
```

Al ejecutar `kubectl get secrets`, se imprimen todos los secretos configurados.

```console
NAME                  TYPE                                  DATA      AGE
containerpreview      kubernetes.io/dockerconfigjson        1         30s
```

## <a name="configure-helm-chart-values-for-deployment"></a>Configuración de los valores del gráfico de Helm para la implementación

Empiece por crear una carpeta llamada *read*. Luego pegue el siguiente contenido de YAML en un nuevo archivo denominado `chart.yaml`:

```yaml
apiVersion: v2
name: read
version: 1.0.0
description: A Helm chart to deploy the microsoft/cognitive-services-read to a Kubernetes cluster
dependencies:
- name: rabbitmq
  condition: read.image.args.rabbitmq.enabled
  version: ^6.12.0
  repository: https://kubernetes-charts.storage.googleapis.com/
- name: redis
  condition: read.image.args.redis.enabled
  version: ^6.0.0
  repository: https://kubernetes-charts.storage.googleapis.com/
```

Para configurar los valores predeterminados del gráfico de Helm, copie y pegue el contenido de YAML en un archivo denominado `values.yaml`. Reemplace los comentarios `# {ENDPOINT_URI}` y `# {API_KEY}` por sus propios valores. Configure resultExpirationPeriod, Redis y RabbitMQ si es necesario.

```yaml
# These settings are deployment specific and users can provide customizations

read:
  enabled: true
  image:
    name: cognitive-services-read
    registry:  containerpreview.azurecr.io/
    repository: microsoft/cognitive-services-read
    tag: latest
    pullSecret: containerpreview # Or an existing secret
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}
      
      # Result expiration period setting. Specify when the system should clean up recognition results.
      # For example, resultExpirationPeriod=1, the system will clear the recognition result 1hr after the process.
      # resultExpirationPeriod=0, the system will clear the recognition result after result retrieval.
      resultExpirationPeriod: 1
      
      # Redis storage, if configured, will be used by read container to store result records.
      # A cache is required if multiple read containers are placed behind load balancer.
      redis:
        enabled: false # {true/false}
        password: password

      # RabbitMQ is used for dispatching tasks. This can be useful when multiple read containers are
      # placed behind load balancer.
      rabbitmq:
        enabled: false # {true/false}
        rabbitmq:
          username: user
          password: password
```

> [!IMPORTANT]
> - Si no se proporcionan los valores `billing` y `apikey`, los servicios expiran pasados 15 minutos. Del mismo modo, se produce un error en la comprobación porque los servicios no están disponibles.
> 
> - Si implementa varios contenedores de Read detrás de un equilibrador de carga en, por ejemplo, Docker Compose o Kubernetes, debe tener una caché externa. Dado que el contenedor de procesamiento y el contenedor de la solicitud GET podrían no ser los mismos, una caché externa almacena los resultados y los comparte entre contenedores. Para más información sobre la configuración de caché, consulte [Configuración de contenedores de Docker de Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/computer-vision-resource-container-config).
>

Cree una carpeta *plantillas* en el directorio *lectura*. Copie y pegue el siguiente YAML en un archivo denominado `deployment.yaml`. El archivo `deployment.yaml` servirá como una plantilla de Helm.

> Las plantillas generan archivos de manifiesto, que son descripciones de recursos con formato YAML que Kubernetes puede entender. [-Guía de plantilla de gráfico de Helm][chart-template-guide]

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: read
  labels:
    app: read-deployment
spec:
  selector:
    matchLabels:
      app: read-app
  template:
    metadata:
      labels:
        app: read-app       
    spec:
      containers:
      - name: {{.Values.read.image.name}}
        image: {{.Values.read.image.registry}}{{.Values.read.image.repository}}
        ports:
        - containerPort: 5000
        env:
        - name: EULA
          value: {{.Values.read.image.args.eula}}
        - name: billing
          value: {{.Values.read.image.args.billing}}
        - name: apikey
          value: {{.Values.read.image.args.apikey}}
        args:        
        - ReadEngineConfig:ResultExpirationPeriod={{ .Values.read.image.args.resultExpirationPeriod }}
        {{- if .Values.read.image.args.rabbitmq.enabled }}
        - Queue:RabbitMQ:HostName={{ include "rabbitmq.hostname" . }}
        - Queue:RabbitMQ:Username={{ .Values.read.image.args.rabbitmq.rabbitmq.username }}
        - Queue:RabbitMQ:Password={{ .Values.read.image.args.rabbitmq.rabbitmq.password }}
        {{- end }}      
        {{- if .Values.read.image.args.redis.enabled }}
        - Cache:Redis:Configuration={{ include "redis.connStr" . }}
        {{- end }}
      imagePullSecrets:
      - name: {{.Values.read.image.pullSecret}}      
--- 
apiVersion: v1
kind: Service
metadata:
  name: read-service
spec:
  type: LoadBalancer
  ports:
  - port: 5000
  selector:
    app: read-app
```

En la misma carpeta *templates*, copie y pegue las siguientes funciones auxiliares en `helpers.tpl`. `helpers.tpl` define funciones útiles para ayudar a generar la plantilla de Helm.

```yaml
{{- define "rabbitmq.hostname" -}}
{{- printf "%s-rabbitmq" .Release.Name -}}
{{- end -}}

{{- define "redis.connStr" -}}
{{- $hostMaster := printf "%s-redis-master:6379" .Release.Name }}
{{- $hostSlave := printf "%s-redis-slave:6379" .Release.Name -}}
{{- $passWord := printf "password=%s" .Values.read.image.args.redis.password -}}
{{- $connTail := "ssl=False,abortConnect=False" -}}
{{- printf "%s,%s,%s,%s" $hostMaster $hostSlave $passWord $connTail -}}
{{- end -}}
```
La plantilla especifica un servicio de equilibrador de carga y la implementación del contenedor o la imagen para lectura.

### <a name="the-kubernetes-package-helm-chart"></a>Paquete de Kubernetes (gráfico de Helm)

El *gráfico de Helm* contiene la configuración de las imágenes de Docker que se van a extraer del registro de contenedor `containerpreview.azurecr.io`.

> Un [gráfico de Helm][helm-charts] es una colección de archivos que describen un conjunto relacionado de recursos de Kubernetes. Un solo gráfico se podría usar para implementar algo sencillo, como un pod almacenado en memoria, o complejo, como una pila de aplicación web completa con servidores HTTP, bases de datos, memorias caché, etc.

Los *gráficos de Helm* proporcionados extraen las imágenes de Docker del servicio Computer Vision y el servicio correspondiente del registro de contenedor `containerpreview.azurecr.io`.

## <a name="install-the-helm-chart-on-the-kubernetes-cluster"></a>Instalación del gráfico de Helm en el clúster de Kubernetes

Para instalar el *gráfico de Helm*, es necesario ejecutar el comando [`helm install`][helm-install-cmd]. Asegúrese de ejecutar el comando de instalación desde el directorio situado encima de la carpeta `read`.

```console
helm install read ./read
```

Este es el resultado de ejemplo que se puede ver tras una ejecución de instalación correcta:

```console
NAME: read
LAST DEPLOYED: Thu Sep 04 13:24:06 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                    READY  STATUS             RESTARTS  AGE
read-57cb76bcf7-45sdh   0/1    ContainerCreating  0         0s

==> v1/Service
NAME     TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)         AGE
read     LoadBalancer  10.110.44.86  localhost    5000:31301/TCP  0s

==> v1beta1/Deployment
NAME    READY  UP-TO-DATE  AVAILABLE  AGE
read    0/1    1           0          0s
```

La implementación de Kubernetes puede tardar varios minutos en completarse. Para confirmar que los pods y los servicios se han implementado correctamente y están disponibles, ejecute el siguiente comando:

```console
kubectl get all
```

Debería ver algo parecido al resultado siguiente:

```console
kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/read-57cb76bcf7-45sdh   1/1     Running   0          17s

NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes     ClusterIP      10.96.0.1      <none>        443/TCP          45h
service/read           LoadBalancer   10.110.44.86   localhost     5000:31301/TCP   17s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/read   1/1     1            1           17s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/read-57cb76bcf7   1         1         1       17s
```
<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la instalación de aplicaciones con Helm en Azure Kubernetes Service (AKS), [visite esta página][installing-helm-apps-in-aks].

> [!div class="nextstepaction"]
> [Contenedores de Cognitive Services][cog-svcs-containers]

<!-- LINKS - external -->
[free-azure-account]: https://azure.microsoft.com/free
[git-download]: https://git-scm.com/downloads
[azure-cli]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[docker-engine]: https://www.docker.com/products/docker-engine
[kubernetes-cli]: https://kubernetes.io/docs/tasks/tools/install-kubectl
[helm-install]: https://helm.sh/docs/using_helm/#installing-helm
[helm-install-cmd]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[helm-charts]: https://helm.sh/docs/topics/charts/
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-test]: https://helm.sh/docs/helm/#helm-test
[chart-template-guide]: https://helm.sh/docs/chart_template_guide

<!-- LINKS - internal -->
[vision-container-host-computer]: computer-vision-how-to-install-containers.md#the-host-computer
[installing-helm-apps-in-aks]: ../../aks/kubernetes-helm.md
[cog-svcs-containers]: ../cognitive-services-container-support.md
