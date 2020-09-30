---
title: Implementación de una aplicación sin estado de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU por medio de kubectl | Microsoft Docs
description: En este artículo se explica cómo crear y administrar una implementación de una aplicación sin estado de Kubernetes en un dispositivo Microsoft Azure Stack Edge Pro con kubectl.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 91a2d08bf9eea2f5af0f6893712515cb2feeab8a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890731"
---
# <a name="deploy-a-kubernetes-stateless-application-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Implementación de una aplicación sin estado de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU por medio de kubectl

En este artículo, se explica cómo se implementa una aplicación sin estado en un clúster de Kubernetes existente utilizando comandos de kubectl. También se describe paso a paso el proceso de creación y configuración de pods en la aplicación sin estado.

## <a name="prerequisites"></a>Requisitos previos

Para poder crear un clúster de Kubernetes y usar la herramienta de línea de comandos `kubectl`, es necesario que compruebe lo siguiente:

- Tiene credenciales de inicio de sesión en un dispositivo Azure Stack Edge Pro de un nodo.

- Tiene Windows PowerShell 5.0 o una versión posterior instalado en un sistema cliente de Windows para poder obtener acceso al dispositivo Azure Stack Edge Pro. Puede utilizar también cualquier otro cliente que tenga un sistema operativo compatible. En este artículo, el procedimiento que se describe emplea un cliente Windows. Para descargar la última versión de Windows PowerShell, vaya a [Instalación de Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-7).

- La funcionalidad de proceso debe estar habilitada en el dispositivo Azure Stack Edge Pro. Para habilitar dicha funcionalidad, vaya a la página **Proceso** de la interfaz de usuario local del dispositivo. A continuación, seleccione la interfaz de red que desee habilitar para el proceso. Seleccione **Habilitar**. Al hacerlo, se crea un conmutador virtual para esa interfaz de red en el dispositivo. Para más información, vea este artículo sobre la [habilitación de la red de proceso en Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).

- El dispositivo Azure Stack Edge Pro debe tener un servidor de clústeres de Kubernetes con la versión 1.9 o posterior. Para más información, vea este artículo sobre la [creación y administración de un clúster de Kubernetes en un dispositivo Microsoft Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).

- Debe tener instalado `kubectl`.

## <a name="deploy-a-stateless-application"></a>Implementación de una aplicación sin estado

Antes de comenzar, es necesario lo siguiente:

1. Crear un clúster de Kubernetes.
2. Configurar un espacio de nombres.
3. Tener un usuario asociado al espacio de nombres.
4. Guardar la configuración de usuario en `C:\Users\<username>\.kube`.
5. Instalar `kubectl`.

Ahora, puede empezar a ejecutar y administrar implementaciones de aplicaciones sin estado en un dispositivo Azure Stack Edge Pro. Para poder empezar a usar `kubectl`, debe comprobar que tiene la versión de `kubectl` adecuada.

### <a name="verify-you-have-the-correct-version-of-kubectl-and-set-up-configuration"></a>Verificación de que la versión de kubectl es la adecuada y establecimiento de la configuración

Para comprobar la versión de `kubectl`:

1. Compruebe que la versión de `kubectl` es 1.9 o superior:

   ```powershell
   kubectl version
   ```
    
   Este es un ejemplo de la salida:
    
   ```powershell
   PS C:\WINDOWS\system32> C:\windows\system32\kubectl.exe version
   Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
   Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
   ```

   En este caso, la versión de cliente de kubectl es 1.15.2, así que podemos continuar.

2. Obtenga una lista de los pods que se ejecutan en el clúster de Kubernetes. Un pod es un contenedor o un proceso de la aplicación que se ejecuta en el clúster de Kubernetes.

   ```powershell
   kubectl get pods -n <namespace-string>
   ```
    
   Este es un ejemplo de uso del comando:
    
   ```powershell
   PS C:\WINDOWS\system32> kubectl get pods -n "test1"
   No resources found.
   PS C:\WINDOWS\system32>
   ```
    
   La salida debe indicar que no se encuentra ningún recurso (pod), ya que no hay ninguna aplicación en ejecución en el clúster.

   El comando rellenará la estructura de directorios de "C:\Users\\&lt;nombreDeUsuario&gt;\\.kube\" con los archivos de configuración. La herramienta de línea de comandos kubectl usará estos archivos para crear y administrar las aplicaciones sin estado del clúster de Kubernetes.

3. Compruebe manualmente la estructura de directorios de "C:\Users\\&lt;nombre de usuario&gt;\\.kube\" para verificar que se han incluido las siguientes subcarpetas en *kubectl*:

   ```powershell
   PS C:\Users\username> ls .kube
    
    
      Directory: C:\Users\user\.kube
    
   Mode                LastWriteTime         Length Name
   ----                -------------         ------ ----
   d-----         2/18/2020 11:05 AM                cache
   d-----         2/18/2020 11:04 AM                http-cache
   -a----         2/18/2020 10:41 AM           5377 config
   ```

