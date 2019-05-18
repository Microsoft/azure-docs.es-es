---
title: 'Tutorial: Configuración de roles de control de acceso basado en rol (RBAC) en Azure Kubernetes Service (AKS) con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para configurar el RBAC en el clúster de Azure Kubernetes Service (AKS).
keywords: ansible, azure, devops, bash, cloudshell, playbook, aks, container, aks, kubernetes, azure active directory, rbac
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: dbef7c2cb8de5a1b4bbb3073f694b8f77c9f441b
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231283"
---
# <a name="tutorial-configure-role-based-access-control-rbac-roles-in-azure-kubernetes-service-aks-using-ansible"></a>Tutorial: Configuración de roles de control de acceso basado en rol (RBAC) en Azure Kubernetes Service (AKS) con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-aks.md](../../includes/open-source-devops-intro-aks.md)]

Es posible configurar AKS para que utilice [Azure Active Directory (AD)](/azure/active-directory/) para la autenticación de usuarios. Una vez configurado, use el token de autenticación de Azure AD para iniciar sesión en el clúster de AKS. El RBAC puede basarse en la identidad de un usuario o en la pertenencia al grupo de directorio.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Crear un clúster de AKS habilitado para Azure AD
> * Configurar un rol de RBAC en el clúster

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]
- **Instalar la biblioteca de RedHat OpenShift** - `pip install openshift`

## <a name="configure-azure-ad-for-aks-authentication"></a>Configurar Azure AD para la autenticación de AKS

