---
title: Conexión a un dispositivo Microsoft Azure Stack Edge Pro y administración mediante la interfaz de Windows PowerShell | Microsoft Docs
description: En este artículo se describe cómo conectarse a Azure Stack Edge Pro y cómo administrarlo mediante la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/10/2020
ms.author: alkohli
ms.openlocfilehash: b0c2b547391efd37fc667b84548d99f1e7385cfb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903521"
---
# <a name="manage-an-azure-stack-edge-pro-gpu-device-via-windows-powershell"></a>Administración de un dispositivo Azure Stack Edge Pro con GPU mediante Windows PowerShell

La solución Azure Stack Edge Pro permite procesar datos y enviarlos a través de la red a Azure. En este artículo se describen algunas de las tareas de configuración y administración del dispositivo Azure Stack Edge Pro. Puede usar Azure Portal, la interfaz de usuario web local o la interfaz de Windows PowerShell para administrar su dispositivo.

Este artículo se centra en cómo puede conectarse a la interfaz de PowerShell del dispositivo y las tareas que puede realizar con esta interfaz. 


## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Crear un paquete de soporte

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

<!--## Upload certificate

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

You can also upload IoT Edge certificates to enable a secure connection between your IoT Edge device and the downstream devices that may connect to it. There are three IoT Edge certificates (*.pem* format) that you need to install:

- Root CA certificate or the owner CA
- Device CA certificate
- Device key certificate

The following example shows the usage of this cmdlet to install IoT Edge certificates:

```
Set-HcsCertificate -Scope IotEdge -RootCACertificateFilePath "\\hcfs\root-ca-cert.pem" -DeviceCertificateFilePath "\\hcfs\device-ca-cert.pem\" -DeviceKeyFilePath "\\hcfs\device-key-cert.pem" -Credential "username"
```
When you run this cmdlet, you will be prompted to provide the password for the network share.

