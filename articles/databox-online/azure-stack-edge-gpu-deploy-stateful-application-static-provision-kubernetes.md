---
title: Uso de kubectl para implementar una aplicación con estado de Kubernetes a través de un recurso compartido aprovisionado de forma estática en un dispositivo Azure Stack Edge Pro | Microsoft Docs
description: Se describe cómo se crea y administra la implementación de una aplicación con estado de Kubernetes a través de un recurso compartido aprovisionado estáticamente utilizando kubectl en un dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/18/2020
ms.author: alkohli
ms.openlocfilehash: 8366c5b7a05b35891bcf87e446229357a5511359
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899548"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-a-persistentvolume-on-your-azure-stack-edge-pro-device"></a>Uso de kubectl para ejecutar una aplicación con estado de Kubernetes con PersistentVolume en el dispositivo Azure Stack Edge Pro

En este artículo se muestra cómo implementar una aplicación con estado de una sola instancia en Kubernetes con PersistentVolume (PV) y una implementación. La implementación usa comandos `kubectl` en un clúster de Kubernetes existente e implementa la aplicación MySQL. 

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
        - Compruebe la compatibilidad entre estas dos versiones con la asignación proporcionada en la versión de Kubernetes admitida <!-- insert link-->. 


Está listo para implementar una aplicación con estado en el dispositivo Azure Stack Edge Pro. 

## <a name="provision-a-static-pv"></a>Aprovisionamiento de un PV estático

Para aprovisionar de forma estática un PV, debe crear un recurso compartido en el dispositivo. Siga estos pasos para aprovisionar un PV en un recurso compartido de archivos SMB. 

> [!NOTE]
> El ejemplo específico que se usa en este artículo no funciona con recursos compartidos de archivos NFS. En general, los recursos compartidos de archivos NFS se pueden aprovisionar en un dispositivo de Azure Stack Edge con aplicaciones que no son base de datos.