Al configurar Azure AD para la autenticación de AKS, se configuran dos aplicaciones de Azure AD. Un administrador del inquilino de Azure debe realizar esta operación. Para más información, vea [Integración de Azure Active Directory con Azure Kubernetes Service](/azure/aks/aad-integration#create-server-application). 

Desde el Administrador de inquilinos de Azure, obtenga los siguientes valores:

- Secreto de la aplicación de servidor
- Id. de la aplicación de servidor
- Id. de la aplicación cliente 
- Id. de inquilino

Estos valores son necesarios para ejecutar el cuaderno de estrategias de ejemplo.  

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

En esta sección, creará una instancia de AKS con la [aplicación de Azure AD](#configure-azure-ad-for-aks-authentication).

Estas son algunas notas claves que se deben tener en cuenta al trabajar con el cuaderno de estrategias de ejemplo:

- El cuaderno de estrategias carga `ssh_key` desde `~/.ssh/id_rsa.pub`. Si lo modifica, utilice el formato de línea única, empezando por "ssh-rsa" (sin las comillas).
- Los valores `client_id` y `client_secret` se cargan desde `~/.azure/credentials`, que es el archivo de credenciales predeterminado. Puede establecer estos valores en su entidad de servicio o cargarlos desde las variables de entorno:

    ```yml
    client_id: "{{ lookup('env', 'AZURE_CLIENT_ID') }}"
    client_secret: "{{ lookup('env', 'AZURE_SECRET') }}"
    ```

Guarde el siguiente cuaderno de estrategias como `aks-create.yml`:

```yml
- name: Create resource group
  azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"

- name: List supported kubernetes version from Azure
  azure_rm_aks_version:
      location: "{{ location }}"
  register: versions

- name: Create AKS cluster with RBAC enabled
  azure_rm_aks:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      dns_prefix: "{{ name }}"
      enable_rbac: yes
      kubernetes_version: "{{ versions.azure_aks_versions[-1] }}"
      agent_pool_profiles:
        - count: 3
          name: nodepool1
          vm_size: Standard_D2_v2
      linux_profile:
          admin_username: azureuser
          ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
      service_principal:
          client_id: "{{ lookup('ini', 'client_id section=default file=~/.azure/credentials') }}"
          client_secret: "{{ lookup('ini', 'secret section=default file=~/.azure/credentials') }}"
      aad_profile:
          client_app_id: "{{ client_app_id }}"
          server_app_id: "{{ server_app_id }}"
          server_app_secret: "{{ server_app_secret }}"
          tenant_id: "{{ app_tenant_id }}"
  register: aks

- name: Save cluster user config
  copy:
      content: "{{ aks.kube_config }}"
      dest: "aks-{{ name }}-kubeconfig-user"

- name: Get admin config of AKS
  azure_rm_aks_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ name }}"
      show_kubeconfig: admin
  register: aks

- name: Save the kubeconfig
  copy:
      content: "{{ aks.aks[0].kube_config }}"
      dest: "aks-{{ name }}-kubeconfig"
```

## <a name="get-the-azure-ad-object-id"></a>Obtener el identificador de objeto de Azure

Para crear un enlace de RBAC, primero debe obtener el identificador de objeto de Azure AD. 

1. Inicie sesión en el [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. En el campo de búsqueda de la parte superior de la página, escriba `Azure Active Directory`. 

1. Haga clic en `Enter`.

1. En el menú **Administrar**, seleccione **Usuarios**.

1. En el campo de nombre, busque su cuenta.

1. En la columna **Nombre**, seleccione el vínculo a su cuenta.

1. En la sección **Identidad**, copie el **Id. de objeto**.

    ![Copie el identificador de objeto de Azure AD.](./media/ansible-aks-configure-rbac/ansible-aad-object-id.png)

## <a name="create-rbac-binding"></a>Creación del enlace de RBAC

En esta sección, creará un enlace de rol o un enlace de rol del clúster en AKS. 

Guarde el siguiente cuaderno de estrategias como `kube-role.yml`:

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: <your-aad-account>
```

Reemplace el marcador de posición `&lt;your-aad-account>` por el [Id. de objeto](#get-the-azure-ad-object-id) del inquilino de Azure AD.

Guarde el siguiente cuaderno de estrategias, que implementa el nuevo rol en AKS, como `aks-kube-deploy.yml`:

```yml
- name: Apply role to AKS
  k8s:
      src: kube-role.yml
      kubeconfig: "aks-{{ name }}-kubeconfig"
```

## <a name="run-the-sample-playbook"></a>Ejecución del cuaderno de estrategias de ejemplo

En esta sección se indica el cuaderno de estrategias de ejemplo completo que llama a las tareas que se crean en este artículo. 

Guarde el siguiente cuaderno de estrategias como `aks-rbac.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: aksansibletest
      name: aksansibletest
      location: eastus
  tasks:
     - name: Ensure resource group exist
       azure_rm_resourcegroup:
           name: "{{ resource_group }}"
           location: "{{ location }}"

     - name: Create AKS
       vars:
           client_app_id: <client id>
           server_app_id: <server id>
           server_app_secret: <server secret>
           app_tenant_id: <tenant id>
       include_tasks: aks-create.yml

     - name: Enable RBAC
       include_tasks: aks-kube-deploy.yml
```

En la sección `vars`, reemplace los marcadores de posición siguientes por la información de Azure AD:

- `<client id>`
- `<server id>`
- `<server secret>`
- `<tenant id>`

Ejecute el cuaderno de estrategias completo con el comando `ansible-playbook`:

```bash
ansible-playbook aks-rbac.yml
```

## <a name="verify-the-results"></a>Verificación de los resultados

En esta sección, usará la lista de nodos de kubectl creada en este artículo.

Escriba el siguiente comando en un símbolo del sistema de terminal:

```bash
kubectl --kubeconfig aks-aksansibletest-kubeconfig-user get nodes
```

El comando le remitirá a una página de autenticación. Inicie sesión con su cuenta de Azure.

Una vez autenticado, kubectl enumera los nodos de una forma similar a la que se muestra en los siguientes resultados:

```txt
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code XXXXXXXX to authenticate.
NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-33413200-0   Ready    agent   49m   v1.12.6
aks-nodepool1-33413200-1   Ready    agent   49m   v1.12.6
aks-nodepool1-33413200-2   Ready    agent   49m   v1.12.6
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
---
- hosts: localhost
  vars:
      name: aksansibletest
      resource_group: aksansibletest
  tasks:
      - name: Clean up resource group
        azure_rm_resourcegroup:
            name: "{{ resource_group }}"
            state: absent
            force: yes
      - name: Remove kubeconfig
        file:
            state: absent
            path: "aks-{{ name }}-kubeconfig"
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ansible en Azure](/azure/ansible/)