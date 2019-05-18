---
title: 'Tutorial: Configuración de un clúster de Azure HDInsight con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo usar Ansible para crear y cambiar el tamaño de un clúster de Azure HDInsight
keywords: ansible, azure, devops, bash, playbook, apache hadoop, hdinsight
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: d6b6dd333d04457a68c3f2452d3cc538a32b61f6
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230295"
---
# <a name="tutorial-configure-a-cluster-in-azure-hdinsight-using-ansible"></a>Tutorial: Configuración de un clúster de Azure HDInsight con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[Azure HDInsight](/azure/hdinsight/) es un servicio de análisis basado en Hadoop para el procesamiento de datos. HDInsight es una herramienta ETL (extracción, transformación y carga) para trabajar con macrodatos, sean estructurados o no. HDInsight admite varios [tipos de clúster](/azure/hdinsight/hadoop/apache-hadoop-introduction#cluster-types-in-hdinsight), donde cada tipo es compatible con un conjunto de componentes diferente. 

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Creación de una cuenta de almacenamiento para HDInsight
> * Configuración de un [clúster de HDInsight Spark](/azure/hdinsight/spark/apache-spark-overview).
> * Cambio del tamaño de un clúster
> * Eliminación de un clúster

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)] 

## <a name="create-a-random-postfix"></a>Creación de un sufijo aleatorio

El código del cuaderno de estrategias de esta sección crea un sufijo aleatorio que se usará como parte del nombre del clúster de Azure HDInsight.

```yml
- hosts: localhost
  vars:
    resource_group: "{{ resource_group_name }}"  
  tasks:
    - name: Prepare random prefix
      set_fact:
        rpfx: "{{ resource_group | hash('md5') | truncate(7, True, '') }}{{ 1000 | random }}"
      run_once: yes
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

El código del cuaderno de estrategias de esta sección crea un grupo de recursos.


```yml
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```

## <a name="create-a-storage-account-and-retrieve-key"></a>Creación de una cuenta de almacenamiento y recuperación de la clave

Se usa una cuenta de almacenamiento de Azure como almacenamiento predeterminado del clúster de HDInsight. 

El código del cuaderno de estrategias de esta sección recupera la clave utilizada para acceder a la cuenta de almacenamiento.

```yml
- name: Create storage account
  azure_rm_storageaccount:
      resource_group: "{{ resource_group }}"
      name: "{{ storage_account_name }}"
      account_type: Standard_LRS
      location: eastus2

- name: Get storage account keys
  azure_rm_resource:
    api_version: '2018-07-01'
    method: POST
    resource_group: "{{ resource_group }}"
    provider: storage
    resource_type: storageaccounts
    resource_name: "{{ storage_account_name }}"
    subresource:
      - type: listkeys
  register: storage_output

- debug:
    var: storage_output
```

## <a name="create-an-hdinsight-spark-cluster"></a>Creación de un clúster de Spark en HDInsight

El código del cuaderno de estrategias de esta sección crea el clúster de Azure HDInsight.

```yml
- name: Create instance of Cluster
  azure_rm_hdinsightcluster:
    resource_group: "{{ resource_group }}"
    name: "{{ cluster_name }}"
    location: eastus2
    cluster_version: 3.6
    os_type: linux
    tier: standard
    cluster_definition:
      kind: spark
      gateway_rest_username: http-user
      gateway_rest_password: MuABCPassword!!@123
    storage_accounts:
      - name: "{{ storage_account_name }}.blob.core.windows.net"
        is_default: yes
        container: "{{ cluster_name }}"
        key: "{{ storage_output['response']['keys'][0]['value'] }}"
    compute_profile_roles:
      - name: headnode
        target_instance_count: 1
        vm_size: Standard_D3
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
      - name: workernode
        target_instance_count: 1
        vm_size: Standard_D3
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
      - name: zookeepernode
        target_instance_count: 3
        vm_size: Medium
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
```

La creación de instancias puede tardar varios minutos en completarse.

## <a name="resize-the-cluster"></a>Cambio de tamaño del clúster

Después de la creación del clúster, la única configuración que puede cambiar es el número de nodos de trabajo. 

El código del cuaderno de estrategias de esta sección incrementa el número de nodos de trabajo mediante la actualización de `target_instance_count` en `workernode`.

```yml
- name: Resize cluster
  azure_rm_hdinsightcluster:
    resource_group: "{{ resource_group }}"
    name: "{{ cluster_name }}"
    location: eastus2
    cluster_version: 3.6
    os_type: linux
    tier: standard
    cluster_definition:
      kind: spark
      gateway_rest_username: http-user
      gateway_rest_password: MuABCPassword!!@123
    storage_accounts:
      - name: "{{ storage_account_name }}.blob.core.windows.net"
        is_default: yes
        container: "{{ cluster_name }}"
        key: "{{ storage_output['response']['keys'][0]['value'] }}"
    compute_profile_roles:
      - name: headnode
        target_instance_count: 1
        vm_size: Standard_D3
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
      - name: workernode
        target_instance_count: 2
        vm_size: Standard_D3
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
      - name: zookeepernode
        target_instance_count: 3
        vm_size: Medium
        linux_profile:
          username: sshuser
          password: MuABCPassword!!@123
    tags:
      aaa: bbb
  register: output