1. Elija si quiere crear un recurso compartido de Edge o un recurso compartido de Edge local. Siga las instrucciones de [Adición de un recurso compartido](azure-stack-edge-manage-shares.md#add-a-share) para crear un recurso compartido. Asegúrese de seleccionar la casilla **Usar el recurso compartido con el proceso perimetral**.

    ![Recurso compartido local perimetral para PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/edge-local-share-static-provision-1.png)

    1. En lugar de crear un nuevo recurso compartido, si decide usar un recurso compartido existente, tendrá que montar el recurso compartido.
    
        En la instancia de Azure Portal del recurso compartido de Azure Stack Edge, vaya a **Recursos compartidos**. En la lista de recursos compartidos existentes, seleccione y haga clic en el recurso compartido que quiere usar.

        ![Selección de recurso compartido local existente para PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-1.png)

    1. Seleccione **Montar** y confirme el montaje cuando se le solicite.  

        ![Montaje de recurso compartido local existente para PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-2.png)

1. Tome nota del nombre del recurso compartido. Cuando se crea este recurso compartido, se crea automáticamente un objeto de volumen persistente en el clúster de Kubernetes correspondiente al recurso compartido de archivos SMB que creó. 

## <a name="deploy-mysql"></a>Implementación de MySQL

Ahora ejecutará una aplicación con estado mediante la creación de una implementación de Kubernetes y su conexión al PV que creo en el paso anterior mediante un elemento PersistentVolumeClaim (PVC). 

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

1. Usará los archivos YAML siguientes. El archivo `mysql-deployment.yml` describe una implementación que ejecuta MySQL y hace referencia a PVC. El archivo define un montaje de volumen para `/var/lib/mysql`y, a continuación, crea un PVC que busca un volumen de 20 GB. 

    Esta reclamación se satisface con cualquier PV existente que se haya aprovisionado estáticamente al crear el recurso compartido en el paso anterior. En el dispositivo, se crea un PV grande de 32 TB para cada recurso compartido. PV cumple los requisitos establecidos por PVC y el PVC debe estar enlazado a este PV.

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
              claimName: mysql-pv-claim
    ```
    
2. Haga una copia y guárdela como archivo `mysql-pv.yml` en la misma carpeta donde guardó el archivo `mysql-deployment.yml`. Para usar el recurso compartido de archivos SMB que creó anteriormente con `kubectl`, establezca el campo `volumeName` del objeto PVC en el nombre del recurso compartido. 

    > [!NOTE] 
    > Asegúrese de que la sangría de los archivos YAML es correcta. Puede comprobarlo con [YAML lint](http://www.yamllint.com/) para la validación y luego guardarlos.
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
    spec:
      volumeName: <smb-share-name-here>
      storageClassName: ""
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Implementación del archivo `mysql-pv.yaml`.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Este es un ejemplo de resultado de la implementación.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pv.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim created
    
    C:\Users\user>
    ```
   Anote el nombre del PVC creado. Lo usará en otro paso más adelante. 

4. Implemente el contenido del archivo `mysql-deployment.yml`.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Este es un ejemplo de resultado de la implementación.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
        service/mysql created
        deployment.apps/mysql created
    ```
    
5. Visualice la información sobre la implementación.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Tue, 18 Aug 2020 09:44:58 -0700
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
        ClaimName:  mysql-pv-claim
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Progressing    True    NewReplicaSetAvailable
      Available      True    MinimumReplicasAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-c85f7f79c (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set mysql-c85f7f79c to 1
    
    C:\Users\user>
    ```
    

6. Enumere los pods creados por la implementación.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Esta es una salida de ejemplo.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                    READY   STATUS    RESTARTS   AGE
    mysql-c85f7f79c-vzz7j   1/1     Running   1          14m
    
    C:\Users\user>
    ```
    
7. Inspeccione PersistentVolumeClaim.

    `kubectl describe pvc <your-pvc-name>`

    Esta es una salida de ejemplo.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim -n userns1
    Name:          mysql-pv-claim
    Namespace:     userns1
    StorageClass:
    Status:        Bound
    Volume:        mylocalsmbshare1
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim","namespace":"userns1"},"spec":{"acc...
                   pv.kubernetes.io/bind-completed: yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      32Ti
    Access Modes:  RWO,RWX
    VolumeMode:    Filesystem
    Mounted By:    mysql-c85f7f79c-vzz7j
    Events:        <none>
    
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Comprobación de que MySQL está en ejecución


Para ejecutar un comando en un contenedor de un pod que ejecuta MySQL, escriba:

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql`

Esta es una salida de ejemplo.

```powershell
C:\Users\user>kubectl exec mysql-c85f7f79c-vzz7j -i -t -n userns1 -- mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
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
C:\Users\user>kubectl delete pvc mysql-pv-claim -n userns1
persistentvolumeclaim "mysql-pv-claim" deleted
C:\Users\user>
```                                                                                         

PV ya no está enlazado al PVC, ya que este se eliminó. Como el PV se aprovisionó cuando se creó el recurso compartido, deberá eliminar el recurso compartido. Siga estos pasos:

1. Desmonte el recurso compartido. En Azure Portal, vaya a su **Recurso de Azure Stack Edge > Recursos compartidos** y seleccione y haga clic en el recurso compartido que quiere desmontar. Seleccione **Desmontar** y confirme la operación. Espere hasta que se desmonte el recurso compartido. El desmontaje libera el recurso compartido (y, por tanto, el elemento PersistentVolume asociado) del clúster de Kubernetes. 

    ![Desmontaje de recurso compartido local para PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/unmount-edge-local-share-1.png)

1. Ahora puede seleccionar **Eliminar** y confirmar la eliminación para eliminar el recurso compartido. También debe eliminar el recurso compartido y el PV correspondiente.

    ![Eliminación del recurso compartido local para PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/delete-edge-local-share-1.png)


## <a name="next-steps"></a>Pasos siguientes

Para saber cómo aprovisionar el almacenamiento de forma dinámica, consulte [Implementación de una aplicación con estado a través del aprovisionamiento dinámico en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md).