> [!NOTE]
> Para ver una lista de todos los comandos de kubectl, escriba `kubectl --help`.

### <a name="create-a-stateless-application-using-a-deployment"></a>Creación de una aplicación sin estado mediante una implementación

Ahora que ha comprobado que la versión de la línea de comandos de kubectl es correcta y que tiene los archivos de configuración necesarios, puede crear una implementación de una aplicación sin estado.

Un pod es la unidad de ejecución básica de una aplicación de Kubernetes, la unidad más pequeña y más sencilla del modelo de objetos de Kubernetes que se puede crear o implementar. Los pods también encapsulan recursos de almacenamiento, una dirección IP de red única y opciones que controlan cómo se deben ejecutar los contenedores.

El tipo de aplicación sin estado que se crea es una implementación de servidor web de Nginx.

Todos los comandos de kubectl que se utilizan para crear y administrar implementaciones de aplicaciones sin estado tienen que especificar el espacio de nombres asociado a la configuración. Este espacio de nombres lo creó mientras estaba conectado al clúster del dispositivo Azure Stack Edge Pro en el tutorial de [creación y administración de un clúster de Kubernetes en un dispositivo Microsoft Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md) con `New-HcsKubernetesNamespace`.

Para especificar el espacio de nombres en un comando de kubectl, use `kubectl <command> -n <namespace-string>`.

Siga estos pasos para crear una implementación de nginx:

1. Asigne una aplicación sin estado creando un objeto de implementación de Kubernetes:

   ```powershell
   kubectl apply -f <yaml-file> -n <namespace-string>
   ```

   En este ejemplo, la ruta de acceso al archivo YAML de la aplicación es un origen externo.

   Este es un ejemplo del uso del comando y su salida:

   ```powershell
   PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment.yaml -n "test1"
    
   deployment.apps/nginx-deployment created
   ```

   Si lo desea, también puede guardar el siguiente código de Markdown en el equipo local y sustituir la ruta de acceso y el nombre de archivo del parámetro *-f*. Por ejemplo, "C:\Kubernetes\deployment.yaml". Esta es la configuración de la implementación de la aplicación:

   ```markdown
   apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     selector:
       matchLabels:
         app: nginx
     replicas: 2 # tells deployment to run 2 pods matching the template
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80
   ```

   Este comando crea una implementación de nginx predeterminada con dos pods que van a ejecutar la aplicación.

2. Obtenga la descripción de la implementación de nginx de Kubernetes que ha creado:

   ```powershell
   kubectl describe deployment nginx-deployment -n <namespace-string>
   ```

   Este es un ejemplo del uso del comando y su salida:
    
   ```powershell
   PS C:\Users\user> kubectl describe deployment nginx-deployment -n "test1"
    
   Name:                   nginx-deployment
   Namespace:              test1
   CreationTimestamp:      Tue, 18 Feb 2020 13:35:29 -0800
   Labels:                 <none>
   Annotations:            deployment.kubernetes.io/revision: 1
                           kubectl.kubernetes.io/last-applied-configuration:
                             {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment","namespace":"test1"},"spec":{"repl...
   Selector:               app=nginx
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   StrategyType:           RollingUpdate
   MinReadySeconds:        0
   RollingUpdateStrategy:  25% max unavailable, 25% max surge
   Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.7.9
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
   Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
   OldReplicaSets:  <none>
   NewReplicaSet:   nginx-deployment-5754944d6c (2/2 replicas created)
   Events:
     Type    Reason             Age    From                   Message
     ----    ------             ----   ----                   -------
     Normal  ScalingReplicaSet  2m22s  deployment-controller  Scaled up replica set nginx-deployment-5754944d6c to 2
   ```

   Si examina atentamente la opción *replicas*, verá:
    
   ```powershell
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   ```

   La opción *replicas* indica que la especificación de la implementación necesitaba dos pods, que esos pods se crearon y actualizaron, y que están listos para que se utilicen.

   > [!NOTE]
   > Los pods que se eliminan o finalizan por alguna razón, como un error en un nodo del dispositivo o una actualización del dispositivo que genere problemas, se sustituyen por conjuntos de réplicas. Por este motivo, es conveniente que utilice un conjunto de réplicas aunque la aplicación solamente necesite un único pod.

