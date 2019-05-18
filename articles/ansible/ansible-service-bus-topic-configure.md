---
title: 'Tutorial: Configuración de temas de Azure Service Bus con Ansible | Microsoft Docs'
description: Aprenda a usar Ansible para crear un tema de Azure Service Bus
keywords: ansible, azure, devops, bash, cuaderno de estrategias, service bus, temas, suscripciones
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: ca8d849796520ac260d888d772c064316db68a30
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230860"
---
# <a name="tutorial-configure-topics-in-azure-service-bus-using-ansible"></a>Tutorial: Configuración de temas de Azure Service Bus con Ansible

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-servicebus.md](../../includes/open-source-devops-intro-servicebus.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * de un tema
> * una suscripción
> * Crear una directiva de SAS
> * Recuperar información del espacio de nombres
> * Recuperar información del tema y de la suscripción
> * Revocar una directiva de SAS

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-the-service-bus-topic"></a>Creación del tema de Service Bus

El código del cuaderno de estrategias de ejemplo crea los siguientes recursos:
- Grupo de recursos de Azure
- El espacio de nombres de Service Bus donde se encuentra el recurso
- Tema de Service Bus con el espacio de nombres

Guarde el siguiente cuaderno de estrategias como `servicebus_topic.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      location: eastus
      namespace: servicebustestns
      topic: servicebustesttopic
  tasks:
    - name: Ensure resource group exist
      azure_rm_resourcegroup:
          name: "{{ resource_group }}"
          location: "{{ location }}"
    - name: Create a namespace
      azure_rm_servicebus:
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
    - name: Create a topic
      azure_rm_servicebustopic:
          name: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
      register: topic
    - debug:
          var: topic
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_topic.yml
```

## <a name="create-the-subscription"></a>Creación de la suscripción

El código del cuaderno de estrategias de ejemplo crea la suscripción en un tema de Service Bus. Los temas de Azure Service Bus pueden tener varias suscripciones. Un suscriptor a un tema puede recibir una copia de cada mensaje enviado a ese tema. Las suscripciones son entidades con nombre, que se crean de forma duradera pero pueden, opcionalmente, expirar.

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      location: eastus
      namespace: servicebustestns
      topic: servicebustesttopic
      subscription: servicebustestsubs
  tasks:
    - name: Create a subscription
      azure_rm_servicebustopicsubscription:
          name: "{{ subscription }}"
          topic: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
      register: subs
    - debug:
          var: subs
```

Guarde el siguiente cuaderno de estrategias como `servicebus_subscription.yml`:

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_subscription.yml
```

## <a name="create-the-sas-policy"></a>Creación de la política SAS

Una [Firma de acceso compartido](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) es un mecanismo de autorización basada en notificaciones que utiliza tokens. 

El código del cuaderno de estrategias de ejemplo crea dos directivas SAS para una cola de Service Bus con privilegios diferentes.

Guarde el siguiente cuaderno de estrategias como `servicebus_topic_policy.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      topic: servicebustesttopic
  tasks:
    - name: Create a policy with send and listen privilege
      azure_rm_servicebussaspolicy:
          name: "{{ topic }}-{{ item }}"
          topic: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          rights: "{{ item }}"
      with_items:
        - send
        - listen
      register: policy
    - debug:
          var: policy
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_topic_policy.yml
```

## <a name="retrieve-namespace-information"></a>Recuperar información del espacio de nombres

El código del cuaderno de estrategias de ejemplo consulta la información del espacio de nombres.

Guarde el siguiente cuaderno de estrategias como `servicebus_namespace_info.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
  tasks:
    - name: Get a namespace's information
      azure_rm_servicebus_facts:
          type: namespace
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          show_sas_policies: yes
      register: ns
    - debug:
          var: ns
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:
- El valor `show_sas_policies` indica si se muestran las directivas SAS en el espacio de nombres especificado. De forma predeterminada, el valor es `False` para evitar más sobrecarga de red.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_namespace_info.yml
```

## <a name="retrieve-topic-and-subscription-information"></a>Recuperar información del tema y de la suscripción

El código del cuaderno de estrategias de ejemplo consulta la siguiente información:
- Información del tema de Service Bus
- Lista de detalles de la suscripción para el tema
 
Guarde el siguiente cuaderno de estrategias como `servicebus_list.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      topic: servicebustesttopic
  tasks:
    - name: Get a topic's information
      azure_rm_servicebus_facts:
          type: topic
          name: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          show_sas_policies: yes
      register: topic_fact
    - name: "List subscriptions under topic {{ topic }}"
      azure_rm_servicebus_facts:
          type: subscription
          topic: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
      register: subs_fact
    - debug:
          var: "{{ item }}"
      with_items:
        - topic_fact.servicebuses[0]
        - subs_fact.servicebuses
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:
- El valor `show_sas_policies` indica si se muestran las directivas SAS en la cola especificada. De forma predeterminada, el valor se establece en `False` para evitar más sobrecarga de red.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_list.yml
```

## <a name="revoke-the-queue-sas-policy"></a>Revocación de la política SAS de cola

El código del cuaderno de estrategias de ejemplo elimina una directiva SAS de cola.

Guarde el siguiente cuaderno de estrategias como `servicebus_queue_policy_delete.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      topic: servicebustesttopic
  tasks:
    - name: Delete a policy
      azure_rm_servicebussaspolicy:
          name: "{{ topic }}-policy"
          topic: "{{ topic }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook servicebus_topic_policy_delete.yml
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente código como `cleanup.yml`:

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      topic: servicebustesttopic
      subscription: servicebustestsubs
  tasks:
    - name: Delete subscription
      azure_rm_servicebustopicsubscription:
          name: "{{ subscription }}"
          topic: "{{ topic }}"
          resource_group: "{{ resource_group }}"
          namespace: "{{ namespace }}"
          state: absent
    - name: Delete topic
      azure_rm_servicebustopic:
          name: "{{ topic }}"
          resource_group: "{{ resource_group }}"
          namespace: "{{ namespace }}"
          state: absent
    - name: Delete namespace
      azure_rm_servicebus:
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          state: absent
    - name: Delete resource group
      azure_rm_resourcegroup:
          name: "{{ resource_group }}"
          state: absent
          force_delete_nonempty: yes
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)