---
title: Creación de un controlador de datos con la CLI de datos de Azure (azdata)
description: Cree un controlador de datos de Azure Arc en un típico clúster de Kubernetes de varios nodos que ya haya creado mediante la CLI de datos de Azure (azdata).
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 8651921dafd5727f79e4ab3919e188d4b8612465
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932380"
---
# <a name="create-azure-arc-data-controller-using-the-azure-data-cli-azdata"></a>Creación de un controlador de datos de Azure Arc mediante la CLI de datos de Azure (azdata)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

Revise el tema [Creación del controlador de datos de Azure Arc](create-data-controller.md) para obtener información general.

Para crear el controlador de datos de Azure Arc con la CLI de datos de Azure, debe tener instalada esta CLI.

   [Instalación de la CLI de datos de Azure](install-client-tools.md)

Independientemente de la plataforma de destino que elija, tendrá que establecer las siguientes variables de entorno antes de la creación para el usuario administrador del controlador de datos. Puede proporcionar estas credenciales a otras personas que necesiten tener acceso de administrador al controlador de datos según sea necesario.

**AZDATA_USERNAME**: nombre de usuario de su elección para el usuario administrador del controlador de datos. Ejemplo: `arcadmin`

**AZDATA_PASSWORD**: contraseña de su elección para el usuario administrador del controlador de datos. La contraseña debe tener al menos 8 caracteres y contener caracteres de tres de los siguientes cuatro conjuntos: mayúsculas, minúsculas, números y símbolos.

### <a name="linux-or-macos"></a>Linux o macOS

```console
export AZDATA_USERNAME="<your username of choice>"
export AZDATA_PASSWORD="<your password of choice>"
```

### <a name="windows-powershell"></a>Windows PowerShell

```console
$ENV:AZDATA_USERNAME="<your username of choice>"
$ENV:AZDATA_PASSWORD="<your password of choice>"
```

Tendrá que conectarse a un clúster de Kubernetes y autenticarse en él, además de tener un contexto de Kubernetes existente seleccionado antes de comenzar la creación del controlador de datos de Azure Arc. La forma en que se conecta a un clúster o servicio de Kubernetes varía; consulte la documentación de la distribución o el servicio de Kubernetes que usa para saber cómo conectarse al servidor de API de Kubernetes.

Puede comprobar que tiene una conexión de Kubernetes y confirmar el contexto actual con los siguientes comandos.

```console
kubectl get namespace

kubectl config current-context
```

## <a name="create-the-azure-arc-data-controller"></a>Creación del controlador de datos de Azure Arc

> [!NOTE]
> Puede usar un valor diferente para el parámetro `--namespace` del comando azdata arc dc create en los ejemplos siguientes, pero asegúrese de usar ese nombre de espacio de nombres en el parámetro `--namespace parameter` de los demás comandos siguientes.

Continúe en la sección siguiente que corresponda a su plataforma de destino para configurar la creación.

