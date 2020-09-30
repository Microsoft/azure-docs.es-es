---
title: Descripción del control de acceso basado en rol de Kubernetes en un dispositivo Azure Stack Edge Pro | Microsoft Docs
description: Describe cómo se produce el control de acceso basado en rol de Kubernetes en un dispositivo Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: alkohli
ms.openlocfilehash: 1f194424a4030a2b829af6c8f5b97a3c200bd2e6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899288"
---
# <a name="kubernetes-role-based-access-control-on-your-azure-stack-edge-pro-gpu-device"></a>Control de acceso basado en roles de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU


En el dispositivo Azure Stack Edge Pro, se crea un clúster de Kubernetes al configurar el rol de proceso. Puede usar el control de acceso basado en roles (RBAC) de Kubernetes para limitar el acceso a los recursos del clúster en el dispositivo.

En este artículo se proporciona información general sobre el sistema RBAC proporcionado por Kubernetes y cómo se implementa en un dispositivo Azure Stack Edge Pro. 

## <a name="rbac-for-kubernetes"></a>Control de acceso basado en roles para Kubernetes

Este control de Kubernetes permite asignar a usuarios o grupos de usuarios permiso para realizar acciones como crear o modificar recursos, o ver registros de cargas de trabajo de aplicaciones en ejecución. Estos permisos se pueden limitar a un único espacio de nombres o conceder en todo el clúster. 

Al configurar el clúster de Kubernetes, se crea un único usuario que corresponde a este clúster y se conoce como usuario administrador del clúster.  Se asocia un archivo `kubeconfig` con este usuario. El archivo `kubeconfig` es un archivo de texto que contiene toda la información de configuración necesaria para conectarse al clúster y autenticar al usuario.

## <a name="namespaces-types"></a>Tipos de espacios de nombres

Los recursos de Kubernetes, como los pods y las implementaciones, se agrupan lógicamente en un espacio de nombres. Estas agrupaciones proporcionan una manera de dividir de forma lógica un clúster de Kubernetes y de restringir el acceso para crear, ver o administrar los recursos. Los usuarios solo pueden interactuar con los recursos dentro de sus espacios de nombres asignados.

Los espacios de nombres están pensados para su uso en entornos con muchos usuarios distribuidos en varios equipos o proyectos. Para más información, consulte los [espacios de nombres de Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).

Un dispositivo Azure Stack Edge Pro tiene los siguientes espacios de nombres:

- **Espacio de nombres de sistema**: este es el espacio de nombres donde se encuentran los recursos principales; por ejemplo, las funciones de red como DNS y proxy o el panel de Kubernetes. Normalmente, el usuario no implementa sus propias aplicaciones en este espacio de nombres. Use este espacio de nombres para depurar cualquier problema de clúster de Kubernetes. 

    Hay varios espacios de nombres de sistema en el dispositivo; los nombres correspondientes a estos espacios están reservados. Esta es una lista de los espacios de nombres de sistema reservados: 
    - kube-system
    - metallb-system
    - dbe-namespace
    - default
    - kubernetes-dashboard
    - kube-node-lease
    - kube-public


    Asegúrese de no usar ningún nombre reservado para los espacios de nombres de usuario que cree. 
<!--- **default namespace** - This namespace is where pods and deployments are created by default when none is provided and you have admin access to this namespace. When you interact with the Kubernetes API, such as with `kubectl get pods`, the default namespace is used when none is specified.-->

- **Espacio de nombres de usuario**: se trata de los espacios de nombres que puede crear a través de **kubectl** o a través de la interfaz de PowerShell del dispositivo para implementar aplicaciones de forma local.
 
- **Espacio de nombres de IoT Edge**: se conecta a este espacio de nombres `iotedge` para administrar las aplicaciones implementadas a través de IoT Edge.

- **Espacio de nombres de Azure Arc**: se conecta a este espacio de nombres `azure-arc` para administrar las aplicaciones implementadas a través de Azure Arc. Con Azure Arc, también puede implementar aplicaciones en otros espacios de nombres de usuario. 

## <a name="namespaces-and-users"></a>Espacios de nombres y usuarios

En el mundo real, es importante dividir el clúster en varios espacios de nombres. 

- **Varios usuarios**: si tiene varios usuarios, contar con varios espacios de nombres permitirá a esos usuarios implementar sus aplicaciones y servicios en sus espacios de nombres específicos, aislados entre sí. 
- **Usuario único**: incluso si hay un solo usuario, contar con varios espacios de nombres permite que ese usuario ejecute varias versiones de las aplicaciones en el mismo clúster de Kubernetes.

### <a name="roles-and-rolebindings"></a>Roles y RoleBindings

En Kubernetes se utilizan los conceptos de rol y enlace de rol, que permiten conceder permisos a usuarios o recursos en un nivel de espacio de nombres y en un nivel de clúster. 

