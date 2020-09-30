---
title: Uso de kubectl para implementar una aplicación con estado de Kubernetes mediante un recurso compartido aprovisionado dinámicamente en un dispositivo Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo se crea y administra la implementación de una aplicación con estado de Kubernetes mediante un recurso compartido aprovisionado dinámicamente utilizando kubectl en un dispositivo Microsoft Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/26/2020
ms.author: alkohli
ms.openlocfilehash: d37152f7dec78d5f5db21fdde9a8ec25c36c4e05
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899478"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-storageclass-on-your-azure-stack-edge-pro-gpu-device"></a>Uso de kubectl para ejecutar una aplicación con estado de Kubernetes con StorageClass en el dispositivo Azure Stack Edge Pro con GPU

En este artículo se muestra cómo implementar una aplicación con estado de una sola instancia en Kubernetes con StorageClass para aprovisionar de forma dinámica el almacenamiento y una implementación. La implementación usa comandos `kubectl` en un clúster de Kubernetes existente e implementa la aplicación MySQL. 

Este procedimiento está dirigido a los usuarios que han revisado [Almacenamiento de Kubernetes en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-storage.md) y están familiarizados con los conceptos de [almacenamiento de Kubernetes](https://kubernetes.io/docs/concepts/storage/).


## <a name="prerequisites"></a>Requisitos previos

Antes de implementar la aplicación con estado, asegúrese de que ha completado los siguientes requisitos previos en el dispositivo y en el cliente que vaya a usar para acceder al dispositivo:

### <a name="for-device"></a>Para el dispositivo

- Tiene credenciales de inicio de sesión en un dispositivo Azure Stack Edge Pro de un nodo.
    - El dispositivo está activado. Consulte [Activación del dispositivo](azure-stack-edge-gpu-deploy-activate.md).
    - El dispositivo tiene el rol de proceso configurado desde Azure Portal y un clúster de Kubernetes. Consulte [Configuración del proceso](azure-stack-edge-gpu-deploy-configure-compute.md).

### <a name="for-client-accessing-the-device"></a>Para el cliente que va a acceder al dispositivo

- Tiene un sistema cliente de Windows que se usará para acceder al dispositivo Azure Stack Edge Pro.
    - El cliente ejecuta Windows PowerShell 5.0 o una versión posterior. Para descargar la última versión de Windows PowerShell, vaya a [Instalación de Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-7).
    
    - También puede utilizar cualquier otro cliente con un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). En este artículo, el procedimiento que se describe emplea un cliente Windows. 
    
    - Ha completado el procedimiento descrito en [Acceso al clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Ha:
      - Creado un espacio de nombres `userns1` con el comando `New-HcsKubernetesNamespace`. 
      - Creado un usuario `user1` con el comando `New-HcsKubernetesUser`. 
      - Concedido el acceso `user1` a `userns1` con el comando `Grant-HcsKubernetesNamespaceAccess`.       
      - Instalado `kubectl` en el cliente y guardado el archivo `kubeconfig` con la configuración de usuario en C:\\Usuarios\\&lt;nombredeusuario&gt;\\.kube. 
    
    - Asegúrese de que la versión del cliente de `kubectl` es como máximo una versión superior o inferior a la versión maestra de Kubernetes que se ejecuta en el dispositivo Azure Stack Edge Pro. 
        - Use `kubectl version` para comprobar la versión de kubectl que se ejecuta en el cliente. Anote la versión completa.
        - En la interfaz de usuario local del dispositivo Azure Stack Edge Pro, vaya a **Información general** y anote el número de software de Kubernetes. 
        - Compruebe la compatibilidad entre estas dos versiones con la asignación proporcionada en la versión de Kubernetes admitida<!-- insert link-->. 


Está listo para implementar una aplicación con estado en el dispositivo Azure Stack Edge Pro. 


## <a name="deploy-mysql"></a>Implementación de MySQL

Ahora ejecutará una aplicación con estado mediante la creación de una implementación de Kubernetes y su conexión a la instancia integrada de StorageClass mediante un elemento PersistentVolumeClaim (PVC). 

Todos los comandos `kubectl` que utiliza para crear y administrar implementaciones de aplicaciones con estado tienen que especificar el espacio de nombres asociado a la configuración. Para especificar el espacio de nombres en un comando kubectl, use `kubectl <command> -n <your-namespace>`.

1. También puede obtener una lista de los pods que se ejecutan en el clúster de Kubernetes en el espacio de nombres. Un pod es un contenedor o un proceso de la aplicación que se ejecuta en el clúster de Kubernetes.

   ```powershell
   kubectl get pods -n <your-namespace>
   ```
    
   Este es un ejemplo de uso del comando:
    
   ```powershell
    C:\Users\user>kubectl get pods -n "userns1"
    No resources found in userns1 namespace.    
    C:\Users\user>
   ```
    
   La salida debe indicar que no se encuentra ningún recurso (pod), ya que no hay ninguna aplicación en ejecución en el clúster.

1. Usará los archivos YAML siguientes. El archivo `mysql-deployment.yml` describe una implementación que ejecuta MySQL y hace referencia a PVC. El archivo define un montaje de volumen para `/var/lib/mysql`y, a continuación, crea un PVC que busca un volumen de 20 GB. Se aprovisiona un PV dinámico y el PVC se enlaza a este PV.

    Copie y guarde el siguiente archivo `mysql-deployment.yml` en una carpeta en el cliente Windows que esté usando para acceder al dispositivo Azure Stack Edge Pro.
    
    ```yml
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
    spec:
      ports:
      - port: 3306
      selector:
        app: mysql
      clusterIP: None
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: mysql
    spec:
      selector:
        matchLabels:
          app: mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - image: mysql:5.6
            name: mysql
            env:
              # Use secret in real usage
            - name: MYSQL_ROOT_PASSWORD
              value: password
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-pv-claim-sc
    ```
    
2. Haga una copia y guárdela como archivo `mysql-pvc.yml` en la misma carpeta donde guardó el archivo `mysql-deployment.yml`. Para usar el elemento StorageClass integrado del dispositivo Azure Stack Edge Pro en un disco de datos conectado, establezca el campo `storageClassName` del objeto PVC en `ase-node-local` y accessModes debe ser `ReadWriteOnce`. 

    > [!NOTE] 
    > Asegúrese de que la sangría de los archivos YAML es correcta. Puede comprobarlo con [YAML lint](http://www.yamllint.com/) para la validación y luego guardarlos.
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim-sc
    spec:
      storageClassName: ase-node-local
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Implementación del archivo `mysql-pvc.yaml`.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Este es un ejemplo de resultado de la implementación.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pvc.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim-sc created
    C:\Users\user>
    ```
   Anote el nombre del PVC creado, aquí es `mysql-pv-claim-sc`. Lo usará en otro paso más adelante. 

4. Implemente el contenido del archivo `mysql-deployment.yml`.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Este es un ejemplo de resultado de la implementación.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
    service/mysql created
    deployment.apps/mysql created
    C:\Users\user>
    ```
    
5. Visualice la información sobre la implementación.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Thu, 20 Aug 2020 11:14:25 -0700
    Labels:             <none>
    Annotations:        deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"mysql","namespace":"userns1"},"spec":{"selector":{"matchL...
    Selector:           app=mysql
    Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:       Recreate
    MinReadySeconds:    0
    Pod Template:
      Labels:  app=mysql
      Containers:
       mysql:
        Image:      mysql:5.6
        Port:       3306/TCP
        Host Port:  0/TCP
        Environment:
          MYSQL_ROOT_PASSWORD:  password
        Mounts:
          /var/lib/mysql from mysql-persistent-storage (rw)
      Volumes:
       mysql-persistent-storage:
        Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName:  mysql-pv-claim-sc
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-695c4d9dcd (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set mysql-695c4d9dcd to 1
    C:\Users\user>
    ```
    

6. Enumere los pods creados por la implementación.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Esta es una salida de ejemplo.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                     READY   STATUS    RESTARTS   AGE
    mysql-695c4d9dcd-rvzff   1/1     Running   0          40s
    C:\Users\user>
    ```
    
7. Inspeccione PersistentVolumeClaim.

    `kubectl describe pvc <your-pvc-name>`

    Esta es una salida de ejemplo.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim-sc -n userns1
    Name:          mysql-pv-claim-sc
    Namespace:     userns1
    StorageClass:  ase-node-local
    Status:        Bound
    Volume:        pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim-sc","namespace":"userns1"},"spec":{"...
                   pv.kubernetes.io/bind-completed: yes
                   pv.kubernetes.io/bound-by-controller: yes
                   volume.beta.kubernetes.io/storage-provisioner: rancher.io/local-path
                   volume.kubernetes.io/selected-node: k8s-3q7lhq2cl-3q7lhq2
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      20Gi
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Mounted By:    mysql-695c4d9dcd-rvzff
    Events:
      Type    Reason                 Age                From                                                                                                Message
      ----    ------                 ----               ----                                                                                                -------
      Normal  WaitForFirstConsumer   71s (x2 over 77s)  persistentvolume-controller                                                                         waiting for first consumer to be created before binding
      Normal  ExternalProvisioning   62s                persistentvolume-controller                                                                         waiting for a volume to be created, either by external provisioner "rancher.io/local-path" or manually created by system administrator
      Normal  Provisioning           62s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  External provisioner is provisioning volume for claim "userns1/mysql-pv-claim-sc"
      Normal  ProvisioningSucceeded  60s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  Successfully provisioned volume pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Comprobación de que MySQL está en ejecución

Para comprobar que la aplicación está en ejecución, escriba:

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql -p`

Cuando se le solicite, proporcione la contraseña. La contraseña está en el archivo `mysql-deployment`.

Esta es una salida de ejemplo.

```powershell
C:\Users\user>kubectl exec mysql-695c4d9dcd-rvzff -i -t -n userns1 -- mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.49 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```

## <a name="delete-a-deployment"></a>Eliminación de una implementación

Para eliminar la implementación, elimine los objetos implementados por nombre. Estos objetos incluyen la implementación, el servicio y el PVC.
 
```powershell
kubectl delete deployment <deployment-name>,svc <service-name> -n <your-namespace>
kubectl delete pvc <your-pvc-name> -n <your-namespace>
```

Este es el resultado de ejemplo de cuando se elimina la implementación y el servicio.

```powershell
C:\Users\user>kubectl delete deployment,svc mysql -n userns1
deployment.apps "mysql" deleted
service "mysql" deleted
C:\Users\user>
```
Este es el resultado de ejemplo de cuando se elimina el PVC.

```powershell
C:\Users\user>kubectl delete pvc mysql-pv-claim-sc -n userns1
persistentvolumeclaim "mysql-pv-claim-sc" deleted
C:\Users\user>
```                                                                                         


## <a name="next-steps"></a>Pasos siguientes

Para saber cómo configurar la red a través de kubectl, consulte [Implementación de una aplicación sin estado en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md).