[Creación en Azure Kubernetes Service (AKS)](#create-on-azure-kubernetes-service-aks)

[Creación en el motor de AKS en Azure Stack Hub](#create-on-aks-engine-on-azure-stack-hub)

[Creación en AKS en Azure Stack HCI](#create-on-aks-on-azure-stack-hci)

[Creación en Red Hat OpenShift en Azure (ARO)](#create-on-azure-red-hat-openshift-aro)

[Creación en Red Hat OpenShift Container Platform (OCP)](#create-on-red-hat-openshift-container-platform-ocp)

[Creación en Kubernetes ascendente de código abierto (kubeadm)](#create-on-open-source-upstream-kubernetes-kubeadm)

[Creación en Elastic Kubernetes Service (EKS) de AWS](#create-on-aws-elastic-kubernetes-service-eks)

[Creación en Kubernetes Engine Service (GKE) de Google Cloud](#create-on-google-cloud-kubernetes-engine-service-gke)

### <a name="create-on-azure-kubernetes-service-aks"></a>Creación en Azure Kubernetes Service (AKS)

De forma predeterminada, el perfil de implementación de AKS usa la clase de almacenamiento `managed-premium`. La clase de almacenamiento `managed-premium` solo funcionará si tiene máquinas virtuales que se implementaron mediante imágenes de máquina virtual con discos premium.

Si va a usar `managed-premium` como clase de almacenamiento, puede ejecutar el siguiente comando para implementar el controlador de datos. Sustituya los marcadores de posición del comando por el nombre del grupo de recursos, el identificador de la suscripción y la ubicación de Azure.

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Si no está seguro de qué clase de almacenamiento debe utilizar, indique la clase de almacenamiento `default`, que es compatible con cualquier tipo de máquina virtual que esté usando. No le ofrecerá el rendimiento más rápido.

Si desea usar la clase de almacenamiento `default`, puede ejecutar este comando:

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aks-engine-on-azure-stack-hub"></a>Creación en el motor de AKS en Azure Stack Hub

De forma predeterminada, el perfil de implementación utiliza la clase de almacenamiento `managed-premium`. La clase de almacenamiento `managed-premium` solo funcionará si tiene máquinas virtuales de trabajo que se implementaron mediante imágenes de máquina virtual con discos premium en Azure Stack Hub.

Puede ejecutar el siguiente comando para implementar el controlador de datos con la clase de almacenamiento managed-premium:

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Si no está seguro de qué clase de almacenamiento debe utilizar, indique la clase de almacenamiento `default`, que es compatible con cualquier tipo de máquina virtual que esté usando. En Azure Stack Hub, los discos premium y los discos estándar cuentan con el respaldo de la misma infraestructura de almacenamiento. Por lo tanto, se espera que proporcionen el mismo rendimiento general, pero con distintos límites de IOPS.

Si desea usar la clase de almacenamiento `default`, puede ejecutar este comando.

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aks-on-azure-stack-hci"></a>Creación en AKS en Azure Stack HCI

De forma predeterminada, el perfil de implementación utiliza una clase de almacenamiento denominada `default` y el tipo de servicio `LoadBalancer`.

Puede ejecutar el siguiente comando para implementar el controlador de datos con la clase de almacenamiento `default` y el tipo de servicio `LoadBalancer`.

```console
azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).


### <a name="create-on-azure-red-hat-openshift-aro"></a>Creación en Red Hat OpenShift en Azure (ARO)

Para crear un controlador de datos en Red Hat OpenShift en Azure, debe ejecutar los siguientes comandos en el clúster para relajar las restricciones de seguridad. Se trata de un requisito temporal que se quitará en el futuro.
> [!NOTE]
>   Use el mismo espacio de nombres aquí y en el comando `azdata arc dc create` siguiente. Un ejemplo es `arc`.

En primer lugar, descargue la restricción de contexto de seguridad (SCC) personalizada de [GitHub](https://raw.githubusercontent.com/microsoft/azure_arc/master/data_services/deployment/yaml/arc-data-scc.yaml) y aplíquela al clúster.

Puede ejecutar el comando siguiente para crear el controlador de datos:
> [!NOTE]
>   Use el mismo espacio de nombres aquí que en los comandos `oc adm policy add-scc-to-user` anteriores. Un ejemplo es `arc`.

```console
azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example
#azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-red-hat-openshift-container-platform-ocp"></a>Creación en Red Hat OpenShift Container Platform (OCP)


> [!NOTE]
> Si usa Red Hat OpenShift Container Platform en Azure, se recomienda usar la versión más reciente disponible.

Para crear un controlador de datos en Azure Red Hat OpenShift Platform, debe ejecutar los siguientes comandos en el clúster para relajar las restricciones de seguridad. Se trata de un requisito temporal que se quitará en el futuro.
> [!NOTE]
>   Use el mismo espacio de nombres aquí y en el comando `azdata arc dc create` siguiente. Un ejemplo es `arc`.

```console
oc adm policy add-scc-to-user arc-data-scc --serviceaccount default --namespace arc
```

También necesitará determinar qué clase de almacenamiento se va a usar ejecutando el siguiente comando.

```console
kubectl get storageclass
```

En primer lugar, empiece por crear un nuevo archivo de perfil de implementación personalizado basado en el perfil de implementación azure-arc-openshift ejecutando el siguiente comando. Este comando creará un directorio `custom` en el directorio de trabajo actual y un archivo de perfil de implementación personalizado `control.json` en ese directorio.

Use el perfil `azure-arc-openshift` para OpenShift Container Platform.

```console
azdata arc dc config init --source azure-arc-openshift --path ./custom
```
Use el perfil `azure-arc-azure-openshift` para Azure RedHat Open Shift.

```console
azdata arc dc config init --source azure-arc-azure-openshift --path ./custom
```

Ahora, establezca la clase de almacenamiento deseada reemplazando `<storageclassname>` en el siguiente comando por el nombre de la clase de almacenamiento que desea usar y que se determinó mediante la ejecución del comando `kubectl get storageclass` anterior.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

De forma predeterminada, el perfil de implementación azure-arc-openshift usa `NodePort` como tipo de servicio. Si usa un clúster de OpenShift que está integrado con un equilibrador de carga, puede cambiar la configuración para usar el tipo de servicio LoadBalancer con el siguiente comando:

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

Cuando usa OpenShift, es posible que desee realizar la ejecución con las directivas de seguridad predeterminadas de OpenShift o que desee bloquear el entorno más de lo habitual. Si lo desea, puede optar por deshabilitar, mediante el uso de los siguientes comandos, algunas características para reducir los permisos que se necesitan al implementar o al ejecutar.

Este comando deshabilita la recopilación de métricas sobre los pods. Si deshabilita esta característica, no podrá ver las métricas sobre los pods en los paneles de Grafana. El valor predeterminado es true.

```console
azdata arc dc config replace -p ./custom/control.json --json-values spec.security.allowPodMetricsCollection=false
```

Este comando deshabilita la recopilación de métricas sobre los nodos. Si deshabilita esta característica, no podrá ver las métricas sobre los nodos en los paneles de Grafana. El valor predeterminado es true.

```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowNodeMetricsCollection=false
```

Este comando deshabilita la posibilidad de realizar volcados de memoria para solucionar problemas.
```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowDumps=false
```

Ahora ya puede crear el controlador de datos con el siguiente comando.
> [!NOTE]
>   Use el mismo espacio de nombres aquí que en los comandos `oc adm policy add-scc-to-user` anteriores. Un ejemplo es `arc`.

> [!NOTE]
>   El parámetro `--path` debe apuntar al _directorio_ que contiene el archivo control.json y no al propio archivo control.json.


```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-open-source-upstream-kubernetes-kubeadm"></a>Creación en Kubernetes ascendente de código abierto (kubeadm)

De forma predeterminada, el perfil de implementación de kubeadm usa una clase de almacenamiento denominada `local-storage` y el tipo de servicio `NodePort`. Si le resulta aceptable, puede omitir las instrucciones que se indican a continuación para establecer la clase de almacenamiento y el tipo de servicio que desee y ejecutar inmediatamente el comando `azdata arc dc create` siguiente.

Si desea personalizar el perfil de implementación para especificar una clase de almacenamiento o un tipo de servicio específicos, empiece por crear un nuevo archivo de perfil de implementación personalizado basado en el perfil de implementación de kubeadm. Para ello, ejecute el siguiente comando. Este comando creará un directorio `custom` en el directorio de trabajo actual y un archivo de perfil de implementación personalizado `control.json` en ese directorio.

```console
azdata arc dc config init --source azure-arc-kubeadm --path ./custom
```

Puede buscar las clases de almacenamiento disponibles mediante la ejecución del siguiente comando.

```console
kubectl get storageclass
```

Ahora, establezca la clase de almacenamiento deseada reemplazando `<storageclassname>` en el siguiente comando por el nombre de la clase de almacenamiento que desea usar y que se determinó mediante la ejecución del comando `kubectl get storageclass` anterior.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

De forma predeterminada, el perfil de implementación de kubeadm usa `NodePort` como tipo de servicio. Si usa un clúster de Kubernetes que está integrado con un equilibrador de carga, puede cambiar la configuración con el siguiente comando.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

Ahora ya puede crear el controlador de datos con el siguiente comando.

```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aws-elastic-kubernetes-service-eks"></a>Creación en Elastic Kubernetes Service (EKS) de AWS

De forma predeterminada, la clase de almacenamiento de EKS es `gp2` y el tipo de servicio es `LoadBalancer`.

Ejecute el siguiente comando para crear el controlador de datos mediante el perfil de implementación de EKS proporcionado.

```console
azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-google-cloud-kubernetes-engine-service-gke"></a>Creación en Kubernetes Engine Service (GKE) de Google Cloud

De forma predeterminada, la clase de almacenamiento de GKE es `standard` y el tipo de servicio es `LoadBalancer`.

Ejecute el siguiente comando para crear el controlador de datos mediante el perfil de implementación de GKE proporcionado.

```console
azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

## <a name="monitoring-the-creation-status"></a>Supervisión del estado de creación

La creación del controlador tardará unos minutos en completarse. Puede supervisar el progreso en otra ventana de terminal con los siguientes comandos:

> [!NOTE]
>  En los siguientes comandos de ejemplo se da por hecho que ha creado un controlador de datos y un espacio de nombres de Kubernetes con el nombre `arc`. Si ha usado otro nombre de controlador de datos o de espacio de nombres, puede reemplazar `arc` por ese nombre.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

También puede comprobar el estado de creación de un pod determinado ejecutando un comando como el siguiente. Esto es especialmente útil para solucionar problemas.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Solución de problemas de creación

Si tiene problemas con la creación, consulte la [guía de solución de problemas](troubleshoot-guide.md).
