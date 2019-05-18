---
title: 'Tutorial: Configuración de aplicaciones en Azure App Service con Ansible | Microsoft Docs'
description: Obtenga información sobre cómo crear una aplicación en Azure App Service con Java 8 y el contenedor de Tomcat en tiempo de ejecución
keywords: ansible, azure, devops, bash, playbook, Azure App Service, Web App, Java
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: aed09baf410ce25f2e5383aa746344a440e2a052
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231240"
---
# <a name="tutorial-configure-apps-in-azure-app-service-using-ansible"></a>Tutorial: Configuración de aplicaciones en Azure App Service con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-app-service.md](../../includes/open-source-devops-intro-app-service.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Crear una aplicación en Azure App Service con Java 8 y el contenedor de Tomcat en tiempo de ejecución
> * Crear un perfil de Azure Traffic Manager
> * Definir un punto de conexión de Traffic Manager con la aplicación creada

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-a-basic-app-service"></a>Creación de un servicio de aplicación básico

El código del cuaderno de estrategias de esta sección define los siguientes recursos:

* Grupo de recursos de Azure dentro del cual se implementan la aplicación y el plan de App Service
* Un servicio de aplicación en Linux con Java 8 y el contenedor de Tomcat en tiempo de ejecución

Guarde el siguiente cuaderno de estrategias como `firstwebapp.yml`:

```yml
- hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    webapp_name: myfirstWebApp
    plan_name: myAppServicePlan
    location: eastus
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"

    - name: Create App Service on Linux with Java Runtime
      azure_rm_webapp:
        resource_group: "{{ resource_group }}"
        name: "{{ webapp_name }}"
        plan:
          resource_group: "{{ resource_group }}"
          name: "{{ plan_name }}"
          is_linux: true
          sku: S1
          number_of_workers: 1
        frameworks:
          - name: "java"
            version: "8"
            settings:
              java_container: tomcat
              java_container_version: 8.5
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook firstwebapp.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create a resource group] 
changed: [localhost]

TASK [Create App Service on Linux with Java Runtime] 
 [WARNING]: Azure API profile latest does not define an entry for WebSiteManagementClient

changed: [localhost]

PLAY RECAP 
localhost                  : ok=3    changed=2    unreachable=0    failed=0
```

## <a name="create-an-app-and-use-azure-traffic-manager"></a>Creación de una aplicación y uso de Azure Traffic Manager

[Azure Traffic Manager](/azure/app-service/web-sites-traffic-manager) permite controlar la manera en que se distribuyen solicitudes de clientes web a aplicaciones en Azure App Service. Cuando se agregan puntos de conexión de App Service a un perfil de Azure Traffic Manager, este realiza un seguimiento del estado de las aplicaciones de App Service. Los estados incluyen en ejecución, detenido y eliminado. Traffic Manager se usa para decidir qué puntos de conexión deben recibir el tráfico.

En App Service, cada aplicación se ejecuta en un [plan de App Service](/azure/app-service/overview-hosting-plans). Un plan de App Service define un conjunto de recursos de proceso para que una aplicación se ejecute. Puede administrar el plan de App Service y la aplicación web en grupos distintos.

El código del cuaderno de estrategias de esta sección define los siguientes recursos:

* Grupo de recursos de Azure dentro del cual se implementa el plan de App Service
* Plan de App Service
* Grupo de recursos de Azure dentro del cual se implementa la aplicación
* Un servicio de aplicación en Linux con Java 8 y el contenedor de Tomcat en tiempo de ejecución
* Perfil de Traffic Manager
* Punto de conexión de Traffic Manager con la aplicación creada

Guarde el siguiente cuaderno de estrategias como `webapp.yml`:

```yml
- hosts: localhost
  connection: local
  vars:
    resource_group_webapp: myResourceGroupWebapp
    resource_group: myResourceGroup
    webapp_name: myLinuxWebApp
    plan_name: myAppServicePlan
    location: eastus
    traffic_manager_profile_name: myTrafficManagerProfile
    traffic_manager_endpoint_name: myTrafficManagerEndpoint

  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
        name: "{{ resource_group_webapp }}"
        location: "{{ location }}"

  - name: Create secondary resource group
    azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"

  - name: Create App Service Plan
    azure_rm_appserviceplan:
      resource_group: "{{ resource_group }}"
      name: "{{ plan_name }}"
      location: "{{ location }}"
      is_linux: true
      sku: S1
      number_of_workers: 1

  - name: Create App Service on Linux with Java Runtime
    azure_rm_webapp:
        resource_group: "{{ resource_group_webapp }}"
        name: "{{ webapp_name }}"
        plan:
          resource_group: "{{ resource_group }}"
          name: "{{ plan_name }}"
          is_linux: true
          sku: S1
          number_of_workers: 1
        app_settings:
          testkey: "testvalue"
        frameworks:
          - name: java
            version: 8
            settings:
              java_container: "Tomcat"
              java_container_version: "8.5"

  - name: Get web app facts
    azure_rm_webapp_facts:
      resource_group: "{{ resource_group_webapp }}"
      name: "{{ webapp_name }}"
    register: webapp
    
  - name: Create Traffic Manager Profile
    azure_rm_trafficmanagerprofile:
      resource_group: "{{ resource_group_webapp }}"
      name: "{{ traffic_manager_profile_name }}"
      location: global
      routing_method: performance
      dns_config:
        relative_name: "{{ traffic_manager_profile_name }}"
        ttl:  60
      monitor_config:
        protocol: HTTPS
        port: 80
        path: '/'

  - name: Add endpoint to traffic manager profile, using created web site
    azure_rm_trafficmanagerendpoint:
      resource_group: "{{ resource_group_webapp }}"
      profile_name: "{{ traffic_manager_profile_name }}"
      name: "{{ traffic_manager_endpoint_name }}"
      type: azure_endpoints
      location: "{{ location }}"
      target_resource_id: "{{ webapp.webapps[0].id }}"
```

Ejecute el comando de estrategias con el comando `ansible-playbook`:

```bash
ansible-playbook webapp.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create resource group] 
changed: [localhost]

TASK [Create resource group for app service plan] 
changed: [localhost]

TASK [Create App Service Plan] 
 [WARNING]: Azure API profile latest does not define an entry for WebSiteManagementClient

changed: [localhost]

TASK [Create App Service on Linux with Java Runtime] 
changed: [localhost]

TASK [Get web app facts] 
ok: [localhost]

TASK [Create Traffic Manager Profile] 
 [WARNING]: Azure API profile latest does not define an entry for TrafficManagerManagementClient

changed: [localhost]

TASK [Add endpoint to traffic manager profile, using the web site created above] 
changed: [localhost]

TASK [Get Traffic Manager Profile facts] 
ok: [localhost]

PLAY RECAP 
localhost                  : ok=9    changed=6    unreachable=0    failed=0
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Tutorial: Escalado de aplicaciones en Azure App Service con Ansible](/azure/ansible/ansible-scale-azure-web-apps)