3. Para mostrar los pods de la implementación:

   ```powershell
   kubectl get pods -l app=nginx -n <namespace-string>
   ```
    
   Este es un ejemplo del uso del comando y su salida:
    
   ```powershell
   PS C:\Users\user> kubectl get pods -l app=nginx -n "test1"
    
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-5754944d6c-7wqjd   1/1     Running   0          3m13s
   nginx-deployment-5754944d6c-nfj2h   1/1     Running   0          3m13s
   ```

   La salida confirma que tenemos dos pods con nombres únicos a los que podemos hacer referencia utilizando kubectl.

4. Para ver información sobre un pod específico de la implementación:

   ```powershell
   kubectl describe pod <podname-string> -n <namespace-string>
   ```

   Este es un ejemplo del uso del comando y su salida:

   ```powershell
   PS C:\Users\user> kubectl describe pod "nginx-deployment-5754944d6c-7wqjd" -n "test1"

   Name:           nginx-deployment-5754944d6c-7wqjd
   Namespace:      test1
   Priority:       0
   Node:           k8s-1d9qhq2cl-n1/10.128.46.184
   Start Time:     Tue, 18 Feb 2020 13:35:29 -0800
   Labels:         app=nginx
                   pod-template-hash=5754944d6c
   Annotations:    <none>
   Status:         Running
   IP:             172.17.246.200
   Controlled By:  ReplicaSet/nginx-deployment-5754944d6c
    Containers:
      nginx:
        Container ID:   docker://280b0f76bfdc14cde481dc4f2b8180cf5fbfc90a084042f679d499f863c66979
        Image:          nginx:1.7.9
        Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Tue, 18 Feb 2020 13:35:35 -0800
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-8gksw (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      default-token-8gksw:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-8gksw
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type    Reason     Age    From                       Message
      ----    ------     ----   ----                       -------
      Normal  Scheduled  4m58s  default-scheduler          Successfully assigned test1/nginx-deployment-5754944d6c-7wqjd to k8s-1d9qhq2cl-n1
      Normal  Pulling    4m57s  kubelet, k8s-1d9qhq2cl-n1  Pulling image "nginx:1.7.9"
      Normal  Pulled     4m52s  kubelet, k8s-1d9qhq2cl-n1  Successfully pulled image "nginx:1.7.9"
      Normal  Created    4m52s  kubelet, k8s-1d9qhq2cl-n1  Created container nginx
      Normal  Started    4m52s  kubelet, k8s-1d9qhq2cl-n1  Started container nginx
   ```

### <a name="rescale-the-application-deployment-by-increasing-the-replica-count"></a>Ajuste del escalado de la implementación de la aplicación mediante un aumento del número de réplicas

Cada pod está diseñado para ejecutar una única instancia de una determinada aplicación. Si desea escalar la aplicación horizontalmente para que se ejecuten varias instancias, puede aumentar el número de pods, uno por cada instancia. En Kubernetes, esto se conoce como "replicación".
Puede aumentar el número de pods en la implementación de la aplicación utilizando un nuevo archivo YAML. El archivo YAML cambia la opción de las réplicas a cuatro, lo que aumenta el número de pods de la implementación en cuatro pods. Para aumentar el número de pods de dos a cuatro:

```powershell
PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml -n "test1"
```

Si lo desea, también puede guardar el siguiente código de Markdown en el equipo local y sustituir la ruta de acceso y el nombre de archivo del parámetro *-f* de `kubectl apply`. Por ejemplo, "C:\Kubernetes\deployment-scale.yaml". Esta es la configuración de escalado de la implementación de la aplicación:

```markdown
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```

Para verificar que la implementación tiene cuatro pods:

```powershell
kubectl get pods -l app=nginx
```

Esta es la salida de ejemplo del nuevo escalado de una implementación en la que se ha pasado de dos a cuatro pods:

```powershell
PS C:\WINDOWS\system32> kubectl get pods -l app=nginx

NAME                               READY     STATUS    RESTARTS   AGE
nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
nginx-deployment-148880595-fxcez   1/1       Running   0          2m
nginx-deployment-148880595-rwovn   1/1       Running   0          2m
```

Como puede ver en la salida, ahora tiene cuatro pods en la implementación que pueden ejecutar la aplicación.

### <a name="delete-a-deployment"></a>Eliminación de una implementación

Para eliminar la implementación, incluidos todos los pods, debe ejecutar `kubectl delete deployment` especificando el nombre de la implementación, *nginx-deployment*, y el nombre del espacio de nombres. Para eliminar la implementación:

   ```powershell
   kubectl delete deployment nginx-deployment -n <namespace-string>
   ```

Este es un ejemplo del uso del comando y su salida:

```powershell
PS C:\Users\user> kubectl delete deployment nginx-deployment -n "test1"
deployment.extensions "nginx-deployment" deleted
```

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Kubernetes](azure-stack-edge-gpu-kubernetes-overview.md)