```

## <a name="delete-the-cluster-instance"></a>Eliminación de la instancia del clúster

La facturación de los clústeres de HDInsight se prorratea por minuto. 

El código del cuaderno de estrategias de esta sección elimina el clúster.

```yml
- name: Delete instance of Cluster
  azure_rm_hdinsightcluster:
    resource_group: "{{ resource_group }}"
    name: "{{ cluster_name }}"
    state: absent
```

## <a name="get-the-sample-playbook"></a>Obtención del cuaderno de estrategias de ejemplo

Hay dos formas de obtener el cuaderno de estrategias de ejemplo completo:
- [Descargar el cuaderno de estrategias](https://github.com/Azure-Samples/ansible-playbooks/blob/master/hdinsight_create.yml) y guardarlo en `hdinsight_create.yml`.
- Crear un archivo llamado `hdinsight_create.yml` y copiar en él el siguiente contenido:

```yml
---
- hosts: localhost
  vars:
    resource_group: "{{ resource_group_name }}"
  tasks:
    - name: Prepare random prefix
      set_fact:
        rpfx: "{{ resource_group | hash('md5') | truncate(7, True, '') }}{{ 1000 | random }}"
      run_once: yes

- hosts: localhost
  #roles:
  #  - azure.azure_preview_modules
  vars:
    resource_group: "{{ resource_group_name }}"
    location: eastus2
    vnet_name: myVirtualNetwork
    subnet_name: mySubnet
    cluster_name: mycluster{{ rpfx }}
    storage_account_name: mystorage{{ rpfx }}

  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"

    - name: Create storage account
      azure_rm_storageaccount:
        resource_group: "{{ resource_group }}"
        name: "{{ storage_account_name }}"
        account_type: Standard_LRS
        location: "{{ location }}"

    - name: Get storage account keys
      azure_rm_resource:
        api_version: '2018-07-01'
        method: POST
        resource_group: "{{ resource_group }}"
        provider: storage
        resource_type: storageaccounts
        resource_name: "{{ storage_account_name }}"
        subresource:
          - type: listkeys
      register: storage_output
    - debug:
        var: storage_output

    - name: Create instance of Cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        location: "{{ location }}"
        cluster_version: 3.6
        os_type: linux
        tier: standard
        cluster_definition:
          kind: spark
          gateway_rest_username: http-user
          gateway_rest_password: MuABCPassword!!@123 
        storage_accounts:
          - name: "{{ storage_account_name }}.blob.core.windows.net" 
            is_default: yes
            container: "{{ cluster_name }}"
            key: "{{ storage_output['response']['keys'][0]['value'] }}"
        compute_profile_roles:
          - name: headnode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: workernode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: zookeepernode
            target_instance_count: 3
            vm_size: Medium
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123

    - name: Resize cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        location: "{{ location }}"
        cluster_version: 3.6
        os_type: linux
        tier: standard
        cluster_definition:
          kind: spark
          gateway_rest_username: http-user
          gateway_rest_password: MuABCPassword!!@123 
        storage_accounts:
          - name: "{{ storage_account_name }}.blob.core.windows.net"
            is_default: yes
            container: "{{ cluster_name }}"
            key: "{{ storage_output['response']['keys'][0]['value'] }}"
        compute_profile_roles:
          - name: headnode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: workernode
            target_instance_count: 2
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: zookeepernode
            target_instance_count: 3
            vm_size: Medium
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
        tags:
          aaa: bbb
      register: output
    - debug:
        var: output

    - name: Assert the state has changed
      assert:
        that:
          - output.changed

    - name: Delete instance of Cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        state: absent
```

## <a name="run-the-sample-playbook"></a>Ejecución del cuaderno de estrategias de ejemplo

En esta sección, ejecute el cuaderno de estrategias para probar varias características que se muestran en este artículo.

Antes de ejecutar el cuaderno de estrategias, haga los siguientes cambios:
- En la sección `vars`, reemplace el marcador de posición `{{ resource_group_name }}` por el nombre del grupo de recursos.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook hdinsight.yml
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        force_delete_nonempty: yes
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)