---
title: Implementación de gráficos de Helm mediante GitOps en clústeres de Kubernetes habilitados para Arc (versión preliminar)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Uso de GitOps con Helm para una configuración de clúster habilitada para Azure Arc (versión preliminar)
keywords: GitOps, Kubernetes, K8s, Azure, Helm, Arc, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: cca48910b679ff8f72ee06f4ed990bd480fb2200
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723646"
---
# <a name="deploy-helm-charts-using-gitops-on-arc-enabled-kubernetes-cluster-preview"></a>Implementación de gráficos de Helm mediante GitOps en clústeres de Kubernetes habilitados para Arc (versión preliminar)

Helm es una herramienta de empaquetado de código abierto que facilita la instalación y administración del ciclo de vida de las aplicaciones de Kubernetes. Helm, que funciona de forma similar a los administradores de paquetes de Linux como APT y Yum, se usa para administrar los gráficos de Kubernetes, que son paquetes de recursos de Kubernetes preconfigurados.

En este artículo se muestra cómo configurar y usar Helm con Kubernetes habilitado para Azure Arc.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster conectado de Kubernetes habilitado para Azure Arc. Si necesita un clúster conectado, vea el [inicio rápido sobre conexión de un clúster](./connect-cluster.md).

En primer lugar se establecerán las variables de entorno que se usarán en este tutorial. Necesitará el nombre del grupo de recursos y el del clúster para el clúster conectado.

```bash
export RESOURCE_GROUP=<Resource_Group_Name>
export CLUSTER_NAME=<ClusterName>
```

## <a name="verify-your-cluster-is-enabled-with-arc"></a>Comprobación de que el clúster está habilitado con Arc

```bash
az connectedk8s list -g $RESOURCE_GROUP -o table
```

Salida:
```bash
Name           Location    ResourceGroup
-------------  ----------  ---------------
arc-helm-demo  eastus      k8s-clusters
```

## <a name="overview-of-using-gitops-and-helm-with-azure-arc-enabled-kubernetes"></a>Introducción al uso de GitOps y Helm con Kubernetes habilitado para Azure Arc

 El operador Helm proporciona una extensión para Flux que automatiza las versiones Chart de Helm. Una versión Chart se describe a través de un recurso personalizado de Kubernetes denominado HelmRelease. Flux sincroniza estos recursos de Git con el clúster, y el operador de Helm se asegura de que los gráficos de Helm se publiquen como se especifica en los recursos.

 En el ejemplo siguiente se muestra la estructura de repositorios de Git que se usará en este tutorial:

```bash
├── charts
│   └── azure-arc-sample
│       ├── Chart.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── values.yaml
└── releases
    └── app.yaml
```

En el repositorio de Git hay dos directorios, uno que contiene un gráfico de Helm y con la configuración de publicaciones. En el directorio `releases`, `app.yaml` contiene la configuración de HelmRelease siguiente:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: azure-arc-sample
  namespace: arc-k8s-demo
spec:
  releaseName: arc-k8s-demo
  chart:
    git: https://github.com/Azure/arc-helm-demo
    ref: master
    path: charts/azure-arc-sample
  values:
    serviceName: arc-k8s-demo
```

La configuración de versión de Helm contiene los campos siguientes:

- `metadata.name` es obligatorio y debe seguir las convenciones de nomenclatura de Kubernetes
- `metadata.namespace` es opcional y determina dónde se crea la versión
- `spec.releaseName` es opcional y, si no se proporciona, el nombre de la versión será $espacio_de_nombres-$nombre
- `spec.chart.path` es el directorio que contiene el gráfico, en relación con la raíz del repositorio.
- `spec.values` son personalizaciones de usuario de los valores de parámetro predeterminados del propio gráfico

Las opciones especificadas en spec.values de HelmRelease reemplazarán las opciones especificadas en values.yaml del origen del gráfico.

Puede obtener más información sobre HelmRelease en la [documentación oficial del operador Helm](https://docs.fluxcd.io/projects/helm-operator/en/stable/).

## <a name="create-a-configuration"></a>Creación de una configuración

Con la extensión de la CLI de Azure para `k8sconfiguration`, se vinculará el clúster conectado al repositorio de Git de ejemplo. A esta configuración se le asigna el nombre `azure-arc-sample` y el operador de Flux se implementa en el espacio de nombres `arc-k8s-demo`.

```bash
az k8sconfiguration create --name azure-arc-sample \
  --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME \
  --operator-instance-name flux --operator-namespace arc-k8s-demo \
  --operator-params='--git-readonly --git-path=releases' \
  --enable-helm-operator --helm-operator-version='0.6.0' \
  --helm-operator-params='--set helm.versions=v3' \
  --repository-url https://github.com/Azure/arc-helm-demo.git  \
  --scope namespace --cluster-type connectedClusters
```

### <a name="configuration-parameters"></a>Parámetros de configuración

Para personalizar la creación de la configuración, [obtenga información sobre los parámetros adicionales que puede usar](./use-gitops-connected-cluster.md#additional-parameters).

## <a name="validate-the-configuration"></a>Validación de la configuración

Con la CLI de Azure, compruebe que `sourceControlConfiguration` se ha creado correctamente.

```console
az k8sconfiguration show --resource-group $RESOURCE_GROUP --name azure-arc-sample --cluster-name $CLUSTER_NAME --cluster-type connectedClusters
```

El recurso `sourceControlConfiguration` se actualiza con el estado de cumplimiento, mensajes y la información de depuración.

**Salida:**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2019-12-05T05:34:41.481000",
    "message": "{\"OperatorMessage\":null,\"ClusterState\":null}",
    "messageLevel": "3"
  },
  "enableHelmOperator": "True",
  "helmOperatorProperties": {
    "chartValues": "--set helm.versions=v3",
    "chartVersion": "0.6.0"
  },
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/azure-arc-sample",
  "name": "azure-arc-sample",
  "operatorInstanceName": "flux",
  "operatorNamespace": "arc-k8s-demo",
  "operatorParams": "--git-readonly --git-path=releases",
  "operatorScope": "namespace",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "",
  "repositoryUrl": "https://github.com/Azure/arc-helm-demo.git",
  "resourceGroup": "AzureArcTest",
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

## <a name="validate-application"></a>Validación de la aplicación

Ejecute el siguiente comando y vaya a `localhost:8080` en el explorador para comprobar que la aplicación se está ejecutando.

```bash
kubectl port-forward -n arc-k8s-demo svc/arc-k8s-demo 8080:8080
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de Azure Policy para controlar la configuración del clúster](./use-azure-policy.md)