For more information on certificates, go to [Azure IoT Edge certificates](https://docs.microsoft.com/azure/iot-edge/iot-edge-certs) or [Install certificates on a gateway](https://docs.microsoft.com/azure/iot-edge/how-to-create-transparent-gateway).-->

## <a name="view-device-information"></a>Ver la información del dispositivo
 
[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="view-gpu-driver-information"></a>Visualización de la información del controlador de GPU

Si el rol de proceso está configurado en su dispositivo, también puede obtener la información del controlador de GPU mediante la interfaz de PowerShell. 

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-HcsGpuNvidiaSmi` para obtener la información del controlador de GPU del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-HcsGpuNvidiaSmi
    ```
    Anote la información del controlador que aparece en la salida de ejemplo de este cmdlet.

    ```powershell    
    +-----------------------------------------------------------------------------+    
    | NVIDIA-SMI 440.64.00    Driver Version: 440.64.00    CUDA Version: 10.2     |    
    |-------------------------------+----------------------+----------------------+    
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |    
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |    
    |===============================+======================+======================|    
    |   0  Tesla T4            On   | 000029CE:00:00.0 Off |                    0 |    
    | N/A   60C    P0    29W /  70W |   1539MiB / 15109MiB |      0%      Default |    
    +-------------------------------+----------------------+----------------------+    
    |   1  Tesla T4           On  | 0000AD50:00:00.0 Off |                    0 |
    | N/A   58C    P0    29W /  70W |    330MiB / 15109MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    ```

## <a name="enable-multi-process-service-mps"></a>Habilitación servicio multiproceso (MPS)

Un servicio multiproceso (MPS) en las GPU de Nvidia proporciona un mecanismo donde varios trabajos pueden compartir las GPU, donde cada trabajo tiene asignado cierto porcentaje de los recursos de la GPU. MPS es una característica en vista previa (GB) en el dispositivo Azure Stack Edge Pro con GPU. Para habilitar MPS en el dispositivo, siga estos pasos:

1. Antes de comenzar, asegúrese de que: 

    1. Ha configurado y [activado el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md) con un recurso de Azure Stack Edge Pro o Data Box Gateway en Azure.
    1. Ha [configurado el proceso en este dispositivo en Azure Portal](azure-stack-edge-deploy-configure-compute.md#configure-compute).
    
1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
1. Use el siguiente comando para habilitar MPS en el dispositivo.

    ```powershell
    Start-HcsGpuMPS
    ```

## <a name="reset-your-device"></a>Restablecer el dispositivo

[!INCLUDE [Reset your device](../../includes/data-box-edge-gateway-deactivate-device.md)]

## <a name="get-compute-logs"></a>Obtener los registros de proceso

Si el rol de proceso está configurado en su dispositivo, también puede obtener los registros del proceso gracias a la interfaz de PowerShell.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-AzureDataBoxEdgeComputeRoleLogs` para obtener los registros del proceso del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-AzureDataBoxEdgeComputeRoleLogs -Path "\\hcsfs\logs\myacct" -Credential "username" -FullLogCollection    
    ```

    Aquí tiene hay una descripción de los parámetros que se usan en el cmdlet:
    - `Path`: proporcione una ruta de red al recurso compartido en el que quiere crear el paquete de registros del proceso.
    - `Credential`: proporcione el nombre de usuario del recurso compartido de red. Cuando ejecute este cmdlet, deberá proporcionar la contraseña del recurso compartido.
    - `FullLogCollection`: este parámetro garantiza que el paquete de registros contendrá todos los registros del proceso. De forma predeterminada, el paquete de registros contiene solo un subconjunto de registros.


## <a name="change-kubernetes-pod-and-service-subnets"></a>Cambio de las subredes de pods y de servicio de Kubernetes

De forma predeterminada, Kubernetes en el dispositivo Azure Stack Edge usa las subredes 172.27.0.0/16 y 172.28.0.0/16 para pods y para el servicio respectivamente. Si estas subredes ya están en uso en la red, puede ejecutar el cmdlet `Set-HcsKubeClusterNetworkInfo` para cambiarlas.

Se debe realizar este cambio antes de configurar el proceso en Azure Portal, ya que el clúster de Kubernetes se crea en este paso.

1. Conéctese a la interfaz de PowerShell del dispositivo.
1. Desde dicha interfaz de PowerShell, ejecute lo siguiente:

    `Set-HcsKubeClusterNetworkInfo -PodSubnet <subnet details> -ServiceSubnet <subnet details>`

    Reemplace <subnet details> por el intervalo de subred que quiera usar. 

1. Una vez que haya ejecutado este comando, puede usar el comando `Get-HcsKubeClusterNetworkInfo` para comprobar que las subredes de pods y de servicio han cambiado.

A continuación se muestra una salida de ejemplo de este comando.

```powershell
[10.100.10.10]: PS>Set-HcsKubeClusterNetworkInfo -PodSubnet 10.96.0.1/16 -ServiceSubnet 10.97.0.1/16
[10.100.10.10]: PS>Get-HcsKubeClusterNetworkInfo

Id                                   PodSubnet    ServiceSubnet
--                                   ---------    -------------
6dbf23c3-f146-4d57-bdfc-76cad714cfd1 10.96.0.1/16 10.97.0.1/16
[10.100.10.10]: PS>
```


## <a name="debug-kubernetes-issues-related-to-iot-edge"></a>Depurar problemas de Kubernetes relacionados con IoT Edge

<!--When the Kubernetes cluster is created, there are two system namespaces created: `iotedge` and `azure-arc`. --> 

<!--### Create config file for system namespace

To troubleshoot, first create the `config` file corresponding to the `iotedge` namespace with `aseuser`.

Run the `Get-HcsKubernetesUserConfig -AseUser` command and save the output as `config` file (no file extension). Save the file in the `.kube` folder of your user profile on the local machine.

Following is the sample output of the `Get-HcsKubernetesUserConfig` command.

```PowerShell
[10.100.10.10]: PS>Get-HcsKubernetesUserConfig -AseUser
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EVXhNekl4TkRRME5sb1hEVE13TURVeE1USXhORFEwTmxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS0M1CjlJbzRSU2hudG90QUdxdjNTYmRjOVd4UmJDYlRzWXU5S0RQeU9xanVoZE1UUE9PcmROOGNoa0x4NEFyZkZaU1AKZithUmhpdWZqSE56bWhucnkvZlprRGdqQzQzRmV5UHZzcTZXeVVDV0FEK2JBdi9wSkJDbkg2MldoWGNLZ1BVMApqU1k0ZkpXenNFbzBaREhoeUszSGN3MkxkbmdmaEpEanBQRFJBNkRWb2pIaktPb29OT1J1dURvUHpiOTg2dGhUCkZaQXJMZjRvZXRzTEk1ZzFYRTNzZzM1YVhyU0g3N2JPYVVsTGpYTzFYSnpFZlZWZ3BMWE5xR1ZqTXhBMVU2b1MKMXVJL0d1K1ArY
===========CUT=========================================CUT===================
    server: https://compute.myasegpu1.wdshcsso.com:6443
    name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aseuser
    name: aseuser@kubernetes
current-context: aseuser@kubernetes
kind: Config
preferences: {}
users:
- name: aseuser
    user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwRENDQWJpZ0F3SUJBZ0lJY1hOTXRPU2VwbG93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMU1UTXlNVFEwTkRaYUZ3MHlNVEExTVRNeU1UVXhNVEphTUJJeApFREFPQmdOVkJBTVRCMkZ6WlhWelpYSXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCCkFRRHVjQ1pKdm9qNFIrc0U3a1EyYmVjNEJkTXdpUEhmU2R2WnNDVVY0aTRRZGY1Yzd0dkE3OVRSZkRLQTY1d08Kd0h0QWdlK3lLK0hIQ1Qyd09RbWtNek1RNjZwVFEzUlE0eVdtRDZHR1cWZWMExBR1hFUUxWWHRuTUdGCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==

[10.100.10.10]: PS>
```
-->

En un dispositivo Azure Stack Edge Pro que tenga configurado el rol de proceso, puede solucionar problemas o supervisar el dispositivo con dos conjuntos diferentes de comandos.

- Uso de comandos de `iotedge` Estos comandos están disponibles para las operaciones básicas del dispositivo.
- Uso de comandos de `kubectl` Estos comandos están disponibles para un amplio conjunto de operaciones para el dispositivo.

Para ejecutar cualquiera de los comandos anteriores, debe [conectarse a la interfaz de PowerShell](#connect-to-the-powershell-interface).

### <a name="use-iotedge-commands"></a>Usar comandos `iotedge`

Para ver una lista de comandos disponibles, [conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface) y utilice la función `iotedge`.

```powershell
[10.100.10.10]: PS>iotedge -?                                                                                                                           
Usage: iotedge COMMAND

Commands:
   list
   logs
   restart

[10.100.10.10]: PS>
```

En la tabla siguiente se proporciona una breve descripción de los comandos disponibles para `iotedge`:

|command  |Descripción |
|---------|---------|
|`list`     | Enumere los módulos         |
|`logs`     | Captura los registros de un módulo.        |
|`restart`     | Detiene y reinicia un módulo.         |


Para enumerar todos los módulos que se ejecutan en el dispositivo, use el comando `iotedge list`.

Esta es una salida de ejemplo del comando. Este comando muestra todos los módulos, la configuración asociada y las direcciones IP externas asociadas a los módulos. Por ejemplo, puede acceder a la aplicación del **servidor web** en `https://10.128.44.244`. 


```powershell
[10.100.10.10]: PS>iotedge list

NAME                   STATUS  DESCRIPTION CONFIG                                             EXTERNAL-IP
----                   ------  ----------- ------                                             -----
gettingstartedwithgpus Running Up 10 days  mcr.microsoft.com/intelligentedge/solutions:latest
iotedged               Running Up 10 days  azureiotedge/azureiotedge-iotedged:0.1.0-beta10    <none>
edgehub                Running Up 10 days  mcr.microsoft.com/azureiotedge-hub:1.0             10.128.44.243
edgeagent              Running Up 10 days  azureiotedge/azureiotedge-agent:0.1.0-beta10
webserverapp           Running Up 10 days  nginx:stable                                       10.128.44.244

[10.100.10.10]: PS>
```


### <a name="use-kubectl-commands"></a>Uso de comandos de kubectl

En un dispositivo de Azure Stack Edge Pro con el rol de proceso configurado, están disponibles todos los comandos de `kubectl` para supervisar o solucionar problemas de los módulos. Para ver una lista de los comandos disponibles, ejecute `kubectl --help` desde la ventana de comandos.

```PowerShell
C:\Users\myuser>kubectl --help

kubectl controls the Kubernetes cluster manager.

Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
    create         Create a resource from a file or from stdin.
    expose         Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
    run            Run a particular image on the cluster
    set            Set specific features on objects
    run-container  Run a particular image on the cluster. This command is deprecated, use "run" instead
==============CUT=============CUT============CUT========================

Usage:
    kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

C:\Users\myuser>
```

Si quiere obtener una lista completa de los comandos de `kubectl`, vaya a la [hoja de referencia de `kubectl`](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).


#### <a name="to-get-ip-of-service-or-module-exposed-outside-of-kubernetes-cluster"></a>Para obtener la dirección IP del servicio o módulo expuesto fuera del clúster de Kubernetes

Para obtener la dirección IP de un servicio de equilibrio de carga o los módulos expuestos fuera del Kubernetes, ejecute el siguiente comando:

`kubectl get svc -n iotedge`

A continuación se muestra una salida de ejemplo de todos los servicios o módulos que se exponen fuera del clúster de Kubernetes. 


```powershell
[10.100.10.10]: PS>kubectl get svc -n iotedge
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                       AGE
edgehub        LoadBalancer   10.103.52.225   10.128.44.243   443:31987/TCP,5671:32336/TCP,8883:30618/TCP   34h
iotedged       ClusterIP      10.107.236.20   <none>          35000/TCP,35001/TCP                           3d8h
webserverapp   LoadBalancer   10.105.186.35   10.128.44.244   8080:30976/TCP                                16h

[10.100.10.10]: PS>
```
La dirección IP de la columna IP externa corresponde al punto de conexión externo del servicio o del módulo. También puede [obtener la dirección IP externa en el panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#get-ip-address-for-services-or-modules).


#### <a name="to-check-if-module-deployed-successfully"></a>Comprobación de si el módulo se ha implementado correctamente

Los módulos de proceso son contenedores que tienen una lógica de negocios implementada. Un pod de Kubernetes puede tener varios contenedores en ejecución. 

Para comprobar si un módulo de proceso se implementa correctamente, conéctese a la interfaz de PowerShell del dispositivo.
Ejecute el comando `get pods` y compruebe si se está ejecutando el contenedor (que corresponde al módulo de proceso).

Para obtener la lista de todos los pods que se ejecutan en un espacio de nombres específico, ejecute el siguiente comando:

`get pods -n <namespace>`

Para comprobar los módulos implementados con IoT Edge, ejecute el siguiente comando:

`get pods -n iotedge`

A continuación se muestra una salida de ejemplo de todos los pods que se ejecutan en el espacio de nombres `iotedge`.

```
[10.100.10.10]: PS>kubectl get pods -n iotedge
NAME                        READY   STATUS    RESTARTS   AGE
edgeagent-cf6d4ffd4-q5l2k   2/2     Running   0          20h
edgehub-8c9dc8788-2mvwv     2/2     Running   0          56m
filemove-66c49984b7-h8lxc   2/2     Running   0          56m
iotedged-675d7f4b5f-9nml4   1/1     Running   0          20h

[10.100.10.10]: PS>
```

El estado **Estado** indica que se están ejecutando todos los pods en el espacio de nombres y el estado **Listo** indica el número de contenedores implementados en un pod. En el ejemplo anterior, todos los pods están en ejecución y todos los módulos implementados en cada uno de los pods se están ejecutando. 

Para comprobar los módulos implementados mediante Azure Arc, ejecute el siguiente comando:

`get pods -n azure-arc`

Como alternativa, puede [conectarse al panel de Kubernetes para ver IoT Edge o implementaciones de Azure Arc](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#view-module-status).


Para obtener una salida más detallada de un pod específico para un espacio de nombres determinado, puede ejecutar el siguiente comando:

`kubectl describe pod <pod name> -n <namespace>` 

A continuación se muestra la salida de ejemplo.

```
[10.100.10.10]: PS>kubectl describe pod filemove-66c49984b7 -n iotedge
Name:           filemove-66c49984b7-h8lxc
Namespace:      iotedge
Priority:       0
Node:           k8s-1hwf613cl-1hwf613/10.139.218.12
Start Time:     Thu, 14 May 2020 12:46:28 -0700
Labels:         net.azure-devices.edge.deviceid=myasegpu-edge
                net.azure-devices.edge.hub=myasegpu2iothub.azure-devices.net
                net.azure-devices.edge.module=filemove
                pod-template-hash=66c49984b7
Annotations:    net.azure-devices.edge.original-moduleid: filemove
Status:         Running
IP:             172.17.75.81
IPs:            <none>
Controlled By:  ReplicaSet/filemove-66c49984b7
Containers:
    proxy:
    Container ID:   docker://fd7975ca78209a633a1f314631042a0892a833b7e942db2e7708b41f03e8daaf
    Image:          azureiotedge/azureiotedge-proxy:0.1.0-beta8
    Image ID:       docker://sha256:5efbf6238f13d24bab9a2b499e5e05bc0c33ab1587d6cf6f289cdbe7aa667563
    Port:           <none>
    Host Port:      <none>
    State:          Running
        Started:      Thu, 14 May 2020 12:46:30 -0700
    Ready:          True
    Restart Count:  0
    Environment:
        PROXY_LOG:  Debug
=============CUT===============================CUT===========================
Volumes:
    config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      iotedged-proxy-config
    Optional:  false
    trust-bundle-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      iotedged-proxy-trust-bundle
    Optional:  false
    myasesmb1local:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  myasesmb1local
    ReadOnly:   false
    myasesmb1:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  myasesmb1
    ReadOnly:   false
    filemove-token-pzvw8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  filemove-token-pzvw8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                    node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>


[10.100.10.10]: PS>
```

#### <a name="to-get-container-logs"></a>Para obtener registros de contenedores

Para obtener los registros de un módulo, ejecute el siguiente comando desde la interfaz de PowerShell del dispositivo:

`kubectl logs <pod_name> -n <namespace> --all-containers` 

Dado que la marca `all-containers` volcará todos los registros de todos los contenedores, una buena manera de ver los errores recientes es usar la opción `--tail 10`.

A continuación se muestra una salida de ejemplo. 

```
[10.100.10.10]: PS>kubectl logs filemove-66c49984b7-h8lxc -n iotedge --all-containers --tail 10
DEBUG 2020-05-14T20:40:42Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 1 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:12Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 1 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
05/14/2020 19:46:44: Info: Opening module client connection.
05/14/2020 19:46:45: Info: Open done.
05/14/2020 19:46:45: Info: Initializing with input: /home/input, output: /home/output, protocol: Amqp.
05/14/2020 19:46:45: Info: IoT Hub module client initialized.

[10.100.10.10]: PS>
```



## <a name="exit-the-remote-session"></a>Salir de la sesión remota

Para salir de la sesión remota de PowerShell, cierre la ventana de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

- Implemente [Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md) en Azure Portal.