- **Roles**: puede definir permisos para los usuarios como **Rol** y, a continuación, usar **Roles** para conceder permisos en un espacio de nombres. 
- **RoleBindings**: una vez que haya definido los roles, puede usar **RoleBindings** para asignar roles en un espacio de nombres determinado. 

Este enfoque permite separar de forma lógica un único clúster de Kubernetes, de modo que los usuarios solo puedan acceder a los recursos de la aplicación en su espacio de nombres asignado. 

## <a name="rbac-on-azure-stack-edge-pro"></a>RBAC en Azure Stack Edge Pro

En la implementación actual del control de acceso basado en rol, Azure Stack Edge Pro permite realizar las siguientes acciones desde un espacio de ejecución de PowerShell restringido:

- Crear espacios de nombres.  
- Crear usuarios adicionales.
- Conceder acceso de administrador a los espacios de nombres que ha creado. Tenga en cuenta que no tendrá acceso al rol de administrador del clúster ni a una vista de los recursos de todo el clúster.
- Obtener el archivo `kubeconfig` con información para acceder al clúster de Kubernetes.


El dispositivo Azure Stack Edge Pro tiene varios espacios de nombres de sistema y permite crear espacios de nombres de usuario con archivos `kubeconfig` para acceder a esos espacios de nombres. Los usuarios tienen control total sobre estos espacios de nombres y pueden crear o modificar usuarios, o conceder acceso a los usuarios. Solo el administrador del clúster tiene acceso total a los espacios de nombres de sistema y a los recursos de todo el clúster. El tipo `aseuser` tiene acceso de solo lectura a los espacios de nombres de sistema.

A continuación, encontrará un diagrama que muestra la implementación del control de acceso basado en rol en un dispositivo Azure Stack Edge Pro.

![RBAC en un dispositivo Azure Stack Edge Pro](./media/azure-stack-edge-gpu-kubernetes-rbac/rbac-view-1.png)

En este diagrama, Alice, Bob y Chuck solo tienen acceso a los espacios de nombres de usuario asignados, que en este caso son `ns1`, `ns2` y `ns3`, respectivamente. Dentro de estos espacios de nombres, tienen acceso de administrador. Por otra parte, el administrador del clúster tiene acceso de administrador a los espacios de nombres de sistema y a los recursos de todo el clúster.

Puede usar comandos `kubectl` para crear espacios de nombres y usuarios, asignar usuarios a espacios de nombres o descargar archivos `kubeconfig`. Este es un flujo de trabajo general:

1. Cree un espacio de nombres y un usuario.  

    `New-HcsKubernetesNamespace -Namespace`  

2. Cree un usuario.  

    `New-HcsKubernetesUser -UserName`  

3. Asocie el espacio de nombres con el usuario que ha creado.  

    `Grant-HcsKubernetesNamespaceAccess -Namespace -UserName`  

4. Guarde la configuración de usuario en `C:\Users\<username>\.kube`.  

5. Instale `kubectl` e comience a implementar aplicaciones en `kubectl`. 

Para obtener instrucciones detalladas, vaya al artículo sobre el [acceso al clúster de Kubernetes mediante kubectl en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).


Cuando se trabaja con espacios de nombres y usuarios en los dispositivos Azure Stack Edge Pro, se aplican las siguientes advertencias:

- No se pueden realizar operaciones como crear usuarios, o conceder o revocar al usuario el acceso al espacio de nombres en ninguno de los espacios de nombres de sistema. Algunos ejemplos de espacios de nombres de sistema son `kube-system`, `metallb-system`, `kubernetes-dashboard`, `default`, `kube-node-lease`, `kube-public`. Los espacios de nombres de sistema también incluyen los espacios de nombres reservados para tipos de implementación, como `iotedge` (espacio de nombres de IoT Edge) y `azure-arc` (espacio de nombres de Azure Arc).
- Puede crear espacios de nombres de usuario y, dentro de esos espacios de nombres, crear usuarios adicionales y conceder o revocar a esos usuarios el acceso a los espacios de nombres.
- No puede crear espacios de nombres que reciban el mismo nombre que un espacio de nombres de sistema. Los nombres de los espacios de nombres de sistema están reservados.  
- No puede crear espacios de nombres de usuario con nombres que ya estén siendo utilizados por otros espacios de nombres de usuario. Por ejemplo, si ha creado un espacio de nombres `test-ns`, no puede crear otro con el nombre `test-ns`.
- No puede crear usuarios con nombres que ya están reservados. Por ejemplo, `aseuser` es un usuario reservado y no se puede usar.


## <a name="next-steps"></a>Pasos siguientes

Para entender cómo puede crear un usuario, crear un espacio de nombres y conceder a los usuarios acceso al espacio de nombres, consulte [Acceso a un clúster de Kubernetes a través de kubectl](azure-stack-edge-gpu-create-kubernetes-cluster.md).

